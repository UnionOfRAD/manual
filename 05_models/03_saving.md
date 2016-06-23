# Saving Entities

Persisting data means that new data is being stored or updated. Before you can save your data, you have to initialize a `Model`. This can either be done with the `find()` methods shown earlier or—if you want to create a new entity with the static `create()` method.  `save()` is a method (called on record and document objects) to create or update the record or document in the database that corresponds to `$entity`.

```php
// Create a new post, add title and author, then save.
$post         = Posts::create();
$post->title  = 'My first blog post.';
$post->author = 'Michael';
$post->save();

// Same as above.
$post = Posts::create([
	'title'  => 'My first blog post.',
	'author' => 'Michael'
]);
$post->save();

// Find the first blog post and change the author
$post = Posts::first();
$post->author = 'John';
$post->save();
```

Note that `save()` also validates your data if you have any validation rules defined in your model. It returns either `true` or `false`, depending on the success of the validation and saving process. You'll normally use this in your controller like so:

```php
public function add() {
	$post = Posts::create();

	if($this->request->data && $post->save($this->request->data)) {
		$this->redirect('Posts::index');
	}
	return compact('post');
}
```
This redirects the user only to the `index` action if the saving process was successful. If not, the form is rendered again and errors are shown in the view.

To override the validation checks and save anyway, you can pass the `'validate'` option:

```php
$post->title = "We Don't Need No Stinkin' Validation";
$post->body = "I know what I'm doing.";
$post->save(null, ['validate' => false]);
```

<div class="note note-hint">
	There is more information about validation and how to use the
	<code>validates()</code> in the <a href="./validation.md">Validation</a> chapter. 
</div>

The `$entity` parameter is the record or document object to be saved in the database. This parameter is implicit and should not be passed under normal circumstances. In the above example, the call to `save()` on the `$post` object is transparently proxied through to the `Posts` model class, and `$post` is passed in as the `$entity` parameter.  The `$data` parameter is for any data that should be assigned to the record before it is saved.

There is also an `$options` parameter that has the following settable elements.

* `'callbacks'` _boolean_: If `false`, all callbacks will be disabled before executing. Defaults to `true`.
* `'validate'` _mixed_: If `false`, validation will be skipped, and the record will be immediately saved. Defaults to `true`. May also be specified as an array, in which case it will replace the default validation rules specified in the `$validates` property of the model.
* `'events'` _mixed_: A string or array defining one or more validation _events_. Events are different contexts in which data events can occur, and correspond to the optional `'on'` key in validation rules. They will be passed to the validates() method if `'validate'` is not `false`.
* `'whitelist'` _array_: An array of fields that are allowed to be saved to this record.


