# 14. Configuration, sessions, cache, and logs

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Queues, events, and scheduled jobs](./13-queues-events-and-scheduled-jobs.md) | [Notes index](../README.md) | [Next: Testing Laravel applications](./15-testing-laravel-applications.md) |

Configuration controls how an application connects to services and behaves in each environment. Sessions keep short-lived state for a visitor, cache stores temporary values for reuse, and logs help operators understand what happened.

## Keep environment values out of source code

Put environment-specific secrets and settings in the local <code>.env</code> file, and commit only safe defaults in <code>.env.example</code>. Never commit real credentials. Read environment values from configuration files, then use Laravel’s config helpers in application code.

~~~php
// config/services.php
return [
    'billing' => [
        'endpoint' => env('BILLING_ENDPOINT', 'https://example.test'),
        'token' => env('BILLING_TOKEN'),
    ],
];
~~~

~~~php
$endpoint = config('services.billing.endpoint');
~~~

After configuration is cached for production, do not call <code>env()</code> from application code. Laravel loads the cached configuration rather than reading the environment file at runtime.

## Store session state deliberately

Sessions are useful for web state such as a temporary status message or a short checkout flow. Do not treat sessions as permanent business records or use them as a place to keep credentials.

~~~php
$request->session()->put('checkout.step', 'address');

$step = $request->session()->get('checkout.step', 'address');
~~~

Choose the session driver and cookie settings for the deployment. Use HTTPS in production and secure cookie settings. Regenerate the session ID after authentication changes.

## Cache computed values

Cache is useful when a value is expensive to compute and can safely be reused for a period. Give keys meaningful names and define a reasonable expiration.

~~~php
use Illuminate\Support\Facades\Cache;

$popularNotes = Cache::remember('notes:popular', now()->addMinutes(10), function () {
    return Note::query()
        ->where('published', true)
        ->orderByDesc('views')
        ->limit(10)
        ->get();
});
~~~

Invalidate or refresh cached data when its underlying source changes. A cached value can be stale, so do not treat cache as the authoritative store for important records.

## Write useful logs

Log enough context to diagnose an issue, but do not write passwords, tokens, session identifiers, payment details, or unnecessary personal data.

~~~php
use Illuminate\Support\Facades\Log;

Log::warning('Report generation failed', [
    'report_id' => $report->id,
    'user_id' => $request->user()?->id,
]);
~~~

Choose the log level that matches the event. Route logs to a destination that production operators monitor, and define retention so logs do not grow without bound.

## Prepare configuration for production

Review environment values before deployment. Laravel provides commands such as <code>php artisan config:cache</code> to build a production configuration cache. Clear or rebuild caches as part of a deployment when configuration changes. Never expose the environment file through the web server.

## Key points

- Keep secrets in environment-specific configuration and out of version control.
- Access environment values through Laravel config after configuration is cached.
- Use sessions for temporary visitor state, not permanent business data.
- Cache can become stale; the database or service remains the source of truth.
- Logs should include useful context without sensitive values.
- Production config and log retention should be reviewed during deployment.

## Practice questions

1. Which environment file should contain real local secrets?
2. Why should application code use config() instead of env()?
3. What kind of data is appropriate for a session?
4. Why is cache not the source of truth?
5. What should happen to cached data when its source changes?
6. Name two kinds of data that should not be logged.
7. What does config:cache prepare?
8. Why should production logs have a retention policy?

## Main references

- [Laravel 13 configuration](https://laravel.com/docs/13.x/configuration)
- [Laravel 13 sessions](https://laravel.com/docs/13.x/session)
- [Laravel 13 cache](https://laravel.com/docs/13.x/cache)
- [Laravel 13 logging](https://laravel.com/docs/13.x/logging)
