# Lithium's Unit Testing Framework

Applications with any amount of complexity or reuse necessitate test coverage. Lithium's unit testing framework is home grown, and is used for the framework's own testing. It's simple, lightweight, and ready for immediate use.

## Getting Started

Since the unit testing framework is built into Lithium, you might already have it up and running. Once you've downloaded and installed Lithium, point your web browser to `/test` under your application's base URL.

The Lithium Unit Test Dashboard is where you'll be able to view test cases, run unit tests, and view reports. Initially, you'll only be seeing Lithium's core tests. Soon enough, however, you'll be managing your own application's unit testing setup.

All of your application's unit tests will reside in `/app/tests/`. There are three main test folders you'll need to be using: `cases`, `integration`, and `mocks`. The `cases` folder holds unit tests for single classes, `integration` holds test cases that span two or more classes, and `mocks` is used to create fake data for use during testing.

## Test Cases

The `cases` folder is used to house all the core logic for your unit tests. If you take a peek inside `/app/tests/cases`, you'll see that you already have three folders used to organize your application's unit tests. This folder structure dictates the namespace for each unit test class, and should generally mirror your application's class/namespace structure.

Let's start out by creating a simple test case as a working example. Our first working example will be a model unit test. Let's start by creating one using the `li3 create` console command.

{{{
$ cd /path/to/lithium/app
$ li3 create model Post

Post created in app\models.
}}}

We can also use the `li3 create` command to create our test case class.

{{{
$ li3 create test model Post

PostTest created for Post in app\tests\cases\models.
}}}

Doing so creates a test file template class that extends `\lithium\test\Unit` and looks like the following:

{{{
<?php

namespace app\tests\cases\models;

use app\models\Post;

class PostTest extends \lithium\test\Unit {

	public function setUp() {}

	public function tearDown() {}

}

?>
}}}

The two initial methods supplied act as they're named. The `setUp()` method is used to perform any preparation work you'll need to perform your unit testing logic. This might be anything from setting up database connections to initializing mock data. Similarly, `tearDown()` is used to clean up anything that might be left over once a unit test has been completed. These methods are called before and after each method in your unit test case.

The meat of the unit test, however, will be housed inside of methods you create. Each piece of your unit testing logic should be placed inside of a method whose name starts with 'test'. Before we make any adjustments to the `Post` model, let's exercise a bit of TDD and write an example test method first.

Since our test case is a subclass of `lithium\test\Unit`, we have easy access to a number of methods that help us validate test assertions. Since they're plainly named, I'll list some here. For for information, please refer to the API documentation for `lithium\test\Unit`.

 - `assertEqual()`
 - `assertNotEqual()`
 - `assertIdentical()`
 - `assertTrue()`
 - `assertFalse()`
 - `assertNull()`
 - `assertNoPattern()`
 - `assertPattern()`
 - `assertTags()`
 - `assertCookie()`
 - `expectException()`

Every post should have a great title, and any editor knows that post titles containing the phrase "top ten" are pure rubbish. We'll eventually need a method in our Post model that searches for this phrase and warns us. Before writing that method, let's establish a test case to cover it. We'll call it `testIsGoodTitle()`. See an example implementation below:

{{{
<?php

namespace app\tests\cases\models;

use app\models\Post;

class PostTest extends \lithium\test\Unit {

	public function setUp() {}

	public function tearDown() {}

	public function testIsGoodTitle() {
		$this->assertTrue(Post::isGoodTitle("How to Win Friends and Influence People"));
		$this->assertFalse(Post::isGoodTitle("The Top 10 Best Top Ten Lists"));
	}
}

?>
}}}

Turn back to your browser showing the Unit Test Dashboard, and refresh it. You should see a new entry at the top of the list on the left hand side that shows our `PostTest` unit test case. Clicking on the `PostTest` test case should show you the test results. At this point you won't get far—the model will likely complain about a missing connection or function: as it should!

Let's start working on the model so we can get that test to pass. First, let's specify our model as not having any connection. We'll adjust this later, but let's do this now for simplicity's sake.

{{{
<?php

namespace app\models;

class Post extends \lithium\data\Model {
	protected $_meta = array('connection' => false);
	public $validates = array();
}

?>
}}}

Once that's in place, running the test again should have it barking about how `isGoodTitle()` hasn't been defined. Let's provide a rudimentary implementation in the model to satisfy it:

{{{
<?php

namespace app\models;

class Post extends \lithium\data\Model {
	protected $_meta = array('connection' => false);
	public $validates = array();

	public static function isGoodTitle($title) {
		return !stristr($title, 'top ten');
	}
}

?>
}}}

At this point, your test cases should run successfully in the Unit Test Dashboard.

## Mocks

Mocks are used in place of actual sources of information. You can create a mock for just about anything: a data source, model data, a console command response... anything. Since we're dealing primarily with the model in this example, let's continue that train of thought, and use some mocks to help us test our new model functionality.

Let's create a MockPost that returns test data we can use to run through our `isGoodTitle()` method. One easy way to do that is to create a new class that just returns a RecordSet (in the case of an SQL database) or a Document (in the case of a document database) collection.

Start by creating a new file in `app/tests/mocks/data/MockPost.php`:

{{{

<?php

namespace app\tests\mocks\data;

use lithium\data\collection\RecordSet;

class MockPost extends \app\models\Post {
	public static function find($type = 'all', array $options = array()) {
		switch ($type) {
			case 'first':
				return new RecordSet(array('data' =>
					array('id' => 1, 'title' => 'Top ten reasons why this is a bad title.')
				));
			break;
			case 'all':
			default :
				return new RecordSet(array('data' => array(
					array('id' => 1, 'title' => 'Top ten reasons why this is a bad title.'),
					array('id' => 2, 'title' => 'Sensationalist Over-dramatization!'),
					array('id' => 3, 'title' => 'Heavy Editorializing!'),
				)));
			break;
		}
	}
}

}}}

What we've got here is essentially a model that spits out hard-coded data when we call `find()`. In some cases, this might really be all we need. Let's use this in our main test case by adding the following function:

{{{

public function testMockTitles() {
	$results = MockPost::find('all');

	$first = $results->current();
	$this->assertFalse(MockPost::isGoodTitle($first['title']));
}

}}}

Head back to the Unit Test Dashboard to make sure this runs successfully, and you're done!



