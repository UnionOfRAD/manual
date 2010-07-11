# Add functionality to your Model.

 1. Static methods: apply to the Model
 2. Instance methods: apply to each Record

Generally, this means that static methods are used in preparation for a accessing data. On the other hand, instance methods are most useful when data has already been accessed.

### Example: Preparing extra data 

{{{
<?php
namespace app\models;

class User extends \lithium\data\Model {

	protected static $_roles = array('admin', 'friend', 'stranger');

	public static function roles() {
		return static::$_roles;
	}
}
}}}



### Example: Adding a name method for displaying in the view

{{{
<?php
namespace app\models;

class User extends \lithium\data\Model {

	public function name($record) {
		return "{$record->firstName} {$record->lastName}";
	}
}
}}}

### Using the new method
{{{
<?php foreach ($users as $user) { ?>
	<p>Name: <?=$user->name()?></p>
<?php } ?>
}}}