# API Method Overrides

**Category:** API Extensions  
**Compatibility:** WP Fusion  
**Source:** [WP Fusion Documentation](https://wpfusion.com/documentation/filters/wpf_api_method_name/)

## Description

Override any of WP Fusion's CRM API methods with custom functionality using the `wpf_api_{$method_name}` filter. This allows you to intercept API calls and implement custom logic, caching, logging, or alternative processing before or instead of the standard CRM communication.

## Use Cases

- Enhanced contact lookup with additional matching criteria
- Implement API call caching for performance
- Add comprehensive logging for debugging
- Temporarily disable API calls during maintenance
- Custom error handling and retry logic

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code Examples

### Cache Contact ID Lookups

Cache contact ID lookups to reduce API calls and improve performance on high-traffic sites.

```php
function cache_contact_id_lookups( $result = null, $args ) {
    
    $email_address = $args[0];
    $cache_key = 'wpf_contact_id_' . md5( $email_address );
    
    // Check cache first
    $cached_id = wp_cache_get( $cache_key );
    if ( false !== $cached_id ) {
        return $cached_id;
    }
    
    // Let WP Fusion do the normal lookup
    return $result; // null = continue with normal process
}

// Cache the result after normal lookup
function cache_contact_id_result( $contact_id, $email ) {
    if ( $contact_id && ! is_wp_error( $contact_id ) ) {
        $cache_key = 'wpf_contact_id_' . md5( $email );
        wp_cache_set( $cache_key, $contact_id, '', HOUR_IN_SECONDS );
    }
}

add_filter( 'wpf_api_get_contact_id', 'cache_contact_id_lookups', 10, 2 );
add_action( 'wpf_got_contact_id', 'cache_contact_id_result', 10, 2 );
```

### Enhanced Contact Lookup with Name Matching

Override the contact ID lookup with ActiveCampaign so the ID is only returned if the email address, first name, and last name match (for registered users).

```php
/**
 * Get the Contact ID for a given email address, with an additional search by name
 * for registered users.
 *
 * @param null  $result The result. Return a non-null value to override the API call.
 * @param array $args   The arguments passed to the CRM API function.
 * @return string The Contact ID, if found.
 */
function custom_wpf_get_contact_id( $result = null, $args ) {

    // $email_address is the only parameter passed to wp_fusion()->crm->get_contact_id()
    $email_address = $args[0];

    $request_uri = wp_fusion()->crm->api_url . 'api/3/contacts?email=' . rawurlencode( $email_address );

    $user = get_user_by( 'email', $email_address );

    if ( $user ) {
        // If it's a registered user, let's also search by their name
        $request_uri .= '&search=' . rawurlencode( $user->first_name . ' ' . $user->last_name );
    }

    // Make the API call
    $response = wp_safe_remote_get( $request_uri, wp_fusion()->crm->get_params() );

    if ( is_wp_error( $response ) ) {
        return $response;
    }

    $response = json_decode( wp_remote_retrieve_body( $response ) );

    if ( empty( $response->contacts ) ) {
        // No matching contact found
        return false;
    } else {
        // Return the ID of the contact
        return $response->contacts[0]->id;
    }
}

add_filter( 'wpf_api_get_contact_id', 'custom_wpf_get_contact_id', 10, 2 );
```

### Prevent Tag Application During Maintenance

Temporarily disable tag applications during site maintenance or bulk operations.

```php
function prevent_tags_during_maintenance( $result = null, $args ) {
    
    // Check if maintenance mode is active
    if ( defined( 'WP_MAINTENANCE_MODE' ) && WP_MAINTENANCE_MODE ) {
        wpf_log( 'notice', 0, 'Tag application blocked - maintenance mode active' );
        return true; // Return success without actually applying tags
    }
    
    // Check for bulk operations
    if ( doing_action( 'wpf_batch_process' ) ) {
        // Allow normal processing during batch operations
        return $result;
    }
    
    // Check if it's during specified maintenance hours (e.g., 2-4 AM)
    $current_hour = (int) date( 'H' );
    if ( $current_hour >= 2 && $current_hour <= 4 ) {
        wpf_log( 'notice', 0, 'Tag application deferred - maintenance window' );
        
        // Queue the tag application for later
        wp_schedule_single_event( time() + HOUR_IN_SECONDS, 'wpf_delayed_apply_tags', $args );
        
        return true;
    }
    
    return $result;
}

add_filter( 'wpf_api_apply_tags', 'prevent_tags_during_maintenance', 10, 2 );
```

### Log All API Calls for Debugging

Override API calls to add comprehensive logging while still executing the original functionality.

```php
function log_all_api_calls( $result = null, $args ) {
    
    // Get the method name from the current filter
    $current_filter = current_filter();
    $method_name = str_replace( 'wpf_api_', '', $current_filter );
    
    // Log the API call attempt
    if ( function_exists( 'wpf_log' ) ) {
        wpf_log( 'info', 0, "API Call: {$method_name}", array( 
            'meta_array' => array(
                'method' => $method_name,
                'args' => $args,
                'timestamp' => current_time( 'mysql' )
            )
        ));
    }
    
    // Return null to continue with normal processing
    return $result;
}

// Apply to all API methods
$api_methods = array( 'get_contact_id', 'get_tags', 'apply_tags', 'remove_tags', 
                     'add_contact', 'update_contact', 'load_contact' );

foreach ( $api_methods as $method ) {
    add_filter( "wpf_api_{$method}", 'log_all_api_calls', 5, 2 );
}
```

## Available API Methods

- `get_contact_id` - Look up contact ID by email
- `get_tags` - Retrieve available tags from CRM
- `apply_tags` - Apply tags to a contact
- `remove_tags` - Remove tags from a contact
- `add_contact` - Create new contact in CRM
- `update_contact` - Update existing contact data
- `load_contact` - Load contact data from CRM
- `load_contacts` - Load multiple contacts
- `track_event` - Track events (with Event Tracking add-on)

## Filter Parameters

- `$result` (null) - Return a non-null value to override the API call
- `$args` (array) - The arguments being passed to the CRM API method

## Related Snippets

- [Non-Blocking Login API Calls](./non-blocking-login-api-calls.md) - Improve login performance
- [Extend HTTP Timeout](./extend-http-timeout.md) - Handle slow API responses
- [Webhook Loopback Prevention](./webhook-loopback-prevention.md) - Prevent infinite loops

## Notes

- Return `null` to continue with normal WP Fusion processing
- Return any other value to override and skip the normal API call
- Always handle errors appropriately in your custom implementations
- Test thoroughly with your specific CRM integration

## Troubleshooting

**Issue:** Override not working  
**Solution:** Ensure you're using the correct filter name format: `wpf_api_{method_name}`

**Issue:** Breaking normal functionality  
**Solution:** Return `null` when you want normal processing to continue

**Issue:** Performance issues  
**Solution:** Implement proper caching and error handling in your overrides
