WP-Logging
==========

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