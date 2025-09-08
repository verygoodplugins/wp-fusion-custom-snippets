# Webhook Loopback Prevention

**Category:** API Extensions  
**Compatibility:** WP Fusion  
**Source:** [GitHub Gist](https://gist.github.com/jack-arturo/5e2c7c64012442e3b67e196a3bb14985)

## Description

Prevents webhook loopbacks by blocking incoming webhooks for one minute after WP Fusion applies tags to a user. This prevents infinite loops where applying tags triggers webhooks that apply more tags.

## Use Cases

- Prevent infinite webhook loops between WordPress and CRM
- Stop duplicate tag applications from webhook responses
- Avoid webhook storms during bulk operations
- Maintain data consistency during automated processes
- Prevent performance issues from webhook loops

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code

```php
<?php

/**
 * Sets a transient to lock a user from receiving webhooks from the CRM for one minute
 * after tags have been applied by WP Fusion.
 *
 * @param int $user_id The user ID.
 */
function wpf_lock_user( $user_id ) {

        $contact_id = wpf_get_contact_id( $user_id );

        set_transient( 'wpf_api_lock_' . $contact_id, 'update', MINUTE_IN_SECONDS );

}

add_action( 'wpf_tags_applied', 'wpf_lock_user' );
```

## How It Works

1. **Tag Application Detection**: Hooks into `wpf_tags_applied` action
2. **Contact ID Lookup**: Gets the CRM contact ID for the user
3. **Transient Lock**: Sets a temporary lock using WordPress transients
4. **Time-Limited**: Lock expires after 60 seconds (MINUTE_IN_SECONDS)
5. **Webhook Blocking**: WP Fusion checks this transient before processing webhooks

## Configuration Options

### Adjust Lock Duration

Change the lock duration by modifying the time constant:

```php
// Lock for 2 minutes
set_transient( 'wpf_api_lock_' . $contact_id, 'update', 2 * MINUTE_IN_SECONDS );

// Lock for 30 seconds
set_transient( 'wpf_api_lock_' . $contact_id, 'update', 30 );

// Lock for 5 minutes
set_transient( 'wpf_api_lock_' . $contact_id, 'update', 5 * MINUTE_IN_SECONDS );
```

### Lock on Other Actions

Extend locking to other WP Fusion actions:

```php
function wpf_extended_lock_user( $user_id ) {
    $contact_id = wpf_get_contact_id( $user_id );
    set_transient( 'wpf_api_lock_' . $contact_id, 'update', MINUTE_IN_SECONDS );
}

// Lock on multiple actions
add_action( 'wpf_tags_applied', 'wpf_extended_lock_user' );
add_action( 'wpf_tags_removed', 'wpf_extended_lock_user' );
add_action( 'wpf_user_updated', 'wpf_extended_lock_user' );
```

### Conditional Locking

Only lock for specific tags or users:

```php
function wpf_conditional_lock_user( $user_id, $tags ) {
    
    // Only lock for specific tags
    $problematic_tags = array( 'tag-id-1', 'tag-id-2' );
    
    if ( array_intersect( $tags, $problematic_tags ) ) {
        $contact_id = wpf_get_contact_id( $user_id );
        set_transient( 'wpf_api_lock_' . $contact_id, 'update', MINUTE_IN_SECONDS );
    }
}

add_action( 'wpf_tags_applied', 'wpf_conditional_lock_user', 10, 2 );
```

## Advanced Implementations

### Logging Lock Events

Add logging to monitor lock behavior:

```php
function wpf_lock_user_with_logging( $user_id ) {
    
    $contact_id = wpf_get_contact_id( $user_id );
    
    set_transient( 'wpf_api_lock_' . $contact_id, 'update', MINUTE_IN_SECONDS );
    
    if ( function_exists( 'wpf_log' ) ) {
        wpf_log( 'info', $user_id, 'User locked from webhooks for 1 minute. Contact ID: ' . $contact_id );
    }
}

add_action( 'wpf_tags_applied', 'wpf_lock_user_with_logging' );
```

### Dynamic Lock Duration

Adjust lock duration based on conditions:

```php
function wpf_dynamic_lock_user( $user_id, $tags ) {
    
    $contact_id = wpf_get_contact_id( $user_id );
    
    // Longer lock for bulk operations
    $lock_duration = doing_action( 'wpf_batch_process' ) ? 5 * MINUTE_IN_SECONDS : MINUTE_IN_SECONDS;
    
    // Longer lock for problematic tags
    $high_risk_tags = array( 'automation-trigger', 'webhook-heavy' );
    if ( array_intersect( $tags, $high_risk_tags ) ) {
        $lock_duration = 3 * MINUTE_IN_SECONDS;
    }
    
    set_transient( 'wpf_api_lock_' . $contact_id, 'update', $lock_duration );
}

add_action( 'wpf_tags_applied', 'wpf_dynamic_lock_user', 10, 2 );
```

### Check Lock Status

Utility function to check if a user is locked:

```php
function wpf_is_user_locked( $user_id ) {
    $contact_id = wpf_get_contact_id( $user_id );
    return get_transient( 'wpf_api_lock_' . $contact_id ) !== false;
}

// Usage example
if ( wpf_is_user_locked( $user_id ) ) {
    wpf_log( 'notice', $user_id, 'Skipping webhook processing - user is locked' );
    return;
}
```

## Troubleshooting

### Common Issues

1. **Locks Not Working**: Ensure WP Fusion is checking the transient in webhook processing
2. **Too Short Duration**: Increase lock time if webhooks are still causing loops
3. **Missing Contact ID**: Verify user has a valid CRM contact ID

### Debug Lock Status

```php
// Add temporarily to check lock status
function debug_lock_status( $user_id ) {
    $contact_id = wpf_get_contact_id( $user_id );
    $lock_status = get_transient( 'wpf_api_lock_' . $contact_id );
    error_log( "User {$user_id} (Contact {$contact_id}) lock status: " . var_export( $lock_status, true ) );
}

add_action( 'wpf_tags_applied', 'debug_lock_status' );
```

## Performance Considerations

- Transients are stored in the database (or object cache if available)
- Minimal performance impact
- Automatic cleanup when transients expire
- Consider object caching for high-traffic sites

## Compatibility Notes

- Works with all WP Fusion CRM integrations
- Compatible with webhook processing systems
- Requires WP Fusion to respect the lock transient
- Works with WordPress transient system

## Related Snippets

- [API Extensions](../api-extensions/) - Other API modifications
- [Debugging & Logging](../debugging-logging/) - Webhook monitoring
- [Automation Workflows](../automation-workflows/) - Preventing automation loops
