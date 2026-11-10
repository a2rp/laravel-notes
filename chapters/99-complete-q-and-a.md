# 99. Complete questions and answers

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Code samples quick reference](./98-all-code-samples.md) | [Notes index](../README.md) | End of notes |

Use these answers to review the eight practice questions at the end of each core chapter. Try answering from memory before reading this appendix.

## Chapter 1: Start with Laravel

1. PHP.
2. Composer installs and manages PHP project dependencies.
3. The command is `php artisan`; Artisan is Laravel’s command-line interface.
4. `public/index.php` is the entry point for browser requests.
5. Web routes are usually defined in `routes/web.php`.
6. The router matches the request’s HTTP method and URL to a route, then runs its action.
7. Run `php artisan route:list`.
8. A real environment file can contain secrets and machine-specific values that must not be published.

## Chapter 2: Routes and controllers

1. A route matches an HTTP method and a URL pattern.
2. GET is normally used to read a resource.
3. Write a required parameter in braces, such as `/notes/{note}`.
4. A route name lets code generate links and redirects without hard-coding the URL.
5. A route group can share middleware, prefixes, names, or other route settings.
6. Artisan generates a controller under `app/Http/Controllers`.
7. Run `php artisan route:list`.
8. Run `php artisan install:api` to add API routing to a fresh Laravel app.

## Chapter 3: Blade views and layouts

1. Blade view files are stored under `resources/views`.
2. Dots in a view name represent nested folders, so `notes.show` maps to `resources/views/notes/show.blade.php`.
3. Double braces escape the value before output, helping prevent injected markup from running.
4. Raw output may execute attacker-controlled HTML or script in the visitor’s browser.
5. `@forelse` is useful when a loop needs a clear empty-state branch.
6. A layout provides shared page structure and named sections for child views.
7. A component slot contains the content placed between the component’s opening and closing tags.
8. `route()` generates a URL from the route name, so links survive URL pattern changes.

## Chapter 4: Forms and validation

1. A user can bypass browser checks, so the server must validate before processing data.
2. `@csrf` adds the token that Laravel checks to protect browser forms from cross-site request forgery.
3. HTML forms support GET and POST directly; `@method` lets Laravel interpret a hidden field as PUT, PATCH, or DELETE.
4. `Request::validate()` returns only the validated values, or redirects/returns validation errors when input is invalid.
5. JSON validation failures normally use HTTP 422.
6. `old('title')` retrieves the previous submitted title so the form can repopulate it after validation fails.
7. A Form Request centralizes authorization and validation rules.
8. Raw input may include invalid or unintended keys; passing only validated fields prevents unsafe model updates.

## Chapter 5: Database migrations and seeders

1. A migration describes a versioned change to the database schema.
2. Migration files are stored in `database/migrations`.
3. `up` applies a schema change; `down` reverses it when possible.
4. Run `php artisan migrate`.
5. `migrate:fresh` drops all tables and recreates the schema from migrations.
6. Destructive reset commands erase data, so use them only on disposable databases.
7. A seeder inserts repeatable sample or initial data into the database.
8. Shared environments may already have run a migration; create a new migration so the change is recorded and applied consistently.

## Chapter 6: Eloquent models and relationships

1. An Eloquent model represents a database table and its rows.
2. By convention, `Note` uses the `notes` table.
3. `fillable` restricts which model attributes can be set through mass assignment.
4. `get()` retrieves all matching records into a collection.
5. The model containing the foreign key usually defines `belongsTo`.
6. `hasMany` describes one model having multiple related records.
7. `comments()` returns a relationship query; `$note->comments` accesses or loads the related collection.
8. `with('comments')` eager loads comments and helps avoid an N+1 query pattern.

## Chapter 7: Queries, scopes, and pagination

1. The query usually runs when a terminal operation such as `get`, `first`, `count`, or `paginate` is called.
2. Concatenated user input can alter SQL and create injection vulnerabilities; parameterized query methods bind values safely.
3. Put the alternatives inside a grouped `where` closure.
4. Laravel 13’s `#[Scope]` attribute marks a protected model method as a local query scope.
5. `paginate` returns one page plus pagination metadata, while `get` returns all matching rows.
6. An N+1 pattern occurs when code loads a list, then issues another relationship query for each item.
7. `withCount` adds a computed relationship count attribute such as `comments_count`.
8. Laravel rolls back the transaction if an exception escapes the transaction callback.

## Chapter 8: Authentication and authorization

1. Authentication answers who is making the request.
2. Authorization answers whether that user may perform a specific action.
3. The `auth` middleware restricts a route to signed-in users.
4. Regenerating the session ID after login reduces the risk of session fixation.
5. Logout should invalidate the session and regenerate its CSRF token.
6. A policy fits rules centered on a model and its actions; a gate fits broader abilities.
7. A user can send a request directly, so the server must enforce permission regardless of what the interface shows.
8. Store a securely generated password hash, never the plain-text password.

