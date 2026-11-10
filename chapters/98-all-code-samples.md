# 98. Code samples quick reference

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Security and deployment basics](./16-security-and-deployment-basics.md) | [Notes index](../README.md) | [Next: Complete questions and answers](./99-complete-q-and-a.md) |

This appendix collects selected patterns from the core chapters in one place. Each example is a focused reference, not a standalone application. Follow the linked chapter for required model, migration, route, and configuration context.

## Create a model and migration

~~~sh
php artisan make:model Note -m
php artisan migrate
~~~

Define only the fields that can safely be mass assigned, and create records from validated data.

~~~php
class Note extends Model
{
    protected $fillable = ['title', 'body'];
}

$validated = $request->validate([
    'title' => ['required', 'string', 'max:150'],
    'body' => ['required', 'string'],
]);

$note = $request->user()->notes()->create($validated);
~~~

See [Chapter 4: Forms and validation](./04-forms-and-validation.md), [Chapter 5: Database migrations and seeders](./05-database-migrations-and-seeders.md), and [Chapter 6: Eloquent models and relationships](./06-eloquent-models-and-relationships.md).

## Paginate and eager load

~~~php
$notes = Note::with('comments')
    ->latest()
    ->paginate(15)
    ->withQueryString();
~~~

~~~blade
@foreach ($notes as $note)
    <article>
        <h2>{{ $note->title }}</h2>
        <ul>
            @foreach ($note->comments as $comment)
                <li>{{ $comment->body }}</li>
            @endforeach
        </ul>
    </article>
@endforeach

{{ $notes->links() }}
~~~

Prefer `withCount('comments')` when the view only needs the count. See [Chapter 7: Queries, scopes, and pagination](./07-queries-scopes-and-pagination.md).

## Protect a model action

~~~php
public function update(Request $request, Note $note)
{
    $this->authorize('update', $note);

    $validated = $request->validate([
        'title' => ['required', 'string', 'max:150'],
        'body' => ['required', 'string'],
    ]);

    $note->update($validated);

    return redirect()->route('notes.show', $note);
}
~~~

The policy must check that the signed-in user may update this note. See [Chapter 8: Authentication and authorization](./08-authentication-and-authorization.md).

## Return an API resource

~~~php
return NoteResource::collection(
    Note::query()->latest()->paginate(20)
);
~~~

Shape public fields in `NoteResource::toArray` instead of exposing every model attribute. See [Chapter 11: Build JSON APIs](./11-build-json-apis.md).

## Store and authorize an upload

~~~php
$validated = $request->validate([
    'document' => ['required', 'file', 'mimes:pdf', 'max:10240'],
]);

$path = $request->file('document')->store('reports', 'private');

$report->update(['file_path' => $path]);
~~~

~~~php
public function download(Report $report)
{
    $this->authorize('view', $report);

    return Storage::disk('private')->download($report->file_path);
}
~~~

See [Chapter 12: Files and storage](./12-files-and-storage.md).

## Queue work and schedule recurring commands

~~~php
SendWeeklyDigest::dispatch($user->id);
~~~

~~~php
Schedule::command('reports:cleanup')
    ->daily()
    ->withoutOverlapping();
~~~

A queue worker must be running for queued work. A system cron entry must invoke `php artisan schedule:run` every minute. See [Chapter 13: Queues, events, and scheduled jobs](./13-queues-events-and-scheduled-jobs.md).

## Cache a computed result

~~~php
$popularNotes = Cache::remember(
    'notes:popular',
    now()->addMinutes(10),
    fn () => Note::where('published', true)
        ->orderByDesc('views')
        ->limit(10)
        ->get()
);
~~~

Choose an expiration and invalidate the cache when its source data changes. See [Chapter 14: Configuration, sessions, cache, and logs](./14-configuration-sessions-cache-and-logs.md).

## Test an important behavior

~~~php
$response = $this->actingAs($user)->post('/notes', [
    'title' => 'First note',
    'body' => 'A useful reminder.',
]);

$response->assertRedirect();

$this->assertDatabaseHas('notes', [
    'user_id' => $user->id,
    'title' => 'First note',
]);
~~~

Use a separate test database and test both successful and denied behavior. See [Chapter 15: Testing Laravel applications](./15-testing-laravel-applications.md).

## Production deployment checklist

~~~sh
composer install --no-dev --optimize-autoloader
npm ci
npm run build
php artisan migrate --force
php artisan optimize
~~~

Use the commands that match the app’s actual dependencies and deployment process. Configure secrets outside source control, set `APP_DEBUG=false`, expose only Laravel’s `public` directory, and confirm that queue workers, scheduled tasks, storage, backups, and rollback are handled. See [Chapter 16: Security and deployment basics](./16-security-and-deployment-basics.md).

## Other chapter references

- [Chapter 1: Start with Laravel](./01-start-with-laravel.md)
- [Chapter 2: Routes and controllers](./02-routes-and-controllers.md)
- [Chapter 3: Blade views and layouts](./03-blade-views-and-layouts.md)
- [Chapter 9: Middleware and request lifecycle](./09-middleware-and-request-lifecycle.md)
- [Chapter 10: Service container and dependency injection](./10-service-container-and-dependency-injection.md)
