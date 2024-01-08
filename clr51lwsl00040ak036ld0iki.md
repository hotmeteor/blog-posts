---
title: "Run a Wordpress blog alongside your Laravel app"
seoDescription: "Learn how to set up a Wordpress blog alongside an existing Laravel app to have a powerful content platform without affecting functionality."
datePublished: Mon Jan 08 2024 14:53:21 GMT+0000 (Coordinated Universal Time)
cuid: clr51lwsl00040ak036ld0iki
slug: run-a-wordpress-blog-alongside-your-laravel-app
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1704724669055/48673c9e-fffa-4df2-800e-d09622a77b23.jpeg
tags: laravel, wordpress, seo

---

## **Many times over the years I've wanted to add a blog to an existing Laravel app.**

Lately, this has especially been the case for "experiments" I've been building, whether they are micro-SaaS projects or pSEO sites. These sites are built to see if they can drive traffic (and potentially revenue) and I don't want to invest a whole lot into them because they could get nuked just as fast.

95% of these experiments are Laravel apps because it's the tool I know best, and I know I can squeeze a ton of **functionality and performance** for my site using the framework.

Also top of mind is **SEO**. SEO is vital for any project with hopes of organic growth. Blogs are an excellent way to build an SEO presence. So with that in mind I know that that blog needs to sit **within** the root domain, not a subdomain, or I'm just shooting myself in the foot (This is [a big topic you can explore](https://www.oncrawl.com/technical-seo/subdomains-impact-seo-performance/)).

This article outlines how I set up Wordpress blogs alongside existing Laravel apps so that I can have a powerful content platform without affecting functionality.

### Considerations

When considering my options a few come to mind:

* **Write my own** - This is fine, but it's also limited. There's so much domain knowledge that has gone into blogging systems and CMSs that I'd want to leverage, but I don't want to have to build.
    
* **Use Statamic** - [Statamic](https://statamic.com) is very awesome, I've used it on a handful of sites. Since it's a package that sits inside of a Laravel installation it's actually well suited for this case. Unfortunately, I'm not always in the position to fork up the money for a license for an experiment I'm running.
    
* **Use Wordpress** - [Wordpress](https://wordpress.org) is free, it's easy to install, and it comes with all the bells and whistles I may or may not need. But it's got one major issue, which is that it is intended to be its own standalone app. Laravel's routing system makes it difficult to "ignore" a folder within the app and you'd need to fiddle with Apache or nginx settings.
    

### Wordpress + Laravel = 🥰

Faced with these options, it's clear none are perfect for my "cheap experiments" scenario.

But for what I'm going for Wordpress comes closest, so I decided that it was time to figure out how to make Wordpress work with an existing Laravel app without adding additional complex overhead.

I'm not 100% sure what I outline in this article is the *easiest or quickest* way that a Laravel and Wordpress marriage can be done. But I do know this set up absolutely works, is pretty elegant, and has allowed me to seamlessly incorporate Wordpress into my existing apps.

And you'll be happy to know that this is all just as possible using PostgreSQL as it is MySQL (I'll show you how)!

## Overview

Before we start I want to lay out the objective so you can decide now if you want to keep reading or not.

What we're going to do is to **use Wordpress as a headless CMS for our blog.** By "headless" I mean Wordpress will only serve as the CMS; we will separately pull that data into our app by other means.

Wordpress will be set up as a **distinct app** from your Laravel app but it will **share the database**. The Wordpress data and content can be pulled into our app just like you would with any other resource.

It's intended that **the Wordpress install will live on a subdomain** (which is completely fine from an SEO perspective because it's just acting as a CMS) and your blog content will be presented within blog pages you create within your app.

For the sake of this example:

* The Laravel app is `mysite.test`
    
* The CMS app is `cms.mysite.test`
    
* The blog will live at the `/blog` path of your Laravel app, `mysite.test/blog`
    

### Implications

The implications of this setup is that **your Laravel app and your Wordpress install are two distinct projects** and will require independent deployments.

Fortunately, once Wordpress is deployed it can be managed entirely from production, including framework and plugin updates, so you shouldn't need to be running two deployment processes after it's first installed.

Hopefully you're still with me. Let's press on.

## Requirements

There's a few things you'll need before getting started:

* An existing Laravel app that's up and running, with a database
    
