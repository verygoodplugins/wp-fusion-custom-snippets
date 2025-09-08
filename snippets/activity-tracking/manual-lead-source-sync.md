# Manual Lead Source Data Sync

**Category:** Activity Tracking  
**Compatibility:** WP Fusion  
**Source:** Custom implementation

## Description

Manually syncs lead source data (UTM parameters and current page) from WP Fusion's lead source tracking cookie to custom CRM fields when a new user is created. This ensures lead source attribution is properly captured even when automatic syncing might not work as expected.

## Use Cases

- Ensure UTM parameter tracking for marketing attribution
- Capture lead source data for new user registrations
- Override or supplement automatic lead source tracking
- Custom field mapping for specific CRM configurations
- Track referral sources and campaign performance
- Marketing ROI analysis and attribution reporting

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code

```php
<?php

function wpf_manually_sync_lead_source_data( $user_id, $contact_id ) {
	
	$field_mapping = array(
		'utm_campaign' => 'field[132,0]',
		'utm_source'   => 'field[133,0]',
		'utm_medium'   => 'field[134,0]',
		'current_page' => 'field[139,0]',
	);

	if ( empty( $_COOKIE['wpf_leadsource'] ) ) {
		return;
	}

	$cookie_data = json_decode( wp_unslash( $_COOKIE['wpf_leadsource'] ), true );

	$update_data = array();

	foreach ( $field_mapping as $cookie_name => $field_id ) {

		if ( ! isset( $cookie_data[ $cookie_name ] ) ) {
			continue;
		}

		$update_data[ $field_id ] = $cookie_data[ $cookie_name ];
	}

	wpf_log( 'info', $user_id, 'Manually syncing lead source data:', array( 'meta_array_nofilter' => $update_data ) );

	wp_fusion()->crm->update_contact( $contact_id, $update_data );

}

add_action( 'wpf_user_created', 'wpf_manually_sync_lead_source_data', 10, 2 );
```

## How It Works

1. **User Creation Hook**: Triggers when WP Fusion creates a new contact record
2. **Cookie Check**: Verifies the WP Fusion lead source cookie exists
3. **Data Extraction**: Parses JSON cookie data for UTM parameters and page info
4. **Field Mapping**: Maps cookie data to specific CRM field IDs
5. **Data Update**: Sends the lead source data directly to the CRM
6. **Logging**: Records the sync operation for debugging

## Configuration

### Update Field Mapping

Replace the field IDs with your actual CRM field IDs:

```php
$field_mapping = array(
    'utm_campaign' => 'your_campaign_field_id',
    'utm_source'   => 'your_source_field_id',
    'utm_medium'   => 'your_medium_field_id',
    'current_page' => 'your_page_field_id',
);
```

### Find Your CRM Field IDs

**For ActiveCampaign:**
- Field IDs look like `field[132,0]` (as shown in example)
- Check your ActiveCampaign custom fields

**For HubSpot:**
- Field IDs are property names like `utm_campaign`
- Use HubSpot property internal names

**For Salesforce:**
- Field IDs are API names like `UTM_Campaign__c`
- Check your Salesforce custom field API names

## Advanced Customizations

### Add Additional UTM Parameters

Track more UTM parameters and custom data:

```php
function wpf_extended_lead_source_sync( $user_id, $contact_id ) {
    
    $field_mapping = array(
        'utm_campaign' => 'field[132,0]',
        'utm_source'   => 'field[133,0]',
        'utm_medium'   => 'field[134,0]',
        'utm_term'     => 'field[135,0]',
        'utm_content'  => 'field[136,0]',
        'current_page' => 'field[139,0]',
        'referrer'     => 'field[140,0]',
        'landing_page' => 'field[141,0]',
        'gclid'        => 'field[142,0]', // Google Ads click ID
        'fbclid'       => 'field[143,0]', // Facebook click ID
    );

    if ( empty( $_COOKIE['wpf_leadsource'] ) ) {
        return;
    }

    $cookie_data = json_decode( wp_unslash( $_COOKIE['wpf_leadsource'] ), true );
    $update_data = array();

    foreach ( $field_mapping as $cookie_name => $field_id ) {
        if ( isset( $cookie_data[ $cookie_name ] ) && ! empty( $cookie_data[ $cookie_name ] ) ) {
            $update_data[ $field_id ] = sanitize_text_field( $cookie_data[ $cookie_name ] );
        }
    }

    if ( ! empty( $update_data ) ) {
        wpf_log( 'info', $user_id, 'Syncing extended lead source data:', array( 'meta_array_nofilter' => $update_data ) );
        wp_fusion()->crm->update_contact( $contact_id, $update_data );
    }
}

add_action( 'wpf_user_created', 'wpf_extended_lead_source_sync', 10, 2 );
```

### Conditional Sync Based on Source

Only sync data for specific traffic sources:

