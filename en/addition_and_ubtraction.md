# Addition and Subtraction

The default DateTime provides a couple of different methods for easily adding and subtracting time. There is `modify()`, `add()` and `sub()`. `modify()` takes a magical date/time format string, `'last day of next month'`, that it parses and applies the modification while `add()` and `sub()` expect a `DateInterval` instance that's not so obvious, (e.g. `new \DateInterval('P6YT5M')` would mean 6 years and 5 minutes). Hopefully using these fluent functions will be more clear and easier to read after not seeing your code for a few weeks. But of course I don't make you choose since the base class functions are still available.
```php
$dt = Carbon::create(2012, 1, 31, 0);

echo $dt->toDateTimeString();            // 2012-01-31 00:00:00

echo $dt->addCenturies(5);               // 2512-01-31 00:00:00
echo $dt->addCentury();                  // 2612-01-31 00:00:00
echo $dt->subCentury();                  // 2512-01-31 00:00:00
echo $dt->subCenturies(5);               // 2012-01-31 00:00:00

echo $dt->addYears(5);                   // 2017-01-31 00:00:00
echo $dt->addYear();                     // 2018-01-31 00:00:00
echo $dt->subYear();                     // 2017-01-31 00:00:00
echo $dt->subYears(5);                   // 2012-01-31 00:00:00

echo $dt->addQuarters(2);                // 2012-07-31 00:00:00
echo $dt->addQuarter();                  // 2012-10-31 00:00:00
echo $dt->subQuarter();                  // 2012-07-31 00:00:00
echo $dt->subQuarters(2);                // 2012-01-31 00:00:00

echo $dt->addMonths(60);                 // 2017-01-31 00:00:00
echo $dt->addMonth();                    // 2017-03-03 00:00:00 equivalent of $dt->month($dt->month + 1); so it wraps
echo $dt->subMonth();                    // 2017-02-03 00:00:00
echo $dt->subMonths(60);                 // 2012-02-03 00:00:00

echo $dt->addDays(29);                   // 2012-03-03 00:00:00
echo $dt->addDay();                      // 2012-03-04 00:00:00
echo $dt->subDay();                      // 2012-03-03 00:00:00
echo $dt->subDays(29);                   // 2012-02-03 00:00:00

echo $dt->addWeekdays(4);                // 2012-02-09 00:00:00
echo $dt->addWeekday();                  // 2012-02-10 00:00:00
echo $dt->subWeekday();                  // 2012-02-09 00:00:00
echo $dt->subWeekdays(4);                // 2012-02-03 00:00:00

echo $dt->addWeeks(3);                   // 2012-02-24 00:00:00
echo $dt->addWeek();                     // 2012-03-02 00:00:00
echo $dt->subWeek();                     // 2012-02-24 00:00:00
echo $dt->subWeeks(3);                   // 2012-02-03 00:00:00

echo $dt->addHours(24);                  // 2012-02-04 00:00:00
echo $dt->addHour();                     // 2012-02-04 01:00:00
echo $dt->subHour();                     // 2012-02-04 00:00:00
echo $dt->subHours(24);                  // 2012-02-03 00:00:00

echo $dt->addMinutes(61);                // 2012-02-03 01:01:00
echo $dt->addMinute();                   // 2012-02-03 01:02:00
echo $dt->subMinute();                   // 2012-02-03 01:01:00
echo $dt->subMinutes(61);                // 2012-02-03 00:00:00

echo $dt->addSeconds(61);                // 2012-02-03 00:01:01
echo $dt->addSecond();                   // 2012-02-03 00:01:02
echo $dt->subSecond();                   // 2012-02-03 00:01:01
echo $dt->subSeconds(61);                // 2012-02-03 00:00:00

echo $dt->addMilliseconds(61);           // 2012-02-03 00:00:00
echo $dt->addMillisecond();              // 2012-02-03 00:00:00
echo $dt->subMillisecond();              // 2012-02-03 00:00:00
echo $dt->subMillisecond(61);            // 2012-02-03 00:00:00

echo $dt->addMicroseconds(61);           // 2012-02-03 00:00:00
echo $dt->addMicrosecond();              // 2012-02-03 00:00:00
echo $dt->subMicrosecond();              // 2012-02-03 00:00:00
echo $dt->subMicroseconds(61);           // 2012-02-03 00:00:00

// and so on for any unit: millenium, century, decade, year, quarter, month, week, day, weekday,
// hour, minute, second, microsecond.

// Generic methods add/sub (or subtract alias) can take many different arguments:
echo $dt->add(61, 'seconds');                      // 2012-02-03 00:01:01
echo $dt->sub('1 day');                            // 2012-02-02 00:01:01
echo $dt->add(CarbonInterval::months(2));          // 2012-04-02 00:01:01
echo $dt->subtract(new DateInterval('PT1H'));      // 2012-04-01 23:01:01
```
For fun you can also pass negative values to `addXXX()`, in fact that's how `subXXX()` is implemented.

P.S. Don't worry if you forget and use `addDay(5)` or `subYear(3)`, I have your back ;)

By default, Carbon relies on the underlying parent class PHP DateTime behavior. As a result adding or subtracting months can overflow, example:

```php
$dt = CarbonImmutable::create(2017, 1, 31, 0);

echo $dt->addMonth();                    // 2017-03-03 00:00:00
echo "\n";
echo $dt->subMonths(2);                  // 2016-12-01 00:00:00
```
Since Carbon 2, you can set a local overflow behavior for each instance:

```php
$dt = CarbonImmutable::create(2017, 1, 31, 0);
$dt->settings([
    'monthOverflow' => false,
]);

echo $dt->addMonth();                    // 2017-02-28 00:00:00
echo "\n";
echo $dt->subMonths(2);                  // 2016-11-30 00:00:00
```
This will apply for methods `addMonth(s)`, `subMonth(s)`, `add($x, 'month')`, `sub($x, 'month')` and equivalent quarter methods. But it won't apply for intervals objects or strings like `add(CarbonInterval::month())` or `add('1 month')`.

Static helpers exist but are deprecated. If you're sure to need to apply global setting or work with version 1 of Carbon, [check the overflow static helpers section](https://carbon.nesbot.com/docs/#overflow-static-helpers)

You also can use `->addMonthsNoOverflow`, `->subMonthsNoOverflow`, `->addMonthsWithOverflow` and `->subMonthsWithOverflow` (or the singular methods with no s to "month") to explicitly add/sub months with or without overflow no matter the current mode and the same for any bigger unit (quarter, year, decade, century, millennium).

```php
$dt = Carbon::createMidnightDate(2017, 1, 31)->settings([
    'monthOverflow' => false,
]);

echo $dt->copy()->addMonthWithOverflow();          // 2017-03-03 00:00:00
// plural addMonthsWithOverflow() method is also available
echo $dt->copy()->subMonthsWithOverflow(2);        // 2016-12-01 00:00:00
// singular subMonthWithOverflow() method is also available
echo $dt->copy()->addMonthNoOverflow();            // 2017-02-28 00:00:00
// plural addMonthsNoOverflow() method is also available
echo $dt->copy()->subMonthsNoOverflow(2);          // 2016-11-30 00:00:00
// singular subMonthNoOverflow() method is also available

echo $dt->copy()->addMonth();                      // 2017-02-28 00:00:00
echo $dt->copy()->subMonths(2);                    // 2016-11-30 00:00:00

$dt = Carbon::createMidnightDate(2017, 1, 31)->settings([
    'monthOverflow' => true,
]);

echo $dt->copy()->addMonthWithOverflow();          // 2017-03-03 00:00:00
echo $dt->copy()->subMonthsWithOverflow(2);        // 2016-12-01 00:00:00
echo $dt->copy()->addMonthNoOverflow();            // 2017-02-28 00:00:00
echo $dt->copy()->subMonthsNoOverflow(2);          // 2016-11-30 00:00:00

echo $dt->copy()->addMonth();                      // 2017-03-03 00:00:00
echo $dt->copy()->subMonths(2);                    // 2016-12-01 00:00:00
```

The same is available for years.

You also can control overflow for any unit when working with unknown inputs:
```php
$dt = CarbonImmutable::create(2018, 8, 30, 12, 00, 00);

// Add hours without overflowing day
echo $dt->addUnitNoOverflow('hour', 7, 'day');     // 2018-08-30 19:00:00
echo "\n";
echo $dt->addUnitNoOverflow('hour', 14, 'day');    // 2018-08-30 23:59:59
echo "\n";
echo $dt->addUnitNoOverflow('hour', 48, 'day');    // 2018-08-30 23:59:59

echo "\n-------\n";

// Substract hours without overflowing day
echo $dt->subUnitNoOverflow('hour', 7, 'day');     // 2018-08-30 05:00:00
echo "\n";
echo $dt->subUnitNoOverflow('hour', 14, 'day');    // 2018-08-30 00:00:00
echo "\n";
echo $dt->subUnitNoOverflow('hour', 48, 'day');    // 2018-08-30 00:00:00

echo "\n-------\n";

// Set hours without overflowing day
echo $dt->setUnitNoOverflow('hour', -7, 'day');    // 2018-08-30 00:00:00
echo "\n";
echo $dt->setUnitNoOverflow('hour', 14, 'day');    // 2018-08-30 14:00:00
echo "\n";
echo $dt->setUnitNoOverflow('hour', 25, 'day');    // 2018-08-30 23:59:59

echo "\n-------\n";

// Adding hours without overflowing month
echo $dt->addUnitNoOverflow('hour', 7, 'month');   // 2018-08-30 19:00:00
echo "\n";
echo $dt->addUnitNoOverflow('hour', 14, 'month');  // 2018-08-31 02:00:00
echo "\n";
echo $dt->addUnitNoOverflow('hour', 48, 'month');  // 2018-08-31 23:59:59

```
Any modifiable unit can be passed as argument of those methods:

```php
$units = [];
foreach (['millennium', 'century', 'decade', 'year', 'quarter', 'month', 'week', 'weekday', 'day', 'hour', 'minute', 'second', 'millisecond', 'microsecond'] as $unit) {
    $units[$unit] = Carbon::isModifiableUnit($unit);
}

echo json_encode($units, JSON_PRETTY_PRINT);
/* {
    "millennium": true,
    "century": true,
    "decade": true,
    "year": true,
    "quarter": true,
    "month": true,
    "week": true,
    "weekday": true,
    "day": true,
    "hour": true,
    "minute": true,
    "second": true,
    "millisecond": true,
    "microsecond": true
} */
```