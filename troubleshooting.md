# Troubleshooting
This is a compilation of common problems and their respective solutions and/or workarounds.

## Auditing not working
You followed the documentation to the letter and yet, audit records aren't being created?

This package relies on Eloquent [events](https://laravel.com/docs/5.5/eloquent#events) and if they don't fire, an `Audit` cannot be created.

The most common mistake is doing an `update` or `delete` using a `Builder` instance, rather than an using an Eloquent `Model` instance.

Usually, something like this happens:
```php
Article::where('id', $id)->update($data);
```

When in fact, it should be:
```php
Article::find($id)->update($data);
```

## Error: Call to undefined method Closure::__set_state()
This error happens when Laravel tries to cache the configuration and you're using a `Closure` as a `User` resolver in the `config/audits.php` file.

Support for `Closure` and `callable` resolvers has been removed from version **5.0.0** onward.

Read the `User` resolver section in the [General Configuration](general-configuration) for more information.

## Attributes are considered modified, when they're not
False positives cause Audit records to be created.
This happens when model attributes are updated with **BOOLEAN** and/or **DATE** values.

The internal data of the Eloquent model will be as follows:

In the **$attributes** array attribute:
- `true` stays `true`
- `false` stays `false`
- `YYYY-MM-DD` stays `YYYY-MM-DD`

In the **$original** array attribute:
- `true` becomes `1`
- `false` becomes `0`
- `YYYY-MM-DD` becomes `YYYY-MM-DD 00:00:00`

This makes the `getDirty()` and `isDirty()` methods to consider wrongful attribute changes when comparing data.

According to a [maintainer](https://github.com/laravel/framework/issues/16823#issuecomment-267573840), this is an **expected behaviour**.

As a workaround, consider passing `1` and `0`, instead of `true` and `false`. For date values, append ` 00:00:00`.

At the time of writing, [this](https://github.com/laravel/internals/issues/349) is the only open issue about this subject.

## Argument 1 passed to Illuminate\Database\Eloquent\Model::serializeDate() must implement interface DateTimeInterface, null given
This might happen in version **4.1.x**, because the `updated_at` column values in the `audits` table are set to `NULL`.

After upgrading the table schema for **4.1.x**, don't forget to set the `updated_at` values to match the ones from `created_at`.

Read the [Upgrading](upgrading) documentation for more details.
