# Vault (Laravel 5 Package)

Vault is a simple yet powerful access control system for the new Laravel 5 Framework. It comes with a backend user interface to manage users, roles, and permissions as well as the relationships between them.

**Examples:**
[Vault User Index](http://i.imgur.com/yZ80ySY.png)
[Vault Create Role](http://i.imgur.com/R4yE7nF.png)
[Vault Edit User](http://i.imgur.com/5ZIRcGV.png)
[Vault Role Index](http://i.imgur.com/zmfGeEr.png)

## Prerequisite

- This package assumes you have an installation of Laravel 5 using the pre-packaged authentication library and functionality. For a brand new project, I recommend using my [Laravel 5 Boilerplate Package](https://github.com/rappasoft/Laravel-5-Boilerplate) and requiring this library.
- User model must have soft deletes enabled.

## Setup

In the `require` key of `composer.json` file add the following

    "rappasoft/vault": "dev-master"
    
Run the Composer update command

    $ composer update

In your `config/app.php` add `'Rappasoft\Vault\VaultServiceProvider'` to the end of the `$providers` array

```php
'providers' => array(

    'App\Providers\EventServiceProvider',
    'App\Providers\RouteServiceProvider',
    ...
    'Rappasoft\Vault\VaultServiceProvider',

),
```

**The Vault Facade is loaded by the service provider by default.**

Run the `vendor:publish` command

    $ php artisan vendor:publish

This will publish the following files to your application:

- app/config/vault.php config file
- Vault Migration File
- Vault Seed File (Will add the seed call to the end of your DatabaseSeeder.php class)
- public/js/vault/*
- public/css/vault/*

Run the `migration` command

    $ php artisan migrate
        
Run the `seed` command

    $ php artisan db:seed --class="VaultTableSeeder"
    
Add the `UserHasRole` trait to your User model:

```php
<?php namespace App;

...
use Illuminate\Database\Eloquent\SoftDeletes;
use Rappasoft\Vault\Traits\UserHasRole;

class User extends Model implements AuthenticatableContract, CanResetPasswordContract {

	use Authenticatable, CanResetPassword, SoftDeletes, UserHasRole;
}
```

Add the `route middleware` to your app/Http/Kernel.php file:

```php
protected $routeMiddleware = [
    'auth' => 'App\Http\Middleware\Authenticate',
    'auth.basic' => 'Illuminate\Auth\Middleware\AuthenticateWithBasicAuth',
    'guest' => 'App\Http\Middleware\RedirectIfAuthenticated',
    ...
    'vault.routeNeedsRole' => 'Rappasoft\Vault\Http\Middleware\RouteNeedsRole',
    'vault.routeNeedsPermission' => 'Rappasoft\Vault\Http\Middleware\RouteNeedsPermission',
    'vault.routeNeedsRoleOrPermission' => 'Rappasoft\Vault\Http\Middleware\RouteNeedsRoleOrPermission',
];
```

###That's it! You should now be able to navigate to http://YOUR_PROJECT/access/users to see the users index.
    
## Configuration

Look through `app/config/vault.php` for all package options.

By default the package works without publishing its views. But if you wanted to publish the vault views to your application to take full control, run the vault:views command:

    $ php artisan vault:views
    
If you do not want vault to use its default routes file you can duplicate it and set the `vault.general.use_vault_routes` configuration to false and it will not load by default.
    
### Utilizing the `status` property

If would would like to enable enabled/disabled users you can simply do a check wherever you are logging in your user:

```php
if ($user->status == 0)
    return Redirect::back()->withMessage("Your account is currently disabled");
```

## Applying the Route Middleware

Laravel 5 is trying to steer away from the filters.php file and more towards using middleware. Here is an example right from the vault routes file of a group of routes that requires the Administrator role:

```php
Route::group([
	'middleware' => 'vault.routeNeedsRole',
	'role' => ['Administrator'],
	'redirect' => '/',
	'with' => ['error', 'You do not have access to do that.']
], function()
{
    Route::group(['prefix' => 'access'], function ()
    	{
    		/*User Management*/
    		Route::resource('users', '\Rappasoft\Vault\Http\Controllers\UserController', ['except' => ['show']]);
    	});
});
```

The above code checks to see if the currently authenticated user has the role `Administrator`, if not redirects to `/` with a session variable that has a key of `message` and value of `You do not have access to do that.`

The following middleware ships with the vault package:

- vault.routeNeedsRole
- vault.routeNeedsPermission
- vault.routeNeedsRoleOrPermission

## Route Parameters

- `middleware` => The middleware name, you can change them in your app/Http/Kernel.php file.
- `role` => A string of one role or an array of roles by name.
- `permission` => A string of one permission or an array of permissions by name.
- `needsAll` => A boolean, false by default, that states whether or not all of the specified roles/permissions are required to authenticate.
- `with` => Sends a session flash on failure. Array with 2 items, first is session key, second is value.
- `redirect` => Redirect to a url if authentication fails.
- `redirectRoute` => Redirect to a route if authentication fails.
- `redirectAction` => Redirect to an action if authentication fails.

**If no redirect is specified a `response('Unauthorized', 401);` will be thrown.**

## License

Vault is free software distributed under the terms of the MIT license.

## Additional information

Any issues, please [report here](https://github.com/rappasoft/vault/issues).