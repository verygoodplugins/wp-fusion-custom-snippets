# WP Fusion Custom Snippets Index

This directory contains organized WP Fusion custom code snippets categorized by functionality. Each snippet includes detailed documentation, installation instructions, and usage examples.

## 📁 Categories

### 📊 [Activity Tracking](./activity-tracking/)
User behavior tracking, analytics, and attribution management.

- **[Manual Lead Source Sync](./activity-tracking/manual-lead-source-sync.md)** - Manually syncs UTM parameters and lead source data from cookies to CRM fields on user creation

### 🔐 [Access Control](./access-control/)
Snippets for managing content access, redirects, and user permissions.

- **[LearnDash Previous Page Redirect](./access-control/learndash-previous-page-redirect.md)** - Redirects users back to the previous page when accessing restricted LearnDash lessons instead of using default redirect
- **[IP-Based Content Unlock](./access-control/ip-based-content-unlock.md)** - Bypasses content protection when accessing from specific IP addresses for office/VIP access
- **[Return After Login Customizations](./access-control/return-after-login-customizations.md)** - Customize redirect behavior after users login to access restricted content

### 🔧 [API Extensions](./api-extensions/)
Modifications and enhancements to WP Fusion's API functionality.

- **[Non-Blocking Login API Calls](./api-extensions/non-blocking-login-api-calls.md)** - Makes HTTP API calls non-blocking during login to improve performance with slow CRMs
- **[Webhook Loopback Prevention](./api-extensions/webhook-loopback-prevention.md)** - Prevents infinite webhook loops by temporarily blocking incoming webhooks after tag applications
- **[Extend HTTP Timeout](./api-extensions/extend-http-timeout.md)** - Increases HTTP timeout to 60 seconds to prevent errors with slow CRM APIs
- **[Query Filter Cache Optimization](./api-extensions/query-filter-cache-optimization.md)** - Optimize Filter Queries caching behavior for better performance
- **[API Method Overrides](./api-extensions/api-method-overrides.md)** - Override WP Fusion's CRM API methods with custom functionality

### 🔑 [Authentication & Login](./authentication-login/)
Custom login behaviors, user authentication, and registration enhancements.

- **[Username-Based Tag Application](./authentication-login/username-based-tag-application.md)** - Automatically applies username as a tag when users register for unique identification
- **[True Auto-Login via URL](./authentication-login/true-auto-login.md)** - Enables seamless auto-login through specially formatted URLs with contact ID and access key

### 🤖 [Automation Workflows](./automation-workflows/)
Snippets for creating custom automation triggers and workflow enhancements.

- **[Pageviews Tag Trigger](./automation-workflows/pageviews-tag-trigger.md)** - Applies tags automatically after users view a specified number of pages across the site
- **[Visit-Based Tag Trigger](./automation-workflows/visit-based-tag-trigger.md)** - Applies tags when users visit specific posts/pages multiple times
- **[Post Terms Tag Application](./automation-workflows/post-terms-tag-application.md)** - Applies tags based on post categories and tags when content is viewed

### 💳 [E-commerce Integration](./ecommerce-integration/)
WooCommerce and other e-commerce platform integrations and customizations.

- **[WooCommerce Subscription Renewal Tags](./ecommerce-integration/woocommerce-subscription-renewal-tags.md)** - Prevents applying "active" tags during subscription renewals
- **[FooEvents Multiple Event Sync](./ecommerce-integration/fooevents-multiple-event-sync.md)** - Syncs multiple FooEvents registrations to numbered custom fields per contact
- **[WooCommerce Custom Order Status Processing](./ecommerce-integration/woocommerce-custom-order-statuses.md)** - Customizes which order statuses trigger WP Fusion processing and CRM sync
- **[WooCommerce Subscription Signup Tracking for HubSpot](./ecommerce-integration/woocommerce-subscription-signup-tracking-hubspot.md)** - Tracks new subscription signups with timestamps for HubSpot reporting

### 🐛 [Debugging & Logging](./debugging-logging/)
Development and troubleshooting tools for WP Fusion implementations.

- **[Show CRM Field Keys](./debugging-logging/show-crm-field-keys.md)** - Displays CRM field internal IDs/keys in WP Fusion dropdowns for development

