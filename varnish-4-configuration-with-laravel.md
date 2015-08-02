In my previous post I talked about my not so delicate introduction to [Varnish](https://www.varnish-cache.org/releases) 3.0. Since that time Varnish has upgraded its offering and I thought I would be useful to have a little tutorial on the site for anyone wandering the Internet looking for something useful.

As the title states, this is technically a tutorial on how to configure Varnish for [Laravel](http://laravel.com/). However, in terms of Varnish configuration there is only 3 lines difference. The rest is in a smart little package you install using composer on [Laravel](http://laravel.com/) called [Session Monster](https://github.com/HaiFangHui/sessionmonster). Which I will come onto later.

## Installing Varnish

So getting started head over to [Varnish](https://www.varnish-cache.org/releases) and download the release for your system. The docs will guide you through the process for you OS. I'm on Ubuntu so from now the file paths might be a little different if your running something else. Im also using [Nginx](http://wiki.nginx.org/Main), so if you are using Apache things might be different for you than explained here.

## Setting up the ports

Once you have Varnish install you need to set a few things up. First we need to go into the varnish script at `/etc/default/varnish` and change where varnish looks for things. So,

```bash
sudo nano /etc/default/varnish
```
and change
```bash
DAEMON_OPTS="-a :6081 \
             -T localhost:6082 \
             -f /etc/varnish/default.vcl \
             -S /etc/varnish/secret \
             -s malloc,256m"
```
in order to get

```bash
DAEMON_OPTS="-a :80 \
             -T localhost:6082 \
             -f /etc/varnish/default.vcl \
             -S /etc/varnish/secret \
             -s malloc,256m"
```
all we have done here is change the `-a` port number in the default deamon options. Varnish runs as a deamon in the background on your server, so we need to tell it which port number to listen on for web traffic. Which by default is port 80. Everything else we keep the same, and we can see in that default config that our configuration file is located where the `-f` says, so `/etc/varnish/default.vcl`. We will need to go there later.

First we need to tweak our Nginx configuration. This is where ever you set up your config when you set up the site. Usually in `sites-available` if you are running a virtual host.

```bash
sudo nano /etc/nginx/sites-available/example.com
```
Here you need to change where Nginx is listening. At the top of your server block I'm guessing you have something like

```bash
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;
```
Just change those port numbers to match where Varnish is sending your backend responses. Which in this example is `8080` and I shall explain why later.

```bash
    listen 8080 default_server;
    listen [::]:8080 default_server ipv6only=on;
```

## Configuring Varnish

Next we come onto the more complex task. Actually configuring Varnish. Now the below configuration is for a generic site. You should learn and tailor your configuration for your particular site. It will depend on your levels of traffic, amount of static resources such as images, CSS file etc, and how well your site performs under strain maybe during peak hours.

The file that contains all your Varnish configuration is the file mentioned as the `-f` parameter earlier. Lets go there.

```bash
sudo nano /etc/varnish/default.vcl
```
You will have some default stuff in there to start you off.

```bash
#
# This is an example VCL file for Varnish.
#
# It does not do anything by default, delegating control to the
# builtin VCL. The builtin VCL is called when there is no explicit
# return statement.
#
# See the VCL chapters in the Users Guide at https://www.varnish-cache.org/docs/
# and http://varnish-cache.org/trac/wiki/VCLExamples for more examples.

# Marker to tell the VCL compiler that this VCL has been adapted to the
# new 4.0 format.
vcl 4.0;

# Default backend definition. Set this to point to your content server.
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}

sub vcl_recv {
    # Happens before we check if we have this in cache already.
    #
    # Typically you clean up the request here, removing cookies you don't need,
    # rewriting the request, etc.
}

sub vcl_backend_response {
    # Happens after we have read the response headers from the backend.
    #
    # Here you clean the response headers, removing silly Set-Cookie headers
    # and other mistakes your backend does.
}

sub vcl_deliver {
    # Happens when we have all the pieces we need, and are about to send the
    # response to the client.
    #
    # You can do accounting or modifying the final object here.
}
```
Varnish is built in a way so that if you customize one of its subroutines, but you don't exit with a `return` statement, it will continue with its default commands for that subroutine. So for example, if you only want to extend Varnish by telling it to drop a `foo` header on a backend response. All you would have to do is add `unset beresp.http.foo;` to your `vcl_backend_response`, make no `return` statement and it would carry on like normal.

In that case your `default.vcl` file would simply be

```bash
# Marker to tell the VCL compiler that this VCL has been adapted to the
# new 4.0 format.
vcl 4.0;

# Default backend definition. Set this to point to your content server.
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}
sub vcl_backend_response {
    unset beresp.http.foo;
}
```
nice and simple. So now lets go through the VCL file and I will explain each bit. First we have
```bash
vcl 4.0;
```
This is self explanatory given the comment in the example file. Just tell Varnish we are using 4.0.

Next we have

```bash
# Default backend definition. Set this to point to your content server.
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}
```
This tells Varnish where our backend is. In this example we are using Nginx explained above, and we told Nginx to listen on port 8080. So when Varnish doesn`t have a cached version of a request it uses port 8080 to ask Nginx for a fresh copy.

#### Handling the request

Next we have the `sub vcl_recv` subroutine. This is the first thing thats hit and it`s purpose is to decide on if Varnish should process the request or not. Rather than explain line by line there are comments throughout. The principle is to definitely cache anything that is static, and definitely not cache anything thats dynamic (POST requests). Also it might get annoying to cache the blog and have to wait for new posts to become live. We could set up dynamic cache purging but that is beyond the scope of this tutorial.

```bash
sub vcl_recv {
    # Happens before we check if we have this in cache already.
    #
    # Typically you clean up the request here, removing cookies you don't need,
    # rewriting the request, etc.

	# Properly handle different encoding types
	if (req.http.Accept-Encoding) {
	  	if (req.url ~ "\.(jpg|jpeg|png|gif|gz|tgz|bz2|tbz|mp3|ogg|swf|woff)$") {
	    		# No point in compressing these
	    		unset req.http.Accept-Encoding;
	  	} elsif (req.http.Accept-Encoding ~ "gzip") {
	    		set req.http.Accept-Encoding = "gzip";
	  	} elsif (req.http.Accept-Encoding ~ "deflate") {
	    		set req.http.Accept-Encoding = "deflate";
	  	} else {
	    		# unknown algorithm (aka crappy browser)
			unset req.http.Accept-Encoding;
	 	}
	}

	# Cache files with these extensions
	if (req.url ~ "\.(js|css|jpg|jpeg|png|gif|gz|tgz|bz2|tbz|mp3|ogg|swf|woff)$") {
		unset req.http.cookie;
		return (hash);
	}

	# Dont cache anything thats on the blog page or thats a POST request
	if (req.url ~ "^/blog" || req.method == "POST") {
    		return (pass);
	}

	# This is Laravel specific, we have session-monster which sets a no-session header if we dont really need the set session cookie.
	# Check for this and unset the cookies if not required
	# Except if its a POST request
	if (req.http.X-No-Session ~ "yeah" && req.method != "POST") {  
     		unset req.http.cookie;  
	}  

	return (hash);
}
```

#### The Laravel Part
One important part is

```bash
if (req.http.X-No-Session ~ "yeah" && req.method != "POST") {  
        unset req.http.cookie;  
}
```
which is the one part that is Laravel specific. Session monster is installed via composer, and will add a `No-Session` header when a session is there, but is not actually required. Therefore we can disregard the `laravel_session` cookie.

```
"require": {
    "HaiFangHui/session-monster": "*", # At time of writing this was the most up-to-date version
}
```

So if we get through those bits then we can cache everything else so pass through to `return(hash)`.

#### Response from the server

So now we have cover what Varnish does when it gets a request. We need to tell Varnish what to do when it needs to talk to the server. There is 2 parts and the first one is mentioned in the configuration of `vcl_recv`. You may have notices `return(pass)` and that is exactly what it sounds like. Pass that request straight onto Nginx. Then part 2 is when Nginx response and that is handled in `vcl_backend_response`. Which the configuration for is below.

```bash
sub vcl_backend_response {
    # Happens after we have read the response headers from the backend.
    #
    # Here you clean the response headers, removing silly Set-Cookie headers
    # and other mistakes your backend does.

	# This is how long Varnish will cache content. Set at top for visibility.
	set beresp.ttl = 1d;

	if ((bereq.method == "GET" && bereq.url ~ "\.(css|js|xml|gif|jpg|jpeg|swf|png|zip|ico|img|wmf|txt)$") ||
                bereq.url ~ "\.(minify).*\.(css|js).*" ||
                bereq.url ~ "\.(css|js|xml|gif|jpg|jpeg|swf|png|zip|ico|img|wmf|txt)\?ver") {
                unset beresp.http.Set-Cookie;
                set beresp.ttl = 5d;
        }

	# Unset all cache control headers bar Age.
	unset beresp.http.etag;
	unset beresp.http.Cache-Control;
        unset beresp.http.Pragma;

	# Unset headers we never want someone to see on the front end
 	unset beresp.http.Server;
        unset beresp.http.X-Powered-By;

        # Set how long the client should keep the item by default
        set beresp.http.cache-control = "max-age = 300";

        # Set how long the client should keep the item by default
        set beresp.http.cache-control = "max-age = 300";

        # Override browsers to keep styling and dynamics for longer
        if (bereq.url ~ ".minify.*\.(css|js).*") { set beresp.http.cache-control = "max-age = 604800"; }
        if (bereq.url ~ "\.(css|js).*") { set beresp.http.cache-control = "max-age = 604800"; }

        # Override the browsers to cache longer for images than for main content
        if (bereq.url ~ ".(xml|gif|jpg|jpeg|swf|css|js|png|zip|ico|img|wmf|txt)$") {
                set beresp.http.cache-control = "max-age = 604800";
        }

	# We're done here, send the data to the browser
	return (deliver);
}
```

Here there are a few important concepts. The first is the `beresp.ttl` and this is how long Varnish should keep a cached copy for. We set this differently for static resources as dynamic for obvious reasons.

We also format the headers that we use. Unsetting most of them and the setting some cache headers to let the browser know how long to keep the static resources for.

#### Delivering to the user

The last thing we do is modify the `vcl_deliver` subroutine, and that is just to remove the varnish headers. Which is more of a vanity thing really but I like to have clean headers. Plus I suppose there is a small security risk of people knowing you have varnish installed on your site. 

```bash
sub vcl_deliver {
    # Happens when we have all the pieces we need, and are about to send the
    # response to the client.
    #
    # You can do accounting or modifying the final object here.

    # Lets not tell the world we are using Varnish in the same principle we set server_tokens off in Nginx
    unset resp.http.Via;  
    unset resp.http.X-Varnish;  
}
```

## Put it all together

So putting everything together we get our finished default.vcl file. Which you can also find in my [gist](https://gist.github.com/davidthingsaker/6b0997b641fdd370a395) on github, along with some other hopefully [useful code snippets](https://gist.github.com/davidthingsaker). 

```bash
# Marker to tell the VCL compiler that this VCL has been adapted to the
# new 4.0 format.
vcl 4.0;

# Default backend definition. Set this to point to your content server.
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}

sub vcl_recv {
    # Happens before we check if we have this in cache already.
    #
    # Typically you clean up the request here, removing cookies you don't need,
    # rewriting the request, etc.

	# Properly handle different encoding types
	if (req.http.Accept-Encoding) {
	  	if (req.url ~ "\.(jpg|jpeg|png|gif|gz|tgz|bz2|tbz|mp3|ogg|swf|woff)$") {
	    		# No point in compressing these
	    		unset req.http.Accept-Encoding;
	  	} elsif (req.http.Accept-Encoding ~ "gzip") {
	    		set req.http.Accept-Encoding = "gzip";
	  	} elsif (req.http.Accept-Encoding ~ "deflate") {
	    		set req.http.Accept-Encoding = "deflate";
	  	} else {
	    		# unknown algorithm (aka crappy browser)
			unset req.http.Accept-Encoding;
	 	}
	}

	# Cache files with these extensions
	if (req.url ~ "\.(js|css|jpg|jpeg|png|gif|gz|tgz|bz2|tbz|mp3|ogg|swf|woff)$") {
		unset req.http.cookie;
		return (hash);
	}

	# Dont cache anything thats on the blog page or thats a POST request
	if (req.url ~ "^/blog" || req.method == "POST") {
    		return (pass);
	}

	# This is Laravel specific, we have session-monster which sets a no-session header if we dont really need the set session cookie.
	# Check for this and unset the cookies if not required
	# Except if its a POST request
	if (req.http.X-No-Session ~ "yeah" && req.method != "POST") {  
     		unset req.http.cookie;  
	}  

	return (hash);
}

sub vcl_backend_response {
    # Happens after we have read the response headers from the backend.
    #
    # Here you clean the response headers, removing silly Set-Cookie headers
    # and other mistakes your backend does.

	# This is how long Varnish will cache content. Set at top for visibility.
	set beresp.ttl = 1d;

	if ((bereq.method == "GET" && bereq.url ~ "\.(css|js|xml|gif|jpg|jpeg|swf|png|zip|ico|img|wmf|txt)$") ||
                bereq.url ~ "\.(minify).*\.(css|js).*" ||
                bereq.url ~ "\.(css|js|xml|gif|jpg|jpeg|swf|png|zip|ico|img|wmf|txt)\?ver") {
                unset beresp.http.Set-Cookie;
                set beresp.ttl = 5d;
        }

	# Unset all cache control headers bar Age.
	unset beresp.http.etag;
	unset beresp.http.Cache-Control;
        unset beresp.http.Pragma;

	# Unset headers we never want someone to see on the front end
 	unset beresp.http.Server;
        unset beresp.http.X-Powered-By;

        # Set how long the client should keep the item by default
        set beresp.http.cache-control = "max-age = 300";

        # Set how long the client should keep the item by default
        set beresp.http.cache-control = "max-age = 300";

        # Override browsers to keep styling and dynamics for longer
        if (bereq.url ~ ".minify.*\.(css|js).*") { set beresp.http.cache-control = "max-age = 604800"; }
        if (bereq.url ~ "\.(css|js).*") { set beresp.http.cache-control = "max-age = 604800"; }

        # Override the browsers to cache longer for images than for main content
        if (bereq.url ~ ".(xml|gif|jpg|jpeg|swf|css|js|png|zip|ico|img|wmf|txt)$") {
                set beresp.http.cache-control = "max-age = 604800";
        }

	# We're done here, send the data to the browser
	return (deliver);
}


sub vcl_deliver {
    # Happens when we have all the pieces we need, and are about to send the
    # response to the client.
    #
    # You can do accounting or modifying the final object here.

	# Lets not tell the world we are using Varnish in the same principle we set server_tokens off in Nginx
	unset resp.http.Via;  
   	unset resp.http.X-Varnish;  
}
```

