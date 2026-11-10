# 10. Service container and dependency injection

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Middleware and request lifecycle](./09-middleware-and-request-lifecycle.md) | [Notes index](../README.md) | [Next: Build JSON APIs](./11-build-json-apis.md) |

The service container creates and connects application objects. Dependency injection makes those needs visible in a constructor or method instead of hiding object creation inside a class.

## Inject concrete dependencies

Laravel resolves many controller dependencies automatically. Constructor injection makes a dependency reusable across the controller’s methods.

~~~php
<?php

namespace App\Http\Controllers;

use App\Services\ReportBuilder;

class ReportController
{
    public function __construct(
        private ReportBuilder $reports,
    ) {}

    public function show()
    {
        return view('reports.show', [
            'report' => $this->reports->build(),
        ]);
    }
}
~~~

This keeps the controller focused on the request and response while the service handles report-building logic.

## Depend on an interface

An interface lets application code describe what it needs without choosing a particular implementation.

~~~php
<?php

namespace App\Contracts;

interface PaymentGateway
{
    public function charge(int $amountInCents): string;
}
~~~

A production implementation can call a provider, while a fake implementation can be used in a test. Bind the interface to its implementation in a service provider.

~~~php
use App\Contracts\PaymentGateway;
use App\Services\StripePaymentGateway;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(PaymentGateway::class, StripePaymentGateway::class);
    }
}
~~~

A transient <code>bind</code> creates a fresh instance when resolved. Use <code>singleton</code> only when one shared instance is intended and safe for the application’s lifecycle.

## Register application services

Laravel registers application service providers from <code>bootstrap/providers.php</code>. The default <code>AppServiceProvider</code> is a natural location for a small number of app-wide bindings. Larger apps can create focused providers.

Place dependency registration in <code>register</code>. Use <code>boot</code> for setup that needs the application’s services to be registered first, such as defining a gate or observer.

~~~php
public function boot(): void
{
    // Register framework hooks or application boot behavior here.
}
~~~

Avoid putting database-dependent work in the provider constructor. Providers are part of bootstrapping and should remain predictable.

## Use method injection

Laravel can resolve dependencies in controller actions and many framework-managed callbacks.

~~~php
use App\Contracts\PaymentGateway;
use Illuminate\Http\Request;

public function checkout(Request $request, PaymentGateway $gateway)
{
    $validated = $request->validate([
        'amount' => ['required', 'integer', 'min:1'],
    ]);

    $reference = $gateway->charge($validated['amount']);

    return response()->json(['reference' => $reference]);
}
~~~

Validate request values before passing them into a service. Authorization and business checks should also happen before a side effect such as charging a payment method.

## Understand facades

A facade provides a concise static-looking interface to a service that Laravel resolves from the container. For example, <code>Cache::get()</code> is a convenient way to use the cache service. Facades are not ordinary static utility methods; they resolve managed services behind the scenes.

Use constructor injection when explicit dependencies help explain a class or when a dependency needs a substitute in a focused test. Use facades when they make a small framework operation clearer. Consistency and clear boundaries matter more than avoiding one style entirely.

## Key points

- The service container constructs objects and supplies their dependencies.
- Constructor injection makes long-lived class dependencies explicit.
- Bind an interface when application code should not depend on one implementation.
- Use service providers to register bindings and boot behavior.
- Put registrations in register and post-registration setup in boot.
- Facades are container-backed shortcuts to framework services.

## Practice questions

1. What problem does the service container solve?
2. What does dependency injection make visible?
3. Why might a class depend on an interface?
4. What is the difference between bind and singleton?
5. Where does a Laravel 13 app list its service providers?
6. What belongs in a provider’s register method?
7. What is a facade backed by?
8. Why should authorization occur before a payment side effect?

## Main references

- [Laravel 13 service container](https://laravel.com/docs/13.x/container)
- [Laravel 13 service providers](https://laravel.com/docs/13.x/providers)
- [Laravel 13 facades](https://laravel.com/docs/13.x/facades)
