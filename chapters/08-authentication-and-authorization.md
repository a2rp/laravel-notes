# 8. Authentication and authorization

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Queries, scopes, and pagination](./07-queries-scopes-and-pagination.md) | [Notes index](../README.md) | [Next: Middleware and request lifecycle](./09-middleware-and-request-lifecycle.md) |

Authentication answers “who is making this request?” Authorization answers “may this user perform this action?” A user being signed in does not automatically grant access to every record.

## Start with an authentication starter kit

Laravel authentication includes guards, providers, password hashing, session handling, and password reset features. A starter kit provides ready-made login, registration, and related screens. For a new application, use the starter kit that fits its frontend and review the generated routes and configuration.

Do not build password storage yourself. Laravel’s authentication features use the configured password hasher. Store only a password hash, never a plain password.

## Protect routes

The auth middleware allows signed-in users through and redirects guests to the login flow for web routes.

~~~php
use App\Http\Controllers\NoteController;
use Illuminate\Support\Facades\Route;

Route::middleware('auth')->group(function () {
    Route::resource('notes', NoteController::class);
});
~~~

Authentication middleware checks identity. It does not decide whether a user owns a particular note.

## Understand the session login flow

When implementing a custom login form, validate the submitted credentials and ask the authentication system to check them. After successful login, regenerate the session ID to reduce session fixation risk.

~~~php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

public function store(Request $request)
{
    $credentials = $request->validate([
        'email' => ['required', 'email'],
        'password' => ['required', 'string'],
    ]);

    if (! Auth::attempt($credentials, $request->boolean('remember'))) {
        return back()->withErrors([
            'email' => 'These credentials do not match our records.',
        ])->onlyInput('email');
    }

    $request->session()->regenerate();

    return redirect()->intended(route('dashboard'));
}
~~~

On logout, invalidate the session and rotate the CSRF token.

~~~php
Auth::logout();

$request->session()->invalidate();
$request->session()->regenerateToken();

return redirect('/');
~~~

Prefer a supported starter kit unless a custom flow is necessary. Authentication involves edge cases such as throttling, verification, password reset, and multi-factor authentication.

## Authorize actions with policies

A policy groups rules for a model. Generate one with Artisan and define actions such as viewing, updating, or deleting a note.

~~~sh
php artisan make:policy NotePolicy --model=Note
~~~

~~~php
<?php

namespace App\Policies;

use App\Models\Note;
use App\Models\User;

class NotePolicy
{
    public function update(User $user, Note $note): bool
    {
        return $user->id === $note->user_id;
    }

    public function delete(User $user, Note $note): bool
    {
        return $user->id === $note->user_id;
    }
}
~~~

Laravel can discover policies that follow its conventional model and policy locations. Check the policy registration if the app uses a custom structure.

Use authorization in a controller or route. A denied action becomes an authorization response rather than relying on a hidden button in the page.

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

A Blade template may hide an action for convenience, but the server must still authorize the request.

~~~blade
@can('update', $note)
    <a href="{{ route('notes.edit', $note) }}">Edit</a>
@endcan
~~~

## Use gates for abilities without a model

A gate suits a small application-wide ability, such as access to an administration dashboard. A policy is usually clearer when rules center on a model.

~~~php
use Illuminate\Support\Facades\Gate;

Gate::define('view-admin-dashboard', function (User $user): bool {
    return $user->is_admin;
});
~~~

Check the ability with <code>Gate::authorize</code>, <code>$user->can()</code>, or Blade’s <code>@can</code> directive. Keep sensitive checks on the server.

## Key points

- Authentication identifies a user; authorization checks a requested action.
- Use Laravel starter kits or its authentication facilities instead of inventing password handling.
- Regenerate the session after login and invalidate it on logout.
- Protect signed-in routes with auth middleware.
- Use policies for model-specific rules and gates for broader abilities.
- A hidden or disabled UI control does not replace server-side authorization.

## Practice questions

1. What question does authentication answer?
2. What does authorization answer?
3. What does auth middleware protect?
4. Why regenerate the session after a successful login?
5. What should a logout flow do to the session?
6. When is a policy a better fit than a gate?
7. Why must a controller authorize a request even if its button is hidden?
8. What should an application store instead of a plain-text password?

## Main references

- [Laravel 13 authentication](https://laravel.com/docs/13.x/authentication)
- [Laravel 13 authorization](https://laravel.com/docs/13.x/authorization)
- [Laravel 13 starter kits](https://laravel.com/docs/13.x/starter-kits)
- [Laravel 13 password hashing](https://laravel.com/docs/13.x/hashing)
