# Securing Applications

## Injection

### SQL Injection

[SQL Injections](https://en.wikipedia.org/wiki/SQL_injection) are a common type of attack
and can take very different forms.

The ORM will quote any values that are passed as condition values automatically in order to 
protect your application from injection attacks.

```php
$author = 'UNTRUSTED USER INPUT';

Posts::find('first', array(
	'conditions' => array('author' => $author)
));
```

However you **cannot rely** on this feature if you use user input in any other place of a query. The 
following example shows how untrusted user input can be used safely used to specify the set of 
fields to be retrieved. As a countermeasure we first check if the provided field actually exists.


```php
$field = 'UNTRUSTED USER INPUT';

if (!Posts::hasField($field)) {
	throw new Exception('Invalid field.');
}

Posts::find('first', array(
	'fields' => array($field, 'title')
));
```

## Mass Assignment

To prevent you application from opening up to the so called [mass assingment vulnerabilty](http://en.wikipedia.org/wiki/Mass_assignment_vulnerability), 
the framework provides you with the _whitelist_ feature. This whitelist can be used to limit the set of fields which get updated during create or update operations.

```php
$data = array(
	'name' => 'John Doe',
	'email' => 'haxx0r@example.org' // Added by the attacker.
);

$user = Users::findById($authed['user']['id']);
$user->save();
```

As a countermeasure additionally use the whitelist feature. So that in this
case really just the name field gets updated.

```php
$user = Users::findById($authed['user']['id']);
$user->save($data, array(
	'whitelist' => array('name')
));
```

<div class="note note-info">
	Always prefer whitelisting over blacklisting.
</div>




