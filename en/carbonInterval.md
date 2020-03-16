## CarbonInterval

The CarbonInterval class is [inherited](http://php.net/manual/en/keyword.extends.php) from the PHP [DateInterval](http://php.net/manual/en/class.dateinterval.php) class.

```
<?php
class CarbonInterval extends \DateInterval
{
    // code here
}
```
You can create an instance in the following ways:

```php

echo CarbonInterval::createFromFormat('H:i:s', '10:20:00'); // 10 hours 20 minutes
echo "\n";
echo CarbonInterval::year();                           // 1 year
echo "\n";
echo CarbonInterval::months(3);                        // 3 months
echo "\n";
echo CarbonInterval::days(3)->seconds(32);             // 3 days 32 seconds
echo "\n";
echo CarbonInterval::weeks(3);                         // 3 weeks
echo "\n";
echo CarbonInterval::days(23);                         // 3 weeks 2 days
echo "\n";
// years, months, weeks, days, hours, minutes, seconds, microseconds
echo CarbonInterval::create(2, 0, 5, 1, 1, 2, 7, 123); // 2 years 5 weeks 1 day 1 hour 2 minutes 7 seconds
echo "\n";
echo CarbonInterval::createFromDateString('3 months'); // 3 months
```

If you find yourself inheriting a `\DateInterval` instance from another library, fear not! You can create a `CarbonInterval `instance via a friendly `instance()` function.

```php
$di = new \DateInterval('P1Y2M'); // <== instance from another API
$ci = CarbonInterval::instance($di);
echo get_class($ci);                                   // 'Carbon\CarbonInterval'
echo $ci;                                              // 1 year 2 months
```

And as the opposite you can extract a raw `DateInterval` from `CarbonInterval` and even cast it in any class that extends `DateInterval`

```php
$ci = CarbonInterval::days(2);
$di = $ci->toDateInterval();
echo get_class($di);   // 'DateInterval'
echo $di->d;           // 2

// Your custom class can also extends CarbonInterval
class CustomDateInterval extends \DateInterval {}

$di = $ci->cast(\CustomDateInterval::class);
echo get_class($di);   // 'CustomDateInterval'
echo $di->d;           // 2
```
You can compare intervals the same way than Carbon objects, using `equalTo()`, `notEqualTo()`,`lessThan()`, `lessThanOrEqualTo()`, `greaterThan()`, `greaterThanOrEqualTo()`, `between()`, `betweenExcluded()`, etc.
Other helpers, but beware the implementation provides helpers to handle weeks but only days are saved. Weeks are calculated based on the total days of the current instance.
```php
echo CarbonInterval::year()->years;                    // 1
echo CarbonInterval::year()->dayz;                     // 0
echo CarbonInterval::days(24)->dayz;                   // 24
echo CarbonInterval::days(24)->daysExcludeWeeks;       // 3
echo CarbonInterval::weeks(3)->days(14)->weeks;        // 2  <-- days setter overwrites the current value
echo CarbonInterval::weeks(3)->weeks;                  // 3
echo CarbonInterval::minutes(3)->weeksAndDays(2, 5);   // 2 weeks 5 days 3 minutes
```
CarbonInterval extends DateInterval and you can create both using [ISO-8601 duration format](https://en.wikipedia.org/wiki/ISO_8601#Durations):

```php
$ci = CarbonInterval::create('P1Y2M3D');
var_dump($ci->isEmpty()); // bool(false)
$ci = new CarbonInterval('PT0S');
var_dump($ci->isEmpty()); // bool(true)
```
Carbon intervals can be created from human-friendly strings thanks to `fromString()` method.
```php
CarbonInterval::fromString('2 minutes 15 seconds');
CarbonInterval::fromString('2m 15s'); // or abbreviated
```

Or create it from an other `DateInterval` / `CarbonInterval` object.

```php
$ci = new CarbonInterval(new DateInterval('P1Y2M3D'));
var_dump($ci->isEmpty()); // bool(false)
```
Note that month abbreviate "mo" to distinguish from minutes and the whole syntax is not case sensitive.

It also has a handy `forHumans()`, which is mapped as the `__toString()` implementation, that prints the interval for humans.


```php
CarbonInterval::setLocale('fr');
echo CarbonInterval::create(2, 1)->forHumans();        // 2 ans 1 mois
echo CarbonInterval::hour()->seconds(3);               // 1 heure 3 secondes
CarbonInterval::setLocale('en');
```

As you can see, you can change the locale of the string using `CarbonInterval::setLocale('fr')`.

As for Carbon, you can use the make method to return a new instance of CarbonInterval from other interval or strings:

```php
$dateInterval = new DateInterval('P2D');
$carbonInterval = CarbonInterval::month();
echo CarbonInterval::make($dateInterval)->forHumans();       // 2 days
echo CarbonInterval::make($carbonInterval)->forHumans();     // 1 month
echo CarbonInterval::make('PT3H')->forHumans();              // 3 hours
echo CarbonInterval::make('1h 15m')->forHumans();            // 1 hour 15 minutes
// forHumans has many options, since version 2.9.0, the recommended way is to pass them as an associative array:
echo CarbonInterval::make('1h 15m')->forHumans(['short' => true]); // 1h 15m
$interval = CarbonInterval::make('1h 15m 45s');
echo $interval->forHumans(['join' => true]);                 // 1 hour, 15 minutes and 45 seconds
$esInterval = CarbonInterval::make('1h 15m 45s');
echo $esInterval->forHumans(['join' => true]);               // 1 hour, 15 minutes and 45 seconds
echo $interval->forHumans(['join' => true, 'parts' => 2]);   // 1 hour and 15 minutes
echo $interval->forHumans(['join' => ' - ']);                // 1 hour - 15 minutes - 45 seconds

// Available syntax modes:
// ago/from now (translated in the current locale)
echo $interval->forHumans(['syntax' => CarbonInterface::DIFF_RELATIVE_TO_NOW]);      // 1 hour 15 minutes 45 seconds ago
// before/after (translated in the current locale)
echo $interval->forHumans(['syntax' => CarbonInterface::DIFF_RELATIVE_TO_OTHER]);    // 1 hour 15 minutes 45 seconds before
// default for intervals (no prefix/suffix):
echo $interval->forHumans(['syntax' => CarbonInterface::DIFF_ABSOLUTE]);             // 1 hour 15 minutes 45 seconds

// Available options:
// transform empty intervals into "just now":
echo CarbonInterval::hours(0)->forHumans([
    'options' => CarbonInterface::JUST_NOW,
    'syntax' => CarbonInterface::DIFF_RELATIVE_TO_NOW,
]); // just now
// transform empty intervals into "1 second":
echo CarbonInterval::hours(0)->forHumans([
    'options' => CarbonInterface::NO_ZERO_DIFF,
    'syntax' => CarbonInterface::DIFF_RELATIVE_TO_NOW,
]); // 1 second ago
// transform "1 day ago"/"1 day from now" into "yesterday"/"tomorrow":
echo CarbonInterval::day()->forHumans([
    'options' => CarbonInterface::ONE_DAY_WORDS,
    'syntax' => CarbonInterface::DIFF_RELATIVE_TO_NOW,
]); // 1 day ago
// transform "2 days ago"/"2 days from now" into "before yesterday"/"after tomorrow":
echo CarbonInterval::days(2)->forHumans([
    'options' => CarbonInterface::TWO_DAY_WORDS,
    'syntax' => CarbonInterface::DIFF_RELATIVE_TO_NOW,
]); // 2 days ago
// options can be piped:
echo CarbonInterval::days(2)->forHumans([
    'options' => CarbonInterface::ONE_DAY_WORDS | CarbonInterface::TWO_DAY_WORDS,
    'syntax' => CarbonInterface::DIFF_RELATIVE_TO_NOW,
]); // 2 days ago

// Before version 2.9.0, parameters could only be passed sequentially:
// $interval->forHumans($syntax, $short, $parts, $options)
// and join parameter was not available
```
The add, sub (or subtract), times, shares, multiply and divide methods allow to do proceed intervals calculations:

```php
$interval = CarbonInterval::make('7h 55m');
$interval->add(CarbonInterval::make('17h 35m'));
$interval->subtract(10, 'minutes');
// add(), sub() and subtract() can take DateInterval, CarbonInterval, interval as string or 2 arguments factor and unit
$interval->times(3);
echo $interval->forHumans();                                       // 72 hours 240 minutes
echo "\n";
$interval->shares(7);
echo $interval->forHumans();                                       // 10 hours 34 minutes
echo "\n";
// As you can see add(), times() and shares() operate naively a rounded calculation on each unit

// You also can use multiply() of divide() to cascade units and get precise calculation:
echo CarbonInterval::make('19h 55m')->multiply(3)->forHumans();    // 2 days 11 hours 45 minutes
echo "\n";
echo CarbonInterval::make('19h 55m')->divide(3)->forHumans();      // 6 hours 38 minutes 20 seconds
```
You get pure calculation from your input unit by unit. To cascade minutes into hours, hours into days etc. Use the cascade method:

```php
echo $interval->forHumans();             // 10 hours 34 minutes
echo $interval->cascade()->forHumans();  // 10 hours 34 minutes
```

Default factors are:

- 1 minute = 60 seconds
- 1 hour = 60 minutes
- 1 day = 24 hour
- 1 week = 7 days
- 1 month = 4 weeks
- 1 year = 12 months

CarbonIntervals do not carry context so they cannot be more precise (no DST, no leap year, no real month length or year length consideration). But you can completely customize those factors. For example to deal with work time logs:

```php
$cascades = CarbonInterval::getCascadeFactors(); // save initial factors

CarbonInterval::setCascadeFactors([
    'minute' => [60, 'seconds'],
    'hour' => [60, 'minutes'],
    'day' => [8, 'hours'],
    'week' => [5, 'days'],
    // in this example the cascade won't go farther than week unit
]);

echo CarbonInterval::fromString('20h')->cascade()->forHumans();              // 2 days 4 hours
echo CarbonInterval::fromString('10d')->cascade()->forHumans();              // 2 weeks
echo CarbonInterval::fromString('3w 18d 53h 159m')->cascade()->forHumans();  // 7 weeks 4 days 7 hours 39 minutes

// You can see currently set factors with getFactor:
echo CarbonInterval::getFactor('minutes', /* per */ 'hour');                 // 60
echo CarbonInterval::getFactor('days', 'week');                              // 5

// And common factors can be get with short-cut methods:
echo CarbonInterval::getDaysPerWeek();                                       // 5
echo CarbonInterval::getHoursPerDay();                                       // 8
echo CarbonInterval::getMinutesPerHour();                                    // 60
echo CarbonInterval::getSecondsPerMinute();                                  // 60
echo CarbonInterval::getMillisecondsPerSecond();                             // 1000
echo CarbonInterval::getMicrosecondsPerMillisecond();                        // 1000

CarbonInterval::setCascadeFactors($cascades); // restore original factors
```
Is it possible to convert an interval into a given unit (using provided cascade factors).

```php
echo CarbonInterval::days(3)->hours(5)->total('hours');    // 77
echo CarbonInterval::days(3)->hours(5)->totalHours;        // 77
echo CarbonInterval::months(6)->totalWeeks;                // 24
echo CarbonInterval::year()->totalDays;                    // 336
```
`->total` method and properties need cascaded intervals, if your interval can have overflow, cascade them before calling these feature:

```php
echo CarbonInterval::minutes(1200)->cascade()->total('hours'); // 20
echo CarbonInterval::minutes(1200)->cascade()->totalHours; // 20
```

You can also get the ISO 8601 spec of the inverval with `spec()`
```php
echo CarbonInterval::days(3)->hours(5)->spec(); // P3DT5H
```
It's also possible to get it from a DateInterval object since to the static helper:
```php
echo CarbonInterval::getDateIntervalSpec(new DateInterval('P3DT6M10S')); // P3DT6M10S
```

List of date intervals can be sorted thanks to the `compare()` and `compareDateIntervals()` methods:

```php
$halfDay = CarbonInterval::hours(12);
$oneDay = CarbonInterval::day();
$twoDay = CarbonInterval::days(2);

echo CarbonInterval::compareDateIntervals($oneDay, $oneDay);   // 0
echo $oneDay->compare($oneDay);                                // 0
echo CarbonInterval::compareDateIntervals($oneDay, $halfDay);  // 1
echo $oneDay->compare($halfDay);                               // 1
echo CarbonInterval::compareDateIntervals($oneDay, $twoDay);   // -1
echo $oneDay->compare($twoDay);                                // -1

$list = [$twoDay, $halfDay, $oneDay];
usort($list, ['Carbon\CarbonInterval', 'compareDateIntervals']);

echo implode(', ', $list);                                     // 12 hours, 1 day, 2 days
```

Dump interval values as an array with:

```php
$interval = CarbonInterval::months(2)->hours(12)->seconds(50);

// All the values:
print_r($interval->toArray());
/*
Array
(
    [years] => 0
    [months] => 2
    [weeks] => 0
    [days] => 0
    [hours] => 12
    [minutes] => 0
    [seconds] => 50
    [microseconds] => 0
)

*/

// Values sequence from the biggest to the smallest non-zero ones:
print_r($interval->getValuesSequence());
/*
Array
(
    [months] => 2
    [weeks] => 0
    [days] => 0
    [hours] => 12
    [minutes] => 0
    [seconds] => 50
)

*/

// Non-zero values:
print_r($interval->getNonZeroValues());
/*
Array
(
    [months] => 2
    [hours] => 12
    [seconds] => 50
)

*/

```


Last, a CarbonInterval instance can be converted into a CarbonPeriod instance by calling `toPeriod()` with complementary arguments.

I hear you ask what is a CarbonPeriod instance. Oh! Perfect transition to our next chapter.



