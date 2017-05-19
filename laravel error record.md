
## Whoops, looks like something went wrong.
打开项目目录下config/app.php修改：'debug' => env('APP_DEBUG', true),原本为'debug' => env('APP_DEBUG', false),

## The only supported ciphers are AES-128-CBC and AES-256-CBC with the correct key lengths.
php artisan key:generate 

