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
WP_Logging::add( $title = '', $message = '', $parent = 0, $type = null );
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

WP_Logging::add( $title = '', $message = '', $parent = 0, $type = null );
``` 
