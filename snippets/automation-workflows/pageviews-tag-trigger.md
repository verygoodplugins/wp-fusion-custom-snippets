# Apply Tags After X Pageviews

**Category:** Automation Workflows  
**Compatibility:** WP Fusion  
**Source:** [GitHub Gist](https://gist.github.com/jack-arturo/97eda7080690cce511b538d6e2ec354b)

## Description

Tracks total page views across the entire site for logged-in users and automatically applies a WP Fusion tag when the view count exceeds a specified threshold (default: 5 pages). This is useful for engagement-based automation and user segmentation.

## Use Cases

- Segment users based on site engagement
- Trigger email sequences for active users
- Apply special offers to engaged visitors
- Track user interest and activity levels
- Create engagement-based automations in your CRM

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code

```php
<?php

/**
 * Track total page views across the site and apply a tag with WP Fusion if the view count exceeds 5.
 */

function wpf_track_total_page_views() {

        // Replace with the WP Fusion tag name you want to apply
        $wp_fusion_tag = 'Example Tag Name';

        $target_view_count = 5;

        //
        // Don't edit after here:
        //

        $wp_fusion_tag = wpf_get_tag_id( $wp_fusion_tag );

        // Ensure user is logged in
        if ( ! is_user_logged_in() ) {
                return;
        }

        $user_id     = get_current_user_id();
        $meta_key    = 'total_page_view_count';
        $page_views  = (int) get_user_meta( $user_id, $meta_key, true );

        // Increment total page view count
        ++$page_views;
        update_user_meta( $user_id, $meta_key, $page_views );

        // Apply WP Fusion tag if view count exceeds the target and tag hasn't been applied yet
        if ( $page_views > $target_view_count ) {
                wp_fusion()->user->apply_tags( array( $wp_fusion_tag ), $user_id );
        }
}

add_action( 'wp', 'wpf_track_total_page_views' );
```

## Configuration

### Required Settings

1. **Tag Name**: Replace `'Example Tag Name'` with your actual WP Fusion tag name
2. **View Count Threshold**: Change `$target_view_count = 5;` to your desired number

### Example Configuration

```php
// Apply "Engaged User" tag after 10 page views
$wp_fusion_tag = 'Engaged User';
$target_view_count = 10;
```

## How It Works

1. **User Check**: Only tracks logged-in users
2. **View Counting**: Increments page view count on each page load using `wp` action
3. **Meta Storage**: Stores count in user meta as `total_page_view_count`
4. **Tag Application**: Applies specified tag when threshold is exceeded
5. **One-Time Trigger**: Tag is applied only once when threshold is first reached

## Advanced Customizations

### Multiple Thresholds

Create multiple engagement levels:

```php
function wpf_track_multiple_engagement_levels() {
    if ( ! is_user_logged_in() ) {
        return;
    }

    $user_id = get_current_user_id();
    $page_views = (int) get_user_meta( $user_id, 'total_page_view_count', true );
    ++$page_views;
    update_user_meta( $user_id, 'total_page_view_count', $page_views );

    $engagement_levels = array(
        5  => 'Low Engagement',
        15 => 'Medium Engagement',
        30 => 'High Engagement',
        50 => 'Super Engaged'
    );

    foreach ( $engagement_levels as $threshold => $tag_name ) {
        if ( $page_views == $threshold ) {
            $tag_id = wpf_get_tag_id( $tag_name );
            wp_fusion()->user->apply_tags( array( $tag_id ), $user_id );
        }
    }
}
```

### Exclude Specific Pages

Skip certain pages from tracking:

```php
// Add this check at the beginning of the function
$excluded_pages = array( 'checkout', 'cart', 'my-account' );
if ( is_page( $excluded_pages ) ) {
    return;
}
```

## Compatibility Notes

- Only tracks logged-in users
- Requires WP Fusion to be active
- Uses WordPress user meta for storage
- Triggers on every page load via `wp` action

## Performance Considerations

- Minimal database impact (one user meta update per page view)
- Efficient tag checking prevents duplicate applications
- Consider caching implications for high-traffic sites

## Related Snippets

- [Visit-Based Tag Application](./visit-based-tag-trigger.md) - Track visits to specific posts
- [User Management](../user-management/) - Other user tracking features
- [Automation Workflows](../automation-workflows/) - Related automation triggers
