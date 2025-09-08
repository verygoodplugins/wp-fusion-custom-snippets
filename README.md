# WP Fusion Custom Snippets

A curated collection of custom code snippets to extend and enhance WP Fusion functionality. This repository provides ready-to-use code examples and modifications that can be easily implemented using various methods.

## 📋 Table of Contents

- [Installation Methods](#installation-methods)
  - [Method 1: functions.php File](#method-1-functionsphp-file)
  - [Method 2: Custom Plugin](#method-2-custom-plugin)
  - [Method 3: FluentSnippets Plugin](#method-3-fluentsnippets-plugin)
- [Snippet Categories](#snippet-categories)
- [Usage Guidelines](#usage-guidelines)
- [Contributing](#contributing)
- [Support](#support)

## 🚀 Installation Methods

There are three primary ways to add these custom code snippets to your WordPress site. Choose the method that best fits your workflow and technical comfort level.

### Method 1: functions.php File

**Best for:** Simple customizations, temporary testing, or when you have a child theme.

**Pros:**
- Quick and easy implementation
- No additional plugins required
- Immediate activation

**Cons:**
- Code is lost when switching themes (unless using a child theme)
- Can break the site if there are PHP errors
- Harder to manage multiple snippets

**How to use:**

1. Navigate to `Appearance > Theme Editor` in your WordPress admin
2. Select your active theme (or preferably, your child theme)
3. Open the `functions.php` file
4. Add your snippet at the end of the file, before the closing `?>` tag (if it exists)
5. Click "Update File"

**Example:**
```php
// Add this to your theme's functions.php file
add_action( 'wp_fusion_user_login', 'my_custom_login_function' );
function my_custom_login_function( $user_id ) {
    // Your custom code here
    error_log( 'User logged in: ' . $user_id );
}
```

**⚠️ Important:** Always use a child theme to prevent losing customizations when the parent theme updates.

### Method 2: Custom Plugin

**Best for:** Permanent customizations, complex functionality, or when managing multiple sites.

**Pros:**
- Survives theme changes
- Easy to activate/deactivate
- Can be version controlled
- Portable across sites

**Cons:**
- Requires creating a plugin file
- Slightly more complex setup

**How to create:**

1. Create a new folder in `/wp-content/plugins/` (e.g., `wp-fusion-custom`)
2. Create a main plugin file (e.g., `wp-fusion-custom.php`)
3. Add the plugin header and your code
4. Activate the plugin from the WordPress admin

**Plugin Template:**
```php
<?php
/**
 * Plugin Name: WP Fusion Custom Snippets
 * Description: Custom functionality for WP Fusion
 * Version: 1.0.0
 * Author: Your Name
 */

// Prevent direct access
if ( ! defined( 'ABSPATH' ) ) {
    exit;
}

// Your custom functions go here
function my_custom_login_function( $user_id ) {
    // Your custom code here
    error_log( 'User logged in: ' . $user_id );
}

add_action( 'wp_fusion_user_login', 'my_custom_login_function' );
```

### Method 3: FluentSnippets Plugin

**Best for:** Non-technical users, easy management, or when you need conditional logic.

**Pros:**
- User-friendly interface
- File-based storage (no database queries)
- Built-in error handling
- Conditional logic support
- Easy to enable/disable snippets
- Survives theme changes

**Cons:**
- Requires installing an additional plugin
- Learning curve for the interface

**How to use:**

1. Install the [FluentSnippets plugin](https://wordpress.org/plugins/easy-code-manager/)
2. Go to `FluentSnippets > Add New` in your WordPress admin
3. Choose snippet type:
   - **Functions (PHP):** For hooks, filters, and custom functions
   - **Content (PHP + HTML):** For output content
   - **CSS:** For styling
   - **JS:** For JavaScript code
4. Add your code and configure conditions if needed
5. Save and activate the snippet

**FluentSnippets Features:**
- **Zero Database Queries:** Snippets are stored as files for maximum performance
- **Conditional Logic:** Execute snippets based on user roles, page types, dates, etc.
- **Error Handling:** Prevents broken sites from PHP errors
- **Stand-alone Mode:** Snippets continue working even if the plugin is deactivated
- **Shortcode Support:** Create reusable content blocks

**Example FluentSnippets Setup:**
1. Create a new "Functions" type snippet
2. Add your WP Fusion code
3. Set conditions (e.g., only on specific pages)
4. Save and activate

## 📁 Snippet Categories

This repository organizes snippets into the following categories:

- **Activity Tracking:** Custom user activity tracking
- **Authentication & Login:** Custom login behaviors, user sync modifications
- **Contact Management:** Contact field customizations, data manipulation
- **E-commerce Integration:** WooCommerce, EDD, and other e-commerce enhancements
- **Email Marketing:** Custom email triggers, list management
- **Form Integration:** Contact form customizations and data handling
- **User Management:** Role modifications, user data sync
- **API Extensions:** Custom API endpoints and webhooks
- **Debugging & Logging:** Development and troubleshooting tools

## 📖 Usage Guidelines

### Before Installing Any Snippet:

1. **Backup Your Site:** Always create a full backup before adding custom code
2. **Test on Staging:** Test snippets on a staging environment first
3. **Read the Documentation:** Understand what each snippet does
4. **Check Compatibility:** Ensure compatibility with your WP Fusion version
5. **Review Dependencies:** Some snippets may require specific plugins or settings

### Best Practices:

- Use descriptive function names with prefixes to avoid conflicts
- Add comments to explain what your code does
- Follow WordPress coding standards
- Test thoroughly before deploying to production
- Keep snippets organized and well-documented

### Error Prevention:

- Always check for function/class existence before defining them
- Use proper WordPress hooks and filters
- Validate and sanitize user input
- Handle errors gracefully

**Example of safe coding:**
```php
// Check if function doesn't already exist
if ( ! function_exists( 'my_custom_wp_fusion_function' ) ) {
    function my_custom_wp_fusion_function() {
        // Your code here
    }
}

// Check if WP Fusion is active
if ( class_exists( 'WP_Fusion' ) ) {
    // WP Fusion specific code
}
```

## 🤝 Contributing

We welcome contributions from the community! To contribute:

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/new-snippet`)
3. Add your snippet with proper documentation
4. Test thoroughly
5. Submit a pull request

**Contribution Guidelines:**
- Include clear documentation for each snippet
- Follow WordPress coding standards
- Test with the latest version of WP Fusion
- Include usage examples
- Specify any dependencies or requirements

## 📞 Support

- **Issues:** Report bugs or request features via [GitHub Issues](../../issues)
- **Documentation:** Check the [WP Fusion documentation](https://wpfusion.com/documentation/)
- **Community:** Join the [WP Fusion Facebook group](https://www.facebook.com/groups/wpfusion/)

## ⚠️ Disclaimer

These snippets are provided as-is for educational and development purposes. Always test in a staging environment before implementing on production sites. The authors are not responsible for any issues that may arise from using these code snippets.

## 📄 License

This project is licensed under the GPL v3 or later - see the [LICENSE](LICENSE) file for details.
