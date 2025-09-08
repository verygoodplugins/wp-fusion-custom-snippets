# Extend HTTP Timeout

**Category:** API Extensions  
**Compatibility:** WP Fusion  
**Source:** [GitHub Gist](https://gist.github.com/jack-arturo/77df68c31084eb8f2f0558ae52cadf69)

## Description

Extends the HTTP timeout for all WordPress HTTP requests to 60 seconds. This prevents timeout errors when communicating with slow CRM APIs or during high-traffic periods.

## Use Cases

- Prevent timeout errors with slow CRM APIs
- Handle large data sync operations
- Improve reliability during peak traffic
- Support complex API operations that take longer
- Resolve intermittent connection issues
- Handle bulk import/export operations

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code

```php
<?php

function wpf_extend_timeout( $args, $url ) {

        $args['timeout'] = 60;

        return $args;

}

add_filter( 'http_request_args', 'wpf_extend_timeout', 100, 2 );
```

## How It Works

1. **HTTP Filter**: Hooks into `http_request_args` filter
2. **Timeout Modification**: Changes the default timeout from 30 to 60 seconds
3. **Universal Application**: Affects all HTTP requests from WordPress
4. **High Priority**: Uses priority 100 to override other timeout settings

## Configuration Options

### Custom Timeout Duration

Adjust the timeout to your needs:

```php
function wpf_extend_timeout( $args, $url ) {
    // 30 seconds (default)
    $args['timeout'] = 30;
    
    // 2 minutes for very slow APIs
    $args['timeout'] = 120;
    
    // 5 minutes for bulk operations
    $args['timeout'] = 300;
    
    return $args;
}

add_filter( 'http_request_args', 'wpf_extend_timeout', 100, 2 );
```

### CRM-Specific Timeouts

Apply different timeouts based on the destination URL:

```php
function wpf_crm_specific_timeout( $args, $url ) {
    
    // Default timeout
    $default_timeout = 30;
    
    // CRM-specific timeouts
    if ( strpos( $url, 'salesforce.com' ) !== false ) {
        $args['timeout'] = 90;  // Salesforce can be slow
    } elseif ( strpos( $url, 'hubapi.com' ) !== false ) {
        $args['timeout'] = 60;  // HubSpot moderate timeout
    } elseif ( strpos( $url, 'infusionsoft.com' ) !== false ) {
        $args['timeout'] = 120; // Infusionsoft often slow
    } elseif ( strpos( $url, 'activehosted.com' ) !== false ) {
        $args['timeout'] = 45;  // ActiveCampaign usually fast
    } else {
        $args['timeout'] = $default_timeout;
    }
    
    return $args;
}

add_filter( 'http_request_args', 'wpf_crm_specific_timeout', 100, 2 );
```

### Conditional Timeout Extension

Only extend timeout for specific operations:

```php
function wpf_conditional_timeout( $args, $url ) {
    
    // Only extend timeout for WP Fusion operations
    if ( strpos( $url, wp_fusion()->crm->url ) !== false ) {
        $args['timeout'] = 90;
    }
    
    // Extend timeout during bulk operations
    if ( doing_action( 'wpf_batch_process' ) || doing_action( 'wpf_import_users' ) ) {
        $args['timeout'] = 180;
    }
    
    // Extend timeout for specific HTTP methods
    if ( isset( $args['method'] ) && in_array( $args['method'], array( 'POST', 'PUT', 'PATCH' ) ) ) {
        $args['timeout'] = 75;
    }
    
    return $args;
}

add_filter( 'http_request_args', 'wpf_conditional_timeout', 100, 2 );
```

## Advanced Implementations

### Dynamic Timeout Based on Data Size

Adjust timeout based on request size:

```php
function wpf_dynamic_timeout( $args, $url ) {
    
    $base_timeout = 30;
    
    // Increase timeout based on request body size
    if ( isset( $args['body'] ) ) {
        $body_size = strlen( $args['body'] );
        
        if ( $body_size > 10000 ) {      // > 10KB
            $args['timeout'] = 90;
        } elseif ( $body_size > 5000 ) { // > 5KB
            $args['timeout'] = 60;
        } else {
            $args['timeout'] = $base_timeout;
        }
    }
    
    return $args;
}

add_filter( 'http_request_args', 'wpf_dynamic_timeout', 100, 2 );
```

### Retry Logic with Timeout

Combine extended timeout with retry logic:

```php
function wpf_timeout_with_retry( $args, $url ) {
    
    // Extend timeout
    $args['timeout'] = 60;
    
    // Add retry logic (this would need additional implementation)
    $args['retry_count'] = 3;
    
    return $args;
}

// Custom retry implementation
function wpf_http_request_with_retry( $response, $args, $url ) {
    
    if ( is_wp_error( $response ) && strpos( $response->get_error_message(), 'timeout' ) !== false ) {
        
        // Log the timeout
        if ( function_exists( 'wpf_log' ) ) {
            wpf_log( 'warning', 0, 'HTTP timeout for URL: ' . $url );
        }
        
        // Could implement retry logic here
    }
    
    return $response;
}

add_filter( 'http_request_args', 'wpf_timeout_with_retry', 100, 2 );
add_filter( 'http_response', 'wpf_http_request_with_retry', 10, 3 );
```

### Environment-Based Timeouts

Different timeouts for different environments:

```php
function wpf_environment_timeout( $args, $url ) {
    
    if ( defined( 'WP_DEBUG' ) && WP_DEBUG ) {
        // Longer timeout for development
        $args['timeout'] = 120;
    } elseif ( defined( 'WP_ENVIRONMENT_TYPE' ) ) {
        switch ( WP_ENVIRONMENT_TYPE ) {
            case 'development':
                $args['timeout'] = 120;
                break;
            case 'staging':
                $args['timeout'] = 90;
                break;
            case 'production':
                $args['timeout'] = 60;
                break;
            default:
                $args['timeout'] = 30;
        }
    }
    
    return $args;
}

add_filter( 'http_request_args', 'wpf_environment_timeout', 100, 2 );
```

## Monitoring and Logging

### Log Timeout Events

Track when requests are taking too long:

```php
function wpf_log_slow_requests( $response, $args, $url ) {
    
    if ( isset( $args['_start_time'] ) ) {
        $duration = microtime( true ) - $args['_start_time'];
        
        // Log requests taking more than 30 seconds
        if ( $duration > 30 && function_exists( 'wpf_log' ) ) {
            wpf_log( 'warning', 0, "Slow HTTP request: {$duration}s for {$url}" );
        }
    }
    
    return $response;
}

function wpf_track_request_start( $args, $url ) {
    $args['_start_time'] = microtime( true );
    $args['timeout'] = 90;
    return $args;
}

add_filter( 'http_request_args', 'wpf_track_request_start', 100, 2 );
add_filter( 'http_response', 'wpf_log_slow_requests', 10, 3 );
```

### Debug Timeout Settings

Show current timeout for debugging:

```php
function wpf_debug_timeout( $args, $url ) {
    
    $args['timeout'] = 60;
    
    // Debug output for administrators
    if ( current_user_can( 'administrator' ) && isset( $_GET['debug_timeout'] ) ) {
        error_log( "HTTP Request to {$url} with timeout: {$args['timeout']}s" );
    }
    
    return $args;
}

add_filter( 'http_request_args', 'wpf_debug_timeout', 100, 2 );
```

## Performance Considerations

### Impact on Site Performance

- **Positive**: Reduces failed requests due to timeouts
- **Negative**: May increase page load time if requests hang
- **Recommendation**: Monitor actual request times and adjust accordingly

### Memory Usage

Longer timeouts can increase memory usage for concurrent requests:

```php
function wpf_memory_aware_timeout( $args, $url ) {
    
    // Check available memory
    $memory_limit = ini_get( 'memory_limit' );
    $memory_usage = memory_get_usage( true );
    
    // Reduce timeout if memory is low
    if ( $memory_usage > ( $memory_limit * 0.8 ) ) {
        $args['timeout'] = 30; // Shorter timeout when memory is low
    } else {
        $args['timeout'] = 90; // Normal extended timeout
    }
    
    return $args;
}

add_filter( 'http_request_args', 'wpf_memory_aware_timeout', 100, 2 );
```

## Troubleshooting

### Still Getting Timeouts?

1. **Check Server Limits**: Verify `max_execution_time` in PHP
2. **Network Issues**: Test CRM connectivity directly
3. **Increase Further**: Try 120+ seconds for very slow APIs
4. **Contact CRM Support**: Report slow API response times

### Debug Current Settings

```php
// Add temporarily to see current timeout settings
function debug_http_timeout( $args, $url ) {
    if ( current_user_can( 'administrator' ) ) {
        error_log( "Timeout for {$url}: " . ( $args['timeout'] ?? 'default' ) );
    }
    return $args;
}
add_filter( 'http_request_args', 'debug_http_timeout', 999, 2 );
```

## Compatibility Notes

- Affects all WordPress HTTP requests
- Compatible with all WP Fusion CRM integrations
- Works with caching plugins
- May interact with server timeout settings

## Related Snippets

- [API Extensions](../api-extensions/) - Other API modifications
- [Non-Blocking Login API Calls](./non-blocking-login-api-calls.md) - Performance optimization
- [Debugging & Logging](../debugging-logging/) - Request monitoring
