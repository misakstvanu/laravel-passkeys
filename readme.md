> [!WARNING]  
> This project is currently under development

# Laravel Passkeys
This package provides a simple way to authenticate users using passkeys. 

Authentication processes are based on `web-auth/webauthn-lib` package. On frontend, the opposite functionality is provided by `@simplewebauthn/browser` package.

## Installation

1. Install the package via composer:

``` bash
composer require misakstvanu/laravel-passkeys
```

2. Service provider will be auto discovered. If you want to register it manually, add the following line to
   your `config/app.php`

``` php
'providers' => [
    /*
     * Package Service Providers...
     */
    // ...
    Misakstvanu\LaravelPasskeys\PasskeysServiceProvider::class,
];
```

3. Publish migration to create `passkeys` table:

``` bash
php artisan vendor:publish --tag=laravel-passkeys-migrations
php artisan migrate
```

4. (optional) Publish the config file:

``` bash
php artisan vendor:publish --tag=laravel-passkeys-config
```

## Configuration

1. Implement an interface `Misakstvanu\LaravelPasskeys\Contracts\PasskeyAuthentication` on your `User` model:
``` php
use Misakstvanu\LaravelPasskeys\Contracts\PasskeyAuthentication;

class User extends Authenticatable implements PasskeyAuthentication {
    // ...
}
```

2. Set up `passkeys` relation on your `User` model:
``` php
use Misakstvanu\LaravelPasskeys\Models\Passkey;

public function passkeys() :HasMany {
    return $this->hasMany(Passkey::class);
}
```

3. Once you have published the config file, you can configure the package by editing the `config/passkeys.php` file. The variables are:

- `user_model` - the model that will be used to authenticate the user. Default: `App\Models\User`
- `route_prefix` - prefix for the 4 routes this package loads. Default: `passkeys`
- `route_middleware` - middleware that will be applied to the routes. Default: `['web']`
- `username_column` - the column that will be used to find the user. Default: `email`
- `relying_party_ids` - an array of domains that will be allowed insecure connection, use with caution. Default: `[]`
- `registration_user_validation` - validation rules that will be applied to the request when registering new user. These values will then be persisted with the new user. Default: `[]`


## Usage

There are 4 named routes that make everything work:

`POST 'passkeys.login.start'` - login route, accepts `email` or other field specified in your config. If a user with the given username/email exists and has a passkey registered, credential request options will be returned. If the user does not exist, HTTP 404 will be returned instead.

`POST 'passkeys.login.verify'` - login route, accepts passkey response. If the passkey authentication passes, the user will be logged in. If the passkey authentication fails, an exception with additional information is thrown.

`POST 'passkeys.register.start'` - registration route, accepts `email` or other field specified in your config. Credential request options is returned.

`POST 'passkeys.register.verify'` - registration route, accepts passkey response. If the passkey registration passes and an user is currently logged in, the passkey will be added to the existing account, if no one is currently logged in, an account will be created from the username/email and any additional data specified in config and sent along with this request. If the passkey registration fails, an exception with additional information is thrown.
