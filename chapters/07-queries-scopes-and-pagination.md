# 7. Queries, scopes, and pagination

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Eloquent models and relationships](./06-eloquent-models-and-relationships.md) | [Notes index](../README.md) | [Next: Authentication and authorization](./08-authentication-and-authorization.md) |

Eloquent builds database queries through a chain of readable methods. The query is sent when you call a terminal method such as get, first, count, or paginate. Add filters and ordering before retrieving the results.

## Filter and order records

Use where for conditions and pass values separately. Laravel uses parameter binding for query values.

~~~php
$notes = Note::where('published', true)
    ->where('title', 'like', '%database%')
    ->orderByDesc('created_at')
    ->get();
~~~

Avoid concatenating user input into raw SQL. If a query needs an OR condition, group it with a closure so it does not accidentally bypass another condition.

~~~php
$notes = Note::where('published', true)
    ->where(function ($query) {
        $query->where('title', 'like', '%database%')
            ->orWhere('body', 'like', '%database%');
    })
    ->get();
~~~

Use aggregate methods when you need a result rather than model objects.

~~~php
$noteCount = Note::where('published', true)->count();
$hasDrafts = Note::where('published', false)->exists();
~~~

## Reuse query rules with a local scope

A local scope names a query condition in the model. Laravel 13 uses the Scope attribute on protected methods.

~~~php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Scope;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class Note extends Model
{
    #[Scope]
    protected function published(Builder $query): void
    {
        $query->whereNotNull('published_at');
    }
}
~~~

Call the scope as part of a query:

~~~php
$recentNotes = Note::published()
    ->orderByDesc('published_at')
    ->limit(10)
    ->get();
~~~

Scopes are useful for conditions that are repeated and have a clear meaning. Avoid hiding a complicated query behind a vague scope name.

## Paginate large result sets

Do not load an unbounded table into memory for a page. paginate retrieves one page and includes total-page information.

~~~php
$notes = Note::published()
    ->orderByDesc('created_at')
    ->paginate(10);
~~~

Render the collection and page links in Blade:

~~~blade
@foreach ($notes as $note)
    <article>
        <h2>{{ $note->title }}</h2>
    </article>
@endforeach

{{ $notes->links() }}
~~~

Use simplePaginate when the interface only needs next and previous links and does not need a total count. Cursor pagination can work well for long, frequently changing lists with a stable order.

## Avoid the N+1 query problem

If a page loops through notes and reads each note's comments, lazy loading may issue one query for the notes and one more per note. Eager load the comments before the loop.

~~~php
$notes = Note::with('comments')
    ->latest()
    ->paginate(10);
~~~

You can also count related rows without loading every comment:

~~~php
$notes = Note::withCount('comments')->latest()->get();

foreach ($notes as $note) {
    echo $note->comments_count;
}
~~~

When performance matters, inspect the queries the page actually runs. Load only the relationships and columns the view needs.

## Group related database changes in a transaction

A transaction treats several database operations as one unit. If an exception escapes the callback, Laravel rolls back the transaction. If the callback finishes, Laravel commits it.

~~~php
use Illuminate\Support\Facades\DB;

DB::transaction(function () use ($noteData, $commentData) {
    $note = Note::create($noteData);
    $note->comments()->create($commentData);
});
~~~

Use a transaction when related writes must succeed or fail together. Do not perform slow external network work inside a database transaction.

## Key points

- Query methods can be chained before a terminal method runs the database query.
- Use parameterized query methods instead of inserting user input into SQL strings.
- Group OR conditions so they do not bypass other filters.
- Laravel 13 local scopes use the Scope attribute.
- Pagination keeps a page from loading every row at once.
- Eager loading prevents repeated relationship queries.
- Transactions keep related database writes together.

## Practice questions

1. When does an Eloquent query usually run?
2. Why should user input not be concatenated into raw SQL?
3. How can an OR condition be grouped with other filters?
4. What does Laravel 13's Scope attribute mark?
5. What does paginate return that a plain get does not?
6. What causes an N+1 query pattern?
7. What does withCount add to each model?
8. What happens to a transaction if an exception escapes its callback?

## Main references

- [Laravel 13 Eloquent ORM](https://laravel.com/docs/13.x/eloquent)
- [Laravel 13 query builder](https://laravel.com/docs/13.x/queries)
- [Laravel 13 pagination](https://laravel.com/docs/13.x/pagination)
- [Laravel 13 database transactions](https://laravel.com/docs/13.x/database#database-transactions)
