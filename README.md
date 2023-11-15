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



## Compressing images
You can upload any WebP, JPEG or PNG image to the Tinify API to compress it. We will automatically detect the type of image and optimise with the TinyPNG or TinyJPG engine accordingly. Compression will start as soon as you upload a file or provide the URL to the image.

You can choose a local file as the source and write it to another file.

```php
$source = Tinify::fromFile("unoptimized.webp")->->toFile("optimized.webp");

```
You can also upload an image from a buffer (a string with binary) and get the compressed image data.

```php
$sourceData = file_get_contents("unoptimized.jpg");
$resultData = Tinify::fromBuffer($sourceData)->toBuffer();
```

You can provide a URL to your image instead of having to upload it.

```php
$source = Tinify::fromUrl("https://tinypng.com/images/panda-happy.png")->toFile("optimized.png");
```






## Resizing images
Use the API to create resized versions of your uploaded images. By letting the API handle resizing you avoid having to write such code yourself and you will only have to upload your image once. The resized images will be optimally compressed with a nice and crisp appearance.

You can also take advantage of intelligent cropping to create thumbnails that focus on the most visually important areas of your image.

Resizing counts as one additional compression. For example, if you upload a single image and retrieve the optimized version plus 2 resized versions this will count as 3 compressions in total.

To resize an image, call the resize method on an image source:

```php
$source = Tinify::fromFile("large.jpg");
$resized = $source->resize(array(
    "method" => "fit",
    "width" => 150,
    "height" => 100
));
$resized->toFile("thumbnail.jpg");
```

If the target dimensions are larger than the original dimensions, the image will not be scaled up. Scaling up is prevented in order to protect the quality of your images.

## Converting images
You can use the API to convert your images to your desired image type. Tinify currently supports converting between WebP, JPEG, and PNG. When you provide more than one image type in your convert request, the smallest version will be returned to you.

Image converting will count as one additional compression.

```php
$source = Tinify::fromFile("panda-sticker.jpg");
$converted = $source->convert(array("type" => ["image/webp","image/png"]));
$extension = $converted->result()->extension();
$converted->toFile("panda-sticker." . $extension);
```

Images with transparency
Images with a transparent background will only be converted to formats that support transparency. In case you want to include formats that do not support transparency (JPEG), one can specify a background color that replaces the transparency using the transform object.

If you wish to convert an image with a transparent background to one with a solid background, specify a background property in the transform object. If this property is provided, the background of a transparent image will be filled.

```php
$source = Tinify::fromFile("panda-sticker.png");
$converted = $source->convert(array("type" => "image/jpeg"))->transform(array("background" => "#000000"));
$converted->toFile("panda-sticker.jpg");
```


# laravel backpack dashboard

If you are using a Laravel Backpack Dashboard, follow these steps


```shell
php artisan make:job TinifyFileJob
php artisan make:job TinifyImageJob
```

```php
<?php
namespace App\Jobs;
use Exception;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Storage;
use mohamed7sameer\LaravelTinify\Facades\Tinify;
use PhpParser\Node\Stmt\Return_;
class TinifyFileJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    /**
    * Create a new job instance.
    */
    public $filename;
    public $filepath;
    public $extension;
    public $disk;
    public function __construct($main,$file,$filename)
    {
        $this->filename = $filename;
        $this->extension = $file->getClientOriginalExtension();
        $this->disk = $main->getDisk();
    }
    /**
     * Execute the job.
     */
    public function handle(): void
    {
        $extensions = ['JPEG','PNG','WebP','JPG'];
        $filename= $this->filename;
        $extension= $this->extension;
        $disk= $this->disk;
        try{
            if (in_array(strtoupper($extension), $extensions)) {
                $image = Storage::disk($disk)->get($filename);
                $resultData = Tinify::fromBuffer($image)->toBuffer();
                if($resultData){
                    Storage::disk($disk)->delete($filename);
                    Storage::disk($disk)->put($filename, $resultData);
                }
            }
        }catch( Exception $e){

        }
    }
}
```


```php
<?php
namespace App\Jobs;
use Exception;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Storage;
use mohamed7sameer\LaravelTinify\Facades\Tinify;
class TinifyImageJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    /**
    * Create a new job instance.
    */
    public $finalPath ;
    public $disk ;
    public function __construct($finalPath,$disk)
    {
        $this->finalPath = $finalPath;
        $this->disk = $disk;
    }
    /**
     * Execute the job.
     */
    public function handle(): void
    {
        try{
            $image = Storage::disk($this->disk)->get($this->finalPath);
            $resultData = Tinify::fromBuffer($image)->toBuffer();
            if($resultData){
                Storage::disk($this->disk)->delete($this->finalPath);
                Storage::disk($this->disk)->put($this->finalPath, $resultData);
            }
        }catch( Exception $e){
        }
    }
}
```

```php
# QUEUE_CONNECTION=sync
QUEUE_CONNECTION=database
```

```shell
php artisan queue:work
```



```php
// config/backpack/crud.php

// 'uploaders' => [
//     'withFiles' => [
//         'image'           => \Backpack\CRUD\app\Library\Uploaders\SingleBase64Image::class,
//         'upload'          => \Backpack\CRUD\app\Library\Uploaders\SingleFile::class,
//         'upload_multiple' => \Backpack\CRUD\app\Library\Uploaders\MultipleFiles::class,
//     ],
// ],
'uploaders' => [
    'withFiles' => [
        'image'           => \mohamed7sameer\LaravelTinify\Backpack\Uploaders\SingleBase64Image::class,
        'upload'          => \mohamed7sameer\LaravelTinify\Backpack\Uploaders\SingleFile::class,
        'upload_multiple' => \mohamed7sameer\LaravelTinify\Backpack\Uploaders\MultipleFiles::class,
    ],
],
```