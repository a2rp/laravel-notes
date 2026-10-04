# 1. Start with Laravel

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Notes index](../README.md) | [Notes index](../README.md) | [Next: Routes and controllers](./02-routes-and-controllers.md) |

Laravel is a PHP framework for building web applications. It gives a project a familiar structure and provides tools for routing, database access, validation, views, queues, and testing. PHP is still the language you write. Laravel supplies conventions and components around it.

This chapter uses Laravel 13. Its current server requirement is PHP 8.3 or newer, plus Composer. A database is needed when you start saving application data. The framework can use SQLite for a simple local setup.

## Create a project

Install PHP, Composer, and the Laravel installer using the official setup instructions. Then create a project and start the local server:

~~~sh
laravel new notes-app
cd notes-app
php artisan serve
~~~

The installer may ask which database and starter options to configure. Choose the defaults while learning, then adjust them when a later chapter introduces each choice. Open the address shown by Artisan in your browser, usually http://127.0.0.1:8000.

Confirm the application and route setup from another terminal:

~~~sh
php artisan about
php artisan route:list
~~~

Artisan is Laravel's command-line interface. It helps inspect an application, generate common files, run migrations, and execute tests. Use php artisan list to see the commands available in the project.

## Find the important folders

A fresh Laravel project has many files. Begin with the folders you will use most:

- app contains the application's PHP classes, including controllers and models.
- bootstrap contains the application bootstrap file.
- config contains application configuration files.
- database contains migrations, factories, seeders, and optionally a local SQLite file.
- public contains index.php, the entry point for web requests, and public assets.
- resources contains Blade views and uncompiled frontend assets.
- routes contains route definitions such as web.php and console.php.
- storage contains generated files, logs, sessions, and caches.
- tests contains automated tests.
- vendor contains Composer-installed dependencies. Do not edit package files there.

The framework follows conventions, but you can organize application code further when a project needs it. Keep secrets in environment configuration and do not commit a real .env file.

## Trace a web request

For a normal browser request, the web server sends the request to public/index.php. That file loads Composer's autoloader and retrieves the Laravel application created by bootstrap/app.php. Laravel configures the application, loads service providers, and passes the request through middleware.

The router matches the URL and HTTP method to a route. The route can run a small closure or call a controller. The result travels back as a response, passing through the middleware stack on the way out.

~~~text
Browser request
  -> public/index.php
  -> bootstrap/app.php
  -> service providers and middleware
  -> router
  -> route or controller
  -> response to the browser
~~~

This path is a useful map when a request does not reach the code you expected. Later chapters explain each part.

## Write a first route

Open routes/web.php and add a GET route. A route connects an HTTP method and URL to behavior.

~~~php
<?php

use Illuminate\Support\Facades\Route;

Route::get('/welcome-note', function () {
    return 'Welcome to my Laravel study notes.';
});
~~~

Run php artisan serve and visit /welcome-note on the local address. The browser should show the returned text. The route can return a string, a response object, or a view. The next chapters explain how to move route behavior into a controller.

## Make a small JSON response

Laravel can also return JSON from a route. This is useful when the page is consumed by JavaScript or another client.

~~~php
Route::get('/api/status', function () {
    return response()->json([
        'status' => 'ready',
        'framework' => 'Laravel',
    ]);
});
~~~

Visit /api/status and inspect the response. Laravel converts the array into JSON and sends an appropriate content type.

## Understand the project root

Run Artisan commands from the project root, the directory containing artisan and composer.json. If a command says it cannot find the application, first check the current directory.

~~~sh
pwd
php artisan route:list
~~~

On Windows PowerShell, Get-Location displays the current directory. Running commands from the wrong folder is a common first setup issue.

## Key points

- Laravel is a PHP framework that organizes common application work.
- Laravel 13 requires PHP 8.3 or newer and Composer.
- Artisan is the command-line tool for inspecting and operating a Laravel project.
- public/index.php is the web request entry point.
- Routes match HTTP methods and URLs to application behavior.
- Keep project secrets out of source control.

## Practice questions

1. Which programming language do you write in a Laravel application?
2. What is Composer used for in a PHP project?
3. Which command starts the Laravel command-line interface?
4. Which file is the entry point for browser requests?
5. Where are web route definitions usually stored?
6. What does the router do with an incoming request?
7. Which command lists registered routes?
8. Why should a real environment file not be committed?

## Main references

- [Laravel 13 installation](https://laravel.com/docs/13.x/installation)
- [Laravel 13 directory structure](https://laravel.com/docs/13.x/structure)
- [Laravel 13 request lifecycle](https://laravel.com/docs/13.x/lifecycle)
- [Laravel 13 Artisan console](https://laravel.com/docs/13.x/artisan)