* [wp-cli](https://make.wordpress.org/cli/handbook/guides/installing/)
    
    * This is the official CLI for Wordpress. I refuse to download and unzip packages, so this is the nicer way to do it.
        
    * You can install wp-cli using curl or composer; I myself did it using brew (`brew install wp-cli`)
        

## Getting Started

Install Wordpress as a new site [using the CLI](https://make.wordpress.org/cli/handbook/how-to/how-to-install/). Note the `path` argument is installing it into a **new folder**; just to remind you, this should be a distinct project.

```bash
wp core download --path=mysite-blog --skip-content
```

Ok, that was easy.

### PostgreSQL

**If you're not using PostgreSQL** feel free to skip ahead to the next section. Wordpress is built for MySQL out of the box.

Otherwise, follow me...

#### Wordpress on PostgreSQL

I've been a PostgreSQL user for years. For most of that time I thought I had to spin up one-off MySQL DBs just to run Wordpress. But thanks to the [PostgreSQL for Wordpress](https://github.com/PostgreSQL-For-Wordpress/postgresql-for-wordpress) project, that's not the case!

1. Go to the `wp-content` folder
    
2. Install [pg4wp](https://github.com/PostgreSQL-For-Wordpress/postgresql-for-wordpress)
    

```bash
git clone https://github.com/kevinoid/postgresql-for-wordpress.git
```

1. Move the `pg4wp` folder to the `wp-content` directory:
    

```bash
mv postgresql-for-wordpress/pg4wp pg4wp
```

1. Move the `db.php` file to the `wp-content` directory
    

```bash
cp pg4wp/db.php db.php
```

That's it! Your Wordpress site will work in blissful harmony with your PostgreSQL database.

## Configuring Wordpress

Return to root of your Wordpress project and finish preparing it for installation.

### Environment Variables

As someone who has reached developer enlightenment I prefer **not** to hard-code environment variables into configuration files. So to handle env variables better, we're going to do some quick setup.

[Eric Barnes's wrote a great post about this back in 2017](https://m.dotdev.co/secure-your-wordpress-config-with-dotenv-d939fcb06e24), so we're going to follow his suggestions. There's been a few updates to the related packages since, but here's the steps:

1. In your Wordpress project run `composer init` to create a `composer.json` file. If you don't want to do that you can just use this snippet (replace the values in `<>`s):
    

```json
{
    "name": "<repo name>",
    "authors": [
        {
            "name": "<your name>",
            "email": "<your email>"
        }
    ],
    "require": {
        "vlucas/phpdotenv": "^5.6"
    }
}
```

1. As you can see we're using the awesome [vlucas/phpdotenv](https://github.com/vlucas/phpdotenv) package. Pull it in using `composer require vlucas/phpdotenv` or `composer install` if you manually created your `composer.json` file.
    
2. Create a `.env` file inside the project folder
    
3. Add the `.env` file to a `.gitignore` file. We don't want that landing in our commit history.
    
4. Set **the same database variables from your Laravel app** into the `.env`:
    

```bash
DB_HOST=127.0.0.1
DB_DATABASE=
DB_USERNAME=
DB_PASSWORD=
```

Now your Wordpress site is leveraging environment variables, which will work both locally and in production.

### Finishing Configuration

1. Create the config file:
    

```bash
cp -rp wp-config-sample.php wp-config.php
```

1. Run `wp config shuffle-salts` to set all your SALT values.
    

<div data-node-type="callout">
<div data-node-type="callout-emoji">📣</div>
<div data-node-type="callout-text">Heads up: Running <code>wp config shuffle-salts</code> is a quick way of setting all of the SALT values in your <code>wp-config.php</code> file, but it completely jacks up the line-spacing. So if you're OCD about your files you may want to do it manually.</div>
</div>

1. Update the new `wp-config.php` file with your DB settings.
    
    1. If you're using environment variables ensure you're including the autoload script at the top, and properly referring to the values using `$_ENV`.
        
    2. If you're not using environment variables just drop the necessary values right in there like a complete savage
        

```php
<?php

require_once(__DIR__ . '/vendor/autoload.php');
$dotenv = Dotenv\Dotenv::createImmutable(__DIR__);
$dotenv->load();

/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the web site, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * Database settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://wordpress.org/documentation/article/editing-wp-config-php/
 *
 * @package WordPress
 */

/** The name of the database for WordPress */
define( 'DB_NAME', $_ENV['DB_DATABASE'] );

/** Database username */
define( 'DB_USER', $_ENV['DB_USERNAME'] );

/** Database password */
define( 'DB_PASSWORD', $_ENV['DB_PASSWORD'] );

/** Database hostname */
define( 'DB_HOST', $_ENV['DB_HOST'] );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```

## Installation

We've made great progress and now it's time to install! First, let's review what we've done:

* ✅ We've created a standalone Wordpress install
    
* ✅ We've configured it to connect to our main Laravel app database
    
* ✅ We're leveraging environment variables to keep secrets where they belong
    

It may seem like it's a lot of work, but after you do it once it only takes a minute.

#### **Now it's time to install Wordpress.**

If you're working in Valet now is a good time to link this domain. I like to set up the Wordpress install on a subdomain of my Laravel site to emulate how it'll work in production, but you don't *have* to do that.

```bash
valet link cms.mysite
```

It should now be available at [http://cms.mysite.test](http://cms.mysite.test).

Visit the new URL to complete setup – it should redirect you to the installation path.

Once the installation flow is complete you should have your site setup and running now, whether you use MySQL or PostgreSQL. Congrats!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704719022441/fa2231fe-28ee-46b0-bbbb-3dbb790fbcb6.png align="center")

## Viewing your Blog

Next is the part we've been waiting for: **pulling in blog content into our Laravel site.**

### Staying Eloquent

One of the great parts of Laravel is having Eloquent models to easily interact with database records. Fortunately for us, the [Corcel](https://github.com/corcel/corcel) library has done the heavy lifting to bring Eloquent structure to Wordpress resources.

> Corcel is a collection of PHP classes built on top of Eloquent ORM (from Laravel framework), that provides a fluent interface to connect and get data directly from a WordPress database.

1. Return to your **Laravel project folder** (not the WP app) and install Corcel:
    

```bash
composer require jgrossi/corcel
```

1. Publish the `corcel.php` config file:
    

```bash
php artisan vendor:publish --provider="Corcel\Laravel\CorcelServiceProvider
```

1. Open your `database.php` file and make a copy of your primary database connection configuration. We want Corcel to use our primary database **but** we need to tell it that the Wordpress resources are found in tables prefixed with `wp_`.
    
    1. Set the key to `wordpress`
        
    2. Change the `prefix` value to `wp_`
        
2. Edit the `corcel.php` config file and change the `connection` value to use your new configuration:
    

```bash
'connection' => 'wordpress'
```

### Creating blog pages

Now you can create a controller and view that will serve up your blog index and posts, or however your app is set up to present views.

Out of the box Corcel comes with `Post` models, with all necessary relationships, but you can extend it to add your own functionality. Check out [https://github.com/corcel/corcel](https://github.com/corcel/corcel#-posts) for a deep dive on all the things you can do.

Here's a quick example of a `BlogController` to serve up your blog and blog posts:

```php
<?php

namespace App\Http\Controllers;

use Corcel\Model\Post;

class BlogController extends Controller
{
    public function index()
    {
        return view('blog.index', [
            'posts' => Post::query()
                ->type('post')
                ->published()
                ->newest()
                ->paginate(10),
        ]);
    }

    public function show(string $slug)
    {
        return view('blog.show', [
            'post' => Post::slug($slug)->first(),
        ]);
    }
}
```

And of course you'll need some routes:

```php
<?php

// routes/web.php

use Illuminate\Support\Facades\Route;

Route::get('/blog', [\App\Http\Controllers\BlogController::class, 'index'])->name('blog.index');
Route::get('/blog/{slug}', [\App\Http\Controllers\BlogController::class, 'show'])->name('blog.show');
```

## Wrap Up

There you have it: **a Wordpress blog deployed against your current infrastructure and ready for you to tap into within your primary app.**

While I think there's definitely another path to this with having the Wordpress installation live **inside** your app in a `/blog` folder it means you'll need to configure server configs to get that path to resolve Wordpress's `index.php` file and related routing.

I am interested in this solution as an alternative though, so if you know how to do it please let me know!

And of course, questions and comments are most welcome, as well as suggestions for improving this post to include your particular scenarios.

### Resources:

Here's some links to sites I referenced while pulling this article together:

* [https://m.dotdev.co/secure-your-wordpress-config-with-dotenv-d939fcb06e24](https://m.dotdev.co/secure-your-wordpress-config-with-dotenv-d939fcb06e24)
    
* [https://inovector.com/blog/wordpress-as-a-headless-cms-for-your-laravel-website](https://inovector.com/blog/wordpress-as-a-headless-cms-for-your-laravel-website)
    
* [https://www.enterprisedb.com/postgres-tutorials/how-deploy-wordpress-highly-available-postgresql](https://www.enterprisedb.com/postgres-tutorials/how-deploy-wordpress-highly-available-postgresql)
    
* [https://www.codeable.io/blog/laravel-wordpress/](https://www.codeable.io/blog/laravel-wordpress/) [https://www.cloudways.com/blog/laravel-wordpress/](https://www.cloudways.com/blog/laravel-wordpress/)