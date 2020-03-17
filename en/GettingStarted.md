# Getting Started
* 1.x is compatible with PHP 5.3+.
* 2.x version requires PHP 7.1.8+.

# Installing

The easiest and recommended method to install Carbon is via [composer](http://getcomposer.org/).

## [With composer](#withcomposer) [Direct download](#directdownload)

<a name="withcomposer">With composer</a>

Use the following command to install with composer.

`composer require nesbot/carbon`

This will automatically get the latest version and configure a `composer.json` file.

```php
<?php
require 'vendor/autoload.php';

use Carbon\Carbon;

printf("Now: %s", Carbon::now());
```

If you wish you can create the following composer.json file and run composer install to install it.
```
{
    "require": {
        "nesbot/carbon": "^2.31.0"
    }
}
```
Carbon 2 is officially supported by Laravel since the version 5.8, if you want to use it on a lower version, you can follow those steps:

Set explicitly the Carbon version and add the adapter in your composer.json:
```php

{
    "require": {
        "nesbot/carbon": "2.31.0 as 1.39.0"
        "kylekatarnls/laravel-carbon-2": "^1.0.0"
    }
}
```

Use 1.25.0 alias for Laravel 5.6, 1.39.0 for other versions as each version of Laravel has its own range of Carbon compatibility, you have to take 2.31.0 as the real used version, then pick an alias among versions Laravel supports. You may have other dependencies with other Carbon restrictions. If so, we cannot ensure Carbon 2 will works with them as Carbon 1 did, but you can still try an other version alias. To be sure an alias will be compatible with all of your current dependencies, you can choose the version of Carbon you have before upgrading to Carbon 2 using `composer show nesbot/carbon`.

Then update your dependencies by running.

```shell
$ composer update
```


### Learn More

Looks good so far. What do I do next? Read the [API docs](https://carbon.nesbot.com/docs).

## Author
Brian Nesbitt

http://nesbot.com

https://twitter.com/NesbittBrian

## Maintainer

kylekatarnls

https://github.com/kylekatarnls

## Contributors
![contributors](https://opencollective.com/Carbon/contributors.svg?width=890&button=false)
