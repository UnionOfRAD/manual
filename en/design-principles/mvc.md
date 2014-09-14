# Model-View-Controller

Understanding the Model View Controller (or MVC) approach to building web applications is fundamental to programming with Lithium. At a basic level, MVC programming allows the programmer to separate the way information is presented from how it is gathered and processed. Maintaining this separation makes adding new features simpler and more intuitive. It also makes fixing bugs and creating entirely new ways to view your application much easier.

While not a comprehensive guide on MVC, this section aims to introduce you to the concept as well as impart a few best practices from the Lithium perspective.

## Models

The primary goal of a model is to manage domain knowledge for a particular object. Model classes in Lithium are typically named after the object they manage. The model is in charge of CRUD operations for the object, data validation, as well as commonly used data manipulation operations upon that object's data.

Most applications contain a model that manages the user domain. Features of this Users model might include:

* Validation of user information before saving (valid email address, required field enforcement)
* Simple saving and fetching of user data
* Concatenation of the first and last names for a "virtual" full name field
* Complicated finder methods for user data (finding a user by account status or age)

## Views

Views present model data and the interface for your application. They contain some logic, but it's usually pretty light things like conditionally showing information, reusable bits of presentational code, or looping through objects in order to present them.

While most views in Lithium contain HTML, MVC programming allows you to switch between presentational technologies easily. A well-built MVC application should be able to show and switch between HTML, PDF, SVG or JSON with minimal effort.

## Controllers

In desktop software, controllers typically respond to user commands. Similarly on the web, a controller handles an HTTP request. Logic in a controller is mainly focused on application flow.

Typical controller functions might include:

* Authenticating a request, and re-routing a user based on permissions
* Using models to hand the view layer processed data
* Handling an error in the application
* Forwarding a user along to a different action based on model state

Controllers are often named after the model they primarily deal with.

## MVC Best Practices

Now that you know the primary purposes of each part of the MVC architecture, you should also be aware of a number of MVC best practices that will keep your code neatly organized and easier to maintain:

* Models should be feature-rich; Controllers should be very thin.
* Views should only contain logic that is presentational in nature. This usually means no access to models.
* Keep presentational languages like HTML out of your models and controllers.
* Actions dealing primarily with a given model should be placed in that model's controller.