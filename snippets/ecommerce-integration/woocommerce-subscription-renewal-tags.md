# WooCommerce Subscription Renewal - Don't Apply Tags

**Category:** E-commerce Integration  
**Compatibility:** WP Fusion + WooCommerce Subscriptions  
**Source:** [GitHub Gist](https://gist.github.com/jack-arturo/45b44c586ad1912e37202c022e5b3dd3)

## Description

Prevents WP Fusion from applying "active" tags during WooCommerce subscription renewals. This is useful when you want different behavior for new subscriptions vs. renewal payments.

## Use Cases

- Prevent duplicate tag applications on subscription renewals
- Different automation workflows for new vs. renewed subscriptions
- Avoid triggering unnecessary automations during renewal processing

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code

```php
<?php
/**
 * Don't apply tags on renewal
 * 
 * @param array $apply_tags The tags to apply.
 * @param string $status The status of the subscription.
 * @return array The tags to apply.
 */
function dont_apply_tags_on_renewal( $apply_tags, $status ) {

	// Only run on subscription status changes
	if ( ! did_action( 'woocommerce_subscription_status_updated' ) ) {
		return $apply_tags;
	}

	// Get the subscription from the current action
	$subscription = wcs_get_subscription( func_get_args()[2] );

	if ( empty( $subscription ) ) {
		return $apply_tags;
	}

	// If this is a renewal, don't apply tags
	if ( $subscription->get_meta( '_wpf_renewal_in_progress' ) ) {
		wpf_log( 'info', 0, 'Not applying tags during subscription renewal.' );
		return array();
	}

	return $apply_tags;

}

add_filter( 'wpf_woocommerce_subscription_status_apply_tags', 'dont_apply_tags_on_renewal', 10, 3 );
```

## How It Works

1. **Hook Detection**: Checks if the function is running during a subscription status update
2. **Subscription Validation**: Retrieves and validates the subscription object
3. **Renewal Check**: Looks for the renewal meta flag to determine if this is a renewal
4. **Tag Prevention**: Returns empty array instead of tags during renewals
5. **Logging**: Logs the action for debugging purposes

## Configuration

No additional configuration required. The snippet automatically detects renewal scenarios and prevents tag application.

## Compatibility Notes

- Requires WooCommerce Subscriptions plugin
- Compatible with WP Fusion's WooCommerce integration
- Works with all subscription status changes

## Related Snippets

- [E-commerce Integration](../ecommerce-integration/) - Other WooCommerce customizations
- [Automation Workflows](../automation-workflows/) - Related automation controls
