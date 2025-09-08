# Tags as Body Classes

**Category:** User Management  
**Compatibility:** WP Fusion  
**Source:** [GitHub Gist](https://gist.github.com/jack-arturo/2dd9464fa22bde8f36f3d1863890bd2f)

## Description

Adds the current logged-in user's WP Fusion tags as CSS classes to the HTML `<body>` element. This enables dynamic styling and content display based on user tags without requiring additional PHP conditionals.

## Use Cases

- Dynamic styling based on user membership levels
- Show/hide content with CSS based on user tags
- Create personalized designs for different user segments
- Implement tag-based UI customizations
- Build conditional layouts without PHP
- Enable JavaScript targeting by user tags

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code

```php
<?php

// Adds a current logged in users tags to the <body> element's classes.

function wpf_tags_body_class( $classes ) {

        if ( ! function_exists( 'wp_fusion' ) ) {
                return $classes;
        }

        if( ! wpf_is_user_logged_in() || wpf_admin_override() ) {
                return $classes;
        }

        $tags = wp_fusion()->user->get_tags();

        if( ! empty( $tags ) ) {

                foreach( $tags as $tag_id ) {

                        $tag_name = sanitize_title( wp_fusion()->user->get_tag_label( $tag_id ) );
                        $classes[] = 'tag-' . $tag_name;

                }
        }
  
 
        return $classes;
}

add_filter( 'body_class', 'wpf_tags_body_class' );
```

## How It Works

1. **User Check**: Only applies to logged-in users (respects admin override)
2. **Tag Retrieval**: Gets all tags for the current user
3. **Tag Processing**: Converts tag names to valid CSS class names
4. **Class Addition**: Adds `tag-{tagname}` classes to the body element
5. **Sanitization**: Uses `sanitize_title()` to ensure valid CSS class names

## Example Output

If a user has tags "Premium Member" and "Course Access", the body element will include:

```html
<body class="... tag-premium-member tag-course-access">
```

## CSS Implementation Examples

### Membership Level Styling

```css
/* Premium member styling */
body.tag-premium-member .header {
    background: linear-gradient(45deg, #gold, #yellow);
}

body.tag-premium-member .content {
    border: 2px solid gold;
}

/* VIP member styling */
body.tag-vip-member .navigation {
    background-color: #purple;
}

body.tag-vip-member::before {
    content: "VIP";
    position: fixed;
    top: 10px;
    right: 10px;
    background: purple;
    color: white;
    padding: 5px;
    border-radius: 3px;
}
```

### Show/Hide Content

```css
/* Hide content by default */
.premium-only {
    display: none;
}

/* Show for premium members */
body.tag-premium-member .premium-only {
    display: block;
}

/* Course-specific content */
.course-content {
    display: none;
}

body.tag-course-access .course-content {
    display: block;
}

/* Multiple tag requirements */
body.tag-premium-member.tag-course-access .exclusive-content {
    display: block;
}
```

### Dynamic Navigation

```css
/* Hide admin links by default */
.admin-nav {
    display: none;
}

/* Show for administrators */
body.tag-administrator .admin-nav {
    display: block;
}

/* Member-specific navigation */
body.tag-member .member-nav {
    background-color: #blue;
}

body.tag-vip-member .member-nav {
    background-color: #purple;
}
```

## JavaScript Implementation

### Tag-Based Functionality

```javascript
// Check if user has specific tag
function userHasTag(tagName) {
    return document.body.classList.contains('tag-' + tagName.toLowerCase().replace(/\s+/g, '-'));
}

// Example usage
if (userHasTag('Premium Member')) {
    // Show premium features
    document.querySelector('.premium-features').style.display = 'block';
}

// Multiple tag check
function userHasAllTags(tags) {
    return tags.every(tag => userHasTag(tag));
}

if (userHasAllTags(['Premium Member', 'Course Access'])) {
    // Enable advanced features
    initAdvancedFeatures();
}
```

### Dynamic Content Loading

```javascript
// Load content based on user tags
document.addEventListener('DOMContentLoaded', function() {
    const bodyClasses = document.body.className;
    
    if (bodyClasses.includes('tag-premium-member')) {
        loadPremiumContent();
    }
    
    if (bodyClasses.includes('tag-course-access')) {
        loadCourseModules();
    }
});

function loadPremiumContent() {
    // Ajax call to load premium-specific content
    fetch('/wp-json/custom/v1/premium-content')
        .then(response => response.json())
        .then(data => {
            document.querySelector('.premium-container').innerHTML = data.content;
        });
}
```

## Advanced Customizations

### Custom Prefix

Change the default "tag-" prefix:

```php
function wpf_custom_prefix_body_class( $classes ) {
    
    if ( ! function_exists( 'wp_fusion' ) || ! wpf_is_user_logged_in() || wpf_admin_override() ) {
        return $classes;
    }

    $tags = wp_fusion()->user->get_tags();
    $prefix = 'membership-'; // Custom prefix

    if ( ! empty( $tags ) ) {
        foreach ( $tags as $tag_id ) {
            $tag_name = sanitize_title( wp_fusion()->user->get_tag_label( $tag_id ) );
            $classes[] = $prefix . $tag_name;
        }
    }

    return $classes;
}

add_filter( 'body_class', 'wpf_custom_prefix_body_class' );
```

### Selective Tag Classes

Only add classes for specific tags:

```php
function wpf_selective_tag_classes( $classes ) {
    
    if ( ! function_exists( 'wp_fusion' ) || ! wpf_is_user_logged_in() || wpf_admin_override() ) {
        return $classes;
    }

    // Only add classes for these specific tags
    $allowed_tags = array( 'Premium Member', 'VIP', 'Course Access', 'Administrator' );
    $user_tags = wp_fusion()->user->get_tags();

    if ( ! empty( $user_tags ) ) {
        foreach ( $user_tags as $tag_id ) {
            $tag_name = wp_fusion()->user->get_tag_label( $tag_id );
            
            if ( in_array( $tag_name, $allowed_tags ) ) {
                $sanitized_name = sanitize_title( $tag_name );
                $classes[] = 'tag-' . $sanitized_name;
            }
        }
    }

    return $classes;
}

add_filter( 'body_class', 'wpf_selective_tag_classes' );
```

### Tag Hierarchy Classes

Add parent/child tag relationships:

```php
function wpf_hierarchical_tag_classes( $classes ) {
    
    if ( ! function_exists( 'wp_fusion' ) || ! wpf_is_user_logged_in() || wpf_admin_override() ) {
        return $classes;
    }

    $user_tags = wp_fusion()->user->get_tags();
    
    // Define tag hierarchy
    $tag_hierarchy = array(
        'Member' => array( 'Premium Member', 'VIP Member' ),
        'Student' => array( 'Course Access', 'Advanced Course' ),
    );

    if ( ! empty( $user_tags ) ) {
        foreach ( $user_tags as $tag_id ) {
            $tag_name = wp_fusion()->user->get_tag_label( $tag_id );
            $sanitized_name = sanitize_title( $tag_name );
            $classes[] = 'tag-' . $sanitized_name;
            
            // Add parent classes
            foreach ( $tag_hierarchy as $parent => $children ) {
                if ( in_array( $tag_name, $children ) ) {
                    $classes[] = 'tag-' . sanitize_title( $parent );
                }
            }
        }
    }

    return $classes;
}

add_filter( 'body_class', 'wpf_hierarchical_tag_classes' );
```

## CSS Framework Integration

### Bootstrap Integration

```css
/* Bootstrap-style conditional display */
.d-premium-none {
    display: none;
}

body.tag-premium-member .d-premium-block {
    display: block;
}

body.tag-premium-member .d-premium-none {
    display: none;
}

/* Bootstrap component modifications */
body.tag-vip-member .navbar {
    background-color: var(--purple) !important;
}

body.tag-premium-member .btn-primary {
    background-color: gold;
    border-color: gold;
}
```

### Tailwind CSS Integration

```html
<!-- Use with Tailwind's group functionality -->
<div class="group">
    <div class="hidden group-[.tag-premium-member]:block">
        Premium content here
    </div>
</div>
```

## Performance Considerations

### Caching Compatibility

The snippet works with caching plugins since classes are added client-side after page load. For better performance with heavy caching:

```php
function wpf_cached_tag_classes( $classes ) {
    
    // Only add classes if not cached or if user is logged in
    if ( ! wpf_is_user_logged_in() || wpf_admin_override() ) {
        return $classes;
    }

    // Cache user tags for the session
    $cache_key = 'wpf_user_tags_' . get_current_user_id();
    $user_tags = wp_cache_get( $cache_key );
    
    if ( false === $user_tags ) {
        $user_tags = wp_fusion()->user->get_tags();
        wp_cache_set( $cache_key, $user_tags, '', 300 ); // 5 minutes
    }

    if ( ! empty( $user_tags ) ) {
        foreach ( $user_tags as $tag_id ) {
            $tag_name = sanitize_title( wp_fusion()->user->get_tag_label( $tag_id ) );
            $classes[] = 'tag-' . $tag_name;
        }
    }

    return $classes;
}

add_filter( 'body_class', 'wpf_cached_tag_classes' );
```

## Troubleshooting

### Common Issues

1. **Classes Not Appearing**: Verify user is logged in and has tags
2. **Invalid CSS Names**: Check tag names don't contain special characters
3. **Styling Not Applied**: Ensure CSS selectors match generated class names
4. **Admin Override**: Snippet respects WP Fusion admin override setting

### Debug Tag Classes

```php
// Add to functions.php temporarily for debugging
function debug_tag_classes() {
    if ( current_user_can( 'administrator' ) && isset( $_GET['debug_tags'] ) ) {
        echo '<div style="background: yellow; padding: 10px; margin: 10px;">';
        echo '<strong>User Tags:</strong><br>';
        $tags = wp_fusion()->user->get_tags();
        foreach ( $tags as $tag_id ) {
            $tag_name = wp_fusion()->user->get_tag_label( $tag_id );
            echo 'Tag: ' . $tag_name . ' → Class: tag-' . sanitize_title( $tag_name ) . '<br>';
        }
        echo '</div>';
    }
}
add_action( 'wp_footer', 'debug_tag_classes' );
```

## Compatibility Notes

- Works with all WordPress themes
- Compatible with page builders
- Works with caching plugins
- Supports all WP Fusion CRM integrations
- Compatible with CSS frameworks

## Related Snippets

- [User Management](../user-management/) - Other user-based customizations
- [Access Control](../access-control/) - Content restriction features
- [Automation Workflows](../automation-workflows/) - Tag-based automations
