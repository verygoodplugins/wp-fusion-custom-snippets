# LearnDash - Redirect to Previous Page

**Category:** Access Control  
**Compatibility:** WP Fusion + LearnDash  
**Source:** [GitHub Gist](https://gist.github.com/jack-arturo/f16dcdbcde3dad7fb36c57d5611a10c8)

## Description

Redirects users back to the previous page when they attempt to access a restricted LearnDash lesson, instead of using the default WP Fusion redirect URL. This provides a better user experience by maintaining navigation context.

## Use Cases

- Improve user experience in LearnDash courses
- Maintain navigation context when users hit restricted content
- Provide more intuitive redirects for course progression
- Custom learning path management

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code

```php
<?php
/**
 * Redirects users back to the previous page when accessing a restricted LearnDash lesson
 *
 * @param string $redirect_url The redirect URL
 * @param int    $post_id      The post ID
 * @return string The modified redirect URL
 */
function my_learndash_redirect( $redirect_url, $post_id ) {

    // Check if it's a LearnDash lesson
    if ( get_post_type( $post_id ) === 'sfwd-lessons' ) {
        
        // Optional, disable the generic redirect and take them back to the course:
        // return false;
        
        // Get the previous page URL
        $previous_url = wp_get_referer();
        
        // If we have a previous URL, use it
        if ( ! empty( $previous_url ) ) {
            return $previous_url;
        }
    }

    return $redirect_url;
}
add_filter( 'wpf_redirect_url', 'my_learndash_redirect', 10, 2 );
```

## How It Works

1. **Post Type Check**: Verifies the restricted content is a LearnDash lesson
2. **Referrer Detection**: Uses `wp_get_referer()` to get the previous page URL
3. **URL Validation**: Checks if a valid previous URL exists
4. **Smart Redirect**: Returns the previous page URL or falls back to default
5. **Course Context**: Maintains the user's position within the course structure

## Configuration Options

### Alternative Redirect to Course

Uncomment the `return false;` line to disable generic redirects and take users back to the course overview instead:

```php
// Disable generic redirect and take them back to the course:
return false;
```

### Extend to Other Post Types

Modify the post type check to include other LearnDash content:

```php
$learndash_post_types = array( 'sfwd-lessons', 'sfwd-topic', 'sfwd-quiz' );
if ( in_array( get_post_type( $post_id ), $learndash_post_types ) ) {
    // Redirect logic
}
```

## Compatibility Notes

- Requires LearnDash plugin
- Works with WP Fusion's access control system
- Compatible with LearnDash course progression
- Respects browser referrer headers

## Related Snippets

- [Access Control](../access-control/) - Other access control customizations
- [User Management](../user-management/) - User experience improvements
