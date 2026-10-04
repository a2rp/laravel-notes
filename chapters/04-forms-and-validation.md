# 4. Forms and validation

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Blade views and layouts](./03-blade-views-and-layouts.md) | [Notes index](../README.md) | [Next: Database migrations and seeders](./05-database-migrations-and-seeders.md) |

A form collects input from a person. Treat every submitted value as untrusted until the server validates it. Laravel can validate input, keep safe values for redisplay, and return useful errors when a field is invalid.

## Add CSRF protection to a web form

Forms that send POST, PUT, PATCH, or DELETE requests through the web middleware group need a CSRF token. Blade's @csrf directive adds the hidden token field.

~~~blade
<form method="POST" action="{{ route('notes.store') }}">
    @csrf

    <label for="title">Title</label>
    <input id="title" name="title" value="{{ old('title') }}">

    <button type="submit">Save note</button>
</form>
~~~

HTML forms support GET and POST directly. For PUT, PATCH, or DELETE, use method spoofing:

~~~blade
<form method="POST" action="{{ route('notes.update', ['note' => $note->id]) }}">
    @csrf
    @method('PUT')

    <label for="title">Title</label>
    <input id="title" name="title" value="{{ old('title', $note->title) }}">

    <button type="submit">Update note</button>
</form>
~~~

The hidden field tells Laravel which HTTP method the form intends to use. Keep CSRF protection enabled for browser forms.

## Validate in a controller

The validate method checks the input and returns only validated values. If browser validation fails, Laravel redirects back with errors and previous input. If the request expects JSON, Laravel returns a 422 response with validation errors.

~~~php
use Illuminate\Http\Request;

public function store(Request $request)
{
    $validated = $request->validate([
        'title' => ['required', 'string', 'max:120'],
        'body' => ['nullable', 'string', 'max:5000'],
    ]);

    // Save the validated values in the next database step.
    return redirect()
        ->route('notes.index')
        ->with('status', 'The note is ready to save.');
}
~~~

Choose rules that match the data model. A title can be required and limited in length. A body may be optional. A date should use a date rule, and a number should use an appropriate numeric or integer rule.

Do not pass the whole request into a model. Use the validated array so unexpected fields cannot be saved by accident.

## Display errors and old values

Laravel shares validation errors with Blade views after a browser redirect. Show a field error close to that field and redisplay the previous safe input.

~~~blade
<label for="title">Title</label>
<input id="title" name="title" value="{{ old('title') }}">

@error('title')
    <p class="error">{{ $message }}</p>
@enderror

@if (session('status'))
    <p role="status">{{ session('status') }}</p>
@endif
~~~

Blade's escaped output keeps the message and old value from being interpreted as HTML. Add a clear label and describe what the person needs to fix.

## Move rules into a Form Request

When validation or authorization grows, a Form Request keeps it out of a controller method. Generate one with Artisan:

~~~sh
php artisan make:request StoreNoteRequest
~~~

Add rules and decide whether the current user may make this request:

~~~php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreNoteRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:120'],
            'body' => ['nullable', 'string', 'max:5000'],
        ];
    }
}
~~~

Type-hint the request in the controller. Laravel validates it before calling the action.

~~~php
use App\Http\Requests\StoreNoteRequest;

public function store(StoreNoteRequest $request)
{
    $validated = $request->validated();

    return response()->json([
        'title' => $validated['title'],
    ], 201);
}
~~~

The authorize method is a separate access check. For resource permissions, use a policy or another clear authorization rule instead of always returning true.

## Use clear validation feedback

Laravel includes many built-in validation rules. For a unique value, the database and application should both enforce the intended rule. For cross-field checks, use the built-in rule that expresses the relationship. Custom messages can be added when a default message does not explain how to fix the input.

Validation is not a substitute for database constraints or authorization. A request can become stale or be sent outside the browser, so enforce critical rules at the layer that owns the data.

## Key points

- Validate all submitted input on the server.
- Use @csrf in browser forms handled by web routes.
- Use @method to send PUT, PATCH, or DELETE from an HTML form.
- Work with the validated array instead of the raw request.
- Show errors and old values with escaped Blade output.
- Form Request classes can hold reusable rules and authorization checks.

## Practice questions

1. Why should submitted values be validated on the server?
2. What does @csrf add to a Blade form?
3. Why does an HTML form use @method for PUT or DELETE?
4. What does Request::validate return?
5. What HTTP status is used for JSON validation failures?
6. What does old('title') provide to a view?
7. What are two responsibilities of a Form Request?
8. Why should raw request data not be passed directly to a model?

## Main references

- [Laravel 13 validation](https://laravel.com/docs/13.x/validation)
- [Laravel 13 CSRF protection](https://laravel.com/docs/13.x/csrf)
- [Laravel 13 requests](https://laravel.com/docs/13.x/requests)
- [Laravel 13 Blade templates](https://laravel.com/docs/13.x/blade)