### 👥 [User Management](./user-management/)
User data handling, personalization, and user experience enhancements.

- **[Tags as Body Classes](./user-management/tags-as-body-classes.md)** - Adds user's WP Fusion tags as CSS classes to the body element for dynamic styling

## 🚀 Quick Start

1. **Browse Categories** - Navigate to the relevant category folder
2. **Choose Installation Method** - Each snippet supports three installation methods:
   - functions.php file
   - Custom plugin
   - FluentSnippets plugin
3. **Follow Documentation** - Each snippet includes detailed setup and customization instructions
4. **Test Thoroughly** - Always test snippets in a staging environment first

## 📋 Installation Methods

All snippets in this repository support three installation methods. Choose the one that best fits your workflow:

### Method 1: functions.php File
- **Best for:** Quick testing, simple customizations
- **Location:** `wp-content/themes/your-theme/functions.php`
- **Note:** Use a child theme to prevent loss during theme updates

### Method 2: Custom Plugin
- **Best for:** Permanent customizations, multiple sites
- **Benefits:** Survives theme changes, easy to manage
- **Template:** See main README for plugin template

### Method 3: FluentSnippets Plugin
- **Best for:** Non-technical users, conditional logic
- **Benefits:** User-friendly interface, built-in error handling
- **Plugin:** [FluentSnippets on WordPress.org](https://wordpress.org/plugins/easy-code-manager/)

## 🔍 Finding Snippets

### By Category
Browse the category folders above to find snippets related to specific functionality areas.

### By Use Case
Common use cases and their relevant snippets:

**Performance Optimization:**
- [Non-Blocking Login API Calls](./api-extensions/non-blocking-login-api-calls.md)

**User Engagement Tracking:**
- [Pageviews Tag Trigger](./automation-workflows/pageviews-tag-trigger.md)
- [Visit-Based Tag Trigger](./automation-workflows/visit-based-tag-trigger.md)

**E-commerce Enhancements:**
- [WooCommerce Subscription Renewal Tags](./ecommerce-integration/woocommerce-subscription-renewal-tags.md)
- [FooEvents Multiple Event Sync](./ecommerce-integration/fooevents-multiple-event-sync.md)
- [WooCommerce Subscription Signup Tracking](./ecommerce-integration/woocommerce-subscription-signup-tracking-hubspot.md)

**Development Tools:**
- [Show CRM Field Keys](./debugging-logging/show-crm-field-keys.md)

**Content Access Control:**
- [LearnDash Previous Page Redirect](./access-control/learndash-previous-page-redirect.md)

**API Reliability:**
- [Webhook Loopback Prevention](./api-extensions/webhook-loopback-prevention.md)

## 📖 Documentation Standards

Each snippet includes:

- **Description** - What the snippet does and why you'd use it
- **Use Cases** - Specific scenarios where the snippet is helpful  
- **Installation** - Step-by-step setup instructions
- **Code** - Complete, ready-to-use code
- **Configuration** - Customization options and settings
- **How It Works** - Technical explanation of the functionality
- **Compatibility** - Plugin requirements and version notes
- **Related Snippets** - Links to complementary functionality

## ⚠️ Safety Guidelines

Before installing any snippet:

1. **Backup Your Site** - Always create a full backup
2. **Test in Staging** - Never test on production sites
3. **Read Documentation** - Understand what the snippet does
4. **Check Compatibility** - Ensure compatibility with your setup
5. **Monitor Performance** - Watch for any performance impacts

## 🤝 Contributing

Found a bug or have an improvement? See the main [README](../README.md) for contribution guidelines.

## 📞 Support

- **Issues:** Report problems via [GitHub Issues](../../../issues)
- **Documentation:** [WP Fusion Documentation](https://wpfusion.com/documentation/)
- **Community:** [WP Fusion Facebook Group](https://www.facebook.com/groups/wpfusion/)

---

**Total Snippets:** 20 organized snippets  
**Last Updated:** December 2024 - Including custom solutions from WP Fusion support tickets  
**Sources:** [Jack Arturo's GitHub Gists](https://gist.github.com/jack-arturo/), WP Fusion Documentation, and custom support implementations
