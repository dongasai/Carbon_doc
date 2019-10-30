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

## Instantiation
---
There are several different methods available to create a new instance of Carbon. First there is a constructor. It overrides the [parent constructor](http://www.php.net/manual/en/datetime.construct.php) and you are best to read about the first parameter from the PHP manual and understand the date/time string formats it accepts. You'll hopefully find yourself rarely using the constructor but rather relying on the explicit static methods for improved readability.

```php
$carbon = new Carbon();                  // equivalent to Carbon::now()
$carbon = new Carbon('first day of January 2008', 'America/Vancouver');
echo get_class($carbon);                 // 'Carbon\Carbon'
```

You'll notice above that the timezone (2nd) parameter was passed as a string rather than a **\DateTimeZone** instance. All DateTimeZone parameters have been augmented so you can pass a DateTimeZone instance, string or integer offset to GMT and the timezone will be created for you. This is again shown in the next example which also introduces the **now()** function.

```php
$now = Carbon::now(); // will use timezone as set with date_default_timezone_set
// PS: we recommend you to work with UTC as default timezone and only use
// other timezones (such as the user timezone) on display

$nowInLondonTz = Carbon::now(new DateTimeZone('Europe/London'));

// or just pass the timezone as a string
$nowInLondonTz = Carbon::now('Europe/London');
echo $nowInLondonTz->tzName;             // Europe/London
echo "\n";

// or to create a date with a custom fixed timezone offset
$date = Carbon::now('+13:30');
echo $date->tzName;                      // +13:30
echo "\n";

// Get/set minutes offset from UTC
echo $date->utcOffset();                 // 810
echo "\n";

$date->utcOffset(180);

echo $date->tzName;                      // +03:00
echo "\n";
echo $date->utcOffset();                 // 180
```

If you really love your fluid method calls and get frustrated by the extra line or ugly pair of brackets necessary when using the constructor you'll enjoy the **parse** method.

```php
echo (new Carbon('first day of December 2008'))->addWeeks(2);     // 2008-12-15 00:00:00
echo "\n";
echo Carbon::parse('first day of December 2008')->addWeeks(2);    // 2008-12-15 00:00:00
```

The string passed to **Carbon::parse** or to **new Carbon** can represent a relative time (next sunday, tomorrow, first day of next month, last year) or an absolute time (first day of December 2008, 2017-01-06). You can test if a string will produce a relative or absolute date with **Carbon::hasRelativeKeywords()**.

```php
$string = 'first day of next month';
if (strtotime($string) === false) {
    echo "'$string' is not a valid date/time string.";
} elseif (Carbon::hasRelativeKeywords($string)) {
    echo "'$string' is a relative valid date/time string, it will returns different dates depending on the current date.";
} else {
    echo "'$string' is an absolute date/time string, it will always returns the same date.";
}
```
