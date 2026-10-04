# 6. Eloquent models and relationships

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Database migrations and seeders](./05-database-migrations-and-seeders.md) | [Notes index](../README.md) | [Next: Queries, scopes, and pagination](./07-queries-scopes-and-pagination.md) |

Eloquent is Laravel's object-relational mapper. A model represents a database table and lets application code retrieve and change rows using PHP objects. By convention, the Note model uses the notes table.

## Generate and configure a model

Generate a model with Artisan:

~~~sh
php artisan make:model Note
~~~

Models are stored in app/Models. A model can define which fields are safe for mass assignment.

~~~php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Note extends Model
{
    protected $fillable = [
        'title',
        'body',
    ];
}
~~~

Mass assignment sets multiple fields from an array. Listing fillable fields prevents unexpected request keys from changing protected columns. Pass only validated data to create or update.

~~~php
$note = Note::create($validated);

$note->update($validated);
~~~

The notes table needs matching columns from the prior migration. Eloquent expects a primary key named id and timestamp columns unless the model is configured differently.

## Retrieve and save models

Eloquent query methods return model objects or collections. find looks up a row by primary key. first returns the first matching row or null. get returns a collection of matches.

~~~php
$note = Note::find(1);
$latestNotes = Note::where('published', true)
    ->orderByDesc('created_at')
    ->get();
~~~

Use create to insert a record when the model allows the supplied fields. Call save after changing a model instance.

~~~php
$note = new Note([
    'title' => 'Eloquent basics',
    'body' => 'A model represents a table row.',
]);

$note->save();
$note->title = 'Eloquent model basics';
$note->save();
~~~

A query is not sent to the database until a terminal operation such as get, first, or create executes it.

## Define a one-to-many relationship

If one note has many comments, define hasMany on Note and belongsTo on Comment.

~~~php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Note extends Model
{
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
}
~~~

~~~php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Comment extends Model
{
    public function note(): BelongsTo
    {
        return $this->belongsTo(Note::class);
    }
}
~~~

Laravel expects the comments table to hold a note_id foreign key. Define that column in a migration. Relationship methods can build queries or return related models.

~~~php
$note = Note::findOrFail(1);
$comments = $note->comments()->latest()->get();

$comment = Comment::findOrFail(3);
$parentNote = $comment->note;
~~~

The relationship method comments() is useful when adding query conditions. The property comments loads related records as a collection.

## Load related records efficiently

If a page displays comments for many notes, loading them one note at a time can create many database queries. Eager load the relationship with with.

~~~php
$notes = Note::with('comments')->latest()->get();

foreach ($notes as $note) {
    foreach ($note->comments as $comment) {
        echo $comment->body;
    }
}
~~~

This pattern helps avoid the N+1 query problem. The next chapter covers query inspection and other ways to keep database work efficient.

## Common relationship types

- belongsTo means this model stores the foreign key for its parent.
- hasOne means the related table has one matching row.
- hasMany means the related table can have several rows.
- belongsToMany uses a pivot table to connect both sides.

Use relationship names that describe the data. Add database constraints in migrations so the database also enforces the link.

## Key points

- Eloquent models represent database tables and their rows.
- Laravel uses naming conventions such as Note and notes.
- Protect mass assignment with fillable or an explicit guarded strategy.
- Use only validated data when creating or updating from requests.
- Relationship methods describe database connections.
- Eager loading can reduce repeated queries when displaying related data.

## Practice questions

1. What does an Eloquent model represent?
2. Which table does the Note model use by convention?
3. What does fillable protect?
4. Which method retrieves all matching records into a collection?
5. Which relationship belongs on a model that owns the foreign key?
6. What does hasMany describe?
7. What is the difference between comments() and the comments property?
8. What problem can with('comments') help avoid?

## Main references

- [Laravel 13 Eloquent ORM](https://laravel.com/docs/13.x/eloquent)
- [Laravel 13 Eloquent relationships](https://laravel.com/docs/13.x/eloquent-relationships)
- [Laravel 13 mass assignment](https://laravel.com/docs/13.x/eloquent#mass-assignment)
