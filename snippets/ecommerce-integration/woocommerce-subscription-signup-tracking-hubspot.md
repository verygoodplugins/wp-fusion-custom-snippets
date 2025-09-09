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

	// Only proceed if this is the initial activation (not a renewal)
	if ( $subscription->get_date_created() ) {
		$created_date = $subscription->get_date_created()->getTimestamp();
		$current_time = current_time( 'timestamp' );
		
		// If subscription was created more than 1 hour ago, this is likely a reactivation
		if ( ( $current_time - $created_date ) > 3600 ) {
			
			// Check if we've already tracked this subscription
			$already_tracked = $subscription->get_meta( '_wpf_signup_tracked' );
			
			if ( $already_tracked ) {
				wpf_log( 'info', $subscription->get_user_id(), 'Subscription #' . $subscription->get_id() . ' signup already tracked, skipping.' );
				return;
			}
		}
	}

	$user_id = $subscription->get_user_id();
	
	if ( empty( $user_id ) ) {
		return;
	}

	// Get subscription details
	$subscription_products = array();
	$subscription_value = 0;
	
	foreach ( $subscription->get_items() as $item ) {
		$product = $item->get_product();
		if ( $product ) {
			$subscription_products[] = $product->get_name();
			$subscription_value += $item->get_total();
		}
	}

	// Prepare data to sync
	$update_data = array(
		'subscription_signup_date'     => $subscription->get_date_created()->getTimestamp() * 1000, // HubSpot uses milliseconds
		'subscription_products'        => implode( ', ', $subscription_products ),
		'subscription_initial_value'   => $subscription_value,
		'subscription_billing_period'  => $subscription->get_billing_period(),
		'subscription_billing_interval' => $subscription->get_billing_interval(),
	);

	// Optional: Add subscription start date if different from signup
	if ( $subscription->get_date( 'start' ) ) {
		$update_data['subscription_start_date'] = strtotime( $subscription->get_date( 'start' ) ) * 1000;
	}

	// Optional: Track trial information
	if ( $subscription->get_date( 'trial_end' ) ) {
		$update_data['subscription_has_trial'] = true;
		$update_data['subscription_trial_end_date'] = strtotime( $subscription->get_date( 'trial_end' ) ) * 1000;
	}

	wpf_log( 'info', $user_id, 'Tracking subscription signup for subscription #' . $subscription->get_id() );

	// Update the contact in HubSpot
	wp_fusion()->crm->update_contact( 
		wpf_get_contact_id( $user_id ), 
		$update_data, 
		false 
	);

	// Fire a custom event in HubSpot (requires WP Fusion 3.40.10+)
	if ( method_exists( wp_fusion()->crm, 'track_event' ) ) {
		
		$event_data = array(
			'event_name' => 'subscription_signup',
			'properties' => array(
				'subscription_id'       => $subscription->get_id(),
				'subscription_products' => implode( ', ', $subscription_products ),
				'subscription_value'    => $subscription_value,
				'billing_period'        => $subscription->get_billing_period(),
				'billing_interval'      => $subscription->get_billing_interval(),
				'currency'              => $subscription->get_currency(),
				'has_trial'             => ! empty( $subscription->get_date( 'trial_end' ) ),
			)
		);

		wp_fusion()->crm->track_event( 
			wpf_get_contact_id( $user_id ), 
			$event_data 
		);
		
		wpf_log( 'info', $user_id, 'Fired subscription_signup event to HubSpot' );
	}

	// Mark this subscription as tracked to prevent duplicate tracking
	$subscription->update_meta_data( '_wpf_signup_tracked', true );
	$subscription->save();

	// Optional: Also apply tags for segmentation
	$settings = wpf_get_option( 'woo_subscription_signup_tags', array() );
	
	if ( ! empty( $settings['apply_tags'] ) ) {
		wp_fusion()->user->apply_tags( $settings['apply_tags'], $user_id );
	}
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

### Required HubSpot Fields

Create these custom properties in HubSpot (Contacts → Actions → Edit properties):

1. **subscription_signup_date** (Date picker)
   - Used for the primary signup timestamp
   - Enables date range filtering in reports

2. **subscription_products** (Single-line text)
   - Stores comma-separated list of subscribed products

3. **subscription_initial_value** (Number)
   - The initial subscription value

4. **subscription_billing_period** (Single-line text)
   - Values: day, week, month, year

5. **subscription_billing_interval** (Number)
   - e.g., 1, 3, 6, 12

### Optional Fields

- **subscription_start_date** (Date picker)
- **subscription_has_trial** (Single checkbox)
- **subscription_trial_end_date** (Date picker)

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