```php
function wpf_conditional_lead_source_sync( $user_id, $contact_id ) {
    
    if ( empty( $_COOKIE['wpf_leadsource'] ) ) {
        return;
    }

    $cookie_data = json_decode( wp_unslash( $_COOKIE['wpf_leadsource'] ), true );
    
    // Only sync for paid traffic sources
    $paid_sources = array( 'google', 'facebook', 'linkedin', 'twitter' );
    
    if ( ! isset( $cookie_data['utm_source'] ) || 
         ! in_array( strtolower( $cookie_data['utm_source'] ), $paid_sources ) ) {
        return;
    }

    $field_mapping = array(
        'utm_campaign' => 'field[132,0]',
        'utm_source'   => 'field[133,0]',
        'utm_medium'   => 'field[134,0]',
        'current_page' => 'field[139,0]',
    );

    $update_data = array();

    foreach ( $field_mapping as $cookie_name => $field_id ) {
        if ( isset( $cookie_data[ $cookie_name ] ) ) {
            $update_data[ $field_id ] = $cookie_data[ $cookie_name ];
        }
    }

    if ( ! empty( $update_data ) ) {
        wpf_log( 'info', $user_id, 'Syncing paid traffic lead source data:', array( 'meta_array_nofilter' => $update_data ) );
        wp_fusion()->crm->update_contact( $contact_id, $update_data );
    }
}

add_action( 'wpf_user_created', 'wpf_conditional_lead_source_sync', 10, 2 );
```

### Sync on Login Instead of Creation

Track lead source for existing users on login:

```php
function wpf_sync_lead_source_on_login( $user_login, $user ) {
    
    if ( empty( $_COOKIE['wpf_leadsource'] ) ) {
        return;
    }

    $contact_id = wpf_get_contact_id( $user->ID );
    if ( empty( $contact_id ) ) {
        return;
    }

    $cookie_data = json_decode( wp_unslash( $_COOKIE['wpf_leadsource'] ), true );
    
    // Only update if we have new campaign data
    if ( empty( $cookie_data['utm_campaign'] ) ) {
        return;
    }

    $field_mapping = array(
        'utm_campaign' => 'field[132,0]',
        'utm_source'   => 'field[133,0]',
        'utm_medium'   => 'field[134,0]',
    );

    $update_data = array();

    foreach ( $field_mapping as $cookie_name => $field_id ) {
        if ( isset( $cookie_data[ $cookie_name ] ) ) {
            $update_data[ $field_id ] = $cookie_data[ $cookie_name ];
        }
    }

    if ( ! empty( $update_data ) ) {
        wpf_log( 'info', $user->ID, 'Syncing lead source data on login:', array( 'meta_array_nofilter' => $update_data ) );
        wp_fusion()->crm->update_contact( $contact_id, $update_data );
    }
}

add_action( 'wp_login', 'wpf_sync_lead_source_on_login', 10, 2 );
```

## Lead Source Cookie Structure

The WP Fusion lead source cookie (`wpf_leadsource`) typically contains:

```json
{
    "utm_source": "google",
    "utm_medium": "cpc",
    "utm_campaign": "summer-sale",
    "utm_term": "wordpress-plugin",
    "utm_content": "text-ad",
    "current_page": "/landing-page/",
    "referrer": "https://google.com/",
    "landing_page": "/home/",
    "gclid": "abc123def456",
}
```

## Debugging and Testing

### Debug Cookie Contents

Add temporary debugging to see what's in the cookie:

```php
function debug_lead_source_cookie() {
    if ( current_user_can( 'administrator' ) && isset( $_GET['debug_leadsource'] ) ) {
        if ( ! empty( $_COOKIE['wpf_leadsource'] ) ) {
            $cookie_data = json_decode( wp_unslash( $_COOKIE['wpf_leadsource'] ), true );
            echo '<pre style="background: yellow; padding: 10px;">';
            echo 'Lead Source Cookie Data:' . "\n";
            print_r( $cookie_data );
            echo '</pre>';
        } else {
            echo '<div style="background: red; color: white; padding: 10px;">No lead source cookie found</div>';
        }
    }
}
add_action( 'wp_footer', 'debug_lead_source_cookie' );
```

## Performance Considerations

- Cookie parsing is lightweight and fast
- Only runs when new users are created
- Minimal CRM API calls (one update per user)

## Troubleshooting

### Common Issues

1. **No Data Syncing**: Check if lead source cookie exists and contains data
2. **Wrong Field IDs**: Verify CRM field IDs are correct
3. **Cookie Parsing Errors**: Ensure cookie contains valid JSON
4. **CRM API Errors**: Check WP Fusion logs for API failures

### Validation Steps

1. Visit your site with UTM parameters: `?utm_source=test&utm_campaign=debug`
2. Check if lead source cookie is set
3. Create a test user account
4. Verify data appears in your CRM
5. Check WP Fusion logs for sync confirmation

## Compatibility Notes

- Works with all WP Fusion CRM integrations
- Requires WP Fusion's lead source tracking to be enabled
- Compatible with custom registration forms
- Works with WooCommerce customer registration

## Related Snippets

- [Activity Tracking](../activity-tracking/) - Other tracking implementations
- [User Management](../user-management/) - User data handling
- [Automation Workflows](../automation-workflows/) - Attribution-based automations
