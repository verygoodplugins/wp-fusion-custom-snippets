# Show CRM Field Keys in Dropdowns

**Category:** Debugging & Logging  
**Compatibility:** WP Fusion  
**Source:** [GitHub Gist](https://gist.github.com/jack-arturo/a40cbf1b7de0e8234d24146ed1826701)

## Description

Appends each CRM field's internal ID/key to the field label in WP Fusion dropdowns. This is extremely helpful for developers and advanced users who need to reference field keys for custom integrations, API calls, or troubleshooting.

## Use Cases

- Development and debugging field mappings
- API integration development
- Troubleshooting field sync issues
- Custom code development requiring field keys
- Documentation of field structures
- CRM migration planning

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code

```php
<?php

function add_field_keys_to_names( $fields ) {

        foreach ( $fields as $i => $category ) {

                foreach ( $category as $key => $label ) {
                        $fields[ $i ][ $key ] = $label .= ' (' . $key . ')';
                }
        }

        return $fields;

}

add_filter( 'wpf_get_setting_crm_fields', 'add_field_keys_to_names' );
```

## How It Works

1. **Filter Hook**: Uses `wpf_get_setting_crm_fields` filter to modify field display
2. **Category Loop**: Iterates through each field category (Contact, Company, etc.)
3. **Field Loop**: Processes each field within categories
4. **Label Modification**: Appends the field key to the existing label
5. **Display Update**: Shows modified labels in all WP Fusion field dropdowns

## Example Output

**Before:**
- First Name
- Last Name
- Email Address

**After:**
- First Name (first_name)
- Last Name (last_name)
- Email Address (email)

## Use Cases in Detail

### API Development

When building custom integrations, you need the exact field keys:

```php
// Now you can easily see that "Email Address (email)" uses key "email"
$contact_data = array(
    'email' => 'user@example.com',
    'first_name' => 'John',
    'last_name' => 'Doe'
);
```

### Troubleshooting

Quickly identify which fields are causing sync issues:

1. Check WP Fusion logs
2. Match error messages to field keys shown in dropdowns
3. Verify field mappings are correct

### Custom Field Mapping

When programmatically setting field mappings:

```php
// You can see the exact keys needed
$field_mappings = array(
    'billing_email' => 'email',
    'billing_first_name' => 'first_name',
    'billing_company' => 'company_name'
);
```

## Advanced Customizations

### Show Only for Administrators

Only display field keys for admin users:

```php
function add_field_keys_to_names( $fields ) {
    
    // Only show keys to administrators
    if ( ! current_user_can( 'administrator' ) ) {
        return $fields;
    }

    foreach ( $fields as $i => $category ) {
        foreach ( $category as $key => $label ) {
            $fields[ $i ][ $key ] = $label .= ' (' . $key . ')';
        }
    }

    return $fields;
}

add_filter( 'wpf_get_setting_crm_fields', 'add_field_keys_to_names' );
```

### Custom Format

Change how keys are displayed:

```php
function add_field_keys_to_names( $fields ) {

    foreach ( $fields as $i => $category ) {
        foreach ( $category as $key => $label ) {
            // Different formats:
            $fields[ $i ][ $key ] = $label . ' [' . $key . ']';  // Square brackets
            // $fields[ $i ][ $key ] = $key . ' - ' . $label;     // Key first
            // $fields[ $i ][ $key ] = $label . ' | ' . $key;     // Pipe separator
        }
    }

    return $fields;
}

add_filter( 'wpf_get_setting_crm_fields', 'add_field_keys_to_names' );
```

### Show Keys for Specific Categories

Only show keys for certain field types:

```php
function add_field_keys_to_names( $fields ) {

    $show_keys_for = array( 'Custom Fields', 'Company Fields' );

    foreach ( $fields as $category_name => $category ) {
        
        if ( ! in_array( $category_name, $show_keys_for ) ) {
            continue;
        }
        
        foreach ( $category as $key => $label ) {
            $fields[ $category_name ][ $key ] = $label .= ' (' . $key . ')';
        }
    }

    return $fields;
}

add_filter( 'wpf_get_setting_crm_fields', 'add_field_keys_to_names' );
```

### Export Field Keys

Create a function to export all field keys:

```php
function export_crm_field_keys() {
    
    if ( ! current_user_can( 'administrator' ) ) {
        return;
    }
    
    $fields = wpf_get_setting( 'crm_fields' );
    
    echo '<h3>CRM Field Keys</h3>';
    echo '<table border="1">';
    echo '<tr><th>Category</th><th>Label</th><th>Key</th></tr>';
    
    foreach ( $fields as $category_name => $category ) {
        foreach ( $category as $key => $label ) {
            echo '<tr>';
            echo '<td>' . esc_html( $category_name ) . '</td>';
            echo '<td>' . esc_html( $label ) . '</td>';
            echo '<td><code>' . esc_html( $key ) . '</code></td>';
            echo '</tr>';
        }
    }
    
    echo '</table>';
}

// Add to admin menu or use in development
```

## Development Workflow

### Field Mapping Development

1. **Enable snippet** to see field keys
2. **Map fields** in WP Fusion settings
3. **Note the keys** for custom code
4. **Test mappings** with sample data
5. **Disable snippet** in production (optional)

### API Integration

1. **Identify required fields** using the dropdown display
2. **Build API calls** with correct field keys
3. **Test data sync** between systems
4. **Document field mappings** for future reference

## Performance Notes

- Minimal performance impact (only modifies display)
- Runs only when field dropdowns are loaded
- No database queries added
- Safe for production use

## Compatibility Notes

- Works with all WP Fusion CRM integrations
- Compatible with custom field types
- Supports field categories and organization
- Works in all WP Fusion admin interfaces

## Related Snippets

- [Debugging & Logging](../debugging-logging/) - Other development tools
- [Contact Management](../contact-management/) - Field management utilities
- [API Extensions](../api-extensions/) - Custom integration helpers
