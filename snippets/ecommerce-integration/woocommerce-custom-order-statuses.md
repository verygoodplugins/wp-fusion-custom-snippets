# WooCommerce Custom Order Status Processing

**Category:** E-commerce Integration  
**Compatibility:** WP Fusion + WooCommerce  
**Source:** [WP Fusion Documentation](https://wpfusion.com/documentation/ecommerce/woocommerce/#developers)

## Description

Customizes which WooCommerce order statuses trigger WP Fusion processing, allowing you to sync customers and apply tags at different points in the order lifecycle. This includes registering additional statuses, excluding specific statuses, and configuring batch export operations.

## Use Cases

- Process orders at custom statuses (pending, on-hold, etc.)
- Skip certain order statuses from syncing
- Handle custom order statuses from plugins
- Control when customers get added to CRM
- Manage batch export operations for specific statuses
- Customize order processing workflows

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Default Behavior

By default, WP Fusion processes WooCommerce orders when their status changes to either `processing` or `completed`. At this point:
- A contact record is created in your CRM
- Tags configured on products are applied
- Customer data is synced

## Register Additional Statuses for Sync

### Basic Additional Status Registration

Process orders at additional statuses like `pending` or custom statuses:

```php
<?php

function wpf_add_custom_order_status() {

	// Process "pending" status orders
	add_action( 'woocommerce_order_status_pending', array( wp_fusion()->integrations->woocommerce, 'process_order' ) );

	// OR, alternatively, on a custom order status "tbh-unpaid"
	// add_action( 'woocommerce_order_status_tbh-unpaid', array( wp_fusion()->integrations->woocommerce, 'process_order' ) );

}

add_action( 'wp_fusion_init', 'wpf_add_custom_order_status' );
```

### Multiple Custom Statuses

Register several custom order statuses at once:

```php
function wpf_register_multiple_order_statuses() {

	$custom_statuses = array(
		'pending',
		'on-hold',
		'awaiting-shipment',
		'partially-shipped',
		'custom-status'
	);

	foreach ( $custom_statuses as $status ) {
		add_action( 'woocommerce_order_status_' . $status, array( wp_fusion()->integrations->woocommerce, 'process_order' ) );
	}

}

add_action( 'wp_fusion_init', 'wpf_register_multiple_order_statuses' );
```

### Conditional Status Processing

Only process certain statuses under specific conditions:

```php
function wpf_conditional_order_processing() {

	// Only process pending orders for VIP customers
	add_action( 'woocommerce_order_status_pending', 'wpf_process_pending_vip_orders' );

}

function wpf_process_pending_vip_orders( $order_id ) {
	
	$order = wc_get_order( $order_id );
	
	if ( ! $order ) {
		return;
	}

	$customer = $order->get_user();
	
	// Check if customer has VIP role or specific meta
	if ( $customer && ( in_array( 'vip_customer', $customer->roles ) || get_user_meta( $customer->ID, 'is_vip', true ) ) ) {
		// Process the order with WP Fusion
		wp_fusion()->integrations->woocommerce->process_order( $order_id );
	}
}

add_action( 'wp_fusion_init', 'wpf_conditional_order_processing' );
```

## Register Additional Order Statuses for Export

### Basic Export Status Registration

Include additional statuses in batch export operations:

```php
add_filter( 'wpf_woocommerce_order_statuses', function( $statuses ) {

	$statuses[] = 'pending';
	$statuses[] = 'failed';

	return $statuses;

} );
```

### Comprehensive Export Status List

Add multiple statuses for complete order lifecycle tracking:

```php
function wpf_comprehensive_export_statuses( $statuses ) {

	$additional_statuses = array(
		'pending',
		'on-hold',
		'failed',
		'cancelled',
		'refunded',
		'partially-refunded',
		'awaiting-payment',
		'awaiting-shipment'
	);

	return array_merge( $statuses, $additional_statuses );

}

add_filter( 'wpf_woocommerce_order_statuses', 'wpf_comprehensive_export_statuses' );
```

### Custom Plugin Status Support

Support statuses from WooCommerce extensions:

```php
function wpf_plugin_order_statuses( $statuses ) {

	// WooCommerce Subscriptions statuses
	$subscription_statuses = array(
		'wc-pending-cancel',
		'wc-switched'
	);

	// WooCommerce Pre-Orders statuses
	$preorder_statuses = array(
		'wc-pre-ordered'
	);

	// Custom shipping plugin statuses
	$shipping_statuses = array(
		'wc-awaiting-shipment',
		'wc-partially-shipped',
		'wc-shipped'
	);

	$all_additional = array_merge( $subscription_statuses, $preorder_statuses, $shipping_statuses );

	return array_merge( $statuses, $all_additional );

}

add_filter( 'wpf_woocommerce_order_statuses', 'wpf_plugin_order_statuses' );
```

## Exclude Order Statuses from Syncing

### Basic Status Exclusion

Prevent WP Fusion from processing specific statuses:

```php
add_filter( 'wpf_woocommerce_order_statuses', function( $statuses ) {

	$ignore_statuses = array( 'invoice' );
	return array_diff( $statuses, $ignore_statuses );

} );
```

### Multiple Status Exclusions

Exclude several statuses from processing:

```php
function wpf_exclude_multiple_statuses( $statuses ) {

	$ignore_statuses = array(
		'processing',
		'pending',
		'on-hold',
		'draft'
	);

	return array_diff( $statuses, $ignore_statuses );

}

add_filter( 'wpf_woocommerce_order_statuses', 'wpf_exclude_multiple_statuses' );
```

### Conditional Status Exclusion

Exclude statuses based on conditions:

```php
function wpf_conditional_status_exclusion( $statuses ) {

	// Only exclude processing status for subscription orders
	if ( class_exists( 'WC_Subscriptions_Order' ) ) {
		
		// Check if current context involves subscription orders
		$ignore_statuses = array( 'processing' );
		$statuses = array_diff( $statuses, $ignore_statuses );
	}

	// Exclude pending status during high traffic periods
	$current_hour = (int) date( 'H' );
	if ( $current_hour >= 12 && $current_hour <= 14 ) { // Lunch rush
		$ignore_statuses = array( 'pending' );
		$statuses = array_diff( $statuses, $ignore_statuses );
	}

	return $statuses;

}

add_filter( 'wpf_woocommerce_order_statuses', 'wpf_conditional_status_exclusion' );
```

## Advanced Implementations

### Status-Specific Tag Application

Apply different tags based on which status triggered the processing:

```php
function wpf_status_specific_processing() {

	// Different processing for different statuses
	add_action( 'woocommerce_order_status_pending', 'wpf_process_pending_orders' );
	add_action( 'woocommerce_order_status_on-hold', 'wpf_process_on_hold_orders' );

}

function wpf_process_pending_orders( $order_id ) {
	
	// Apply specific tags for pending orders
	add_filter( 'wpf_woocommerce_apply_tags_checkout', function( $apply_tags, $order ) {
		$apply_tags[] = 'Pending Payment';
		return $apply_tags;
	}, 10, 2 );

	wp_fusion()->integrations->woocommerce->process_order( $order_id );
}

function wpf_process_on_hold_orders( $order_id ) {
	
	// Apply specific tags for on-hold orders
	add_filter( 'wpf_woocommerce_apply_tags_checkout', function( $apply_tags, $order ) {
		$apply_tags[] = 'Payment Review';
		return $apply_tags;
	}, 10, 2 );

	wp_fusion()->integrations->woocommerce->process_order( $order_id );
}

add_action( 'wp_fusion_init', 'wpf_status_specific_processing' );
```

### Delayed Processing

Process orders with a delay for certain statuses:

```php
function wpf_delayed_order_processing() {

	// Process pending orders with a 5-minute delay
	add_action( 'woocommerce_order_status_pending', 'wpf_schedule_delayed_processing' );

}

function wpf_schedule_delayed_processing( $order_id ) {
	
	// Schedule processing in 5 minutes
	wp_schedule_single_event( time() + 300, 'wpf_delayed_order_processing', array( $order_id ) );
}

function wpf_process_delayed_order( $order_id ) {
	
	$order = wc_get_order( $order_id );
	
	// Only process if order is still pending
	if ( $order && $order->get_status() === 'pending' ) {
		wp_fusion()->integrations->woocommerce->process_order( $order_id );
	}
}

add_action( 'wp_fusion_init', 'wpf_delayed_order_processing' );
add_action( 'wpf_delayed_order_processing', 'wpf_process_delayed_order' );
```

### Status Change Logging

Log when orders are processed at different statuses:

```php
function wpf_log_status_processing() {

	$statuses_to_log = array( 'pending', 'on-hold', 'processing', 'completed' );

	foreach ( $statuses_to_log as $status ) {
		add_action( 'woocommerce_order_status_' . $status, 'wpf_log_order_status_change', 5 ); // Priority 5 to run before processing
	}

}

function wpf_log_order_status_change( $order_id ) {
	
	$order = wc_get_order( $order_id );
	
	if ( $order ) {
		$status = $order->get_status();
		$customer_email = $order->get_billing_email();
		
		if ( function_exists( 'wpf_log' ) ) {
			wpf_log( 'info', 0, "Order #{$order_id} status changed to '{$status}' for customer: {$customer_email}" );
		}
	}
}

add_action( 'wp_fusion_init', 'wpf_log_status_processing' );
```

## Payment Gateway Specific Processing

### PayPal Pending Orders

Handle PayPal pending payments differently:

```php
function wpf_paypal_pending_processing() {

	add_action( 'woocommerce_order_status_pending', 'wpf_handle_paypal_pending' );

}

function wpf_handle_paypal_pending( $order_id ) {
	
	$order = wc_get_order( $order_id );
	
	if ( ! $order ) {
		return;
	}

	// Only process PayPal pending orders
	if ( $order->get_payment_method() === 'paypal' ) {
		
		// Apply specific tags for PayPal pending
		add_filter( 'wpf_woocommerce_apply_tags_checkout', function( $apply_tags, $order ) {
			$apply_tags[] = 'PayPal Pending';
			return $apply_tags;
		}, 10, 2 );

		wp_fusion()->integrations->woocommerce->process_order( $order_id );
	}
}

add_action( 'wp_fusion_init', 'wpf_paypal_pending_processing' );
```

### Bank Transfer Processing

Handle bank transfer orders at pending status:

```php
function wpf_bank_transfer_processing() {

	add_action( 'woocommerce_order_status_pending', 'wpf_handle_bank_transfer_orders' );

}

function wpf_handle_bank_transfer_orders( $order_id ) {
	
	$order = wc_get_order( $order_id );
	
	if ( ! $order ) {
		return;
	}

	// Process bank transfer orders immediately at pending status
	if ( $order->get_payment_method() === 'bacs' ) {
		
		// Apply bank transfer specific tags
		add_filter( 'wpf_woocommerce_apply_tags_checkout', function( $apply_tags, $order ) {
			$apply_tags[] = 'Bank Transfer Order';
			$apply_tags[] = 'Awaiting Payment';
			return $apply_tags;
		}, 10, 2 );

		wp_fusion()->integrations->woocommerce->process_order( $order_id );
	}
}

add_action( 'wp_fusion_init', 'wpf_bank_transfer_processing' );
```

## Subscription-Specific Processing

### WooCommerce Subscriptions Integration

Handle subscription-specific order statuses:

```php
function wpf_subscription_order_processing() {

	if ( ! class_exists( 'WC_Subscriptions' ) ) {
		return;
	}

	// Process subscription renewal orders
	add_action( 'woocommerce_order_status_processing', 'wpf_handle_subscription_renewals' );

}

function wpf_handle_subscription_renewals( $order_id ) {
	
	$order = wc_get_order( $order_id );
	
	if ( ! $order ) {
		return;
	}

	// Check if this is a subscription renewal
	if ( wcs_order_contains_renewal( $order ) ) {
		
		// Apply renewal-specific tags
		add_filter( 'wpf_woocommerce_apply_tags_checkout', function( $apply_tags, $order ) {
			$apply_tags[] = 'Subscription Renewal';
			return $apply_tags;
		}, 10, 2 );

		// Process normally (this will run in addition to default processing)
		wp_fusion()->integrations->woocommerce->process_order( $order_id );
	}
}

add_action( 'wp_fusion_init', 'wpf_subscription_order_processing' );
```

## Debugging and Testing

### Debug Order Status Processing

Add logging to see which statuses are being processed:

```php
function wpf_debug_order_status_processing( $statuses ) {
	
	if ( current_user_can( 'administrator' ) && isset( $_GET['debug_order_statuses'] ) ) {
		error_log( 'WP Fusion Order Statuses: ' . print_r( $statuses, true ) );
	}

	return $statuses;
}

add_filter( 'wpf_woocommerce_order_statuses', 'wpf_debug_order_status_processing' );
```

### Test Custom Status Processing

Test if custom statuses are working:

```php
function wpf_test_custom_status_processing( $order_id ) {
	
	$order = wc_get_order( $order_id );
	
	if ( $order ) {
		$status = $order->get_status();
		error_log( "WP Fusion processing order #{$order_id} with status: {$status}" );
	}
}

// Uncomment to test
// add_action( 'woocommerce_order_status_pending', 'wpf_test_custom_status_processing' );
```

## Performance Considerations

### Avoid Processing Pending Orders in High-Traffic Sites

For high-traffic sites, processing pending orders can create performance issues:

```php
// Only process pending orders for high-value orders
function wpf_selective_pending_processing( $order_id ) {
	
	$order = wc_get_order( $order_id );
	
	if ( ! $order ) {
		return;
	}

	// Only process pending orders over $100
	if ( $order->get_total() >= 100 ) {
		wp_fusion()->integrations->woocommerce->process_order( $order_id );
	}
}

add_action( 'woocommerce_order_status_pending', 'wpf_selective_pending_processing' );
```

## Important Notes

### Order Status Hooks

- The `woocommerce_order_status_*` hooks are triggered when an order status is **changed** to that status
- They don't trigger for orders that are created with that status initially
- Use caution with `pending` status as it can create performance issues

### Batch Export Considerations

- The `wpf_woocommerce_order_statuses` filter affects batch export operations
- Including too many statuses can slow down batch exports
- Test batch operations after modifying status lists

## Troubleshooting

### Common Issues

1. **Duplicate Processing**: Orders being processed multiple times
2. **Performance Issues**: Too many pending orders being processed
3. **Missing Orders**: Custom statuses not being recognized
4. **Plugin Conflicts**: Third-party order status plugins interfering

### Debug Steps

1. Check which statuses are registered: `?debug_order_statuses=1`
2. Monitor order processing in WP Fusion logs
3. Test with small batches first
4. Verify custom statuses exist in WooCommerce

## Compatibility Notes

- Works with WooCommerce High Performance Order Storage (HPOS)
- Compatible with WooCommerce Subscriptions
- Supports custom order status plugins
- Works with all WP Fusion CRM integrations

## Related Snippets

- [E-commerce Integration](../ecommerce-integration/) - Other WooCommerce customizations
- [WooCommerce Subscription Renewal Tags](./woocommerce-subscription-renewal-tags.md) - Subscription-specific modifications
- [API Extensions](../api-extensions/) - Performance optimizations
