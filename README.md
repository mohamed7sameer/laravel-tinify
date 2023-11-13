# laravel-tinify

This package provides integration with the Tinify a.k.a TinyPNG API.

The package simply provides a Tinify facade that acts as a wrapper to the [tinify/tinfiy-php](https://github.com/tinify/tinify-php)



## Installation

Install the package via Composer:

```
   composer require mohamed7sameer/laravel-tinify
```

### Laravel 5.5+:

If you don't use auto-discovery, add the ServiceProvider to the providers array in ```config/app.php```


```php
    ...
    mohamed7sameer\LaravelTinify\LaravelTinifyServiceProvider::class
    ...
```

Add alias to ```config/app.php```:

```php
    ...
    'Tinify' => mohamed7sameer\LaravelTinify\Facades\Tinify::class
    ...
```

## Configuration
Publish the Configuration for the package which will create the config file tinify.php inside config directory

`php artisan vendor:publish --provider="mohamed7sameer\LaravelTinify\LaravelTinifyServiceProvider"`

Set a env variable "TINIFY_APIKEY" with your issued apikey or set api_key into config/tinify.php 

This package is available under the [MIT license](http://opensource.org/licenses/MIT).
