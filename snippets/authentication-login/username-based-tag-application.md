# Username-Based Tag Application

**Category:** Authentication & Login  
**Compatibility:** WP Fusion  
**Source:** [GitHub Gist](https://gist.github.com/jack-arturo/ffaf723ef190b7dcb2c17e0d8db0ac22)

## Description

Automatically applies a tag to users in the CRM using their WordPress username when they register. This creates a unique identifier tag for each user based on their login credentials.

## Use Cases

- Create unique user identifiers in CRM
- Track user-specific automations
- Generate personalized content based on username
- Debugging and user identification
- Custom user segmentation strategies

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code

```php
<?php

function tag_based_on_username( $user_id ) {

        $user = get_user_by( 'id', $user_id );

        wp_fusion()->user->apply_tags( array( $user->user_login ), $user_id );

}

add_action( 'wpf_user_created', 'tag_based_on_username' );
```

## How It Works

1. **User Creation Hook**: Triggers when WP Fusion creates a new user
2. **User Lookup**: Gets the user object by ID
3. **Username Extraction**: Retrieves the user's login name
4. **Tag Application**: Applies the username as a tag in the CRM

## Configuration Options

### Add Prefix to Username Tags

Prevent conflicts with other tags by adding a prefix:

```php
function tag_based_on_username( $user_id ) {
    $user = get_user_by( 'id', $user_id );
    $username_tag = 'user_' . $user->user_login;
    wp_fusion()->user->apply_tags( array( $username_tag ), $user_id );
}

add_action( 'wpf_user_created', 'tag_based_on_username' );
```

### Apply on Login Instead

Apply username tags when users log in rather than register:

```php
function tag_based_on_username_login( $user_login, $user ) {
    wp_fusion()->user->apply_tags( array( $user->user_login ), $user->ID );
}

add_action( 'wp_login', 'tag_based_on_username_login', 10, 2 );
```

### Sanitize Username for CRM

Clean up usernames for better CRM compatibility:

```php
function tag_based_on_username( $user_id ) {
    $user = get_user_by( 'id', $user_id );
    
    // Remove special characters and convert to lowercase
    $clean_username = preg_replace( '/[^a-zA-Z0-9]/', '_', $user->user_login );
    $clean_username = strtolower( $clean_username );
    
    wp_fusion()->user->apply_tags( array( $clean_username ), $user_id );
}

add_action( 'wpf_user_created', 'tag_based_on_username' );
```

## Advanced Implementations

### Username with Role Combination

Combine username with user role:

```php
function tag_based_on_username_role( $user_id ) {
    $user = get_user_by( 'id', $user_id );
    $user_roles = $user->roles;
    $primary_role = $user_roles[0] ?? 'subscriber';
    
    $combined_tag = $user->user_login . '_' . $primary_role;
    wp_fusion()->user->apply_tags( array( $combined_tag ), $user_id );
}

add_action( 'wpf_user_created', 'tag_based_on_username_role' );
```

### Conditional Username Tagging

Only apply username tags for specific user roles:

```php
function conditional_username_tagging( $user_id ) {
    $user = get_user_by( 'id', $user_id );
    
    // Only tag customers and subscribers
    $allowed_roles = array( 'customer', 'subscriber', 'contributor' );
    
    if ( array_intersect( $allowed_roles, $user->roles ) ) {
        wp_fusion()->user->apply_tags( array( $user->user_login ), $user_id );
    }
}

add_action( 'wpf_user_created', 'conditional_username_tagging' );
```

### Batch Apply to Existing Users

Apply username tags to all existing users:

```php
function batch_apply_username_tags() {
    // Only run for administrators
    if ( ! current_user_can( 'administrator' ) ) {
        return;
    }
    
    $users = get_users( array( 'number' => -1 ) );
    
    foreach ( $users as $user ) {
        // Check if user already has a contact ID
        if ( wpf_get_contact_id( $user->ID ) ) {
            wp_fusion()->user->apply_tags( array( $user->user_login ), $user->ID );
        }
    }
}

// Run once manually or via WP-CLI
// batch_apply_username_tags();
```

## CRM Considerations

### Tag Management

Username tags can quickly accumulate in your CRM. Consider:

- **Naming Convention**: Use consistent prefixes (e.g., `user_johnsmith`)
- **Tag Categories**: Create a "Username" category in your CRM
- **Cleanup Strategy**: Periodically review and clean unused username tags

### Automation Setup

Use username tags for:

- **Personalized Emails**: Reference specific users in automations
- **Custom Redirects**: Send users to personalized landing pages
- **Support Ticketing**: Automatically route support requests by username
- **A/B Testing**: Segment users for testing based on username patterns

## Security Considerations

### Username Privacy

Be aware that usernames might contain sensitive information:

```php
function secure_username_tagging( $user_id ) {
    $user = get_user_by( 'id', $user_id );
    
    // Hash username for privacy
    $hashed_username = 'user_' . substr( md5( $user->user_login ), 0, 8 );
    
    wp_fusion()->user->apply_tags( array( $hashed_username ), $user_id );
}

add_action( 'wpf_user_created', 'secure_username_tagging' );
```

## Troubleshooting

### Tags Not Appearing

1. **Check CRM Connection**: Ensure WP Fusion is connected to your CRM
2. **Verify Hook**: Confirm the hook is firing with logging
3. **Tag Creation**: Some CRMs require manual tag creation first

### Debug Username Tags

```php
function debug_username_tagging( $user_id ) {
    $user = get_user_by( 'id', $user_id );
    
    error_log( 'Applying username tag: ' . $user->user_login . ' for user ID: ' . $user_id );
    
    wp_fusion()->user->apply_tags( array( $user->user_login ), $user_id );
}

add_action( 'wpf_user_created', 'debug_username_tagging' );
```

## Compatibility Notes

- Works with all WP Fusion CRM integrations
- Compatible with custom user registration forms
- Supports WooCommerce customer registration
- Works with membership plugins

## Related Snippets

- [Authentication & Login](../authentication-login/) - Other login customizations
- [User Management](../user-management/) - User data management
- [Automation Workflows](../automation-workflows/) - Tag-based automations