## Chapter 9: Middleware and request lifecycle

1. Middleware wraps route handling and runs before, and sometimes after, the route action.
2. It can return a response immediately or pass the request onward and return the resulting response.
3. Run `php artisan make:middleware EnsureAccountIsActive`.
4. Laravel 13 configures application middleware in `bootstrap/app.php`.
5. An alias gives middleware a short name that routes can apply.
6. Focused middleware is easier to understand, test, and reuse.
7. A client can forge a header; a trusted authentication layer must verify identity.
8. Middleware can inspect the response after it calls `$next($request)`.

## Chapter 10: Service container and dependency injection

1. The container constructs objects and resolves the dependencies they need.
2. Dependency injection makes a class’s required collaborators explicit.
3. An interface lets code depend on a capability rather than one concrete implementation.
4. `bind` resolves a new instance as needed; `singleton` shares one container-managed instance.
5. Laravel 13 lists application service providers in `bootstrap/providers.php`.
6. The `register` method is for registering bindings with the container.
7. A facade is a convenient static-looking interface to a container-managed service.
8. Authorization before charging prevents a user from causing a side effect they are not allowed to trigger.

## Chapter 11: Build JSON APIs

1. A stateless API route does not rely on the normal browser session for its client state; requests carry the credentials and data needed to process them.
2. Run `php artisan install:api`.
3. Validation rejects malformed or invalid values before they can create bad records or trigger side effects.
4. HTTP 201 commonly represents a successfully created resource.
5. An API resource controls how model data is represented in a public response.
6. A model may contain internal fields that should not become part of the API contract.
7. Eager loading avoids extra queries for each item in the paginated result.
8. Never return stack traces, SQL, secrets, file paths, or internal exception details to a public client.

## Chapter 12: Files and storage

1. A client can bypass browser validation and send a crafted request, so the server must check the file.
2. The form needs `multipart/form-data` encoding.
3. `store()` returns the generated path of the saved file.
4. Keeping a path avoids placing large binary contents in the database and keeps storage replaceable.
5. A public disk allows files to be served publicly; a private disk requires controlled access.
6. Without authorization, a user might download another person’s document by guessing or changing an identifier.
7. Local files may disappear when an instance is redeployed or replaced on an ephemeral filesystem.
8. Consider storage errors, database consistency, and removing the old file only after the new one is saved successfully.

## Chapter 13: Queues, events, and scheduled jobs

1. A queue lets the request finish without waiting for slow work such as sending a digest email.
2. Run `php artisan queue:work`.
3. A queued job may run later in another process where the request object no longer exists.
4. Workers may retry a job after a failure, so duplicate execution must not cause duplicate harmful side effects.
5. An event represents a meaningful fact that has happened in the application.
6. A slow listener should implement `ShouldQueue` so its work runs in the background.
7. Laravel 13’s default app can define scheduled tasks in `routes/console.php`.
8. The cron entry runs `php artisan schedule:run` each minute.

## Chapter 14: Configuration, sessions, cache, and logs

1. Real local secrets belong in the uncommitted `.env` file or a deployment secrets manager.
2. After config is cached, application code should use `config()` because `.env` values are not loaded as a runtime API.
3. A session is appropriate for short-lived visitor state, such as a temporary checkout step.
4. Cache is temporary and can expire or be cleared, so durable business records must live in their authoritative store.
5. Invalidate or refresh cached values when the underlying source changes.
6. Do not log passwords, tokens, session identifiers, or sensitive payment data.
7. `config:cache` builds a cached configuration set for production startup.
8. Retention controls disk use and limits how long operational data is kept.

## Chapter 15: Testing Laravel applications

1. A feature test boots Laravel and exercises behavior such as routes, middleware, database access, and responses.
2. A unit test is better for a small isolated piece of logic that does not need the full app.
3. `RefreshDatabase` gives tests a clean database state using Laravel’s configured test database.
4. A separate test database prevents tests from altering real application data.
5. It verifies the authorization boundary and prevents insecure direct-object access.
6. `Mail::fake()` intercepts email so a test can assert behavior without delivering a real message.
7. Real services introduce network dependence, cost, secrets, side effects, and nondeterministic results.
8. Inspect the failure and actual application behavior before deciding whether the assertion or the implementation is wrong.

## Chapter 16: Security and deployment basics

1. Browser validation can be bypassed, so the server must check all submitted data.
2. Blade’s `{{ $value }}` syntax escapes displayed values.
3. Users can send requests without clicking the interface, so the server must still check permission.
4. CSRF protection helps prevent a third-party site from causing a browser to submit an unwanted authenticated request.
5. Revoke or rotate the exposed secret immediately, then remove it from source history where appropriate.
6. The web server should expose Laravel’s `public` directory.
7. Set `APP_DEBUG=false` in production.
8. Backups help recover data; a rollback plan helps restore service when a deployment fails.
