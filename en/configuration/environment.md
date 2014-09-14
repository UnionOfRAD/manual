# Environment Configuration

Most applications use a set of different environments to validate new features and test the fixes for bugs before letting the end users see it. Lithium's `Environment` class offers a way to organize settings and make logic in your application conditional based on where the application thinks it resides.

Rather than wrapping conditionals around all of your database settings and API keys and endpoints, you can use the `Environment` class to direct the application automatically, based on context.

Lithium offers a default set of environments (development, test, and production) as well as a starting point for determining which one the application resides in. This guide covers the process of showing Lithium how to detect and define your different environments and use them in configuration details.

## Getting Started: Detection

The first step in using different environmental settings is telling Lithium how to detect where it is operating from. There are a number of clues often used in determining the environment: IP address or ranges, hostname pattern matching, or possibly even looking at the filesystem or database.

Environment detection is determined by default as follows:

 * The request is local (IP address 127.0.0.1, etc.): _development_.
 * The request URL or hostname starts with "test": _test_.
 * Neither of the above: _production_.

To customize this behavior, use the `Environment::is()` method inside a bootstrap file. Supply a closure that inspects an incoming `Request` and returns the name of the environment deteted. Here's a simple example:

{{{
Environment::is(function($request){
  $host = $request->env('HTTP_HOST');
  if ($host == 'myapp.local' || $host == 'localhost') {
    return 'development';
  }
  if (preg_match('/^qe/', $host)) {
    return 'qe';
  }
  if(preg_match('/beta/', $host)) {
    return 'staging';
  }
  return 'production';
});
}}}

This code defines four custom environments: _development_ if the hostname matches a predetermined set of hostnames, _qe_ if the hostname begins with "qe" (qe1.example.com, qe2.example.com), and _staging_ if the hostname contains "beta" (beta.example.com, www.example-beta.com).

If none of those conditions are met, the default is production.

## Environment-Specific Settings

Once detection is configured, you're able to set environment-specific variables inside of your bootstrap files.

{{{
Environment::set('development', array('service_endpoint', 'dev.service.example.com'));
Environment::set('staging'    , array('service_endpoint', 'beta.service.example.com'));
Environment::set('qe'         , array('service_endpoint', 'qe1.service.example.com'));
Environment::set('production' , array('service_endpoint', 'www.service.example.com'));

// If run on my local system:
Environment::get('service_endpoint'); // 'dev.service.example.com'

}}}

## Adaptable Environment Settings

Subclasses of the core `Adaptable` class (`Logger`, `Connections`, `Cache`, `Session`, etc.) can also be configured on an environmental basis. For core functionality, this is much cleaner than large sets of `Environment::set` calls in your bootstrap files.

The most common example is configuring different database connections based on environment. Here's a quick example:

{{{
Connections::add('default', array(
    'production' => array(
        'type'     => 'database',
        'adapter'  => 'MySql',
        'host'     => 'live.example.com',
        'login'    => 'mysqluser',
        'password' => 's3cr3tg035h3rE',
        'database' => 'app-production'
    ),
    'development' => array(
        'type'     => 'database',
        'adapter'  => 'MySql',
        'host'     => 'localhost',
        'login'    => 'root',
        'password' => '4d1fFeR3n75eCr37',
        'database' => 'app'
    )
));
}}}

While is is handy for using the same types of technologies in each environment, this mechanism allows you to switch between engines as well. For example, using different kinds of Cache engines in each environment:

{{{
Cache::config(array(
    'userData' => array(
        'development' => array('adapter' => 'File'),
        'production' => array('adapter' => 'Memcache')
    )
));
}}}

For more information about creating your own classes that utilize `Adaptable` and `Environment` for automatically changing settings, see the Adaptable guide in the Advanced Tasks section.
