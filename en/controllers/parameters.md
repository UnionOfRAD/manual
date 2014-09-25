# Parameters

An important part of a controller's role in an application is processing incoming request data to show the correct response. In this section, we'll show a few examples that should help you to get data into your controller actions.

## GET Parameters

One of the most user-friendly ways to handle incoming data is through the URL. Information passed along with a GET request is handled a number of different ways, based on your preferences.

The easiest way to handle incoming GET data is by realizing that URL segments that follow the action name are mapped to controller action parameters. Here's a few examples:

```
http://example.com/users/view/1  --> UsersController::view($userId);
http://example.com/posts/show/using+controllers/8384/  -->  PostsController::show($title, $postId);
```

GET information passed this way is also accessible via the incoming request object:

```
http://example.com/users/view/1  --> $this->request->args[0]
```

While we'll always recommend using clear URL-based variables, it's important to mention that GET parameters passed as raw query string variables are also available as an attribute of the incoming request:

```
http://example.com/users/view?userId=1  --> $this->request->query['userId']
```

## POSTed Data

POSTed data is also gathered from the request object. Form field data is found on the request object inside an array with keys named after the input elements generated on the referring page. For example, consider an HTML form that included these elements:

```html
<input type="text" name="title" value="How to Win Friends and Influence People" />
<input type="text" name="category" value="Self-Help" />
```

Accessing these values when submitted to a controller action is as easy as:

```php
$this->request->data['title'];
$this->request->data['category'];
```
