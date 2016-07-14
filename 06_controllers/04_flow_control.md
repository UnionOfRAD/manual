# Flow Control


Occasionally a controller action will want to divert, re-route, or automatically configure the view layer based on an incoming request. There are various controller methods to help facilitate request flow handling.

## Redirection

The most basic type of flow control at the controller level is redirection. It's common to redirect a user to a new URL once an action has been performed. This type of control is done through the controller's `redirect()` method. Here's an example of a controller action that redirects the request:

```php
public function add() {
	// Validate and save user data POSTed to the
	// controller action found in $this->request->data...

	return $this->redirect(["Users::view", "id" => $user->id, "?" => "welcome"]);
}
```

The URL specified can be relative to the application or point to an outside resource. The `redirect()` function also features a second `$options` parameter that also allows you to set HTTP status headers, and make decisions about whether or not to `exit()` after a redirect. Be sure to check the API for `lithium\action\Controller::redirect()` for more details.

## Other Types of Flow Control

li3 provides other options for request flow control, such as [exception and error handling](exceptions-errors) and [type rendering & detection](type-rendering-detection), both of which are explained in more detail in their respective manual pages.
