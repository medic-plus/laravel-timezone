# Laravel Timezone

[![Latest Version on Packagist](https://img.shields.io/packagist/v/medicplus/laravel-timezone.svg?style=flat-square)](https://packagist.org/packages/medicplus/laravel-timezone)
[![GitHub Tests Action Status](https://img.shields.io/github/actions/workflow/status/medicplus/laravel-timezone/run-tests.yml?branch=main&label=tests&style=flat-square)](https://github.com/medicplus/laravel-timezone/actions?query=workflow%3Arun-tests+branch%3Amain)
[![GitHub Code Style Action Status](https://img.shields.io/github/actions/workflow/status/medicplus/laravel-timezone/fix-php-code-style-issues.yml?branch=main&label=code%20style&style=flat-square)](https://github.com/medicplus/laravel-timezone/actions?query=workflow%3A"Fix+PHP+code+style+issues"+branch%3Amain)
[![Total Downloads](https://img.shields.io/packagist/dt/medicplus/laravel-timezone.svg?style=flat-square)](https://packagist.org/packages/medicplus/laravel-timezone)

An easy way to set a timezone for a user in your application and then show date/times to them in their local timezone.

This is a reimplementation of [jamesmills/laravel-timezone](https://github.dev/jamesmills/laravel-timezone) and the cast feature from [amandiobm/laravel-timezone](https://github.com/amandiobm/laravel-timezone/tree/feature-casts)

## Requirements

Supported Laravel versions 9.x to 13.x

## How does it work

This package listens for the `\Illuminate\Auth\Events\Login` event and will then automatically set a `timezone` on your `user` model (stored in the database).

This package uses the [torann/geoip](http://lyften.com/projects/laravel-geoip/doc/) package which looks up the users location based on their IP address. The package also returns information like the users currency and users timezone. [You can configure this package separately if you require](#custom-configuration).

## How to use

You can show dates to your user in their timezone by using

```php
{{ Timezone::convertToLocal($post->created_at) }}
```

Or use our nice blade directive

```php
@displayDate($post->created_at)
```

## Installation

You can install the package via composer:

```bash
composer require medicplus/laravel-timezone
```

You can publish and run the migrations with:

```bash
php artisan vendor:publish --provider="MedicPlus\LaravelTimezone\LaravelTimezoneServiceProvider" --tag=migrations
php artisan migrate
```

This will add a `timezone` (can be changed in the configuration file) column to your `users` table.

## Examples

### Showing date/time to the user in their timezone

Default will use the format `jS F Y g:i:a` and will not show the timezone

```php
{{ Timezone::convertToLocal($post->created_at) }}

// 4th July 2018 3:32:am
```

If you wish you can set a custom format and also include a nice version of the timezone

```php
{{ Timezone::convertToLocal($post->created_at, 'Y-m-d g:i', true) }}

// 2018-07-04 3:32 New York, America
```

### Using blade directive

Making your life easier one small step at a time

```php
@displayDate($post->created_at)

// 4th July 2018 3:32:am
```

And with custom formatting

```php
@displayDate($post->created_at, 'Y-m-d g:i', true)

// 2018-07-04 3:32 New York, America
```

### Using models casting class

You can use the casting class for your models columns, this will let you save the dates in UTC format and then use the attribute accessor to get them in the user timezone.

```php
<?php
namespace App;
use Illuminate\Database\Eloquent\Model;
use MedicPlus\LaravelTimezone\Casts\Timezone;
class Foo extends Model
{
    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = [
        'created_at' => Timezone::class,
    ];
}
```

### Saving the users input to the database in UTC

This will take a date/time, set it to the users timezone then return it as UTC in a Carbon instance.

```php
$post = Post::create([
    'publish_at' => Timezone::convertFromLocal($request->get('publish_at')),
    'description' => $request->input('description'),
]);
```

## Custom Configuration

Publishing the config file is optional.

```php
php artisan vendor:publish --provider="MedicPlus\LaravelTimezone\LaravelTimezoneServiceProvider" --tag=config
```

### Flash Messages

When the timezone has been set, we display a flash message, By default, is configured to use Laravel default flash messaging, here are some of the optional integrations.

[laracasts/flash](https://github.com/laracasts/flash) - `'flash' => 'laracasts'`

[mercuryseries/flashy](https://github.com/mercuryseries/flashy) - `'flash' => 'mercuryseries'`

[spatie/laravel-flash](https://github.com/spatie/laravel-flash) - `'flash' => 'spatie'`

[mckenziearts/laravel-notify](https://github.com/mckenziearts/laravel-notify) - `'flash' => 'mckenziearts'`

[usernotnull/tall-toasts](https://github.com/usernotnull/tall-toasts) - `'flash' => 'tall-toasts'`

To override this configuration, you just need to change the `flash` property inside the configuration file `config/timezone.php` for the desired package. You can disable flash messages by setting `'flash' => 'off'`.

### Overwrite existing timezones in the database

By default, the timezone will be overwritten at each login with the current user timezone. This behavior can be restricted to only update the timezone if it is blank by setting the `'overwrite' => false,` config option.

### Default Format

By default, the date format will be `jS F Y g:i:a`. To override this configuration, you just need to change the `format` property inside the configuration file `config/timezone.php` for the desired format.

### Column Name

By default, this package uses the column `timezone` in the users model, if you want to use a different column name you can change the `column_name` property inside the configuration file `config/timezone.php`. If you already executed the migrations, make sure to rollback the changes before applying this change.

### Lookup Array

This lookup array configuration makes it possible to find the remote address of the user in any attribute inside the Laravel `request` helper, by any key. Having in mind when the key is found inside the attribute, that key will be used. By default, we use the `server` attribute with the key `REMOTE_ADDR`. To override this configuration, you just need to change the `lookup` property inside the configuration file `config/timezone.php` for the desired lookup.

### User Message

You may configure the message shown to the user when the timezone is set by changing the `message` property inside the configuration file `config/timezone.php`

### Underlying GeoIp Package

If you wish to customise the underlying `torann/geoip` package you can publish the config file by using the command below.

```php
php artisan vendor:publish --provider="Torann\GeoIP\GeoIPServiceProvider" --tag=config
```

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Security Vulnerabilities

Please review [our security policy](../../security/policy) on how to report security vulnerabilities.

## Credits

- [James Mills](https://github.com/jamesmills)
- [Amandio Magalhães](https://github.com/amandiobm)
- [Ivan Vasquez](https://github.com/ivanvasquez)
- [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
