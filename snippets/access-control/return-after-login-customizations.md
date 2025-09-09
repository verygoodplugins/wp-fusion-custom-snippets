# Return After Login URL Customizations

**Category:** Filters & Hooks  
**Compatibility:** WP Fusion  
**Source:** [WP Fusion Documentation](https://wpfusion.com/documentation/filters/wpf_return_after_login_url/)

## Description

Customizes the redirect URL when users return to restricted content after logging in using WP Fusion's Return After Login feature. These examples show various ways to modify the redirect behavior based on user roles, content types, and other conditions.

## Use Cases

- Role-based redirects after login
- Content-specific redirect customizations
- Adding tracking parameters to return URLs
- Integration with LMS platforms like LearnDash
- Custom redirect logic for different post types

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code Examples

### Role-Based Redirect After Login

Redirect users to different pages based on their WordPress user role after successfully accessing restricted content.

```php
function wpf_role_based_return_redirect( $url, $post_id, $user ) {
    
    // Redirect customers to their account page
    if ( in_array( 'customer', $user->roles ) ) {
        return wc_get_page_permalink( 'myaccount' );
    }
    
    // Redirect subscribers to a welcome page
    if ( in_array( 'subscriber', $user->roles ) ) {
        return home_url( '/welcome/' );
    }
    
    // Redirect administrators to the admin dashboard
    if ( in_array( 'administrator', $user->roles ) ) {
        return admin_url();
    }
    
    return $url;
}

add_filter( 'wpf_return_after_login_url', 'wpf_role_based_return_redirect', 10, 3 );
```

### Add Tracking Parameters to Return URL

Add UTM parameters or other tracking data to the return URL for analytics purposes.

```php
function wpf_add_tracking_to_return_url( $url, $post_id, $user ) {
    
    $tracking_params = array(
        'utm_source'   => 'wp_fusion',
        'utm_medium'   => 'return_login',
        'utm_campaign' => 'content_access',
        'user_id'      => $user->ID,
        'post_id'      => $post_id
    );
    
    return add_query_arg( $tracking_params, $url );
}

add_filter( 'wpf_return_after_login_url', 'wpf_add_tracking_to_return_url', 10, 3 );
```

### LearnDash Course Overview Redirect

When users access a restricted LearnDash lesson, redirect them to the course overview page instead of the individual lesson.

```php
function wpf_learndash_course_redirect( $url, $post_id, $user ) {
    
    // Check if this is a LearnDash lesson
    if ( 'sfwd-lessons' === get_post_type( $post_id ) ) {
        
        // Get the associated course
        $course_id = learndash_get_course_id( $post_id );
        
        if ( $course_id ) {
            return get_permalink( $course_id );
        }
    }
    
    return $url;
}

add_filter( 'wpf_return_after_login_url', 'wpf_learndash_course_redirect', 10, 3 );
```

### Download Manager PDF Viewer Redirect

Redirect users directly to the PDF viewer for Download Manager files instead of to the download page after login.

```php
function wpf_modify_return_after_login_url( $url, $post_id, $user ) {
    
    // Check if this is a Download Manager file
    if ( 'wpdmpro' === get_post_type( $post_id ) ) {
        
        // Instead of returning to the download page, go directly to the PDF viewer
        $pdf_viewer_url = home_url( '/?__wpdm_pdf_viewer=' . $post_id );
        
        return $pdf_viewer_url;
    }
    
    return $url;
}

add_filter( 'wpf_return_after_login_url', 'wpf_modify_return_after_login_url', 10, 3 );
```

## Filter Parameters

- `$url` - The URL that the user will be redirected to (typically the permalink of the restricted content)
- `$post_id` - The ID of the post that was originally restricted  
- `$user` - The WP_User object of the user who has logged in

## Related Snippets

- [IP-Based Content Unlock](../access-control/ip-based-content-unlock.md) - Bypass content protection for specific IPs
- [True Auto-Login via URL](../authentication-login/true-auto-login.md) - Seamless auto-login functionality

## Notes

- This filter runs after WP Fusion has confirmed the user now has access to the restricted content
- For redirecting users who still don't have access, use the `wpf_redirect_url` filter instead
- Always return the original `$url` parameter if your conditions aren't met
- Test thoroughly with different user roles and content types

## Troubleshooting

**Issue:** Redirect not working  
**Solution:** Ensure your conditional logic is correct and you're returning a valid URL

**Issue:** Infinite redirect loop  
**Solution:** Make sure your redirect URL isn't also protected by WP Fusion

**Issue:** Role detection not working  
**Solution:** Verify the user role names match exactly (case-sensitive)
