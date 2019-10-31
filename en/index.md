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

To accompany **now()**, a few other static instantiation helpers exist to create widely known instances. The only thing to really notice here is that **today()**, **tomorrow()** and **yesterday()**, besides behaving as expected, all accept a timezone parameter and each has their time value set to **00:00:00**.

```php
$now = Carbon::now();
echo $now;                               // 2019-10-14 14:49:03
echo "\n";
$today = Carbon::today();
echo $today;                             // 2019-10-14 00:00:00
echo "\n";
$tomorrow = Carbon::tomorrow('Europe/London');
echo $tomorrow;                          // 2019-10-15 00:00:00
echo "\n";
$yesterday = Carbon::yesterday();
echo $yesterday;                         // 2019-10-13 00:00:00
```

The next group of static helpers are the **createXXX()** helpers. Most of the static **create** functions allow you to provide as many or as few arguments as you want and will provide default values for all others. Generally default values are the current date, time or timezone. Higher values will wrap appropriately but invalid values will throw an **InvalidArgumentException** with an informative message. The message is obtained from an [DateTime::getLastErrors()](http://php.net/manual/en/datetime.getlasterrors.php) call.

```php
$year = 2000; $month = 4; $day = 19;
$hour = 20; $minute = 30; $second = 15; $tz = 'Europe/Madrid';
echo Carbon::createFromDate($year, $month, $day, $tz)."\n";
echo Carbon::createMidnightDate($year, $month, $day, $tz)."\n";
echo Carbon::createFromTime($hour, $minute, $second, $tz)."\n";
echo Carbon::createFromTimeString("$hour:$minute:$second", $tz)."\n";
echo Carbon::create($year, $month, $day, $hour, $minute, $second, $tz)."\n";
```

**createFromDate()** will default the time to now. **createFromTime()** will default the date to today. **create()** will default any null parameter to the current respective value. As before, the $tz defaults to the current timezone and otherwise can be a DateTimeZone instance or simply a string timezone value. The only special case is for **create()** that has minimum value as default for missing argument but default on current value when you pass explicitly **null**.

```php
$xmasThisYear = Carbon::createFromDate(null, 12, 25);  // Year defaults to current year
$Y2K = Carbon::create(2000, 1, 1, 0, 0, 0); // equivalent to Carbon::createMidnightDate(2000, 1, 1)
$alsoY2K = Carbon::create(1999, 12, 31, 24);
$noonLondonTz = Carbon::createFromTime(12, 0, 0, 'Europe/London');
$teaTime = Carbon::createFromTimeString('17:00:00', 'Europe/London');

try { Carbon::create(1975, 5, 21, 22, -2, 0); } catch(InvalidArgumentException $x) { echo $x->getMessage(); }
// minute must be between 0 and 99, -2 given

// Be careful, as Carbon::createFromDate() default values to current date, it can trigger overflow:
// For example, if we are the 15th of June 2020, the following will set the date on 15:
Carbon::createFromDate(2019, 4); // 2019-04-15
// If we are the 31th of October, as 31th April does not exist, it overflows to May:
Carbon::createFromDate(2019, 4); // 2019-05-01
// That's why you simply should not use Carbon::createFromDate() with only 2 parameters (1 or 3 are safe, but no 2)
```

Create exceptions occurs on such negative values but not on overflow, to get exceptions on overflow, use **createSafe()**

```php
echo Carbon::create(2000, 1, 35, 13, 0, 0);
// 2000-02-04 13:00:00
echo "\n";

try {
    Carbon::createSafe(2000, 1, 35, 13, 0, 0);
} catch (\Carbon\Exceptions\InvalidDateException $exp) {
    echo $exp->getMessage();
}
// day : 35 is not a valid value.
```

Note 1: 2018-02-29 produces also an exception while 2020-02-29 does not since 2020 is a leap year.

Note 2: **Carbon::createSafe(2014, 3, 30, 1, 30, 0, 'Europe/London')** also produces an exception as this time is in an hour skipped by the daylight saving time.

Note 3: The PHP native API allow consider there is a year _0_ between _-1_ and _1_ even if it doesn't regarding to Gregorian calendar. That's why years lower than 1 will throw an exception using **c0reateSafe**. Check [isValid()](https://carbon.nesbot.com/docs/#doc-method-Carbon-isValid) for year-0 detection.

```php
Carbon::createFromFormat($format, $time, $tz);
```

