# 2. Routes and controllers

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Start with Laravel](./01-start-with-laravel.md) | [Notes index](../README.md) | [Next: Blade views and layouts](./03-blade-views-and-layouts.md) |

A route connects an HTTP method and URL to code that handles the request. Laravel loads web routes from routes/web.php. A simple route can use a closure. As behavior grows, a controller gives related actions a clear home.

## Match a method and URL

Use the method that matches the action. GET reads a page or resource. POST submits new data. PUT replaces a resource, PATCH changes part of it, and DELETE removes it.

~~~php
use Illuminate\Support\Facades\Route;

Route::get('/status', function () {
    return 'The application is ready.';
});

Route::post('/notes', function () {
    return 'A note was submitted.';
});

Route::delete('/notes/{note}', function (string $note) {
    return 'Delete note ' . $note;
});
~~~

Route parameters use braces. Laravel passes a required parameter to the closure or controller action. Add a type declaration so the expected value is clear.

~~~php
Route::get('/members/{member}/notes/{note}', function (string $member, string $note) {
    return 'Member ' . $member . ', note ' . $note;
});
~~~

If a parameter is optional, mark it with a question mark and provide a default value. Use a constraint when only a known pattern should match.

~~~php
Route::get('/archive/{year?}', function (?string $year = null) {
    return $year ?? 'All years';
});

Route::get('/receipts/{receipt}', function (string $receipt) {
    return 'Receipt ' . $receipt;
})->whereNumber('receipt');
~~~

## Give routes names

A named route gives a stable name to a URL. Generate links with route() instead of repeating a path in templates or controllers. If the URL changes, update the route definition without searching every use of the old string.

~~~php
Route::get('/notes/{note}', function (string $note) {
    return 'Note ' . $note;
})->name('notes.show');

$url = route('notes.show', ['note' => 24]);
~~~

Route names should be unique and describe the action, such as notes.index or notes.show.

## Group related routes

A route group can share a URL prefix, name prefix, or middleware. This keeps related routes together.

~~~php
Route::prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/reports', function () {
            return 'Reports';
        })->name('reports');
    });
~~~

The route in this group has the URL /admin/reports and the name admin.reports. Authentication middleware can be added when an area should only be available to signed-in users.

## Move actions into a controller

Generate a controller with Artisan:

~~~sh
php artisan make:controller NoteController
~~~

The generated file is in app/Http/Controllers. Add an action method and point a route to the class and method:

~~~php
<?php

namespace App\Http\Controllers;

class NoteController extends Controller
{
    public function show(string $note)
    {
        return 'Note ' . $note;
    }
}
~~~

~~~php
use App\Http\Controllers\NoteController;
use Illuminate\Support\Facades\Route;

Route::get('/notes/{note}', [NoteController::class, 'show'])
    ->name('notes.show');
~~~

The route describes how a request is matched. The controller action handles the work and returns a response. Keep route closures short so the routing file remains easy to scan.

## Return a view or JSON

A controller can return a view with data or a JSON response. The Blade chapter explains views.

~~~php
public function index()
{
    return view('notes.index', [
        'title' => 'My notes',
    ]);
}

public function status()
{
    return response()->json([
        'status' => 'ready',
    ]);
}
~~~

For an API route file, a fresh Laravel application can run php artisan install:api. This creates routes/api.php and installs Laravel Sanctum. API routes are stateless and receive the /api prefix by default. Add authentication only when the endpoint needs it.

## Inspect the route table

Use Artisan to confirm that Laravel registered the routes you expect.

~~~sh
php artisan route:list
php artisan route:list --path=notes
php artisan route:list -v
~~~

The verbose option shows middleware. Route caching can help production boot performance. Run php artisan route:cache during deployment after confirming your route definitions do not rely on unsupported dynamic behavior.

## Key points

- A route matches an HTTP method and URL.
- Braced segments capture route parameters.
- Named routes let code generate URLs without hard-coding paths.
- Route groups share prefixes, names, or middleware.
- Controllers keep request actions organized as the application grows.
- In Laravel 13, API routes can be enabled with php artisan install:api.

## Practice questions

1. What two parts does a route match?
2. Which HTTP method is normally used to read a resource?
3. How is a required route parameter written?
4. Why give a route a name?
5. What can a route group share?
6. Where does Artisan generate a basic controller?
7. Which command lists registered routes?
8. Which command adds the API route file to a fresh application?

## Main references

- [Laravel 13 routing](https://laravel.com/docs/13.x/routing)
- [Laravel 13 controllers](https://laravel.com/docs/13.x/controllers)
- [Laravel 13 responses](https://laravel.com/docs/13.x/responses)
- [Laravel 13 Artisan console](https://laravel.com/docs/13.x/artisan)
