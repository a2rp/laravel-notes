# 3. Blade views and layouts

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Routes and controllers](./02-routes-and-controllers.md) | [Notes index](../README.md) | [Next: Forms and validation](./04-forms-and-validation.md) |

A view holds the presentation for a response. Laravel's Blade template engine lets a view combine HTML with readable directives and values from PHP. Blade templates are compiled to PHP and cached by Laravel.

## Return a view with data

Store a view in resources/views. Dots in the view name represent directory separators. Pass the data a view needs from a controller or route.

~~~php
Route::get('/notes', function () {
    $notes = ['Arrays', 'Routing', 'Blade'];

    return view('notes.index', [
        'pageTitle' => 'Study notes',
        'notes' => $notes,
    ]);
});
~~~

This route looks for resources/views/notes/index.blade.php. A controller can return the same view response.

~~~php
public function index()
{
    return view('notes.index', [
        'pageTitle' => 'Study notes',
        'notes' => ['Arrays', 'Routing', 'Blade'],
    ]);
}
~~~

In the template, Blade can print values and loop over the list:

~~~blade
<h1>{{ $pageTitle }}</h1>

<ul>
    @foreach ($notes as $note)
        <li>{{ $note }}</li>
    @endforeach
</ul>
~~~

## Escape output by default

The double curly brace syntax escapes output before it is placed into HTML. This helps prevent user-provided text from being interpreted as markup.

~~~blade
<p>{{ $noteTitle }}</p>
~~~

Avoid the raw output syntax for values that can come from a user, request, or database. Raw HTML can run unsafe markup in a browser. If formatted HTML is required, sanitize it with a trusted sanitizer before rendering.

## Use conditions and empty states

Blade directives make common conditions easy to read.

~~~blade
@if ($notes->isEmpty())
    <p>No notes have been added yet.</p>
@else
    <p>{{ $notes->count() }} notes are available.</p>
@endif
~~~

For a list, @forelse handles both the items and the empty state in one block.

~~~blade
<ul>
    @forelse ($notes as $note)
        <li>{{ $note->title }}</li>
    @empty
        <li>No notes found.</li>
    @endforelse
</ul>
~~~

Blade directives control presentation. Keep database work and complicated business rules in controllers or dedicated application classes.

## Share a layout

A layout gives multiple pages a shared document structure. Create resources/views/layouts/app.blade.php:

~~~blade
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>@yield('title', 'Study Notes')</title>
</head>
<body>
    <header>
        <a href="{{ route('notes.index') }}">Notes</a>
    </header>

    <main>
        @yield('content')
    </main>
</body>
</html>
~~~

A child view can extend the layout and fill its named sections:

~~~blade
@extends('layouts.app')

@section('title', 'All notes')

@section('content')
    <h1>All notes</h1>
    <p>Choose a topic to continue.</p>
@endsection
~~~

This keeps the page shell in one file and page-specific content in another.

## Build a reusable component

A Blade component packages repeated markup. An anonymous component can live at resources/views/components/alert.blade.php.

~~~blade
@props(['type' => 'info'])

<div class="alert alert-{{ $type }}">
    {{ $slot }}
</div>
~~~

Use it with an attribute and slot content:

~~~blade
<x-alert type="success">
    Your changes were saved.
</x-alert>
~~~

Components are useful for repeated interface elements such as alerts, cards, and form fields. Keep their inputs small and their markup focused.

## Generate links through routes

Use the route helper to create links from named routes. This avoids repeating a URL in many views.

~~~blade
<a href="{{ route('notes.index') }}">All notes</a>
<a href="{{ route('notes.show', ['note' => $note->id]) }}">
    {{ $note->title }}
</a>
~~~

Keep display text escaped with Blade's normal double braces. The URL helper generates a path, while the browser follows it when the link is selected.

## Key points

- Blade views are stored in resources/views and end in .blade.php.
- Controllers pass data into a view response.
- Double braces escape output. Raw HTML output requires careful sanitization.
- Blade provides conditions, loops, sections, and reusable components.
- Layouts keep shared page structure in one file.
- Named route helpers prevent URLs from being copied throughout templates.

## Practice questions

1. Where are Blade view files stored?
2. How does a view name with dots map to folders?
3. What does double brace output do?
4. Why is raw HTML output risky for user-provided content?
5. When is @forelse useful?
6. What does a layout provide to child views?
7. What does a Blade component's slot contain?
8. Why use route() when building a link?

## Main references

- [Laravel 13 views](https://laravel.com/docs/13.x/views)
- [Laravel 13 Blade templates](https://laravel.com/docs/13.x/blade)
- [Laravel 13 responses](https://laravel.com/docs/13.x/responses)
