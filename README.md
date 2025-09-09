# WP Fusion Custom Snippets

<div align="center">

![WP Fusion Logo](https://wpfusion.com/wp-content/uploads/2019/08/logo-top-svg.svg)

**🚀 Extend the Power of WordPress's #1 Marketing Automation Plugin**

[![WordPress Sites](https://img.shields.io/badge/Powering-38%2C000%2B%20Sites-orange)](https://wpfusion.com?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets)
[![CRMs Supported](https://img.shields.io/badge/CRMs%20Supported-60%2B-blue)](https://wpfusion.com/documentation/getting-started/crm-compatibility/?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets)
[![Plugin Integrations](https://img.shields.io/badge/Plugin%20Integrations-150%2B-green)](https://wpfusion.com/documentation/?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets)
[![License](https://img.shields.io/badge/License-GPL%20v3-lightgrey)](LICENSE)

A curated collection of custom code snippets to extend and enhance WP Fusion functionality. Ready-to-use solutions that unlock even more possibilities for your marketing automation workflows.

[Get WP Fusion](https://wpfusion.com/pricing/?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets) | [Documentation](https://wpfusion.com/documentation/?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets) | [Support](https://wpfusion.com/contact/?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets) | [Join Community](https://www.facebook.com/groups/wpfusion/)

</div>

---

## 🎯 What is WP Fusion?

**WP Fusion** is the most powerful and flexible marketing automation plugin for WordPress. It connects your WordPress site to your CRM or marketing automation platform, creating seamless two-way data sync that transforms how you engage with your audience.

### Why 38,000+ Sites Trust WP Fusion

- **🔌 60+ CRM Integrations** - From HubSpot to ActiveCampaign, Salesforce to Mailchimp, we've got you covered
- **🧩 150+ Plugin Integrations** - Deep integration with WooCommerce, LearnDash, MemberPress, BuddyBoss, and more
- **⚡ Real-Time Sync** - Instant data synchronization keeps your CRM always up-to-date
- **🎯 Smart Automation** - Trigger actions based on user behavior, purchases, course progress, and more
- **🔒 Content Personalization** - Show different content to different users based on CRM tags
- **🛠️ Developer Friendly** - Extensive hooks, filters, and this snippet library for endless customization

### The Power of Open Source

WP Fusion isn't just a plugin—it's a platform. Built on WordPress's open architecture, it's infinitely extensible through custom code. This repository showcases that extensibility with battle-tested snippets from our community of developers and power users.

## 📋 Table of Contents

- [Quick Start](#-quick-start)
- [Installation Methods](#-installation-methods)
- [Snippet Categories](#-snippet-categories)
- [Featured Snippets](#-featured-snippets)
- [Usage Guidelines](#-usage-guidelines)
- [Contributing](#-contributing)
- [Resources & Support](#-resources--support)

## 🚀 Quick Start

Get up and running in minutes:

1. **Have WP Fusion installed** ([Get it here](https://wpfusion.com/pricing/?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets) or start with [WP Fusion Lite](https://wordpress.org/plugins/wp-fusion-lite/))
2. **Browse our [snippet categories](#-snippet-categories)** to find what you need
3. **Choose an [installation method](#-installation-methods)** that fits your workflow
4. **Copy, customize, and deploy** - Each snippet includes detailed documentation

## 🛠️ Installation Methods

Three ways to add these snippets to your site - choose what works best for you:

### Method 1: functions.php File

**Best for:** Quick testing, simple customizations

Add snippets directly to your theme's `functions.php`:

```php
// Add to your theme's functions.php (preferably in a child theme)
add_action( 'wpf_tags_modified', 'my_custom_function', 10, 2 );
function my_custom_function( $user_id, $tags ) {
    // Your custom logic here
}
```

⚠️ **Pro tip:** Always use a child theme to preserve customizations during updates

### Method 2: Custom Plugin

**Best for:** Production sites, version control, portability

Create a custom plugin for your snippets:

```php
<?php
/**
 * Plugin Name: WP Fusion Custom Extensions
 * Description: Custom functionality extending WP Fusion
 * Version: 1.0.0
 * Author: Your Name
 */

// Prevent direct access
if ( ! defined( 'ABSPATH' ) ) {
    exit;
}

// Your custom snippets here
```

Save as `/wp-content/plugins/wpf-custom/wpf-custom.php` and activate

### Method 3: FluentSnippets Plugin

**Best for:** Non-developers, conditional logic, easy management

1. Install [FluentSnippets](https://wordpress.org/plugins/easy-code-manager/)
2. Add snippets through the visual interface
3. Set conditions and manage activation states
4. Zero database queries - snippets stored as files

## 📁 Snippet Categories

Organized by functionality for easy discovery:

### 🔐 [Access Control](./snippets/access-control/)
Manage content restrictions, redirects, and user permissions
- LearnDash redirects, IP-based unlocks, login customizations

### 📊 [Activity Tracking](./snippets/activity-tracking/)
Advanced user behavior tracking and attribution
- UTM tracking, lead source sync, engagement metrics

### 🔧 [API Extensions](./snippets/api-extensions/)
Enhance WP Fusion's API capabilities
- Performance optimizations, webhook management, custom methods

### 🤖 [Automation Workflows](./snippets/automation-workflows/)
Create sophisticated automation triggers
- Pageview tracking, visit-based tags, content engagement

### 💳 [E-commerce Integration](./snippets/ecommerce-integration/)
WooCommerce and e-commerce enhancements
- Subscription tracking, custom order statuses, event sync

### 👥 [User Management](./snippets/user-management/)
User data handling and personalization
- Dynamic styling, custom fields, role management

[View all 20+ snippets →](./snippets/)

## ⭐ Featured Snippets

Popular solutions from our community:

### 🛒 [WooCommerce Subscription Signup Tracking](./snippets/ecommerce-integration/woocommerce-subscription-signup-tracking-hubspot.md)
Track new subscriptions in HubSpot with timestamps for time-based reporting

### 🎓 [LearnDash Previous Page Redirect](./snippets/access-control/learndash-previous-page-redirect.md)
Smart redirects that remember where users came from

### ⚡ [Non-Blocking Login API Calls](./snippets/api-extensions/non-blocking-login-api-calls.md)
Speed up login times with slow CRMs

### 🔄 [Webhook Loopback Prevention](./snippets/api-extensions/webhook-loopback-prevention.md)
Prevent infinite webhook loops in your automations

## 📖 Usage Guidelines

### Before Installing Any Snippet:

✅ **Always:**
- Create a full backup
- Test on staging first
- Read the documentation
- Check WP Fusion version compatibility

❌ **Never:**
- Test on production sites
- Mix incompatible snippets
- Ignore error messages

### Best Practices:

```php
// Always check dependencies
if ( ! class_exists( 'WP_Fusion' ) ) {
    return;
}

// Use proper namespacing
if ( ! function_exists( 'wpf_custom_function' ) ) {
    function wpf_custom_function() {
        // Your code here
    }
}
```

## 🤝 Contributing

We love contributions from our community! Here's how to share your snippets:

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/amazing-snippet`)
3. Add your snippet with documentation
4. Test thoroughly
5. Submit a pull request

**Contribution must include:**
- Clear documentation
- Usage examples
- Compatibility notes
- Test coverage

## 📚 Resources & Support

### Documentation
- 📖 [Getting Started Guide](https://wpfusion.com/documentation/getting-started/?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets)
- 🔌 [CRM Setup Guides](https://wpfusion.com/documentation/getting-started/installation-guide/?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets)
- 🛠️ [Developer Documentation](https://wpfusion.com/documentation/?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets#advanced-developer-tutorials)
- 🎨 [Brand Assets](https://wpfusion.com/brand-assets/?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets)

### Getting Help
- 💬 [Facebook Community](https://www.facebook.com/groups/wpfusion/) - 5,000+ members strong
- 📧 [Priority Support](https://wpfusion.com/support/?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets) - For license holders
- 🐛 [GitHub Issues](../../issues) - Bug reports and feature requests
- 📺 [YouTube Tutorials](https://www.youtube.com/@wpfusion) - Video walkthroughs

## 💰 Pricing & Plans

### WP Fusion Lite (Free)
- ✅ 60+ CRM integrations
- ✅ User data sync
- ✅ Basic automation
- [Get it on WordPress.org](https://wordpress.org/plugins/wp-fusion-lite/)

### WP Fusion Pro
- ✅ Everything in Lite
- ✅ 150+ plugin integrations
- ✅ Advanced automation
- ✅ E-commerce tracking
- ✅ Priority support
- ✅ 30-day money-back guarantee

[View Pricing →](https://wpfusion.com/pricing/?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets)

## ⚠️ Disclaimer

These snippets are provided as-is for educational and development purposes. Always test in a staging environment before implementing on production sites. The authors are not responsible for any issues that may arise from using these code snippets.

## 📄 License

This project is licensed under the GPL v3 or later - see the [LICENSE](LICENSE) file for details.

---

<div align="center">

**Built with 🧡 by [Jack Arturo](https://github.com/verygoodplugins) at [Very Good Plugins](https://verygoodplugins.com?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets)**

*Made with love for the WordPress open-source community*

[Website](https://wpfusion.com?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets) • [Documentation](https://wpfusion.com/documentation/?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets) • [Support](https://wpfusion.com/contact/?utm_source=github&utm_medium=readme&utm_campaign=custom-snippets) • [Facebook Group](https://www.facebook.com/groups/wpfusion/)

</div>