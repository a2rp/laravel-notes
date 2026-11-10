# 11. Build JSON APIs

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Service container and dependency injection](./10-service-container-and-dependency-injection.md) | [Notes index](../README.md) | [Next: Files and storage](./12-files-and-storage.md) |

An API lets another client communicate with an application over HTTP. A useful API has predictable URLs, validated input, consistent response shapes, and meaningful status codes.

## Define an API route

Laravel API routes are commonly placed in <code>routes/api.php</code>. In a fresh app, install API routing with the Artisan command if it is not present.

~~~sh
php artisan install:api
~~~

API routes are stateless by default and receive the <code>/api</code> prefix. Use resource routes for conventional CRUD endpoints.

~~~php
use App\Http\Controllers\Api\NoteController;
use Illuminate\Support\Facades\Route;

Route::apiResource('notes', NoteController::class)
    ->middleware('auth:sanctum');
~~~

Choose an authentication method that fits the client. A browser-based first-party client and an external mobile client may need different authentication arrangements. Do not assume that hiding an endpoint makes it private.

## Validate input and return a response

Validate incoming data before creating or changing a record. JSON clients receive validation errors in a structured response when the request expects JSON.

~~~php
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => ['required', 'string', 'max:150'],
        'body' => ['required', 'string'],
    ]);

    $note = $request->user()->notes()->create($validated);

    return response()->json([
        'data' => $note,
    ], 201);
}
~~~

HTTP status codes communicate the outcome: use 201 for a created resource, 200 for a successful read or update, 204 for a successful response with no body, 404 when a resource is unavailable, and 422 for validation errors.

## Shape output with API resources

Returning an Eloquent model directly can expose fields you did not intend to make part of the API. An API resource defines the public representation.

~~~sh
php artisan make:resource NoteResource
~~~

~~~php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class NoteResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'body' => $this->body,
            'created_at' => $this->created_at?->toISOString(),
        ];
    }
}
~~~

Return one resource or a resource collection:

~~~php
return new NoteResource($note);
~~~

~~~php
return NoteResource::collection(
    Note::query()->latest()->paginate(20)
);
~~~

Resources keep response formatting separate from database columns. Add related data intentionally and eager load relationships to avoid unnecessary queries.

## Keep API errors useful

Use Laravel’s exception handling and response conventions rather than returning a successful status for a failed operation. Do not send stack traces, SQL, credentials, or internal exception messages to public clients. Log enough context on the server to diagnose failures.

For a public API, document authentication, validation rules, response fields, pagination, and rate limits. Make changes to a response contract deliberately because clients may depend on existing fields.

## Key points

- API routes are stateless and normally use JSON responses.
- Validate request input before performing writes.
- Use meaningful HTTP status codes.
- API resources define the fields clients are allowed to see.
- Authenticate and authorize every protected operation.
- Do not expose internal error details to clients.

## Practice questions

1. What does a stateless API route mean at a high level?
2. What Artisan command adds API routing to a fresh app?
3. Why validate before creating a model?
4. Which status code commonly represents successful creation?
5. What does an API resource control?
6. Why might returning an Eloquent model directly be risky?
7. Why should a paginated API eager load needed relationships?
8. Which details should never be returned in a public error response?

## Main references

- [Laravel 13 routing](https://laravel.com/docs/13.x/routing)
- [Laravel 13 requests](https://laravel.com/docs/13.x/requests)
- [Laravel 13 validation](https://laravel.com/docs/13.x/validation)
- [Laravel 13 responses](https://laravel.com/docs/13.x/responses)
- [Laravel 13 API resources](https://laravel.com/docs/13.x/eloquent-resources)
- [Laravel Sanctum](https://laravel.com/docs/13.x/sanctum)
