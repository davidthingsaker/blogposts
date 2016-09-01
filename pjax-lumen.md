If you follow Jeffrey Way on [Laracasts](https://laracasts.com/) you may have seen recently he added a video on how to add [Pjax](https://github.com/defunkt/jquery-pjax) to your [Laravel](http://laravel.com/) application. Well, if you're using [Lumen](http://lumen.laravel.com/) you may have noticed that the middleware doesn't quite work the same. 

If you havent already, firstly go check out the [videos on Laracasts](https://laracasts.com/lessons/faster-page-loads-with-pjax). There are two, one for setting up Pjax, the other for rewriting the Middleware so you get to know exactly whats going on.

You can also take a look at [Jeffrey's Middleware](https://gist.github.com/JeffreyWay/8526696b6f29201c4e33) on Github's Gists as I will be referencing it below. 

When go through the tutorial and run this, you will find that Lumen gives you an error:

```
RuntimeException in Crawler.php

line 693: Unable to filter with a CSS selector as the Symfony CssSelector is not installed (you can use filterXPath instead).
```

Thats because Lumen is a cut down version of Laravel to make it super quick and obviously one of the bits Taylor decided to cut out was the symfony `CssSelector` class. No big deal, we could add it. Then we would just be adding bulk to our lean Lumen application. Instead we can adjust the Middleware. 

First take a look at exactly what the error says, `you can use filterXPath instead` - really, its giving you the answer. Lets use `filterXPath` instead. 

Jeffrey uses the `filter` function in two places, and thats whats causing your error. The first is in `makeTitle()` on line 58. This one is easy to change we just need to swap out `filter` for `filterXPath` and modify the selector from `'head > title'` to `'descendant-or-self::head/title'`. Job done. 

That gives us a new `makeTitle()` function that looks like this,

```php
protected function makeTitle($crawler)
{
    $pageTitle = $crawler->filterXPath('descendant-or-self::head/title')->html();

    return "<title>{$pageTitle}</title>";
}
```
The next occurance is in `fetchContents` just below. We have 2 issues here rolled into one. We need to do the same as before and switch out `filter` for `filterXPath` but we need to make sure we are still referencing the right container. In Jeffrey's Middleware `$container = '#pjax-container'` but we cant use that in `filterXPath`. 

Without going into too much detail, we can chop off the `#` bit using `ltrim` and then concatenate the string using XPaths `@id` selector like so,

```php
protected function fetchContents($crawler, $container)
{
    $container = ltrim($container, '#');
    $content = $crawler->filterXPath('//div[@id = "'. $container .'"]');

    if (! $content->count()) {
        abort(422);
    }

    return $content->html();
}
```


