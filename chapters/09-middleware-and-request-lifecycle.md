# 9. Middleware and request lifecycle

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Authentication and authorization](./08-authentication-and-authorization.md) | [Notes index](../README.md) | [Next: Service container and dependency injection](./10-service-container-and-dependency-injection.md) |

A Laravel request passes through several layers before a response reaches the browser. Middleware is a convenient place for checks or transformations that apply across a route or group of routes.

## Trace a typical request

A web request enters the public entry point, is bootstrapped by the application, and is matched to a route. The route’s middleware and controller run, then Laravel sends the response back through middleware. Exact internals vary by application setup, but the mental model is useful when locating behavior.

Route middleware can reject a request early, such as redirecting a guest to sign in. Middleware can also inspect the response after the next layer returns.

## Create middleware

Generate a middleware class with Artisan.

~~~sh
php artisan make:middleware EnsureAccountIsActive
~~~

A middleware receives the request and a continuation. It may return a response immediately or call the continuation and return the resulting response.

~~~php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureAccountIsActive
{
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->user() && ! $request->user()->is_active) {
            abort(403);
        }

        return $next($request);
    }
}
~~~

Keep middleware focused. Business rules that belong to a model policy or application service are often easier to test and reuse there.

## Register and apply middleware

Laravel 13 configures middleware in <code>bootstrap/app.php</code>. Append application middleware to the web group when it should apply to browser routes, or alias it when routes need to opt in.

~~~php
use App\Http\Middleware\EnsureAccountIsActive;
use Illuminate\Foundation\Configuration\Middleware;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'active' => EnsureAccountIsActive::class,
    ]);
})
~~~

Then use the alias on selected routes.

~~~php
Route::middleware(['auth', 'active'])->group(function () {
    Route::get('/billing', BillingController::class)->name('billing');
});
~~~

Use framework-provided middleware for standard behavior such as authentication, throttling, and CSRF protection. Avoid replacing security middleware casually.

## Work with request and response data

Middleware runs before a controller and can read route, user, and request information. It should not trust client-provided headers for identity unless a trusted proxy or authentication layer validates them.

A middleware can add a response header after the controller runs.

~~~php
public function handle(Request $request, Closure $next): Response
{
    $response = $next($request);

    $response->headers->set('X-Content-Type-Options', 'nosniff');

    return $response;
}
~~~

Set security headers at the appropriate application or infrastructure layer, and verify that reverse proxies and caches preserve the expected behavior.

## Key points

- Middleware wraps route handling and can stop or continue a request.
- Laravel 13 application middleware configuration lives in bootstrap/app.php.
- Use middleware for cross-cutting request checks, not every business rule.
- Apply authentication before middleware that depends on the signed-in user.
- Never trust identity or permissions supplied only by a client header.
- Framework middleware already handles many common concerns.

## Practice questions

1. Where does middleware sit in a request?
2. What are the two basic choices middleware has when handling a request?
3. How do you create middleware with Artisan?
4. Where is application middleware configured in Laravel 13?
5. What is an alias useful for?
6. Why should middleware classes stay focused?
7. Why is a client-provided identity header unsafe by itself?
8. When can middleware inspect a response?

## Main references

- [Laravel 13 request lifecycle](https://laravel.com/docs/13.x/lifecycle)
- [Laravel 13 middleware](https://laravel.com/docs/13.x/middleware)
- [Laravel 13 routing](https://laravel.com/docs/13.x/routing)
- [Laravel 13 CSRF protection](https://laravel.com/docs/13.x/csrf)
