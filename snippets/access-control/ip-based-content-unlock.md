# IP-Based Content Unlock

**Category:** Access Control  
**Compatibility:** WP Fusion  
**Source:** [GitHub Gist](https://gist.github.com/jack-arturo/932218b7b30c96f8f135225207c39fd0)

## Description

Bypasses WP Fusion's content protection when someone accesses your site from a specific IP address. This is useful for allowing office access, developer testing, or VIP user access without requiring CRM tags.

## Use Cases

- Office or company network access bypass
- Developer and testing environment access
- VIP client access from specific locations
- Emergency content access during CRM downtime
- Demo site access for prospects
- Client preview access during development

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code

```php
<?php

function unlock_by_ip( $can_access, $user_id, $post_id ) {

        if ( ! empty( $_SERVER['HTTP_CLIENT_IP'] ) ) {

                // IP from share internet
                $ip = $_SERVER['HTTP_CLIENT_IP'];

        } elseif ( ! empty( $_SERVER['HTTP_X_FORWARDED_FOR'] ) ) {

                // IP passed from proxy
                $ip = $_SERVER['HTTP_X_FORWARDED_FOR'];

        } else {

                // Normal IP
                $ip = $_SERVER['REMOTE_ADDR'];

        }

        if ( $ip == '123.456.789.001' ) {
                $can_access = true;
        }

        return $can_access;

}

add_filter( 'wpf_user_can_access', 'unlock_by_ip', 10, 3 );
```

## Configuration

### Set Your IP Address

Replace `'123.456.789.001'` with your actual IP address:

```php
// Find your IP at whatismyipaddress.com
if ( $ip == '192.168.1.100' ) {
    $can_access = true;
}
```

### Multiple IP Addresses

Allow access from multiple IPs:

```php
function unlock_by_ip( $can_access, $user_id, $post_id ) {
    
    // Get user's IP address
    if ( ! empty( $_SERVER['HTTP_CLIENT_IP'] ) ) {
        $ip = $_SERVER['HTTP_CLIENT_IP'];
    } elseif ( ! empty( $_SERVER['HTTP_X_FORWARDED_FOR'] ) ) {
        $ip = $_SERVER['HTTP_X_FORWARDED_FOR'];
    } else {
        $ip = $_SERVER['REMOTE_ADDR'];
    }
    
    // Array of allowed IPs
    $allowed_ips = array(
        '192.168.1.100',    // Office IP
        '203.0.113.45',     // Home office
        '198.51.100.78',    // Client office
        '10.0.0.5'          // Local development
    );
    
    if ( in_array( $ip, $allowed_ips ) ) {
        $can_access = true;
    }
    
    return $can_access;
}

add_filter( 'wpf_user_can_access', 'unlock_by_ip', 10, 3 );
```

## Advanced Implementations

### IP Range Support

Allow entire IP ranges:

```php
function unlock_by_ip_range( $can_access, $user_id, $post_id ) {
    
    // Get user's IP
    $ip = $_SERVER['REMOTE_ADDR'] ?? '';
    
    // Allow entire office network (192.168.1.x)
    if ( strpos( $ip, '192.168.1.' ) === 0 ) {
        $can_access = true;
    }
    
    // Allow specific range 203.0.113.1 to 203.0.113.50
    $ip_long = ip2long( $ip );
    $range_start = ip2long( '203.0.113.1' );
    $range_end = ip2long( '203.0.113.50' );
    
    if ( $ip_long >= $range_start && $ip_long <= $range_end ) {
        $can_access = true;
    }
    
    return $can_access;
}

add_filter( 'wpf_user_can_access', 'unlock_by_ip_range', 10, 3 );
```

### Conditional IP Unlock

Only unlock specific content types or posts:

```php
function conditional_ip_unlock( $can_access, $user_id, $post_id ) {
    
    // Only allow IP unlock for specific post types
    $allowed_post_types = array( 'page', 'documentation' );
    
    if ( ! in_array( get_post_type( $post_id ), $allowed_post_types ) ) {
        return $can_access;
    }
    
    // Get IP address
    $ip = $_SERVER['REMOTE_ADDR'] ?? '';
    
    // Office IP gets access to documentation
    if ( $ip == '192.168.1.100' ) {
        $can_access = true;
    }
    
    return $can_access;
}

add_filter( 'wpf_user_can_access', 'conditional_ip_unlock', 10, 3 );
```

### Time-Based IP Access

Combine IP access with time restrictions:

```php
function time_based_ip_unlock( $can_access, $user_id, $post_id ) {
    
    $ip = $_SERVER['REMOTE_ADDR'] ?? '';
    $current_hour = (int) date( 'H' );
    
    // Office IP only gets access during business hours (9 AM - 6 PM)
    if ( $ip == '192.168.1.100' && $current_hour >= 9 && $current_hour <= 18 ) {
        $can_access = true;
    }
    
    return $can_access;
}

add_filter( 'wpf_user_can_access', 'time_based_ip_unlock', 10, 3 );
```

### Logging IP Access

Track when IP-based access is granted:

```php
function logged_ip_unlock( $can_access, $user_id, $post_id ) {
    
    $ip = $_SERVER['REMOTE_ADDR'] ?? '';
    $allowed_ips = array( '192.168.1.100', '203.0.113.45' );
    
    if ( in_array( $ip, $allowed_ips ) ) {
        $can_access = true;
        
        // Log the access
        if ( function_exists( 'wpf_log' ) ) {
            wpf_log( 'info', $user_id, "IP-based access granted for post {$post_id} from IP: {$ip}" );
        }
        
        // Alternative: Use error_log
        error_log( "WP Fusion IP Access: User {$user_id} accessed post {$post_id} from IP {$ip}" );
    }
    
    return $can_access;
}

add_filter( 'wpf_user_can_access', 'logged_ip_unlock', 10, 3 );
```

## Security Considerations

### IP Spoofing Protection

Be aware that IP addresses can be spoofed:

```php
function secure_ip_unlock( $can_access, $user_id, $post_id ) {
    
    // Only allow IP unlock for logged-in users
    if ( ! is_user_logged_in() ) {
        return $can_access;
    }
    
    // Additional security checks
    $ip = $_SERVER['REMOTE_ADDR'] ?? '';
    
    // Verify the IP is from a trusted source
    if ( $ip == '192.168.1.100' && isset( $_SERVER['HTTP_USER_AGENT'] ) ) {
        $can_access = true;
    }
    
    return $can_access;
}

add_filter( 'wpf_user_can_access', 'secure_ip_unlock', 10, 3 );
```

### Environment-Based Access

Different IPs for different environments:

```php
function environment_ip_unlock( $can_access, $user_id, $post_id ) {
    
    $ip = $_SERVER['REMOTE_ADDR'] ?? '';
    
    // Development environment
    if ( defined( 'WP_DEBUG' ) && WP_DEBUG ) {
        $dev_ips = array( '127.0.0.1', '::1', '192.168.1.100' );
        if ( in_array( $ip, $dev_ips ) ) {
            $can_access = true;
        }
    }
    
    // Staging environment
    if ( defined( 'WP_ENVIRONMENT_TYPE' ) && WP_ENVIRONMENT_TYPE === 'staging' ) {
        $staging_ips = array( '203.0.113.45' );
        if ( in_array( $ip, $staging_ips ) ) {
            $can_access = true;
        }
    }
    
    return $can_access;
}

add_filter( 'wpf_user_can_access', 'environment_ip_unlock', 10, 3 );
```

## Finding Your IP Address

### Methods to Find Your IP:

1. **Online Tools**: Visit [whatismyipaddress.com](https://whatismyipaddress.com)
2. **Command Line**: Run `curl ifconfig.me` in terminal
3. **WordPress Debug**: Add temporary logging to see your IP

### Debug Your IP

Add this temporarily to see your current IP:

```php
function debug_current_ip() {
    if ( current_user_can( 'administrator' ) ) {
        $ip = $_SERVER['REMOTE_ADDR'] ?? 'Unknown';
        echo '<div style="background: yellow; padding: 10px;">Your IP: ' . esc_html( $ip ) . '</div>';
    }
}
add_action( 'wp_footer', 'debug_current_ip' );
```

## Compatibility Notes

- Works with all WP Fusion content protection
- Compatible with proxy servers and load balancers
- Supports IPv4 and IPv6 addresses
- Works with CDN services (may need header adjustments)

## Related Snippets

- [Access Control](../access-control/) - Other content protection modifications
- [Debugging & Logging](../debugging-logging/) - Access monitoring tools
- [API Extensions](../api-extensions/) - CRM integration customizations
