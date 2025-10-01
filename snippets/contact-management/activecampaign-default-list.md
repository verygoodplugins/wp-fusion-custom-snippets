# ActiveCampaign Default List Assignment

**Category:** Contact Management  
**Compatibility:** WP Fusion + ActiveCampaign  

## Description

Automatically adds all new contacts to a specified ActiveCampaign list. This ensures every contact created through WP Fusion is added to a default list for organization and automation purposes.

## Use Cases

- Ensure all contacts are added to a master list in ActiveCampaign
- Maintain a universal "All Contacts" list for segmentation
- Automatically organize new contacts into specific campaigns
- Simplify list management by having a consistent default list
- Enable list-based automations that should apply to all contacts

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Prerequisites

1. Have an ActiveCampaign list created (or create a new one)
2. Get the list ID from ActiveCampaign (Lists → Your List → Settings → List ID)

## Code

```php
<?php
/**
 * Automatically add all new contacts to a specific ActiveCampaign list.
 *
 * @param array $args The API arguments for adding a contact.
 * @return array Modified arguments with the list ID added.
 */
function my_add_contacts_to_default_list( $args ) {
	
	// Replace 123 with your actual ActiveCampaign list ID.
	$default_list_id = 123;
	
	// Add the list to the contact data.
	if ( empty( $args[0]['lists'] ) ) {
		$args[0]['lists'] = array();
	}
	
	$args[0]['lists'][] = $default_list_id;
	
	return $args;
}
add_filter( 'wpf_api_add_contact_args', 'my_add_contacts_to_default_list' );
```

## Configuration

1. Find your ActiveCampaign list ID:
   - Log into ActiveCampaign
   - Go to **Lists** → Select your list
   - Click **Settings**
   - Copy the **List ID** number

2. Replace `123` in the code with your actual list ID

3. The snippet will now automatically add every new contact to this list

## How It Works

1. **Filter Hook**: Uses the `wpf_api_add_contact_args` filter to modify contact creation arguments
2. **List Array Check**: Verifies if a lists array already exists in the arguments
3. **List Addition**: Appends the default list ID to the lists array
4. **API Sync**: ActiveCampaign receives the contact with the list assignment included

## Troubleshooting

### Contacts Not Added to List

- Verify the list ID is correct in ActiveCampaign
- Check WP Fusion logs (WP Fusion → Settings → Advanced → Logs) for API errors
- Ensure the list is active and not deleted in ActiveCampaign

### Contacts Added to Wrong List

- Double-check the list ID number matches your intended list
- ActiveCampaign list IDs are numeric values found in list settings

### Existing Contacts

- This snippet only affects newly created contacts
- Existing contacts won't be retroactively added to the list
- Use ActiveCampaign's bulk tools to add existing contacts to the list

## Notes

- This filter runs on **new contact creation only** (not on updates)
- Contacts can be added to multiple lists (this doesn't replace existing list assignments)
- The list assignment happens automatically whenever WP Fusion creates a new contact

