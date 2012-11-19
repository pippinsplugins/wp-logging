WP-Logging - Introduction
=========================

A WordPress class that provides a general logging system. This class is designed to be used it large systems that need a way of persistently tracking events, errors, actions, etc.

This class was first developed for [Easy Digital Downloads](https://github.com/pippinsplugins/Easy-Digital-Downloads/) and used to track payment creation, file downloads, purchase errors, and more.

Log Types
=========


Log entries are designed to be separated into "types". By default the class has "error" and event" log types. You can change these to anything you want by modifying the array in the log_types() method:

```php
private function log_types() {
	$terms = array(
		'error', 'event'
	);

	return apply_filters( 'wp_log_types', $terms );
}
```

You could also register additional log types using a filter:

```php
function pw_add_log_types( $types ) {
	
	$types[] = 'registration';
	return $types;

}
add_filter( 'wp_log_types', 'pw_add_log_types' );
```

Recording Log Entries
=====================

There are two ways to record a log entry, one is quick and simple and the one is more involved, but also gives more control.

Using `add()`:

```php
$log_entry = WP_Logging::add( $title = '', $message = '', $parent = 0, $type = null );
```

* _$title_ - The log entry title (string)
* _$message_ - The log entry message. (string)
* _$parent_ - The post object ID that you want this log entry connected to, if any (int)
* _$type_ - The type classification to give this log entry. Must be one of the types registered in log_types() (string)

A sample log entry creation might look like this:

```php
$title 		= 'Payment Error';
$message 	= 'There was an error processing the payment. Here are details of the transaction: (details shown here)';
$parent 	= 46; // This might be the ID of a payment post type item we want this log item connected to
$type 		= 'error';

WP_Logging::add( $title, $message, $parent, $type );
```


Using `insert_log()`:

```php
$log_entry = WP_logging::insert_log( $log_data = array(), $log_meta = array() );
```

This method requires that all log data be passed via arrays. One array is used for the main post object data and one for additional log meta to be recorded with the log entry.

The `$log_data` array accepts all arguments that can be passed to [wp_insert_post()](http://codex.wordpress.org/Function_Reference/wp_insert_post), with one additional parameter for `log_type`.

The $log_data array expects key/value pairs for any meta data that should be recorded with the log entry. The meta data is stored is normal post meta, though all meta data is prefixed with `_wp_log_`.

Creating a log entry with `insert_log()` might look like this:

```php
$log_data = array(
	'post_title' 	=> 'Payment Error',
	'post_content' 	=>  'There was an error processing the payment. Here are details of the transaction: (details shown here)',
	'post_parent'	=> 46, // This might be the ID of a payment post type item we want this log item connected to
	'log_type'		=> 'error'
);

$log_meta = array(
	'customer_ip' 	=> 'xxx.xx.xx.xx' // the customer's IP address
	'user_id' 		=> get_current_user_id() // the ID number of the currently logged-in user
);

$log_entry = WP_Logging::insert_log( $log_data, $log_meta );
```

Retrieving Log Entries
======================

There are two methods for retrieving entries that have been stored via this logging class:

* WP_Logging::get_logs( $object_id = 0, $type = null, $paged = null )
* WP_Logging::get_connected_logs( $args = array() )