[Lumen](http://lumen.laravel.com/) is the new micro PHP framework offering from [Laravel](http://laravel.com/). Those of you who know me know that I am a big Laravel advocate and therefore I may be slightly biased on the Lumen front. However, I will try and stay impartial. 

A few months back I started building a project in Laravel 4.2 as Laravel 5 hadn't been released as a stable version yet. The project was fairly simple, in terms of complexity, but fairly large in the amount of code that had to be written. Some front end stuff using [bootstrap](http://getbootstrap.com/), and the back end was a lot of [Guzzle](https://github.com/guzzle/guzzle) requests to existing APIs to a sister project (also in L4.2), then a lot of JSON manipulation. So when Lumen came along this seemed like a perfect project to port over to the new framework and learn something new. 

As my projects requirements had changed a lot since inception, a lot of the code needed refactoring and rewriting anyway so I decided to go for a fresh install and rewrite from scratch. Copy and pasting chunks of code over that I still liked to save time. 

To start with my impressions of Lumen were great, reading on the website that if needed I could upgrade to full L5 but simply dropping it into a L5 install is a great safety net. I noticed the speed of the requests increasing as the framework had to do less work chugging through less of the heavy [Symfony](https://symfony.com/) modules so all seemed well. 

I did however have a few niggles. One being the name spacing, which coming from a L4.2 facades and auto loading world, I had got lazy and not been declaring. Which meant I sometimes spent a while looking at error pages thinking 'it definitely does exist' when all I had done was forget the use declarations at the top of my class. 

My other little niggle is the documentation, which Laravel is usually spectacular at in my opinion. Especially when you include the [Laracast](https://laracasts.com/) team. However, I started on L4 once it was already mature, and I think all thats needed with Lumen is a little time, love and attention and the holes in the documentation will soon be filled. 

If you too are coming from the L4 background Lumen is a great place to start learning L5 as you have less to learn, but the syntax is the same. Its the same basic level just slightly different formatting, for example calling config variables is no longer,

```
Config::get('myappconfig.somevariable');
```

Instead its, 

```
config('myappconfig.somevariable');
```

You also need to watch out in Lumen for loading in of files and classes. For example, those config files wont auto load like they do in Laravel. You need to pop over to `/bootsrap/app.php` and load them in. Same with your `.env` file (which is `.env` now by the way not `.env.environment.php` as in L4).

Eloquent and facades can still be used in Lumen, but arent activated but default. Again `app.php` is you friend and you will find some commented out lines in there to load them in. 

Probably my biggest issue with Lumen, and this is with L5 as well, is the removal of the HTML and FORM macros, and Macros in general. I loved macros and found them really useful. Now, they do give you the option to load these in and tell you [how to do so in the docs](http://laravel.com/docs/5.1/upgrade#upgrade-5.0). However, with Lumen I found that if I wanted to even create a small HTML macro, without loading these in I needed to create my own HTML class and do it myself. Which is no real bother, just extra time. 

Other than that though its a lovely framework to work with, it has all the basic features you could want and its relatively bug free even in its early state. I had one bug found with namespacing in routes but they cleared that up quite quickly. Overall I would definitely recommend it and would even go as far to say that from now on my projects will most likely start off as Lumen projects, with that L5 safety net there is required.   