Laravel has a lot of cool and useful helper functions built into it. However sometimes we need to create custom helper functions, and we need them all over our application. Here I'll explain how to create a Helper class to generate custom helper functions in Laravel 5 and Lumen. 

To start with create a directory in your `app` directory called `Libraries` and inside create a new PHP file called `Helpers.php`. This will be where we build our helper class. 

The contents of the `Helper` class should look something like the below, but obviously with a more complex function that `hello_world`.

```php

class Helpers
{
	public static function hello_world()
	{
		return 'Hello World';
	}   
}

```

Next we just need to load in our `Helpers` class. So head over to your `composer.json` file and add in the following,

```json

"autoload": {
	"classmap": [
		"app/Libraries"
	]
}

```

You will likely already have a `autoload` object there so just add `"app/Libraries",` to the `classmap` object within it. 

Now when your app loads you will have access to the `Helpers` class. So all we have to do now is use our new custom Laravel helper function. In a view you can use it like this,

```php
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Hello World</title>
</head>
<body>
	<h1>{{ Helpers::hello_world() }}</h1>
</body>
</html>

```

and in a controller we can use it like this,

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Helpers;

class LoginController extends Controller
{
	public function index()
	{
		return view('helloworld', array(
			'page_title' => Helpers::hello_world()
		));
	}
}
```

Remember in Laravel 5 and Lumen you need to have the `use Helpers` statement at the top due to name spacing. 

Thats all it takes to create customer helper functions in Laravel 5 and Lumen. These can then be used anywhere in your app.
