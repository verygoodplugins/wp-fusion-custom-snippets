# True Auto-Login via URL

**Category:** Authentication & Login  
**Compatibility:** WP Fusion  
**Source:** [GitHub Gist](https://gist.github.com/jack-arturo/4e636fc0e8c68b0d30b42bf8c840290c)

## Description

Enables true auto-login functionality where users can be automatically logged in via a specially formatted URL containing their contact ID, email address, and access key. This provides seamless authentication from email campaigns or external systems.

## Use Cases

- Email campaign click-through authentication
- Seamless login from CRM links
- Customer portal access from external systems
- Support ticket auto-authentication
- Marketing automation login flows
- Mobile app to website authentication

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code

```php
<?php

// Disable "Allow URL Login" in WP Fusion's settings for this to work properly.

/**
 * True auto login. Format your URLs like: https://mysite.com/?cid=%CONTACTID%&email=%EMAILADDRESS%&key=%ACCESSKEY%                                                                                               
 *
 * Where %CONTACTID% is the ID of the contact
 * %EMAIL% is the email address of the contact
 * and %ACCESSKEY% is your access key from the bottom of the WP Fusion General settings tab
 */

function wpf_true_auto_login() {

    if ( ! isset( $_GET['cid'] ) || ! isset( $_GET['email'] ) || ! isset( $_GET['key'] ) ) {
        return;
    }

    if ( is_user_logged_in() ) {
        return;
    }

    $access_key = wpf_get_setting( 'access_key' );

    if ( empty( $access_key ) || $_GET['key'] != $access_key ) {
        return;
    }

    $contact_id = sanitize_text_field( $_GET['cid'] );
    $email      = sanitize_email( $_GET['email'] );

    $user_id = wp_fusion()->user->get_user_id( $contact_id );

    if ( empty( $user_id ) ) {

        // Try to find by email.
        $user = get_user_by( 'email', $email );

        if ( ! empty( $user ) ) {
            $user_id = $user->ID;
        }
    }

    if ( ! empty( $user_id ) ) {

        $user = get_user_by( 'id', $user_id );

        if ( $user->user_email == $email ) {

            wp_set_current_user( $user_id );
            wp_set_auth_cookie( $user_id, true );

            do_action( 'wp_login', $user->user_login, $user );

            // Redirect to remove the login parameters from the URL
            $redirect_url = remove_query_arg( array( 'cid', 'email', 'key' ) );
            wp_redirect( $redirect_url );
            exit;

        }
    }
}

add_action( 'init', 'wpf_true_auto_login' );
```

## Setup Requirements

### 1. WP Fusion Configuration

**Important:** Disable the built-in "Allow URL Login" setting in WP Fusion:
1. Go to WP Fusion → Settings → General
2. Uncheck "Allow URL Login"
3. Save settings

### 2. Access Key Setup

1. Navigate to WP Fusion → Settings → General
2. Scroll to the bottom to find your Access Key
3. Copy this key for use in your URLs

## URL Format

Create auto-login URLs using this format:

```
https://yoursite.com/?cid=CONTACT_ID&email=EMAIL_ADDRESS&key=ACCESS_KEY
```

### Example URL

```
https://mysite.com/?cid=12345&email=john@example.com&key=abc123def456
```

### Email Campaign Integration

Most email platforms support merge tags for dynamic URLs:

**ActiveCampaign:**
```
https://mysite.com/?cid=%CONTACTID%&email=%EMAIL%&key=YOUR_ACCESS_KEY
```

**Infusionsoft/Keap:**
```
https://mysite.com/?cid=~Contact.Id~&email=~Contact.Email~&key=YOUR_ACCESS_KEY
```

**HubSpot:**
```
https://mysite.com/?cid={{ contact.hs_object_id }}&email={{ contact.email }}&key=YOUR_ACCESS_KEY
```

## Advanced Implementations

### Custom Redirect After Login

Redirect users to specific pages after auto-login:

```php
function wpf_true_auto_login_with_redirect() {
    
    // ... existing validation code ...
    
    if ( ! empty( $user_id ) ) {
        $user = get_user_by( 'id', $user_id );
        
        if ( $user->user_email == $email ) {
            wp_set_current_user( $user_id );
            wp_set_auth_cookie( $user_id, true );
            do_action( 'wp_login', $user->user_login, $user );
            
            // Custom redirect logic
            $redirect_url = home_url( '/dashboard/' );
            
            // Redirect based on user role
            if ( in_array( 'customer', $user->roles ) ) {
                $redirect_url = home_url( '/my-account/' );
            } elseif ( in_array( 'subscriber', $user->roles ) ) {
                $redirect_url = home_url( '/members-area/' );
            }
            
            // Allow custom redirect via URL parameter
            if ( ! empty( $_GET['redirect'] ) ) {
                $redirect_url = esc_url_raw( $_GET['redirect'] );
            }
            
            wp_redirect( $redirect_url );
            exit;
        }
    }
}

add_action( 'init', 'wpf_true_auto_login_with_redirect' );
```

### Enhanced Security

Add additional security measures:

```php
function wpf_secure_auto_login() {
    
    if ( ! isset( $_GET['cid'] ) || ! isset( $_GET['email'] ) || ! isset( $_GET['key'] ) ) {
        return;
    }

    if ( is_user_logged_in() ) {
        return;
    }

    // Check if auto-login is enabled (custom setting)
    if ( ! get_option( 'wpf_allow_auto_login', false ) ) {
        return;
    }

    // Rate limiting - only allow one auto-login per IP per minute
    $ip = $_SERVER['REMOTE_ADDR'];
    $rate_limit_key = 'wpf_auto_login_' . md5( $ip );
    
    if ( get_transient( $rate_limit_key ) ) {
        wp_die( 'Too many auto-login attempts. Please try again later.' );
    }
    
    set_transient( $rate_limit_key, true, 60 ); // 1 minute

    $access_key = wpf_get_setting( 'access_key' );
    
    if ( empty( $access_key ) || $_GET['key'] !== $access_key ) {
        // Log failed attempt
        error_log( 'WP Fusion auto-login failed: Invalid access key from IP ' . $ip );
        return;
    }

    // ... rest of login logic ...
    
    // Log successful login
    error_log( 'WP Fusion auto-login successful for user: ' . $email );
}

add_action( 'init', 'wpf_secure_auto_login' );
```

### Temporary Login Links

Create time-limited auto-login links:

```php
function wpf_temporary_auto_login() {
    
    if ( ! isset( $_GET['cid'] ) || ! isset( $_GET['email'] ) || ! isset( $_GET['key'] ) || ! isset( $_GET['expires'] ) ) {
        return;
    }

    if ( is_user_logged_in() ) {
        return;
    }

    // Check if link has expired
    $expires = (int) $_GET['expires'];
    if ( time() > $expires ) {
        wp_die( 'This login link has expired.' );
    }

    // ... rest of validation and login logic ...
}

add_action( 'init', 'wpf_temporary_auto_login' );
```

### Generate Auto-Login URLs

Helper function to generate auto-login URLs:

```php
function wpf_generate_auto_login_url( $user_id, $redirect_to = '' ) {
    
    $contact_id = wpf_get_contact_id( $user_id );
    $user = get_user_by( 'id', $user_id );
    $access_key = wpf_get_setting( 'access_key' );
    
    if ( empty( $contact_id ) || empty( $user ) || empty( $access_key ) ) {
        return false;
    }
    
    $args = array(
        'cid'   => $contact_id,
        'email' => $user->user_email,
        'key'   => $access_key
    );
    
    if ( ! empty( $redirect_to ) ) {
        $args['redirect'] = urlencode( $redirect_to );
    }
    
    return add_query_arg( $args, home_url() );
}

// Usage example:
// $login_url = wpf_generate_auto_login_url( 123, '/premium-content/' );
```

## Security Considerations

### Important Security Notes

1. **Access Key Protection**: Keep your access key secure and regenerate it periodically
2. **HTTPS Required**: Always use HTTPS to protect login credentials in transit
3. **Rate Limiting**: Implement rate limiting to prevent abuse
4. **Logging**: Log auto-login attempts for security monitoring
5. **Expiration**: Consider time-limited links for sensitive applications

### Best Practices

- Use auto-login sparingly and only when necessary
- Monitor logs for suspicious activity
- Consider IP restrictions for high-security applications
- Implement user consent for auto-login functionality
- Regular security audits of auto-login usage

## Troubleshooting

### Common Issues

1. **Login Fails**: Verify access key matches WP Fusion settings
2. **Email Mismatch**: Ensure email in URL matches user's WordPress email
3. **Contact ID Issues**: Verify contact ID exists in CRM and WP Fusion
4. **Already Logged In**: Function won't work if user is already authenticated

### Debug Auto-Login

```php
function debug_auto_login() {
    if ( isset( $_GET['debug_login'] ) && current_user_can( 'administrator' ) ) {
        echo '<pre>';
        echo 'Contact ID: ' . ( $_GET['cid'] ?? 'Not set' ) . "\n";
        echo 'Email: ' . ( $_GET['email'] ?? 'Not set' ) . "\n";
        echo 'Key: ' . ( $_GET['key'] ?? 'Not set' ) . "\n";
        echo 'Access Key: ' . wpf_get_setting( 'access_key' ) . "\n";
        echo 'User ID: ' . wp_fusion()->user->get_user_id( $_GET['cid'] ?? '' ) . "\n";
        echo '</pre>';
    }
}
add_action( 'wp_footer', 'debug_auto_login' );
```

## Compatibility Notes

- Requires WP Fusion to be active and configured
- Works with all WP Fusion CRM integrations
- Compatible with membership plugins
- Works with caching plugins (URLs are dynamic)

## Related Snippets

- [Authentication & Login](../authentication-login/) - Other login customizations
- [User Management](../user-management/) - User handling features
- [API Extensions](../api-extensions/) - CRM integration enhancements
