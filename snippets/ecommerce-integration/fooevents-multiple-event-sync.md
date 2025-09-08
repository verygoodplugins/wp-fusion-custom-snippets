# FooEvents - Multiple Event Data Sync

**Category:** E-commerce Integration  
**Compatibility:** WP Fusion + WooCommerce + FooEvents  
**Source:** [GitHub Gist](https://gist.github.com/jack-arturo/0529cc21c8d51b98acc8c0f65e7e460f)

## Description

Syncs FooEvents event data to separate custom fields on the attendee's contact record for each event. This allows you to track multiple event registrations per contact with individual event details including name, date, time, and Zoom URLs.

## Use Cases

- Track multiple event registrations per contact
- Sync event-specific details to CRM
- Manage Zoom meeting URLs for virtual events
- Create event-specific automations in CRM
- Generate attendee reports with full event history

## Installation

Choose one of the three methods described in the main [README](../../README.md):

1. **functions.php** - Add to your theme's functions.php file
2. **Custom Plugin** - Create a custom plugin file
3. **FluentSnippets** - Use the FluentSnippets plugin interface

## Code

```php
<?php
/**
 * Utility function for getting any FooEvents attendees from a WooCommerce order
 * @param WC_Order $order
 * @return array Attendees
 */
function get_foo_attendees_from_order( $order ) {

    $attendees = array();

    $order_data = $order->get_data();

    foreach ( $order_data['meta_data'] as $meta ) {

        if ( ! is_a( $meta, 'WC_Meta_Data' ) ) {
            continue;
        }

        $data = $meta->get_data();

        if ( 'WooCommerceEventsOrderTickets' != $data['key'] ) {
            continue;
        }

        foreach ( $data['value'] as $sub_value ) {

            if ( ! is_array( $sub_value ) ) {
                continue;
            }

            foreach ( $sub_value as $attendee ) {

                $attendees[] = $attendee;

            }
        }
    }

    return $attendees;
}

/**
 * Retrieves attendees from the given order
 * @param mixed $order_id
 * @return array Processed Attendee Date
 */
function get_foo_event_attendees( $order_id){

    $order     = wc_get_order( $order_id );
    $attendees = get_foo_attendees_from_order( $order );

    $event_count = [];
    $wpf_processed_data = [];

    foreach($attendees as $attendee){

        $email = trim(strtolower($attendee['WooCommerceEventsAttendeeEmail']));
        $product_id = $attendee['WooCommerceEventsProductID'];
        $date = str_replace("/","-",$attendee['WooCommerceEventsBookingOptions']['date_label']);
        $slot = $attendee['WooCommerceEventsBookingOptions']['slot_label'];

        if (isset($event_count[$email])){
            $event_count[$email]++;
        }else{
            $event_count[$email] = 1;
        }

        $wpf_processed_data[$email][$product_id][$date][$slot] = [
          'index' => $event_count[$email],
        ];
    }

    return $wpf_processed_data;

}

/**
 * Pushes updated data in to relevant WP Fusion fields
 * @param $update_data
 * @param $attendee
 * @param $order_id
 * @return mixed
 */
function sync_booking_information( $update_data, $attendee, $order_id ) {

    $attendee_data = get_foo_event_attendees($order_id);

    $email      = trim(strtolower($attendee['WooCommerceEventsAttendeeEmail']));
    $product_id = $attendee['WooCommerceEventsProductID'];
    $date       = str_replace("/","-",$attendee['WooCommerceEventsBookingOptions']['date_label']);
    $slot       = $attendee['WooCommerceEventsBookingOptions']['slot_label'];

    $index          = 1;
    if (isset($attendee_data[$email][$product_id][$date][$slot]['index'])){
        $index = $attendee_data[$email][$product_id][$date][$slot]['index'];
    }

    $event_name     = $update_data['event_name'];
    $booking_date   = date('Y-m-d' , strtotime(str_replace("/","-",$update_data['booking_date'])));
    $booking_time   = $update_data['booking_time'];
    $zoom_join_url  = "";

    if (isset($update_data['zoom_join_url'])){
        $zoom_join_url  = $update_data['zoom_join_url'];
    }

    $update_data[ 'ac_foo_event_name_' . $index ] = $event_name;
    $update_data[ 'ac_foo_event_date_' . $index ] = $booking_date;
    $update_data[ 'ac_foo_event_time_' . $index ] = $booking_time;
    $update_data[ 'ac_foo_event_zoom_' . $index ] = $zoom_join_url;

    return $update_data;

}

add_action( 'wpf_woocommerce_attendee_data', 'sync_booking_information', 10, 3 );
```

## How It Works

1. **Order Processing**: Extracts FooEvents attendee data from WooCommerce order meta
2. **Data Organization**: Groups attendees by email, product, date, and time slot
3. **Index Assignment**: Assigns unique index numbers for multiple events per contact
4. **Field Mapping**: Maps event data to numbered custom fields in the CRM
5. **Data Sync**: Syncs all event details including Zoom URLs when available

## CRM Field Structure

The snippet creates numbered fields for each event:

- `ac_foo_event_name_1` - First event name
- `ac_foo_event_date_1` - First event date (Y-m-d format)
- `ac_foo_event_time_1` - First event time
- `ac_foo_event_zoom_1` - First event Zoom URL

- `ac_foo_event_name_2` - Second event name
- `ac_foo_event_date_2` - Second event date
- etc.

## CRM Setup Requirements

### Create Custom Fields

Create these custom fields in your CRM:

```
ac_foo_event_name_1, ac_foo_event_name_2, ac_foo_event_name_3...
ac_foo_event_date_1, ac_foo_event_date_2, ac_foo_event_date_3...
ac_foo_event_time_1, ac_foo_event_time_2, ac_foo_event_time_3...
ac_foo_event_zoom_1, ac_foo_event_zoom_2, ac_foo_event_zoom_3...
```

### Map Fields in WP Fusion

1. Go to WP Fusion settings
2. Navigate to field mapping
3. Map the custom fields to your CRM fields
4. Save settings

## Customization Options

### Change Field Prefix

Modify the field prefix from `ac_foo_event_` to your preference:

```php
// Change field prefix
$update_data[ 'event_name_' . $index ] = $event_name;
$update_data[ 'event_date_' . $index ] = $booking_date;
$update_data[ 'event_time_' . $index ] = $booking_time;
$update_data[ 'event_zoom_' . $index ] = $zoom_join_url;
```

### Add Additional Data

Include more event information:

```php
// Add venue information
$venue = $update_data['venue'] ?? '';
$update_data[ 'ac_foo_event_venue_' . $index ] = $venue;

// Add ticket type
$ticket_type = $attendee['WooCommerceEventsTicketType'] ?? '';
$update_data[ 'ac_foo_event_ticket_type_' . $index ] = $ticket_type;

// Add order ID for reference
$update_data[ 'ac_foo_event_order_' . $index ] = $order_id;
```

### Limit Number of Events

Prevent unlimited field creation:

```php
// Only sync first 5 events per contact
$max_events = 5;
if ( $index <= $max_events ) {
    $update_data[ 'ac_foo_event_name_' . $index ] = $event_name;
    // ... other fields
}
```

## Debugging

### Log Event Data

Add logging to troubleshoot:

```php
function sync_booking_information( $update_data, $attendee, $order_id ) {
    
    // Log the attendee data
    if ( function_exists( 'wpf_log' ) ) {
        wpf_log( 'info', 0, 'FooEvents attendee data: ' . print_r( $attendee, true ) );
    }
    
    // ... rest of function
}
```

### Check Field Mapping

Verify fields are being created:

```php
// Add temporary logging
add_action( 'wpf_woocommerce_attendee_data', function( $data ) {
    error_log( 'Event fields being synced: ' . print_r( $data, true ) );
}, 999 );
```

## Compatibility Notes

- Requires FooEvents plugin
- Works with WooCommerce integration
- Compatible with virtual events (Zoom)
- Supports multiple time slots per event

## Related Snippets

- [E-commerce Integration](../ecommerce-integration/) - Other WooCommerce customizations
- [Contact Management](../contact-management/) - Field sync customizations
