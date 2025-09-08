# Apply Tags Based on Post/Page Visits

**Category:** Automation Workflows  
**Compatibility:** WP Fusion  
**Source:** [GitHub Gist](https://gist.github.com/jack-arturo/59bcbf0bd6c2e077c336d60980122d65)

## Description

Tracks visits to specific posts or pages for logged-in users and automatically applies a WP Fusion tag when the visit count exceeds a specified threshold (default: 5 visits). This is perfect for identifying highly engaged users interested in specific content.

## Use Cases

- Identify users highly interested in specific products/services
- Segment users based on content consumption patterns
- Trigger targeted email sequences for specific topics
- Track engagement with key landing pages
- Create interest-based automation workflows

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code

```php
<?php
/**
 * Track visits to a specific post or page and apply a tag with WP Fusion if the visit count exceeds 5.
 */

function wpf_track_post_visits() {

        // Replace with the IDs of the posts or pages you want to track
        $target_post_ids = array( 123, 456 );

        // Replace with the WP Fusion tag name you want to apply
        $wp_fusion_tag = 'Example Tag Name';

        // Replace with how many visits should apply the tag
        $target_visit_count = 5;

        //
        // Don't edit after here:
        //

        $wp_fusion_tag = wpf_get_tag_id( $wp_fusion_tag );

        // Ensure user is logged in
        if ( ! is_user_logged_in() ) {
                return;
        }

        // Ensure we're on either a post or page
        if ( ! is_singular( 'post' ) && ! is_singular( 'page' ) ) {
                return;
        }

        // Ensure we're on the right post or page ID
        if ( ! in_array( get_the_ID(), $target_post_ids ) ) {
                return;
        }

        $user_id     = get_current_user_id();
        $meta_key    = 'post_visit_count_' . get_the_ID();
        $visit_count = (int) get_user_meta( $user_id, $meta_key, true );

        // Increment visit count
        ++$visit_count;
        update_user_meta( $user_id, $meta_key, $visit_count );

        // Apply WP Fusion tag if visit count exceeds 5 and tag hasn't been applied yet
        if ( $visit_count > $target_visit_count ) {
                wp_fusion()->user->apply_tags( array( $wp_fusion_tag ), $user_id );
        }
}

add_action( 'wp', 'wpf_track_post_visits' );
```

## Configuration

### Required Settings

1. **Target Post IDs**: Replace the array `array( 123, 456 )` with your actual post/page IDs
2. **Tag Name**: Replace `'Example Tag Name'` with your actual WP Fusion tag name
3. **Visit Threshold**: Change `$target_visit_count = 5;` to your desired number

### Finding Post/Page IDs

You can find post IDs by:
- Hovering over posts in the admin list (shows in URL)
- Editing a post and looking at the URL (`post=123`)
- Using a plugin like "Show IDs by 99 Robots"

### Example Configuration

```php
// Track visits to pricing page (ID: 42) and services page (ID: 67)
$target_post_ids = array( 42, 67 );

// Apply "Highly Interested" tag after 3 visits
$wp_fusion_tag = 'Highly Interested';
$target_visit_count = 3;
```

## How It Works

1. **User Check**: Only tracks logged-in users
2. **Content Type Check**: Ensures we're on a post or page
3. **ID Matching**: Verifies the current post/page is in the target list
4. **Visit Counting**: Increments visit count using post-specific user meta
5. **Tag Application**: Applies tag when threshold is reached

## Advanced Customizations

### Multiple Posts with Different Tags

Track different content with different tags:

```php
function wpf_track_multiple_post_interests() {
    if ( ! is_user_logged_in() || ! is_singular() ) {
        return;
    }

    $tracking_config = array(
        42 => array( 'tag' => 'Pricing Interest', 'threshold' => 3 ),
        67 => array( 'tag' => 'Services Interest', 'threshold' => 2 ),
        89 => array( 'tag' => 'Blog Reader', 'threshold' => 5 )
    );

    $current_post_id = get_the_ID();
    
    if ( ! isset( $tracking_config[ $current_post_id ] ) ) {
        return;
    }

    $config = $tracking_config[ $current_post_id ];
    $user_id = get_current_user_id();
    $meta_key = 'post_visit_count_' . $current_post_id;
    $visit_count = (int) get_user_meta( $user_id, $meta_key, true );
    
    ++$visit_count;
    update_user_meta( $user_id, $meta_key, $visit_count );

    if ( $visit_count >= $config['threshold'] ) {
        $tag_id = wpf_get_tag_id( $config['tag'] );
        wp_fusion()->user->apply_tags( array( $tag_id ), $user_id );
    }
}
```

### Include Custom Post Types

Extend tracking to custom post types:

```php
// Replace the post type check with:
if ( ! is_singular( array( 'post', 'page', 'product', 'course' ) ) ) {
    return;
}
```

### Time-Based Visit Tracking

Track visits within specific time periods:

```php
// Add this after getting visit count
$last_visit = get_user_meta( $user_id, $meta_key . '_last_visit', true );
$current_time = current_time( 'timestamp' );

// Only count if last visit was more than 1 hour ago
if ( empty( $last_visit ) || ( $current_time - $last_visit ) > 3600 ) {
    ++$visit_count;
    update_user_meta( $user_id, $meta_key, $visit_count );
    update_user_meta( $user_id, $meta_key . '_last_visit', $current_time );
}
```

## Data Storage

Visit counts are stored as user meta with the key pattern:
- `post_visit_count_{POST_ID}`

This allows for:
- Individual tracking per post/page
- Easy querying and reporting
- Clean data organization

## Compatibility Notes

- Only tracks logged-in users
- Works with posts, pages, and custom post types
- Requires WP Fusion to be active
- Uses WordPress user meta for storage

## Related Snippets

- [Pageviews Tag Trigger](./pageviews-tag-trigger.md) - Track total site engagement
- [User Management](../user-management/) - Other user tracking features
- [Automation Workflows](../automation-workflows/) - Related automation triggers
