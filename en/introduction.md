##  Introduction

The Carbon class is [inherited](http://php.net/manual/en/keyword.extends.php) from the PHP [DateTime](http://www.php.net/manual/en/class.datetime.php) class.

```php
<?php

namespace Carbon;

class Carbon extends \DateTime
{
    // code here
}

```

You can see from the code snippet above that the Carbon class is declared in the Carbon namespace. You need to import the namespace to use Carbon without having to provide its fully qualified name each time.

```php
use Carbon\Carbon;
```

Examples in this documentation will assume you imported classes of the Carbon namespace this way.

We also provide CarbonImmutable class extending [DateTimeImmutable](http://www.php.net/manual/en/class.datetimeimmutable.php). The same methods are available on both classes but when you use a modifier on a Carbon instance, it modifies and returns the same instance, when you use it on CarbonImmutable, it returns a new instances with the new value.

```php
$mutable = Carbon::now();
$immutable = CarbonImmutable::now();
$modifiedMutable = $mutable->add(1, 'day');
$modifiedImmutable = CarbonImmutable::now()->add(1, 'day');

var_dump($modifiedMutable === $mutable);             // bool(true)
var_dump($mutable->isoFormat('dddd D'));             // string(10) "Tuesday 15"
var_dump($modifiedMutable->isoFormat('dddd D'));     // string(10) "Tuesday 15"
// So it means $mutable and $modifiedMutable are the same object
// both set to now + 1 day.
var_dump($modifiedImmutable === $immutable);         // bool(false)
var_dump($immutable->isoFormat('dddd D'));           // string(9) "Monday 14"
var_dump($modifiedImmutable->isoFormat('dddd D'));   // string(10) "Tuesday 15"
// While $immutable is still set to now and cannot be changed and
// $modifiedImmutable is a new instance created from $immutable
// set to now + 1 day.

$mutable = CarbonImmutable::now()->toMutable();
var_dump($mutable->isMutable());                     // bool(true)
var_dump($mutable->isImmutable());                   // bool(false)
$immutable = Carbon::now()->toImmutable();
var_dump($immutable->isMutable());                   // bool(false)
var_dump($immutable->isImmutable());                 // bool(true)
```

The library also provides CarbonInterface interface extends [DateTimeImmutable](http://www.php.net/manual/en/class.datetimeimmutable.php) and [JsonSerializable](http://www.php.net/manual/en/class.jsonserializable.php), [CarbonInterval](https://carbon.nesbot.com/docs/#api-interval) class extends [DateInterval](http://www.php.net/manual/en/class.dateinterval.php), [CarbonTimeZone](https://carbon.nesbot.com/docs/#api-timezone) class extends [DateTimeZone](http://www.php.net/manual/en/class.datetimezone.php) and [CarbonPeriod](https://carbon.nesbot.com/docs/#api-period) class polyfills [DatePeriod](http://www.php.net/manual/en/class.dateperiod.php).

Carbon has all of the functions inherited from the base DateTime class. This approach allows you to access the base functionality such as [modify](http://php.net/manual/en/datetime.modify.php), [format](http://php.net/manual/en/datetime.format.php) or [diff](http://php.net/manual/en/datetime.diff.php).

Now, let see how cool is this doc. Click on the code below:

```php
$dtToronto = Carbon::create(2012, 1, 1, 0, 0, 0, 'America/Toronto');
$dtVancouver = Carbon::create(2012, 1, 1, 0, 0, 0, 'America/Vancouver');
// Try to replace the 4th number (hours) or the last argument (timezone) with
// Europe/Paris for example and see the actual result on the right hand.
// It's alive!

echo $dtVancouver->diffInHours($dtToronto); // 3
// Now, try to double-click on "diffInHours" or "create" to open
// the References panel.
// Once the references panel is open, you can use the search field to
// filter the list or click the (<) button to close it.
```

Some examples are static snippets, some other are editable (when there is a top right hand corner expand button). You can also click on this button to open the snippet in a new tab. You can double-click on methods name in both static and dynamic examples.