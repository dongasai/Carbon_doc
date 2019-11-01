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

**createFromFormat()** is mostly a wrapper for the base php function [DateTime::createFromFormat](http://php.net/manual/en/datetime.createfromformat.php). The difference being again the *$tz* argument can be a DateTimeZone instance or a string timezone value. Also, if there are errors with the format this function will call the **DateTime::getLastErrors()** method and then throw a **InvalidArgumentException** with the errors as the message.

```php
echo Carbon::createFromFormat('Y-m-d H', '1975-05-21 22')->toDateTimeString(); // 1975-05-21 22:00:00
```

The final three create functions are for working with [unix timestamps](http://en.wikipedia.org/wiki/Unix_time). The first will create a Carbon instance equal to the given timestamp and will set the timezone as well or default it to the current timezone. The second, **createFromTimestampUTC()**, is different in that the timezone will remain UTC (GMT), it acts the same as `Carbon::parse('@'.$timestamp)` but I have just made it a little more explicit. The third, `createFromTimestampMs()`, accepts a timestamp in milliseconds instead of seconds. Negative timestamps are also allowed.

```php
echo Carbon::createFromTimestamp(-1)->toDateTimeString();                                  // 1969-12-31 23:59:59
echo Carbon::createFromTimestamp(-1, 'Europe/London')->toDateTimeString();                 // 1970-01-01 00:59:59
echo Carbon::createFromTimestampUTC(-1)->toDateTimeString();                               // 1969-12-31 23:59:59
echo Carbon::createFromTimestampMs(1)->format('Y-m-d\TH:i:s.uP T');                        // 1970-01-01T00:00:00.001000+00:00 UTC
echo Carbon::createFromTimestampMs(1, 'Europe/London')->format('Y-m-d\TH:i:s.uP T');       // 1970-01-01T01:00:00.001000+01:00 BST
```
You can also create a `copy()` of an existing Carbon instance. As expected the date, time and timezone values are all copied to the new instance.

```php
$dt = Carbon::now();
echo $dt->diffInYears($dt->copy()->addYear());  // 1

// $dt was unchanged and still holds the value of Carbon:now()
```

You can use `nowWithSameTz()` on an existing Carbon instance to get a new instance at now in the same timezone.

```php
$meeting = Carbon::createFromTime(19, 15, 00, 'Africa/Johannesburg');

// 19:15 in Johannesburg
echo 'Meeting starts at '.$meeting->format('H:i').' in Johannesburg.';                  // Meeting starts at 19:15 in Johannesburg.
// now in Johannesburg
echo "It's ".$meeting->nowWithSameTz()->format('H:i').' right now in Johannesburg.';    // It's 16:49 right now in Johannesburg.
```

Finally, if you find yourself inheriting a `\DateTime` instance from another library, fear not! You can create a `Carbon` instance via a friendly `instance()` method. Or use the even more flexible method `make()` which can return a new Carbon instance from a DateTime, Carbon or from a string, else it just returns null.

```php
$dt = new \DateTime('first day of January 2008'); // <== instance from another API
$carbon = Carbon::instance($dt);
echo get_class($carbon);                               // 'Carbon\Carbon'
echo $carbon->toDateTimeString();                      // 2008-01-01 00:00:00
```

Carbon 2 (requiring PHP >= 7.1) perfectly supports microseconds. But if you use Carbon 1 and PHP < 7.1, read our [section about partial microseconds support](https://carbon.nesbot.com/docs/#partial-microseconds-support-v1).

Ever need to loop through some dates to find the earliest or latest date? Didn't know what to set your initial maximum/minimum values to? There are now two helpers for this to make your decision simple:

```php
echo Carbon::maxValue();                               // '9999-12-31 23:59:59'
echo Carbon::minValue();                               // '0001-01-01 00:00:00'
```

Min and max value mainly depends on the system (32 or 64 bit).

With a 32-bit OS system or 32-bit version of PHP (you can check it in PHP with **PHP_INT_SIZE === 4**), the minimum value is the 0-unix-timestamp (1970-01-01 00:00:00) and the maximum is the timestamp given by the constant **PHP_INT_MAX**.

With a 64-bit OS system and 64-bit version of PHP, the minimum is 01-01-01 00:00:00 and maximum is 9999-12-31 23:59:59. It's even possible to use negative year up to -9999 but be aware you may not have accurate results with some operations as the year 0 exists in PHP but not in the Gregorian calendar.

## Localization
---

With Carbon 2, localization changed a lot, 751 new locales are supported and we now embed locale formats, day names, month names, ordinal suffixes, meridiem, week start and more. While Carbon 1 provided partial support and relied on third-party like IntlDateFormatter class and language packages for advanced translation, you now benefit of a wide internationalization support. You still use Carbon 1? I hope you would consider to upgrade, version 2 has really cool new features. Else you still read the [version 1 documentation of Localization by clicking here](https://carbon.nesbot.com/docs/#localization-v1).

Since 2.9.0, you can easily customize translations:

```php
// we recommend to use custom language name/variant
// rather than overriding an existing language
// to avoid conflict such as "en_Boring" in the example below:
$boringLanguage = 'en_Boring';
$translator = \Carbon\Translator::get($boringLanguage);
$translator->setTranslations([
    'day' => ':count boring day|:count boring days',
]);
// as this language starts with "en_" it will inherit from the locale "en"

$date1 = Carbon::create(2018, 1, 1, 0, 0, 0);
$date2 = Carbon::create(2018, 1, 4, 4, 0, 0);

echo $date1->locale($boringLanguage)->diffForHumans($date2); // 3 boring days before

$translator->setTranslations([
    'before' => function ($time) {
        return '['.strtoupper($time).']';
    },
]);

echo $date1->locale($boringLanguage)->diffForHumans($date2); // [3 BORING DAYS]
```

You can use fallback locales by passing in order multiple ones to `locale()`:

```php
\Carbon\Translator::get('xx')->setTranslations([
    'day' => ':count Xday',
]);
\Carbon\Translator::get('xy')->setTranslations([
    'day' => ':count Yday',
    'hour' => ':count Yhour',
]);

$date = Carbon::now()->locale('xx', 'xy', 'es')->sub('3 days 6 hours 40 minutes');

echo $date->ago(['parts' => 3]); // hace 3 Xday 6 Yhour 40 minutos
```

In the example above, it will try to find translations in "xx" in priority, then in "xy" if missing, then in "es", so here, you get "Xday" from "xx", "Yday" from "xy", and "hace" and "minutos" from "es".

Note that you can also use an other translator with `Carbon::setTranslator($custom)` as long as the given translator implements `Symfony\Component\Translation\TranslatorInterface`. And you can get the global default translator using `Carbon::getTranslator()` (and `Carbon::setFallbackLocale($custom)` and `Carbon::getFallbackLocale()` for the fallback locale, setFallbackLocale can be called multiple times to get multiple fallback locales) but as those method will change the behavior globally (including third-party libraries you may have in your app), it might cause unexpected results. You should rather customize translation using custom locales as in the example above.

Carbon embed a default translator that extends [Symfony\Component\Translation\Translator](https://api.symfony.com/2.7/Symfony/Component/Translation/Translator.html) You can [check here the methods we added to it](https://carbon.nesbot.com/docs/#translator-details).

So the support of a locale for `formatLocalized`, getters such as `localeMonth`, `localeDayOfWeek` and short variants is driven by locales installed in your operating system. For other translations, it's supported internally thanks to Carbon community. You can check what's supported with the following methods:

```php
echo implode(', ', array_slice(Carbon::getAvailableLocales(), 0, 3)).'...';      // aa, aa_DJ, aa_ER...

// Support diff syntax (before, after, from now, ago)
var_dump(Carbon::localeHasDiffSyntax('en'));                                     // bool(true)
var_dump(Carbon::localeHasDiffSyntax('zh_TW'));                                  // bool(true)
// Support 1-day diff words (just now, yesterday, tomorrow)
var_dump(Carbon::localeHasDiffOneDayWords('en'));                                // bool(true)
var_dump(Carbon::localeHasDiffOneDayWords('zh_TW'));                             // bool(false)
// Support 2-days diff words (before yesterday, after tomorrow)
var_dump(Carbon::localeHasDiffTwoDayWords('en'));                                // bool(true)
var_dump(Carbon::localeHasDiffTwoDayWords('zh_TW'));                             // bool(false)
// Support short units (1y = 1 year, 1mo = 1 month, etc.)
var_dump(Carbon::localeHasShortUnits('en'));                                     // bool(true)
var_dump(Carbon::localeHasShortUnits('zh_TW'));                                  // bool(false)
// Support period syntax (X times, every X, from X, to X)
var_dump(Carbon::localeHasPeriodSyntax('en'));                                   // bool(true)
var_dump(Carbon::localeHasPeriodSyntax('zh_TW'));                                // bool(false)
```

So, here is the new recommended way to handle internationalization with Carbon.

```php
$date = Carbon::now()->locale('fr_FR');

echo $date->locale();            // fr_FR
echo "\n";
echo $date->diffForHumans();     // il y a 1 seconde
echo "\n";
echo $date->monthName;           // octobre
echo "\n";
echo $date->isoFormat('LLLL');   // lundi 14 octobre 2019 14:49
```

`The ->locale()` method only change the language for the current instance and has precedence over global settings. We recommend you this approach so you can't have conflict with other places or third-party libraries that could use Carbon. Nevertheless, to avoid calling `->locale()` each time, you can use factories.

```php
// Let say Martin from Paris and John from Chicago play chess
$martinDateFactory = new Factory([
    'locale' => 'fr_FR',
    'timezone' => 'Europe/Paris',
]);
$johnDateFactory = new Factory([
    'locale' => 'en_US',
    'timezone' => 'America/Chicago',
]);
// Each one will see date in his own language and timezone

// When Martin moves, we display things in French, but we notify John in English:
$gameStart = Carbon::parse('2018-06-15 12:34:00', 'UTC');
$move = Carbon::now('UTC');
$toDisplay = $martinDateFactory->make($gameStart)->isoFormat('lll')."\n".
    $martinDateFactory->make($move)->calendar()."\n";
$notificationForJohn = $johnDateFactory->make($gameStart)->isoFormat('lll')."\n".
    $johnDateFactory->make($move)->calendar()."\n";
echo $toDisplay;
/*
15 juin 2018 12:34
Aujourd’hui à 14:49
*/

echo $notificationForJohn;
/*
Jun 15, 2018 12:34 PM
Today at 2:49 PM
*/
```

You can call any static Carbon method on a factory (make, now, yesterday, tomorrow, parse, create, etc.) Factory (and FactoryImmutable that generates CarbonImmutable instances) are the best way to keep things organized and isolated. As often as possible we recommend you to work with UTC dates, then apply locally (or with a factory) the timezone and the language before displaying dates to the user.

What factory actually do is using the method name as static constructor then call `settings()` method which is a way to group in one call settings of locale, timezone, months/year overflow, etc. ([See references for complete list](https://carbon.nesbot.com/docs/#doc-method-Carbon-settings).)

```php
$factory = new Factory([
    'locale' => 'fr_FR',
    'timezone' => 'Europe/Paris',
]);
$factory->now(); // You can recall $factory as needed to generate new instances with same settings
// is equivalent to:
Carbon::now()->settings([
    'locale' => 'fr_FR',
    'timezone' => 'Europe/Paris',
]);
// Important note: timezone setting calls ->shiftTimezone() and not ->setTimezone(),
// It means it does not just set the timezone, but shift the time too:
echo Carbon::today()->setTimezone('Asia/Tokyo')->format('d/m G\h e');        // 14/10 9h Asia/Tokyo
echo "\n";
echo Carbon::today()->shiftTimezone('Asia/Tokyo')->format('d/m G\h e');      // 14/10 0h Asia/Tokyo
```

Factory settings can be changed afterward with `setSettings(array $settings) `or to merge new settings with existing ones `mergeSettings(array $settings)`and the class to generate can be initialized as the second argument of the construct then changed later with `setClassName(string $className)`.

```php
$factory = new Factory(['locale' => 'ja'], CarbonImmutable::class);
var_dump($factory->now()->locale);                                           // string(2) "ja"
var_dump(get_class($factory->now()));                                        // string(22) "Carbon\CarbonImmutable"

class MyCustomCarbonSubClass extends Carbon { /* ... */ }
$factory
    ->setSettings(['locale' => 'zh_CN'])
    ->setClassName(MyCustomCarbonSubClass::class);
var_dump($factory->now()->locale);                                           // string(5) "zh_CN"
var_dump(get_class($factory->now()));                                        // string(22) "MyCustomCarbonSubClass"
```

Previously there was `Carbon::setLocale` that set globally the locale. But as for our other static setters, we highly discourage you to use it. It breaks the principle of isolation because the configuration will apply for every class that uses Carbon.

You also may know `formatLocalized()` method from Carbon 1. This method still works the same in Carbon 2 but you should better use `isoFormat()` instead.

`->isoFormat(string $format)`: string use ISO format rather than PHP-specific format and use inner translations rather than language packages you need to install on every machine where you deploy your application. isoFormat method is compatible with [momentjs format method](https://momentjs.com/), it means you can use same format strings as you may have used in moment from front-end or node.js. Here are some examples:

```php
$date = Carbon::parse('2018-06-15 17:34:15.984512', 'UTC');
echo $date->isoFormat('MMMM Do YYYY, h:mm:ss a'); // June 15th 2018, 5:34:15 pm
echo "\n";
echo $date->isoFormat('dddd');           // Friday
echo "\n";
echo $date->isoFormat('MMM Do YY');      // Jun 15th 18
echo "\n";
echo $date->isoFormat('YYYY [escaped] YYYY'); // 2018 escaped 2018
```
You can also create date from ISO formatted strings:

```php
$date = Carbon::createFromIsoFormat('!YYYY-MMMM-D h:mm:ss a', '2019-January-3 6:33:24 pm', 'UTC');
echo $date->isoFormat('M/D/YY HH:mm'); // 1/3/19 18:33
```
`->isoFormat` use contextualized methods for day names and month names as they can have multiple forms in some languages, see the following examples:

```php
$date = Carbon::parse('2018-03-16')->locale('uk');
echo $date->getTranslatedDayName('[в] dddd'); // п’ятницю
// By providing a context, we're saying translate day name like in a format such as [в] dddd
// So the context itself has to be translated first consistently.
echo "\n";
echo $date->getTranslatedDayName('[наступної] dddd'); // п’ятниці
echo "\n";
echo $date->getTranslatedDayName('dddd, MMM'); // п’ятниця
echo "\n";
// The same goes for short/minified variants:
echo $date->getTranslatedShortDayName('[наступної] dd'); // пт
echo "\n";
echo $date->getTranslatedMinDayName('[наступної] ddd'); // пт
echo "\n";

// And the same goes for months
$date->locale('ru');
echo $date->getTranslatedMonthName('Do MMMM'); // марта
echo "\n";
echo $date->getTranslatedMonthName('MMMM YYYY'); // март
echo "\n";
// Short variant
echo $date->getTranslatedShortMonthName('Do MMM'); // мар
echo "\n";
echo $date->getTranslatedShortMonthName('MMM YYYY'); // мар
echo "\n";

// And so you can force a different context to get those variants:
echo $date->isoFormat('Do MMMM');        // 16-го марта
echo "\n";
echo $date->isoFormat('MMMM YYYY');      // март 2018
echo "\n";
echo $date->isoFormat('Do MMMM', 'MMMM YYYY'); // 16-го март
echo "\n";
echo $date->isoFormat('MMMM YYYY', 'Do MMMM'); // марта 2018
echo "\n";
```
Here is the complete list of available replacements (examples given with `$date = Carbon::parse('2017-01-05 17:04:05.084512')`;):


|Code|	Example|	Description|
|-|-|-|
|OD	|5	|Day number with alternative numbers such as 三 for 3 if locale is ja_JP|
|OM	|1|	Month number with alternative numbers such as ၀၂ for 2 if locale is my_MM|
|OY	|2017	|Year number with alternative numbers such as ۱۹۹۸ for 1998 if locale is fa|
|OH	|17	|24-hours number with alternative numbers such as ႑႓ for 13 if locale is shn_MM|
|Oh	|5	|12-hours number with alternative numbers such as 十一 for 11 if locale is lzh_TW|
|Om|	4|	Minute number with alternative numbers such as ୫୭ for 57 if locale is or|
|Os	|5	|Second number with alternative numbers such as 十五 for 15 if locale is ja_JP|
|D|	5|	Day of month number (from 1 to 31)|
|DD	|05	|Day of month number with trailing zero (from 01 to 31)|
|Do	|5th	|Day of month with ordinal suffix (from 1st to 31th), translatable|
|d	|4|	Day of week number (from 0 (Sunday) to 6 (Saturday))|
|dd	|Th|	Minified day name (from Su to Sa), transatable|
|ddd	|Thu	|Short day name (from Sun to Sat), transatable|
|dddd|	Thursday|	Day name (from Sunday to Saturday), transatable|
|DDD	|5	|Day of year number (from 1 to 366)|
|DDDD	|005|	Day of year number with trailing zeros (3 digits, from 001 to 366)|
|DDDo|	5th	|Day of year number with ordinal suffix (from 1st to 366th), translatable|
|e|	4	|Day of week number (from 0 (Sunday) to 6 (Saturday)), similar to "d" but this one is translatable (takes first day of week of the current locale)|
|E	|4	|Day of week number (from 1 (Monday) to 7 (Sunday))|
|H|	17	|Hour from 0 to 23|
|HH|	17|	Hour with trailing zero from 00 to 23|
|h	|5	|Hour from 0 to 12|
|hh	|05	|Hour with trailing zero from 00 to 12|
|k	|17	|Hour from 1 to 24|
|kk	|17	|Hour with trailing zero from 01 to 24|
|m	|4	|Minute from 0 to 59|
|mm|	04	|Minute with trailing zero from 00 to 59|
|a|	pm|	Meridiem am/pm|
|A	|PM|	Meridiem AM/PM|
|s|	5	|Second from 0 to 59|
|ss	|05	|Second with trailing zero from 00 to 59|
|S	|0	|Second tenth|
|SS	|08	|Second hundredth (on 2 digits with trailing zero)|
|SSS|	084	|Millisecond (on 3 digits with trailing zeros)|
|SSSS	|0845	|Second ten thousandth (on 4 digits with trailing zeros)|
|SSSSS	|08451	|Second hundred thousandth (on 5 digits with trailing zeros)|
|SSSSSS	|084512	|Microsecond (on 6 digits with trailing zeros)|
|SSSSSSS	|0845120|	Second ten millionth (on 7 digits with trailing zeros)|
|SSSSSSSS	|08451200|	Second hundred millionth (on 8 digits with trailing zeros)|
|SSSSSSSSS	|084512000|	Nanosecond (on 9 digits with trailing zeros)|
|M	|1	|Month from 1 to 12|
|MM	|01	|Month with trailing zero from 01 to 12|
|MMM	|Jan	|Short month name, translatable|
|MMMM	|January	|Month name, translatable|
|Mo	|1st	|Month with ordinal suffix from 1st to 12th, translatable|
|Q	|1	|Quarter from 1 to 4|
|Qo	|1st	|Quarter with ordinal suffix from 1st to 4th, translatable|
|G	|2017	|ISO week year (see ISO week date)|
|GG	|2017	|ISO week year (on 2 digits with trailing zero)|
|GGG	|2017	|ISO week year (on 3 digits with trailing zeros)|
|GGGG	|2017	|ISO week year (on 4 digits with trailing zeros)|
|GGGGG	|02017	|ISO week year (on 5 digits with trailing zeros)|
|g	|2017|	Week year according to locale settings, translatable|
|gg	|2017|	Week year according to locale settings (on 2 digits with trailing zero), translatable|
|ggg	|2017	|Week year according to locale settings (on 3 digits with trailing zeros), translatable|
|gggg	|2017	|Week year according to locale settings (on 4 digits with trailing zeros), translatable|
|ggggg	|02017	|Week year according to locale settings (on 5 digits with |trailing zeros), translatable|
|W	|1	|ISO week number in the year (see ISO week date)|
|WW	|01	| ISO week number in the year (on 2 digits with trailing zero)
|Wo	|1st	| ISO week number in the year with ordinal suffix, translatable|
|w	|1	|Week number in the year according to locale settings, translatable|
|ww	|01	|Week number in the year according to locale settings (on 2 digits with trailing zero)|
|wo	|1st|	Week number in the year according to locale settings with ordinal suffix, translatable|
|x	|1483635845085|	Millisecond-precision timestamp (same as date.getTime() in JavaScript)|
|X	|1483635845	|Timestamp (number of seconds since 1970-01-01)|
|Y	|2017|	Full year from -9999 to 9999|
|YY	|17	|Year on 2 digits from 00 to 99|
|YYYY	|2017|	Year on 4 digits from 0000 to 9999|
|YYYYY	|02017	| Year on 5 digits from 00000 to 09999|
|YYYYYY	|+002017 |	Year on 5 digits with sign from -09999 to +09999|
|z	|utc	| Abbreviated time zone name|
|zz	|UTC	| Time zone name|
|Z	|+00:00	| Time zone offset HH:mm|
|ZZ	|+0000	| Time zone offset HHmm|