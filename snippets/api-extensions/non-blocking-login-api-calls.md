# Non-Blocking API Calls During Login

**Category:** API Extensions  
**Compatibility:** WP Fusion  
**Source:** [GitHub Gist](https://gist.github.com/jack-arturo/0ec8942f06c0fb781594913be93af519)

## Description

Makes all HTTP API calls non-blocking during user login to prevent slow CRM APIs from delaying the login process. This significantly improves user experience when your CRM has response time issues.

## Use Cases

- Improve login performance with slow CRM APIs
- Prevent login timeouts due to CRM connectivity issues
- Enhance user experience during peak traffic
- Handle unreliable third-party API connections
- Maintain site responsiveness during login processes

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code

```php
<?php

/**
 * Sends API calls non-blocking during login.
 *
 * This is useful if your CRM has a slow API, it will allow the user to log in
 * without any noticable delay. However, if the API is offline or fails to process
 * the request, no error will be logged and there will be no indication it failed.
 *
 * @param array $parsed_args The HTTP API request args.
 */
function set_login_to_async( $parsed_args ) {

        if ( did_action( 'wp_login' ) ) {
                $parsed_args['blocking'] = false;
        }

        return $parsed_args;

}

add_filter( 'http_request_args', 'set_login_to_async', 100 );
```

## How It Works

1. **Login Detection**: Uses `did_action( 'wp_login' )` to detect when login has occurred
2. **HTTP Modification**: Modifies all HTTP requests to be non-blocking during login
3. **Performance Boost**: Allows login to complete without waiting for API responses
4. **Filter Priority**: Uses priority 100 to run after other modifications

## Important Considerations

### ⚠️ Trade-offs

**Benefits:**
- Faster login experience
- No login delays from slow APIs
- Better user experience
- Prevents timeouts

**Limitations:**
- No error logging for failed API calls
- No indication if sync fails
- Potential data inconsistency if APIs fail
- Debugging becomes more difficult

### When to Use

**Good for:**
- Sites with consistently slow CRM APIs
- High-traffic sites where login speed is critical
- CRMs with frequent connectivity issues
- User experience is prioritized over data consistency

**Avoid when:**
- Data synchronization is critical
- You need immediate error feedback
- CRM responses are generally fast
- Debugging API issues frequently

## Advanced Customizations

### Selective Non-Blocking

Only make specific APIs non-blocking:

```php
function selective_non_blocking_apis( $parsed_args, $url ) {
    
    if ( ! did_action( 'wp_login' ) ) {
        return $parsed_args;
    }
    
    // List of APIs to make non-blocking
    $non_blocking_apis = array(
        'infusionsoft.com',
        'activehosted.com',
        'mailchimp.com'
    );
    
    foreach ( $non_blocking_apis as $api_domain ) {
        if ( strpos( $url, $api_domain ) !== false ) {
            $parsed_args['blocking'] = false;
            break;
        }
    }
    
    return $parsed_args;
}

add_filter( 'http_request_args', 'selective_non_blocking_apis', 100, 2 );
```

### Time-Limited Non-Blocking

Only use non-blocking for a short period after login:

```php
function time_limited_non_blocking( $parsed_args ) {
    
    // Only non-blocking for 30 seconds after login
    $login_time = get_transient( 'wpf_user_login_time_' . get_current_user_id() );
    
    if ( $login_time && ( time() - $login_time ) < 30 ) {
        $parsed_args['blocking'] = false;
    }
    
    return $parsed_args;
}

// Store login time
function store_login_time( $user_login, $user ) {
    set_transient( 'wpf_user_login_time_' . $user->ID, time(), 60 );
}

add_filter( 'http_request_args', 'time_limited_non_blocking', 100 );
add_action( 'wp_login', 'store_login_time', 10, 2 );
```

### Logging Non-Blocking Calls

Add basic logging for non-blocking requests:

```php
function log_non_blocking_calls( $parsed_args, $url ) {
    
    if ( did_action( 'wp_login' ) ) {
        $parsed_args['blocking'] = false;
        
        // Log the non-blocking call
        if ( function_exists( 'wpf_log' ) ) {
            wpf_log( 'info', 0, 'Non-blocking API call during login: ' . $url );
        }
    }
    
    return $parsed_args;
}

add_filter( 'http_request_args', 'log_non_blocking_calls', 100, 2 );
```

## Monitoring and Testing

### Test Login Performance

```php
// Add to functions.php temporarily for testing
function log_login_performance() {
    $start_time = microtime( true );
    
    add_action( 'wp_login', function() use ( $start_time ) {
        $end_time = microtime( true );
        $duration = $end_time - $start_time;
        error_log( "Login duration: " . $duration . " seconds" );
    }, 999 );
}
add_action( 'init', 'log_login_performance' );
```

### Monitor API Success

Check your CRM for successful data sync after implementing this snippet to ensure data integrity.

## Compatibility Notes

- Works with all CRM integrations
- Compatible with WP Fusion's API system
- May affect other plugins using HTTP requests during login
- High priority filter (100) to run last

## Related Snippets

- [API Extensions](../api-extensions/) - Other API modifications
- [Debugging & Logging](../debugging-logging/) - API monitoring tools
- [Authentication & Login](../authentication-login/) - Login customizations
