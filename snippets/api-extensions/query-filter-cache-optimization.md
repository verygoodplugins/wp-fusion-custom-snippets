# Query Filter Cache Time Optimization

**Category:** Filters & Hooks  
**Compatibility:** WP Fusion  
**Source:** [WP Fusion Documentation](https://wpfusion.com/documentation/filters/wpf_query_filter_cache_time/)

## Description

Optimizes WP Fusion's Filter Queries caching behavior by customizing cache expiration times. The Filter Queries feature excludes restricted content from database queries, and this filter allows you to control how long the list of accessible posts is cached to balance performance with real-time access updates.

## Use Cases

- Improve performance by extending cache times for stable content
- Real-time access updates for VIP users
- Environment-specific cache configurations
- Content-type specific cache durations
- User-role based cache optimization

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code Examples

### User-Specific Cache Times

Set different cache times based on user roles or specific users. VIP users get real-time updates, while regular users get longer cache times.

```php
function user_specific_cache_time( $time, $user_id, $post_types ) {
    
    $user = get_user_by( 'id', $user_id );
    
    if ( ! $user ) {
        return $time;
    }
    
    // VIP users get real-time access updates
    if ( in_array( 'vip', $user->roles ) ) {
        return 0;
    }
    
    // Premium users get 30-second cache
    if ( in_array( 'premium_member', $user->roles ) ) {
        return 30;
    }
    
    // Regular users get 5-minute cache
    return 5 * MINUTE_IN_SECONDS;
}

add_filter( 'wpf_query_filter_cache_time', 'user_specific_cache_time', 10, 3 );
```

### Post Type Specific Cache Times

Set different cache durations based on the content type being queried. Critical content gets shorter cache times.

```php
function post_type_specific_cache_time( $time, $user_id, $post_types ) {
    
    $post_type = $post_types[0] ?? '';
    
    switch ( $post_type ) {
        case 'sfwd-courses':
        case 'sfwd-lessons':
            // LearnDash content - 30 seconds
            return 30;
            
        case 'product':
            // WooCommerce products - 2 minutes
            return 2 * MINUTE_IN_SECONDS;
            
        case 'download':
            // EDD downloads - 5 minutes
            return 5 * MINUTE_IN_SECONDS;
            
        default:
            // Default for other content
            return MINUTE_IN_SECONDS;
    }
}

add_filter( 'wpf_query_filter_cache_time', 'post_type_specific_cache_time', 10, 3 );
```

### Environment-Based Cache Times

Use different cache durations for development, staging, and production environments.

```php
function environment_based_cache_time( $time ) {
    
    if ( defined( 'WP_DEBUG' ) && WP_DEBUG ) {
        // Development - no caching for immediate testing
        return 0;
    }
    
    if ( defined( 'WP_ENVIRONMENT_TYPE' ) ) {
        switch ( WP_ENVIRONMENT_TYPE ) {
            case 'staging':
                // Staging - short cache
                return 30;
                
            case 'production':
                // Production - longer cache for performance
                return 10 * MINUTE_IN_SECONDS;
        }
    }
    
    return $time;
}

add_filter( 'wpf_query_filter_cache_time', 'environment_based_cache_time' );
```

### Extended Cache Time

Extend the cache time to one hour for better performance. Note that new content access may take up to an hour to become visible.

```php
function extend_query_filter_cache_time( $time ) {
    return HOUR_IN_SECONDS;
}

add_filter( 'wpf_query_filter_cache_time', 'extend_query_filter_cache_time' );
```

### Disable Caching Entirely

Disable caching completely for real-time access rule checking. Use with caution as this can impact performance.

```php
function disable_query_filter_cache( $time ) {
    return 0;
}

add_filter( 'wpf_query_filter_cache_time', 'disable_query_filter_cache' );
```

## Filter Parameters

- `$seconds` (int) - The number of seconds until the cache expires (default: 60)
- `$user_id` (int) - The user ID for whom the posts are being cached
- `$post_types` (array) - The post types used in the query (first post type is used in cache key)

## Performance Considerations

### Cache Behavior by Setup:

1. **No Object Caching**: Cache lasts for current page load only
2. **Object Caching** (Redis/Memcached): Cache persists across page views for the specified duration

### Recommendations:

- **High-traffic sites**: Use longer cache times (5-15 minutes)
- **Frequently changing access**: Use shorter cache times (30-60 seconds)  
- **VIP/Premium users**: Consider real-time updates (0 seconds)
- **Development**: Disable caching for immediate testing

## Related Snippets

- [API Extensions](../api-extensions/) - Performance optimizations for API calls
- [Non-Blocking Login API Calls](../api-extensions/non-blocking-login-api-calls.md) - Login performance improvements

## Notes

- Cache keys are unique per user and post type
- Returning 0 disables caching entirely
- Changes take effect immediately for new queries
- Object caching plugins (Redis, Memcached) extend cache persistence

## Troubleshooting

**Issue:** New content not appearing immediately  
**Solution:** Reduce cache time or clear object cache

**Issue:** Poor performance with real-time updates  
**Solution:** Increase cache time or implement user-specific caching

**Issue:** Cache not working  
**Solution:** Verify object caching is properly configured
