# Models 

Models play a key role in nearly every web application. Their main purpose is to abstract business logic and database operations from higher levels (controllers and views). They also act as a gatekeeper and—if properly implemented—make sure that only valid and allowed data gets passed through them. Models in li3 have three main purposes:

 1. Provide an abstraction layer to the underlying data source(s)
 2. Perform common data operations (like [fetching](querying.md) and [storing](saving.md) data)
 3. Help with [validating](validation.md) data.

Also, li3 makes it easy to extend models so that they fit your application's needs. Thanks to the nifty autoloading mechanism, models are lazy-loaded and are only initialized when you need them. In the next sections you will learn how to use models and perform common operations on them. Later sections will provide you with a more detailed look on models like [relationships](relationships.md) between them and how to [extend models](adding-functions-to-models.md) to fit your application's needs.

The `Model` class is the starting point for the domain logic of your application.  Models are tasked with providing meaning to otherwise raw and unprocessed data (e.g. user profile).  Models expose a consistent and unified API to interact with an underlying datasource (e.g. MongoDB, CouchDB, MySQL) for operations such as querying, saving, updating and deleting data from the persistent storage.

<div class="note note-info">
	As the framework is capable of working with document oriented data sources as it is with relational databases, we use the term <em>entity</em> to refer to what might be considered a document in one data source type or a record/row in another type.
</div>

Models allow you to interact with your data in two fundamentally different ways: [querying](querying.md) and [data mutation](data-mutation.md) (saving/updating/deleting). All query-related operations may be done through the static `find()` method, along with some additional utility methods provided for convenience. Classes extending the `Model` class should, conventionally, be named as plural, CamelCase and be placed in the `models` directory. i.e. a posts model would be `model/Posts.php`.

## Where To Go Next

Read the section below on how to _create a model_ first, then continue with a quick look into the basic of creating [connections](connections.md), creating/updating/deleting entites in [data mutation](data-mutation.md), persisting entities by [saving](saving.md) them to the datastore. Finally [querying](querying.md) shows how to get all the precious data back.

## Creating a Model

li3 provides you with a general-purpose class that all your models should extend. You can find the `Model` class in the `lithium\data` namespace. If you do nothing more than extend it, you instantly get a bunch of functionality that covers basic CRUD as well as more complex tasks.

Let's say you want to store and manage blog posts in your database. According to our conventions, you create a new file called `Posts.php` in `app/models`. The basic structure looks like this:

```
namespace app\models;

class Posts extends \lithium\data\Model {}
```

<div class="note note-hint">
	li3 also allows model creation via the console:  You can enter <code>li3 model create Posts</code> into the command line (assuming you have configured the command line for use) and the code above will automatically be created in a file called <code>\app\models\Posts.php</code>.
</div>

