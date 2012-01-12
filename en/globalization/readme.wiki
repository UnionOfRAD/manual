# Globalization Tutorial

## Introduction

Globalization, or [ g11n](https://secure.wikimedia.org/wikipedia/en/wiki/G11n), is a term that combines the features found in internationalization and localization. Globalization makes it possible to adapt a single piece of software to different languages, scripts and regional language differences. This tutorial describes the approach Lithium takes towards globalizing software.

Globalization in Lithium is a core framework feature, and Lithium has been developed with g11n in mind from the start.

## Bootstrapping g11n

Enabling g11n in your application starts by loading it in the Lithium bootstrap process. Start by uncommenting the g11n line in `bootstrap.php`. The `g11n.php` file contains your application's globalization rules, including inflections, transliterations, localized validation, and how localized text should be loaded.

{{{
require __DIR__ . '/bootstrap/g11n.php';
}}}

## Creating Default Locale Settings

The settings for the current locale (and available locales) are kept as environment settings inside `g11n.php`. This allows for a central place to switch, set, and retrieve g11n-related settings. It also makes very clear what the current locale settings are.

Here, you can decide which locale is your default effective locale, and also which locales your application will support. Do so by defining environment variables like so:

{{{
Environment::set('development', array(
    'locale' => 'en', // the default effective locale
    'locales' => array('en' => 'English') // locales available for your application
));
}}}

## Locale Detection

The _effective locale_ depends on the availability of globalized application content and the locale preferred by the client.

__But how do you identify the preferred locale(s)?__

### General Approach

There are two main ways to make guesses about which locale to make effective. First, you can guess based on the name of the resource (parts of the request URL). Second, you can make some informed guesses based on information you receive from the client (user agent information, etc.).

In order to keep the process as transparent as possible, we recommend inspecting resources first, and falling back on client information as necessary.


### Technical Implementation

The detection of the effective locale works differently for controller action and console application requests.

For controller action requests, the `Locale` class parses the locales contained within the ```Accept-Language``` header (which is sent by the user agent on behalf of the user). For console requests, the preferred locale is retrieved by looking at certain environment settings.

While determining the effective locale in an semi-automatic way works well for console requests, it is a less successful approach with controller action requests. As such, we recommend using pieces of the request URL to set the effective locale. The side benefit is that the locale is also in plain view to the user, making the decision completely transparent. We'll cover more on localized routing in a bit, but for now, understand that URL-based g11n is going to be your best bet as far as user experience goes.

### g11n Filters

One of the last things that `g11n.php` does in the bootstrap process is apply a filters to both the console command and controller action dispatchers. Here is the perfect place to decide how we set and switch between locales based on the information we have from the request and user agent.

{{{
Dispatcher::applyFilter('_callable', function($self, $params, $chain) {
    $request = $params['request'];
    $controller = $chain->next($self, $params, $chain);

    if (!$request->locale) {
        $request->params['locale'] = Locale::preferred($request);
    }
    Environment::set(Environment::get(), array('locale' => $request->locale));
    return $controller;
});
}}}

This filter does the following:

1. Checks to see if the request explicitly states the locale to use
2. In case it does not, detects the locale from request
3. Sets the effective locale from the request

Generally speaking, if the preferred locale is available, then it is set as the effective locale. If it's not, the next locale accepted by the client is matched against the available ones. If none of the accepted locales matches, the detection fails.

While this setup should meet most g11n needs, feel free to implement your own according to the needs of your application. For example, you could alter this filter to search an IP-to-Geo database, and set the effective locale based on the location of the IP address of the request.

## Localized routing

There is [ a great deal of thought](http://h3h.net/2007/01/designing-urls-for-multilingual-web-sites/) on the different approaches to embedding locale information in URLs. While each have their respective advantages, the Lithium g11n framework uses prefixed routes.

A localized route is configured by connecting a continuation route. This example shows how such a route looks like.

Once the route has been connected, all the other application routes become localized and may now carry a locale:

{{{
<?php

Router::connect('/{:locale:[a-zA-Z_]+}/{:args}', array(), array('continue' => true));

?>
}}}

Once this is in place, our application's routes will illustrate the effective locale, i.e.:

 - `http://example.com/en/controller/action/param1`
 - `http://example.com/fr/controller/action/param1`

As a final example, here's a bit of view code you could use to switch between locales:

{{{
<?php use lithium\core\Environment; ?>
 <div id="locale-navigation">
	<ul>
		<?php foreach (Environment::get('locales') as $locale => $name): ?>
			<li><?=$this->html->link($name, compact('locale') + $this->_request->params); ?></li>
		<?php endforeach; ?>
	</ul>
</div>
}}}

## Globalization Catalog Adapters

G11n data is not just translated messages: it's informed validation rules, date/number/currency formats, and a lot more. Data is grouped into four different kinds of categories: _inflection_ (not yet used), _validation_, _message_ and _list_.

The `Catalog` is a class that allows us to retrieve and store globalized data, providing low-level functionality to other classes. Because it extends `Adaptable`, its interface is similar to classes like `Session` or `Cache`, and is extensible through adapters.

The class is able to aggregate globalized data from different sources, which allows a developer to complement sparse data. Example usage:

{{{
<?php

use lithium\g11n\Catalog;

// Configures the runtime source.
Catalog::config(array('runtime' => array('adapter' => 'Memory')));

$data = array(
	'laboratory' => 'Labor',
	'life' => 'Leben'
);
Catalog::write('message', 'de', $data, array('name' => 'runtime'));

// Reads from the runtime g11n configuration
Catalog::read('runtime', 'message', 'de');

// Results for the `de_DE` locale are merged with those for `de`
Catalog::read('runtime', 'message', 'de_DE');

// Reads from just the runtime source.
Catalog::read('runtime', 'message', 'de', array('name' => 'runtime'));

?>
}}}

By default, the g11n bootstrap configures a `runtime` (using the `Memory` adapter) and a `lithium` catalog (using the php adapter for validations and translation shipped with the framework).

For an ad-hoc solution (i.e. translating API messages) it's workable to utilize the `runtime` catalog, but most other use cases will require storing translated messages in files.

In order to allow the application to read write gettext resource files (PO and MO) we configure `Catalog` as follows:

{{{
Catalog::config(array(
	'runtime' => array('adapter' => 'Memory'),
	'app' => array('adapter' => 'Gettext', 'path' => LITHIUM_APP_PATH . '/resources/g11n'),
	'lithium' => array(
		'adapter' => 'Php',
		'path' => LITHIUM_LIBRARY_PATH . '/lithium/g11n/resources/php'
	)
) + Catalog::config());
}}}


### Available Catalog Adapters

The **gettext adapter** reads and writes PO and MO files. While the current implementation of gettext is not thread safe, this PHP only implementation is thread safe and allows reading binary MO files independent of the machine it was created on (and its endian-ness). Both 32bit and 64bit systems are also supported.
 _Note:_ Currently contexts are not supported.

The **memory adapter** allows for writing and reading globalized data needed at runtime, and as such is helpful in testing.

The **code adapter** extracts message IDs for creating message catalog templates from source code. PHP is supported through a parser (using the built-in tokenizer) but future support for other formats (i.e. JavaScript) is possible too.

The **php adapter** allows for reading from files containing data in PHP format.
{{{
return array('the artists' => 'die Künstler');
}}}

### G11n Data Storage

Nearly all adapters require a path to a directory containing globalized data. Data is conventionally stored in the following default locations:

- `libraries/lithium/g11n/resources[/{php,po}]`
- `app/resources/g11n[/{php,po}]`
- `app/plugins/PLUGIN/resources/g11n[/{php,po}]`

Data can either be stored directly below `resources/g11n` or in a subdirectory. If data is stored in a subdirectory, adapters expect a `php` subdirectory to contain data for usage with the PHP adapter and `po` to contain files for usage with the gettext adapter.

More information on the directory structures required by the different adapters can be found in the docblocks of the adapters in `g11n\catalog\adapter\*` namespace.

## Localized Validation Settings

Lithium's g11n framework ships with some localized validation rules for phone number, postal code and social security number validation rules. The `g11n.php` bootstrap file defines these additions so they're available before you use them:

{{{
use lithium\util\Validator;

foreach (array('phone', 'postalCode', 'ssn') as $name) {
    Validator::add($name, Catalog::read('runtime', "validation.{$name}", 'en_US'));
}
}}}

Once those changes have been bootstrapped, you can use this g11n-aware validation logic in your application logic like so:

{{{
Validator::isPhone('PHONE NUMBER US', 'en_US');
}}}

## Using Localized Content

While it may seem like a simple process from the outside, message globalization is actually a multi-step process. This section covers those steps, enabling you to fully use globalized content in in your views. Here are the steps we'll cover:

1. Marking messages as translatable.  The `$t()` and `$tn()` shortcut functions help the extraction tools identify where messages have been in views.

1. Extracting marked messages.  Messages can be extracted through the `g11n` command, and placed in a template.

1. Translating messages. Once a template has been created, translators add localized content to them. The completed templates are often stored on disk.

1. Retrieving the translation for a message. The `$t()` and `$tn()` functions are used to output translated content for the correct locale in a view.

## Marking Messages as Translatable

The two convenience aliases for `Message::translate()`—`$t()` and `$tn()`—are injected into the view as output filters by default. This allows for the following syntax throughout templates:

{{{
<?= $t('green'); ?>
<?= $tn('House', 'Houses', array('count' => 3)); ?>
<?= $t('Everything is so {:color}.', array('color' => $t('green'))); ?>
}}}

If you need access to translated content outside of a view, use `extract()` to use these aliases like so:

{{{
<?php

namespace app\controllers;

use lithium\g11n\Message;

class PostsController extends \lithium\action\Controller {

	public function add() {
		extract(Message::aliases());
		// ...
		$message = $t('Post successfully added.');
		// ...
	}
}

?>
}}}

### Best Practices

Since the marked messages will later be translated by many others, it's important to keep a few best practices in mind.

When embedding translated messages into your application, it's best to use entire sentences as identifiers rather than just single words. Since a translator is often viewing a long list of strings needing to be translated, context is everything.

{{{

// Not so great:
<?= $t('welcome'); ?>

// Better:
<?= $t('Welcome to Gary\'s Fine Clarinets™. How can we help you today?'); ?>

}}}

It's also best to split paragraphs into single sentences as well. This makes things more atomic and granular where large chunks of content may be difficult to work with:

Next, treating text like you would with `String::insert()` can make things easier in the long run, especially in contrast to using bare string concatenation with the `.` operator.

{{{

// Not so great:
<?= "Gary's Fine Clarinets " . $t('is really great'); ?>

// Better:
<?= $t('Everything is so {:color}.', array('color' => $t('green'))); ?>

}}}

Also, avoid the use of escaped characters, or markup of any kind inside of translated messages. This makes it much less error prone, as your language team may not be as keen on well-structured HTML as you are.

### No Operation

Passing the `'noop'` option to `Message::translate()` will result in the default message being returned. Since the short-hand translation functions use `translate()` internally, you can use the option to just mark a string for translation without it actually being translated during runtime:

{{{
<?= $t('foo', array('noop' => true)); ?>
<?= $tn('foo', 'bar', array('noop' => true)); /* we don't need to pass `'count'` in this case */ ?>
}}}

File: `a.php`:
{{{
<?= $t('foo', array('noop' => true)); /* the extractor picks up `foo` */ ?>
}}}

File: `b.php`:
{{{
<?php $section = 'foo'; ?>
<?= $t($section); ?>
}}}

### Knock Out

If you're using templates which use the aliased translation functions but don't want to do any globalization, you can disable the lookup of translations by using a filter.

{{{
Message::applyFilter('_translated', function($self, $params, $chain) {
	return null;
});
}}}

## Extracting Marked Messages & Creating Templates

Marked messages are extracted using the `g11n` command. This allows for extracting messages and comments from source files, creating and updating files containing message templates, creating and updating files containing translated messages and compilation of those.

{{{
li3 g11n extract [--source=DIRECTORY] [--destination=DIRECTORY]
}}}
