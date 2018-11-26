This was a [question I actually answered on StackOverflow](http://stackoverflow.com/questions/26015256/laravel-ajax-call-to-a-function-in-controller/26015321#26015321) a few weeks ago. However, it got me thinking about a nice post that I could do where I could elaborate a little more on how to make an AJAX call to a [Laravel](http://laravel.com/) controller function / method. 

If you are new to Laravel or AJAX this could seem a little tricky, but trust me once you understand it it'll become really easy. 

First we need to set up our controller in [Laravel](http://laravel.com/) so that we have something for the AJAX to call. So assuming you are using Laravel 5 that would be in `app/Http/Controllers` lets make a PHP file called `AjaxController.php`.

To start with we need to put the bare bones of a Laravel controller in there,

```php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class AjaxController extends Controller
{

}

```

I have put the `use Illuminate\Http\Request;` statement in there because we are going to be using Laravel's `Request` object to get our data. 

Next we need to create our method which AJAX will call. Let's assume you want to call the controller function via AJAX to update a customer record. So lets create a method `updateCustomerRecord`,

```php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class AjaxController extends Controller
{
	public function updateCustomerRecord(Request $request)
	{
		// do something here
	}
}

```

Now notice in the parameters of the function I have put in `Request $request`. For those of you that don't know what that means, what I am doing is passing in the request object as a variable called `$request` and I am saying it has to be of type `Request`. Which uses our `use` statement we put in earlier to expand to say it must be of type `Illuminate\Http\Request`. Laravel will by default pass this into our method. So we don't need to worry about doing that, we just need to catch it to use it. 

So now we need to do something in our method. For now, lets just dump out what ever data we are going to send via AJAX to the controller.

```php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class AjaxController extends Controller
{
	public function updateCustomerRecord(Request $request)
	{
		$data = $request->all(); // This will get all the request data.

		dd($data); // This will dump and die
	}
}

```

That way we will be able to test to see that our data is making it to our controller method. 

Next we need a way of ensuring AJAX is able to call our controller method. We do this by creating a route, in `routes/web.php`.

```php
Route::post('/customer/ajaxupdate', 'AjaxController@updateCustomerRecord');
```

Couple of bits to note here. We are using a `POST` route, as we will be posting data to the URL using AJAX. The second is that we are saying to Laravel, if you get a `POST` request to the url `/customer/ajaxupdate` then go to controller `AjaxController` and use method `updateCustomerRecord`.

So now we have our method, and our way of calling it. Now we just need the AJAX to call the Laravel function. We will use JQuery's `$.ajax` function for this,

```javascript
var id = 12; // A random variable for this example

$.ajax({
	method: 'POST', // Type of response and matches what we said in the route
	url: '/customer/ajaxupdate', // This is the url we gave in the route
	data: {'id' : id}, // a JSON object to send back
	success: function(response){ // What to do if we succeed
		console.log(response); 
	},
	error: function(jqXHR, textStatus, errorThrown) { // What to do if we fail
        console.log(JSON.stringify(jqXHR));
        console.log("AJAX error: " + textStatus + ' : ' + errorThrown);
	}
});
```
I will assume that you can put this AJAX in a function a call it when you actually want to send the request. If not, I think that's another tutorial. 

So there we have it. In 3 simple steps we have called a Laravel controller function / method using AJAX. First creating the method in the controller, then creating a route allowing the AJAX to call it, finishing creating the AJAX itself. Simple. 
