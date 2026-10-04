# 5. Database migrations and seeders

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Forms and validation](./04-forms-and-validation.md) | [Notes index](../README.md) | [Next: Eloquent models and relationships](./06-eloquent-models-and-relationships.md) |

A migration describes a database schema change in PHP. It records a change in source control so each environment can build the same tables in the same order. A seeder inserts known starting or sample data.

Laravel 13 creates an SQLite database for a new application by default. You can use another supported database by setting the connection values in the environment file, then applying migrations.

## Create a migration

Use Artisan to generate a migration file:

~~~sh
php artisan make:migration create_notes_table
~~~

Migration files are stored in database/migrations and include a timestamp in the name. Laravel applies pending files in order.

~~~php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('notes', function (Blueprint $table) {
            $table->id();
            $table->string('title', 120);
            $table->text('body')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('notes');
    }
};
~~~

The up method makes the change. The down method describes how Laravel can reverse it. Keep each migration focused so its purpose is easy to understand.

Common column helpers include id, string, text, boolean, integer, date, and timestamps. Add constraints that reflect the data rules, such as a unique index or foreign key.

~~~php
Schema::create('comments', function (Blueprint $table) {
    $table->id();
    $table->foreignId('note_id')->constrained()->cascadeOnDelete();
    $table->string('body', 2000);
    $table->timestamps();
});
~~~

The foreign key links each comment to a note. The cascade rule removes related comments when the referenced note is deleted. Choose delete behavior that matches the application's data policy.

## Apply and inspect migrations

Use Artisan to see status and apply pending changes:

~~~sh
php artisan migrate:status
php artisan migrate
~~~

Run migrations in development after reviewing the generated SQL and checking the target database. Laravel remembers which migrations ran in the migrations table.

For local work, rollback can reverse the most recent batch. A full reset command can drop tables, so use it only with disposable development data.

~~~sh
php artisan migrate:rollback
php artisan migrate:fresh
~~~

migrate:fresh drops all tables and runs every migration again. Never run it against a database containing data you need.

## Add seed data

Generate a seeder with Artisan:

~~~sh
php artisan make:seeder NotesTableSeeder
~~~

Write sample records with the query builder:

~~~php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;

class NotesTableSeeder extends Seeder
{
    public function run(): void
    {
        DB::table('notes')->insert([
            'title' => 'Database basics',
            'body' => 'Migrations describe schema changes.',
            'created_at' => now(),
            'updated_at' => now(),
        ]);
    }
}
~~~

Register it from DatabaseSeeder, then run the seed command:

~~~php
public function run(): void
{
    $this->call(NotesTableSeeder::class);
}
~~~

~~~sh
php artisan db:seed
php artisan migrate:fresh --seed
~~~

The second command rebuilds the database before seeding it. Use it only for a local database that can safely be cleared.

## Keep schema changes safe

Do not edit an old migration after it has run in shared environments. Create a new migration to change the schema so each environment can apply the same sequence. Before removing a column or table, check whether application code still reads or writes it.

Use seeders for stable lookup data and development examples. Do not put real user records or production secrets in a seeder committed to a public repository.

## Key points

- A migration records a versioned database schema change.
- up applies a migration and down reverses it.
- Foreign keys and indexes should express important data rules.
- migrate applies outstanding migrations.
- migrate:fresh removes all tables before rebuilding them.
- Seeders add repeatable starting or sample data.

## Practice questions

1. What does a migration describe?
2. Where are migration files stored?
3. What is the purpose of up and down?
4. Which command applies pending migrations?
5. What does migrate:fresh do?
6. Why should destructive reset commands only target disposable data?
7. What is a seeder used for?
8. Why create a new migration instead of editing one already applied in shared environments?

## Main references

- [Laravel 13 database migrations](https://laravel.com/docs/13.x/migrations)
- [Laravel 13 database seeding](https://laravel.com/docs/13.x/seeding)
- [Laravel 13 database configuration](https://laravel.com/docs/13.x/database)
