# 15. Testing Laravel applications

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Configuration, sessions, cache, and logs](./14-configuration-sessions-cache-and-logs.md) | [Notes index](../README.md) | [Next: Security and deployment basics](./16-security-and-deployment-basics.md) |

Tests describe expected behavior and catch regressions when code changes. A focused test should verify an outcome a user or another part of the application can observe.

## Choose a test level

A unit test checks a small piece of PHP logic without booting the full application. A feature test boots Laravel and can exercise routes, database behavior, middleware, and responses. Use the simplest test that gives confidence in the behavior being changed.

Generate a feature test with Artisan:

~~~sh
php artisan make:test NoteTest
~~~

Laravel applications may use Pest or PHPUnit. Follow the test framework already configured in the repository.

## Test an HTTP behavior

A feature test can submit a request and assert the response. Authenticate a user when the route requires it.

~~~php
<?php

namespace Tests\Feature;

use App\Models\Note;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class NoteTest extends TestCase
{
    use RefreshDatabase;

    public function test_signed_in_user_can_create_a_note(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->post('/notes', [
            'title' => 'First note',
            'body' => 'A useful reminder.',
        ]);

        $response->assertRedirect();

        $this->assertDatabaseHas('notes', [
            'user_id' => $user->id,
            'title' => 'First note',
        ]);
    }
}
~~~

<code>RefreshDatabase</code> gives tests a clean database state using Laravel’s test database setup. Configure a separate test database or in-memory database so running tests cannot change local or production data.

## Test validation and authorization

Test both allowed and denied behavior. A user should not be able to update another user’s note simply by changing an identifier in the URL.

~~~php
public function test_user_cannot_update_another_users_note(): void
{
    $owner = User::factory()->create();
    $otherUser = User::factory()->create();
    $note = Note::factory()->for($owner)->create();

    $response = $this->actingAs($otherUser)->put("/notes/{$note->id}", [
        'title' => 'Changed',
        'body' => 'Not allowed.',
    ]);

    $response->assertForbidden();

    $this->assertDatabaseHas('notes', [
        'id' => $note->id,
        'title' => $note->title,
    ]);
}
~~~

Also check invalid input, unauthenticated requests, and successful authorized actions. Assert the state that matters, not internal implementation details.

## Fake external services

Tests should not send real email, make real HTTP calls, or charge a live payment account. Laravel provides fakes for many services.

~~~php
use Illuminate\Support\Facades\Mail;

Mail::fake();

// Run the application action that should send the message.

Mail::assertSent(WeeklyDigestMail::class);
~~~

Keep tests independent of external networks and production credentials. Prefer fakes for boundaries that are not the behavior under test.

## Run the suite when working in a project

Use the configured test command, commonly:

~~~sh
php artisan test
~~~

A useful suite is deterministic: it should not depend on execution order, current real time, or a live third-party service. When a test fails, inspect the failure and application behavior rather than weakening an assertion without understanding why.

## Key points

- Unit tests focus on isolated logic; feature tests exercise Laravel behavior.
- Keep a test database separate from real application data.
- Test successful, invalid, unauthenticated, and unauthorized cases.
- Assert externally meaningful results and database state.
- Fake email, HTTP, and payment boundaries in automated tests.
- Follow the test framework and commands already configured in the project.

## Practice questions

1. What does a feature test exercise?
2. When is a unit test the better choice?
3. What does RefreshDatabase help with?
4. Why use a separate test database?
5. Why test that one user cannot change another user’s record?
6. What does Mail::fake prevent?
7. Why should automated tests avoid real third-party services?
8. What should you inspect before changing a failing assertion?

## Main references

- [Laravel 13 testing](https://laravel.com/docs/13.x/testing)
- [Laravel 13 HTTP tests](https://laravel.com/docs/13.x/http-tests)
- [Laravel 13 database testing](https://laravel.com/docs/13.x/database-testing)
- [Laravel 13 mocking](https://laravel.com/docs/13.x/mocking)
