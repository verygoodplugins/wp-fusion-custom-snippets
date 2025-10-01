# WooCommerce Subscription Signup Tracking for HubSpot

**Category:** E-commerce Integration  
**Compatibility:** WP Fusion + WooCommerce Subscriptions + HubSpot  
**Source:** Custom solution for ticket #35677

## Description

Tracks new WooCommerce Subscription signups and syncs the signup date to HubSpot, enabling time-based reporting on subscription acquisitions. This snippet specifically addresses the need to track "Number of signups within a specific timeframe" in HubSpot reports.

## Use Cases

- Track subscription acquisition metrics in HubSpot
- Create reports showing subscription signups by date range
- Monitor subscription growth trends over time
- Segment customers based on subscription signup date
- Track conversions at the Company level in HubSpot
- Differentiate between new subscriptions and renewals

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Prerequisites

1. Ensure you have a custom field in HubSpot for the subscription signup date (e.g., `subscription_signup_date`)
2. Map this field in WP Fusion settings (Settings → WP Fusion → Contact Fields)
3. Optionally, create a custom field for subscription product names

## Code

```php
<?php
/**
 * Track WooCommerce Subscription signups in HubSpot
 * 
 * This snippet syncs subscription signup dates and optionally fires events
 * to enable time-based reporting in HubSpot.
 */

/**
 * Track subscription signups when status changes to active
 * 
 * @param WC_Subscription $subscription The subscription object.
 */
function wpf_track_subscription_signup_hubspot( $subscription ) {

	$user_id = $subscription->get_user_id();
	
	if ( empty( $user_id ) ) {
		return;
	}

	// Prepare data to sync
	$update_data = array(
		'subscription_signup_date'  => $subscription->get_date_created()->getTimestamp();
	);

	wpf_log( 'info', $user_id, 'Tracking subscription signup for subscription #' . $subscription->get_id() );

	// Update the contact in HubSpot
	wp_fusion()->user->push_user_meta( $user_id, $update_data );
}

add_action( 'woocommerce_subscription_status_active', 'wpf_track_subscription_signup_hubspot' );

/**
 * Also track when transitioning from pending to active
 */
function wpf_track_subscription_activation( $subscription, $new_status, $old_status ) {
	
	// Track when going from pending/on-hold to active for the first time
	if ( 'active' === $new_status && in_array( $old_status, array( 'pending', 'on-hold' ) ) ) {
		
		// Check if this is the first activation
		$first_activation = $subscription->get_meta( '_wpf_first_activation' );
		
		if ( ! $first_activation ) {
			wpf_track_subscription_signup_hubspot( $subscription );
			$subscription->update_meta_data( '_wpf_first_activation', current_time( 'timestamp' ) );
			$subscription->save();
		}
	}
}

add_action( 'woocommerce_subscription_status_updated', 'wpf_track_subscription_activation', 10, 3 );

/**
 * Add settings to WP Fusion for subscription signup tracking
 */
function wpf_subscription_signup_settings( $settings, $options ) {

	$settings['woo_header']['subsection_title_subscription_signups'] = array(
		'title'   => __( 'Subscription Signup Tracking', 'wp-fusion' ),
		'type'    => 'heading',
		'section' => 'integrations',
	);

	$settings['woo_header']['woo_subscription_signup_tags'] = array(
		'title'   => __( 'Subscription Signup Tags', 'wp-fusion' ),
		'desc'    => __( 'Apply these tags when a new subscription signup is tracked.', 'wp-fusion' ),
		'type'    => 'assign_tags',
		'section' => 'integrations',
	);

	return $settings;
}

add_filter( 'wpf_configure_settings', 'wpf_subscription_signup_settings', 20, 2 );
```

## Configuration

### Creating HubSpot Reports

After implementing this snippet:

1. Go to HubSpot → Reports → Create custom report
2. Choose "Single object" → Contacts
3. Add filters:
   - **subscription_signup_date** is known
   - **subscription_signup_date** between [date range]
4. Group by subscription_products or billing_period for additional insights
5. For Company-level reports, ensure contacts are associated with companies

## How It Works

1. **Initial Detection**: Hooks into subscription status changes
2. **Duplicate Prevention**: Tracks whether signup has been recorded
3. **Data Collection**: Gathers subscription details including products and values
4. **HubSpot Sync**: Updates contact properties with timestamp (in milliseconds for HubSpot)
5. **Event Tracking**: Optionally fires custom events for advanced analytics
6. **Tag Application**: Can apply tags for additional segmentation

## Troubleshooting

### Subscriptions Not Being Tracked

- Check WP Fusion logs (WP Fusion → Settings → Advanced → Logs)
- Verify HubSpot field mappings are configured
- Ensure the subscription is genuinely new (not a renewal)

### Date Format Issues

- HubSpot expects timestamps in milliseconds
- The snippet automatically multiplies Unix timestamps by 1000

### Company-Level Tracking

For company-level tracking, ensure:
1. Contacts are associated with companies in HubSpot
2. Use HubSpot workflows to copy contact properties to company properties
3. Or create company-level custom properties and update via HubSpot API

## Related Snippets

- [WooCommerce Subscription Renewal Tags](./woocommerce-subscription-renewal-tags.md)
- [WooCommerce Custom Order Statuses](./woocommerce-custom-order-statuses.md)