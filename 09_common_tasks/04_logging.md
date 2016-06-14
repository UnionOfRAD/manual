# Logging

Logging in li3 is handled through the `Logger` class.  This class is designed to provide a consistent interface for writing log messages across the application.  The `Logger` class can also be set up with a series of named configurations that contain a log adapter to write to.

## Configuration

When configuring adapters, you may specify one or more priorities for each, using the `'priority'` key. This key can be a single priority level (string), or an array of multiple levels. When a log message is written, all adapters that are configured to accept the priority level with which the message was written will receive the message.

```php
Logger::config(array(
	'default' => array('adapter' => 'Syslog'),
 	'badnews' => array(
 		'adapter' => 'File',
 		'priority' => array('emergency', 'alert', 'critical', 'error')
 	)
 ));
```

## Usage

The `Logger` class has a single public function called `write()`.  This is the method used to write log messages.  li3 also allows you to use the priority as the method name.  For instance, in the above configuration example, you could call the `Logger::critical()` method

### Example

```php
// Using the `write()` method:
Logger::write('critical', 'This is a critical message');

// is the same as:
Logger::critical('This is a critical message');
```

By default, the logger will write a message to any adapter that has the specified priority in the log message in its configuration.  To write to an adapter other than the default adapter(s), you can use the options parameter which expects an array.

```php
Logger::write('critical', 'This is a critical message', array('name' => 'badnews'));
```

<div class="note note-info">
	Attempting to use an undefined priority level will raise an exception. See the list of available adapters for more information on what adapters are available, and how to configure them.
</div>

## See Also

* [List of Logger Adapters](/docs/lithium/analysis/logger/adapter)
* [Logger API Documentation](/docs/lithium/analysis/Logger)
