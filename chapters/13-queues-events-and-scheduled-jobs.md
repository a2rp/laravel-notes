# 13. Queues, events, and scheduled jobs

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Files and storage](./12-files-and-storage.md) | [Notes index](../README.md) | [Next: Configuration, sessions, cache, and logs](./14-configuration-sessions-cache-and-logs.md) |

Queues move slow work out of the request that a user is waiting on. Events describe something that happened; listeners respond to it. The scheduler runs recurring commands from one system cron entry.

## Queue a slow task

Generate a job and mark it as queueable. The job should contain the data needed for the work, not an open request or response object.

~~~sh
php artisan make:job SendWeeklyDigest
~~~

~~~php
<?php

namespace App\Jobs;

use App\Models\User;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Support\Facades\Mail;

class SendWeeklyDigest implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public int $userId,
    ) {}

    public function handle(): void
    {
        $user = User::find($this->userId);

        if ($user === null) {
            return;
        }

        Mail::to($user->email)->send(new WeeklyDigestMail($user));
    }
}
~~~

Dispatch it from application code:

~~~php
SendWeeklyDigest::dispatch($user->id);
~~~

A queue needs a configured backend and a worker process. In local development, start a worker with <code>php artisan queue:work</code>. Production workers should be supervised and restarted during deployments. Configure retries, timeouts, and failed-job handling for the work’s needs.

Jobs may be retried, so design side effects to be safe if they happen more than once. Do not assume a job runs exactly one time.

## Use events to describe changes

An event names a meaningful thing that happened, such as a note being published. A listener handles a separate response, such as sending an email or updating a search index. This keeps the original action from knowing every follow-up task.

~~~php
event(new NotePublished($note));
~~~

A listener can be queued if its work is slow. Keep the event payload small and avoid including secrets or unnecessary personal data. If an event is dispatched while a database transaction is open, consider whether listeners should run only after the transaction commits.

## Schedule recurring work

Define tasks in <code>routes/console.php</code> for the default Laravel 13 application structure.

~~~php
use Illuminate\Support\Facades\Schedule;

Schedule::command('reports:cleanup')
    ->daily()
    ->withoutOverlapping();
~~~

The application server needs one cron entry that invokes Laravel’s scheduler each minute:

~~~sh
* * * * * cd /path/to/app && php artisan schedule:run >> /dev/null 2>&1
~~~

The scheduler decides which due task should run. For multiple servers, use the scheduler’s coordination options so a task intended to run once does not run concurrently on every server.

## Key points

- Queues let users receive a response without waiting for slow background work.
- Configure and monitor a worker as well as the queue backend.
- Jobs can retry; make their side effects safe to repeat.
- Events describe facts; listeners perform follow-up work.
- Define recurring tasks with Laravel’s scheduler.
- A system cron entry must invoke schedule:run every minute.

## Practice questions

1. Why queue a slow task?
2. What command starts a local queue worker?
3. Why should a job avoid storing a request object?
4. Why must jobs tolerate retries?
5. What does an event represent?
6. What should a listener do when its work is slow?
7. Where can Laravel 13 schedule definitions be placed?
8. What does the system cron entry run?

## Main references

- [Laravel 13 queues](https://laravel.com/docs/13.x/queues)
- [Laravel 13 events](https://laravel.com/docs/13.x/events)
- [Laravel 13 task scheduling](https://laravel.com/docs/13.x/scheduling)
- [Laravel 13 mail](https://laravel.com/docs/13.x/mail)
