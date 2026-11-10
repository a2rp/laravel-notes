# 16. Security and deployment basics

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Testing Laravel applications](./15-testing-laravel-applications.md) | [Notes index](../README.md) | [Next: All code samples](./98-all-code-samples.md) |

Security is part of every application feature. Validate data, authorize access, protect secrets, and configure the production environment before exposing the app to users.

## Validate input and escape output

Validate every value that enters through a request, including IDs and uploaded files. Use query builder and Eloquent methods that bind values rather than concatenating user input into SQL.

Blade escapes ordinary output by default:

~~~blade
<h1>{{ $note->title }}</h1>
~~~

Avoid rendering untrusted content with raw Blade output. If an application intentionally supports rich HTML, sanitize it with a well-maintained sanitizer before rendering.

## Protect records and forms

Use policies or gates to authorize each protected action. Checking ownership only in the interface does not prevent a user from sending a request manually.

For browser forms, keep Laravel’s CSRF protection enabled and include the CSRF token. Use HTTPS in production, secure session cookies, and regenerate the session after authentication.

## Protect secrets and dependencies

Never commit credentials, API tokens, private keys, or production environment files. Rotate any secret that was accidentally published. Use environment variables or a secrets manager in deployment and grant each service only the access it needs.

Review dependencies and keep the framework and packages on supported releases. Apply security updates, inspect release notes, and test upgrades in a controlled environment.

## Prepare the production environment

A production deployment commonly needs to:

1. Configure a supported PHP version and required extensions.
2. Install locked production dependencies with Composer.
3. provide correct database, mail, cache, queue, and storage settings.
4. Run reviewed database migrations.
5. Build frontend assets when the application uses them.
6. Point the web server document root at Laravel’s <code>public</code> directory.
7. Set <code>APP_DEBUG=false</code> and use HTTPS.
8. Ensure storage and cache directories are writable by the app process.
9. Run queue workers and the scheduler when the app uses them.
10. Configure backups, monitoring, and a rollback plan.

Use Laravel’s deployment commands and optimizations according to the hosting environment. Do not expose the repository root or <code>.env</code> through the web server.

## Keep errors private and observable

Production error pages should not disclose stack traces, database queries, local paths, or secrets. Keep detailed logs on the server with controlled access and retention. Monitor application health and failed queue jobs so operational problems are visible.

## Key points

- Validate input and authorize every protected operation.
- Escape untrusted output; avoid raw HTML rendering.
- Keep CSRF protection enabled for browser forms and require HTTPS.
- Never commit secrets; rotate a secret after accidental exposure.
- Deploy from a supported, locked dependency set.
- Use the public directory as the web root and disable debug output in production.
- Plan for persistent storage, queues, scheduled tasks, backups, and rollback.

## Practice questions

1. Why is client-side validation not enough?
2. Which Blade syntax escapes displayed values?
3. Why does hiding an action in the interface not authorize it?
4. What does CSRF protection defend against?
5. What should you do if a secret is committed publicly?
6. Which Laravel directory should a web server expose?
7. What should APP_DEBUG be set to in production?
8. Why are backups and rollback part of deployment planning?

## Main references

- [Laravel 13 deployment](https://laravel.com/docs/13.x/deployment)
- [Laravel 13 security](https://laravel.com/docs/13.x/security)
- [Laravel 13 CSRF protection](https://laravel.com/docs/13.x/csrf)
- [Laravel 13 validation](https://laravel.com/docs/13.x/validation)
- [Laravel 13 authorization](https://laravel.com/docs/13.x/authorization)
- [Laravel 13 configuration](https://laravel.com/docs/13.x/configuration)
