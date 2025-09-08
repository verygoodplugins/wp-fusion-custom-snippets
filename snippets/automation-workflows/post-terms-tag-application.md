# Apply Tags Based on Post Categories and Tags

**Category:** Automation Workflows  
**Compatibility:** WP Fusion  
**Source:** [GitHub Gist](https://gist.github.com/jack-arturo/29e917d94ca9f3463eba0b42dd3d17d6)

## Description

Automatically applies WP Fusion tags to logged-in users when they view posts, based on the categories and post tags assigned to that content. This creates dynamic user segmentation based on content consumption patterns.

## Use Cases

- Segment users by content interests (categories/tags)
- Track reading habits and preferences
- Create interest-based email marketing segments
- Trigger targeted follow-up sequences
- Build user personas based on content consumption
- Enable personalized content recommendations

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code

**Note:** The original gist has a small bug. Here's the corrected version:

```php
<?php

function wpf_apply_tags_on_view( $post ) {

        if( ! function_exists( 'wp_fusion' ) ) {
                return;
        }

        if ( is_admin() || ! is_singular() || ! is_user_logged_in() || $post->post_type != 'post' ) {
                return;
        }

        $tags_to_apply = array();

        // Get post categories
        $categories = get_the_terms( $post, 'category' );

        if( ! empty( $categories ) && ! is_wp_error( $categories ) ) {

                foreach( $categories as $category ) {

                        $tags_to_apply[] = $category->name;

                }

        }

        // Get post tags
        $tags = get_the_terms( $post, 'post_tag' );

        if( ! empty( $tags ) && ! is_wp_error( $tags ) ) {

                foreach( $tags as $tag ) {

                        $tags_to_apply[] = $tag->name;

                }

        }

        // Apply the tags (fixed variable name)
        if ( ! empty( $tags_to_apply ) ) {
                wp_fusion()->user->apply_tags( $tags_to_apply );
        }

}

add_action( 'the_post', 'wpf_apply_tags_on_view' );
```

## How It Works

1. **Post View Detection**: Triggers when a post is viewed on the frontend
2. **User Check**: Only applies to logged-in users
3. **Content Type Filter**: Only processes regular posts (not pages/CPTs)
4. **Category Processing**: Extracts all categories from the post
5. **Tag Processing**: Extracts all post tags from the post
6. **Tag Application**: Applies category and tag names as WP Fusion tags

## Example Scenario

If a user views a post with:
- **Categories:** "Marketing", "SEO"
- **Post Tags:** "Content Strategy", "Analytics"

The user will receive these WP Fusion tags:
- Marketing
- SEO
- Content Strategy
- Analytics

## Advanced Customizations

### Add Prefix to Applied Tags

Prevent conflicts with existing tags by adding prefixes:

```php
function wpf_apply_prefixed_tags_on_view( $post ) {
    
    if( ! function_exists( 'wp_fusion' ) || is_admin() || ! is_singular() || ! is_user_logged_in() ) {
        return;
    }

    if ( $post->post_type != 'post' ) {
        return;
    }

    $tags_to_apply = array();

    // Categories with prefix
    $categories = get_the_terms( $post, 'category' );
    if( ! empty( $categories ) && ! is_wp_error( $categories ) ) {
        foreach( $categories as $category ) {
            $tags_to_apply[] = 'Category: ' . $category->name;
        }
    }

    // Post tags with prefix
    $tags = get_the_terms( $post, 'post_tag' );
    if( ! empty( $tags ) && ! is_wp_error( $tags ) ) {
        foreach( $tags as $tag ) {
            $tags_to_apply[] = 'Interest: ' . $tag->name;
        }
    }

    if ( ! empty( $tags_to_apply ) ) {
        wp_fusion()->user->apply_tags( $tags_to_apply );
    }
}

add_action( 'the_post', 'wpf_apply_prefixed_tags_on_view' );
```

### Include Custom Post Types

Extend functionality to custom post types:

```php
function wpf_apply_tags_all_content( $post ) {
    
    if( ! function_exists( 'wp_fusion' ) || is_admin() || ! is_singular() || ! is_user_logged_in() ) {
        return;
    }

    // Include multiple post types
    $allowed_post_types = array( 'post', 'product', 'course', 'event' );
    
    if ( ! in_array( $post->post_type, $allowed_post_types ) ) {
        return;
    }

    $tags_to_apply = array();

    // Get all taxonomies for the post type
    $taxonomies = get_object_taxonomies( $post->post_type );
    
    foreach ( $taxonomies as $taxonomy ) {
        $terms = get_the_terms( $post, $taxonomy );
        
        if ( ! empty( $terms ) && ! is_wp_error( $terms ) ) {
            foreach ( $terms as $term ) {
                // Add taxonomy prefix for clarity
                $tags_to_apply[] = ucfirst( $taxonomy ) . ': ' . $term->name;
            }
        }
    }

    if ( ! empty( $tags_to_apply ) ) {
        wp_fusion()->user->apply_tags( $tags_to_apply );
    }
}

add_action( 'the_post', 'wpf_apply_tags_all_content' );
```

### One-Time Tag Application

Prevent duplicate tag applications for the same content:

```php
function wpf_apply_tags_once_per_post( $post ) {
    
    if( ! function_exists( 'wp_fusion' ) || is_admin() || ! is_singular() || ! is_user_logged_in() ) {
        return;
    }

    if ( $post->post_type != 'post' ) {
        return;
    }

    $user_id = get_current_user_id();
    $viewed_posts_key = 'wpf_viewed_posts_' . $user_id;
    $viewed_posts = get_user_meta( $user_id, $viewed_posts_key, true );
    
    if ( ! is_array( $viewed_posts ) ) {
        $viewed_posts = array();
    }

    // Check if user has already been tagged for this post
    if ( in_array( $post->ID, $viewed_posts ) ) {
        return;
    }

    $tags_to_apply = array();

    // Get categories and tags
    $categories = get_the_terms( $post, 'category' );
    if( ! empty( $categories ) && ! is_wp_error( $categories ) ) {
        foreach( $categories as $category ) {
            $tags_to_apply[] = $category->name;
        }
    }

    $tags = get_the_terms( $post, 'post_tag' );
    if( ! empty( $tags ) && ! is_wp_error( $tags ) ) {
        foreach( $tags as $tag ) {
            $tags_to_apply[] = $tag->name;
        }
    }

    if ( ! empty( $tags_to_apply ) ) {
        wp_fusion()->user->apply_tags( $tags_to_apply );
        
        // Mark this post as viewed
        $viewed_posts[] = $post->ID;
        update_user_meta( $user_id, $viewed_posts_key, $viewed_posts );
    }
}

add_action( 'the_post', 'wpf_apply_tags_once_per_post' );
```

### Selective Tag Application

Only apply tags for specific categories or tags:

```php
function wpf_apply_selective_tags( $post ) {
    
    if( ! function_exists( 'wp_fusion' ) || is_admin() || ! is_singular() || ! is_user_logged_in() ) {
        return;
    }

    if ( $post->post_type != 'post' ) {
        return;
    }

    // Define which categories/tags should trigger tag application
    $tracked_categories = array( 'Marketing', 'SEO', 'WordPress', 'Business' );
    $tracked_tags = array( 'Tutorial', 'Case Study', 'News', 'Tips' );

    $tags_to_apply = array();

    // Check categories
    $categories = get_the_terms( $post, 'category' );
    if( ! empty( $categories ) && ! is_wp_error( $categories ) ) {
        foreach( $categories as $category ) {
            if ( in_array( $category->name, $tracked_categories ) ) {
                $tags_to_apply[] = 'Category: ' . $category->name;
            }
        }
    }

    // Check post tags
    $tags = get_the_terms( $post, 'post_tag' );
    if( ! empty( $tags ) && ! is_wp_error( $tags ) ) {
        foreach( $tags as $tag ) {
            if ( in_array( $tag->name, $tracked_tags ) ) {
                $tags_to_apply[] = 'Content Type: ' . $tag->name;
            }
        }
    }

    if ( ! empty( $tags_to_apply ) ) {
        wp_fusion()->user->apply_tags( $tags_to_apply );
    }
}

add_action( 'the_post', 'wpf_apply_selective_tags' );
```

### Time-Based Tag Application

Apply different tags based on when content is viewed:

```php
function wpf_apply_time_based_tags( $post ) {
    
    if( ! function_exists( 'wp_fusion' ) || is_admin() || ! is_singular() || ! is_user_logged_in() ) {
        return;
    }

    if ( $post->post_type != 'post' ) {
        return;
    }

    $tags_to_apply = array();
    $current_time = current_time( 'timestamp' );
    $post_time = get_the_time( 'U', $post );

    // Apply different tags based on content age
    $content_age = $current_time - $post_time;
    
    if ( $content_age < WEEK_IN_SECONDS ) {
        $tags_to_apply[] = 'Recent Content Reader';
    } elseif ( $content_age < MONTH_IN_SECONDS ) {
        $tags_to_apply[] = 'Current Content Reader';
    } else {
        $tags_to_apply[] = 'Archive Content Reader';
    }

    // Get regular categories and tags
    $categories = get_the_terms( $post, 'category' );
    if( ! empty( $categories ) && ! is_wp_error( $categories ) ) {
        foreach( $categories as $category ) {
            $tags_to_apply[] = $category->name;
        }
    }

    if ( ! empty( $tags_to_apply ) ) {
        wp_fusion()->user->apply_tags( $tags_to_apply );
    }
}

add_action( 'the_post', 'wpf_apply_time_based_tags' );
```

## CRM Integration Examples

### ActiveCampaign Automation

Create automations triggered by these tags:
- **Marketing** tag → Add to "Marketing Interested" list
- **SEO** tag → Send SEO tips email series
- **Tutorial** tag → Tag as "Learner" persona

### Email Segmentation

Use applied tags for targeted email campaigns:
- Users with "WordPress" tags get WordPress-related newsletters
- Users with "Business" tags receive business growth content
- Users with multiple category tags get comprehensive updates

## Performance Considerations

### Caching Compatibility

The snippet works with caching plugins since it only triggers on actual post views. For better performance:

```php
// Add caching for taxonomy terms
function wpf_cached_terms_tags( $post ) {
    
    // ... validation code ...
    
    $cache_key = 'wpf_post_terms_' . $post->ID;
    $cached_terms = wp_cache_get( $cache_key );
    
    if ( false === $cached_terms ) {
        $cached_terms = array();
        
        $categories = get_the_terms( $post, 'category' );
        if( ! empty( $categories ) && ! is_wp_error( $categories ) ) {
            foreach( $categories as $category ) {
                $cached_terms[] = $category->name;
            }
        }
        
        wp_cache_set( $cache_key, $cached_terms, '', HOUR_IN_SECONDS );
    }
    
    if ( ! empty( $cached_terms ) ) {
        wp_fusion()->user->apply_tags( $cached_terms );
    }
}
```

## Troubleshooting

### Common Issues

1. **Tags Not Applied**: Check if user is logged in and post has categories/tags
2. **Duplicate Tags**: Use one-time application or check existing tags first
3. **Wrong Variable Name**: Original gist has `$apply_tags` instead of `$tags_to_apply`

### Debug Tag Application

```php
function debug_post_terms_tags( $post ) {
    if ( current_user_can( 'administrator' ) && isset( $_GET['debug_terms'] ) ) {
        echo '<div style="background: yellow; padding: 10px;">';
        echo '<strong>Post Terms Debug:</strong><br>';
        
        $categories = get_the_terms( $post, 'category' );
        if ( $categories ) {
            echo 'Categories: ' . implode( ', ', wp_list_pluck( $categories, 'name' ) ) . '<br>';
        }
        
        $tags = get_the_terms( $post, 'post_tag' );
        if ( $tags ) {
            echo 'Tags: ' . implode( ', ', wp_list_pluck( $tags, 'name' ) ) . '<br>';
        }
        
        echo '</div>';
    }
}
add_action( 'wp_footer', 'debug_post_terms_tags' );
```

## Compatibility Notes

- Works with all WordPress themes
- Compatible with custom post types (with modifications)
- Supports custom taxonomies
- Works with caching plugins
- Compatible with all WP Fusion CRM integrations

## Related Snippets

- [Automation Workflows](../automation-workflows/) - Other automation triggers
- [User Management](../user-management/) - User segmentation features
- [Visit-Based Tag Trigger](./visit-based-tag-trigger.md) - Related content tracking
