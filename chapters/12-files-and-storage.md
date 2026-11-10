# 12. Files and storage

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Build JSON APIs](./11-build-json-apis.md) | [Notes index](../README.md) | [Next: Queues, events, and scheduled jobs](./13-queues-events-and-scheduled-jobs.md) |

Laravel’s filesystem layer gives an application one API for local files and remote storage disks. Validate uploads before storing them, and decide whether each file is public or private.

## Validate an uploaded file

A form must use <code>enctype="multipart/form-data"</code> so the browser sends its file. Validate the file’s type and size on the server; client-side checks are only a usability aid.

~~~php
$validated = $request->validate([
    'avatar' => ['required', 'image', 'max:2048'],
]);
~~~

The <code>image</code> rule accepts common image formats. A size value is measured in kilobytes. Restrict accepted formats further if the application only supports a particular set.

## Store the upload on a disk

Use the uploaded file’s <code>store</code> method to generate a safe path and save it to a configured disk.

~~~php
$path = $request->file('avatar')->store('avatars', 'public');

$request->user()->update([
    'avatar_path' => $path,
]);
~~~

Store the returned path in the database, not the entire file contents. In a production system, use a disk that persists across deployments and application instances.

For local public storage, run the storage link command once for the environment:

~~~sh
php artisan storage:link
~~~

A public disk file can then be given a URL:

~~~php
$url = Storage::disk('public')->url($path);
~~~

## Keep private files private

Sensitive documents should not be placed on a public disk. Store them on a private disk and authorize each download.

~~~php
public function download(Report $report)
{
    $this->authorize('view', $report);

    return Storage::disk('private')->download($report->file_path);
}
~~~

A file URL is not a substitute for authorization. For private cloud objects, use a short-lived temporary URL only after checking that the requesting user may access the object.

## Replace and remove files safely

When replacing an avatar, save the new file and update the database, then remove the old file if the operation succeeds. Consider what should happen if either storage or the database write fails. Avoid allowing an untrusted client to choose an arbitrary storage path.

~~~php
if ($oldPath && Storage::disk('public')->exists($oldPath)) {
    Storage::disk('public')->delete($oldPath);
}
~~~

Check the disk configuration and backup strategy before using local storage in production. Many deployment platforms use temporary filesystems that are cleared during redeploys.

## Key points

- Laravel disks provide a consistent interface for file storage.
- Validate upload type and size on the server.
- Store generated paths and disk names rather than file bytes in a database row.
- Public and private files need different access patterns.
- Authorize downloads of private files on the server.
- Confirm production storage persists and is backed up.

## Practice questions

1. Why is browser-side file validation insufficient?
2. What encoding does an HTML form need for file uploads?
3. What does the store method return?
4. Why save a path rather than file bytes in a database?
5. What is the difference between a public and private disk?
6. Why must a private download route authorize the user?
7. What can happen to local files on some deployment platforms?
8. What should an app consider when replacing an existing file?

## Main references

- [Laravel 13 filesystem](https://laravel.com/docs/13.x/filesystem)
- [Laravel 13 validation](https://laravel.com/docs/13.x/validation)
- [Laravel 13 authorization](https://laravel.com/docs/13.x/authorization)
