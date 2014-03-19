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

* _$title_ (string) - The log entry title
* _$message_ - The log entry message (string)
* _$parent_ (int) - The post object ID that you want this log entry connected to, if any
* _$type_ (string) - The type classification to give this log entry. Must be one of the types registered in log_types(). This is optional.

A sample log entry creation might look like this:

```php
$title 		= 'Payment Error';
$message 	= 'There was an error processing the payment. Here are details of the transaction: (details shown here)';
$parent 	= 46; // This might be the ID of a payment post type item we want this log item connected to
$type 		= 'error';

WP_Logging::add( $title, $message, $parent, $type );
```
Or, without a type:
```php
$title 		= 'Payment Error';
$message 	= 'There was an error processing the payment. Here are details of the transaction: (details shown here)';
$parent 	= 46; // This might be the ID of a payment post type item we want this log item connected to

WP_Logging::add( $title, $message, $parent );
```
- - -

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

* `WP_Logging::get_logs( $object_id = 0, $type = null, $paged = null )`
* `WP_Logging::get_connected_logs( $args = array() )`

`get_logs()` is the simple method that lets you quickly retrieve logs that are connected to a specific post object. For example, to retrieve error logs connected to post ID 57, you'd do:

```php
$logs = WP_Logging::get_logs( 57, 'error' );
```

This will retrieve the first 10 entries related to ID 57. Note that the third parameter is for `$paged`. This allows you to pass a page number (in the case you're building an admin UI for showing logs) and WP_Logging will adjust the logs retrieved to match the page number.

If you need more granular control, you will want to use `get_connected_logs()`. This method takes a single array of key/value pairs as the only parameter. The `$args` array accepts all arguments that can be passed to [wp_insert_post()](http://codex.wordpress.org/Function_Reference/wp_insert_post), with one additional parameter for `log_type`.

Here's an example of using `get_connected_logs()`:

```php
$args = array(
	'post_parent' 	=> 57,
	'posts_per_page'=> 10,
	'paged'			=> get_query_var( 'paged' ),
	'log_type'		=> 'error'
);
$logs = WP_Logging::get_connected_logs( $args );
```

If you want to retrieve all log entries and ignore pagination, you can do this:

```php
$args = array(
	'post_parent' 	=> 57,
	'posts_per_page'=> -1,
	'log_type'		=> 'error'
);
$logs = WP_Logging::get_connected_logs( $args );
```

Both `get_logs()` and `get_connected_logs()` will return a typical array of post objects, just like [get_posts()](http://codex.wordpress.org/Template_Tags/get_posts)

Get Log Entry Counts
======================

The `get_log_count()` method allows you to retrieve a count for the total number of log entries stored in the database. It allows you to retrieve logs attached to a specific post object ID, of a particular type, and also allows you to pass optional meta options for only counting log entries that have meta values stored.

The method looks like this:

```php
WP_Logging:: get_log_count( $object_id = 0, $type = null, $meta_query = null )
````

To retrieve the total number of `error` logs attached to post 57, you can do this:

```php
$count = WP_Logging::get_log_count( 57, 'error' );
```

To retrieve the total number of logs (regardless of type) attached to post object ID 57, you can do this:

```php
$count = WP_Logging::get_log_count( 57 );
```
The third parameter is for passing a meta query array. This array should be in the same form as meta queries passed to [WP_Query](http://codex.wordpress.org/Class_Reference/WP_Query). For example, to retrieve a count of error log entries that have a user IP that match a specific IP address, you can do this:

```php
$meta_query = array(
	array(
		'key' 	=> '_wp_log_customer_ip',	// the meta key
		'value' => 'xxx.xx.xx.xx'			// the IP address to retrieve logs for
	)
);
$count = WP_Logging::get_log_count( 57, 'error', $meta_query );
```

Pruning Logs
======================

To prune older logs you need to first set the pruning conditional to true then set up a cron job to perform the pruning. It's recommended you set the cron job to hourly so that you can stay on top of pruning your logs. Running daily and deleting 100 logs (the default number) means that many sites would never actually stay caught up with log prunning.

```php
function activate_pruning( $should_we_prune ){
	return true;
} // rapid_activate_pruning
add_filter( 'wp_logging_should_we_prune', 'activate_pruning', 10 );


$scheduled = wp_next_scheduled( 'wp_logging_prune_routine' );
if ( $scheduled == false ){
	wp_schedule_event( time(), 'hourly', 'wp_logging_prune_routine' );
}
```

The default time period is to prune logs that are over 2 weeks old. To change that use `wp_logging_prune_when`. If we wanted to prune logs older than 1 month.

```php
function change_prune_time( $time ){
	return '1 month ago';
}
add_filter( 'wp_logging_prune_when', 'change_prune_time' );
```

The pruning query is run via `get_posts` and you can filter any arguement in the array with the `wp_logging_prune_query_args` filter. By default we prune 100 logs each time the pruning routine is run. You could certainly up this number if your server will handle the load.

Logs are set to bypass the WordPress trash system. If you want to have logs hit the WordPress trash system then you'd need to filter `wp_logging_force_delete_log` and return false.

```php
function hit_trash(){
	return false;
}
add_filter( 'wp_logging_force_delete_log', 'hit_trash' );
```

If you make your logs hit the WordPress trash then you'll need to write your own routine to clear the trash so logs don't build up for you.
