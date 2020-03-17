## Difference

As `Carbon` extends `DateTime` it inherit its methods such as `diff()` that take a second date object as argument and returns a DateInterval instance.

We also provide `diffAsCarbonInterval()` act like `diff()` but returns a `CarbonInterval` instance. Check [CarbonInterval chapter](https://carbon.nesbot.com/docs/#api-interval) for more information. Carbon add diff methods for each unit too such as `diffInYears()`, `diffInMonths()` and so on. diffAsCarbonInterval() and diffIn*() and floatDiffIn*() methods all can take 2 optional arguments: date to compare with (if missing, now is used instead), and an absolute boolean option (true by default) that make the method return an absolute value no matter which date is greater than the other. If set to false, it returns negative value when the instance the method is called on is greater than the compared date (first argument or now). Note that diff() prototype is different: its first argument (the date) is mandatory and its second argument (the absolute option) defaults to false.

These functions always return the total difference expressed in the specified time requested. This differs from the base class `diff()` function where an interval of 122 seconds would be returned as 2 minutes and 2 seconds via a DateInterval instance. The `diffInMinutes()` function would simply return 2 while `diffInSeconds()` would return 122. All values are truncated and not rounded. Each function below has a default first parameter which is the Carbon instance to compare to, or null if you want to use `now()`. The 2nd parameter again is optional and indicates if you want the return value to be the absolute value or a relative value that might have a - (negative) sign if the passed in date is less than the current instance. This will default to true, return the absolute value.

```php
echo Carbon::now('America/Vancouver')->diffInSeconds(Carbon::now('Europe/London')); // 0

$dtOttawa = Carbon::createMidnightDate(2000, 1, 1, 'America/Toronto');
$dtVancouver = Carbon::createMidnightDate(2000, 1, 1, 'America/Vancouver');
echo $dtOttawa->diffInHours($dtVancouver);                             // 3
echo $dtVancouver->diffInHours($dtOttawa);                             // 3

echo $dtOttawa->diffInHours($dtVancouver, false);                      // 3
echo $dtVancouver->diffInHours($dtOttawa, false);                      // -3

$dt = Carbon::createMidnightDate(2012, 1, 31);
echo $dt->diffInDays($dt->copy()->addMonth());                         // 31
echo $dt->diffInDays($dt->copy()->subMonth(), false);                  // -31

$dt = Carbon::createMidnightDate(2012, 4, 30);
echo $dt->diffInDays($dt->copy()->addMonth());                         // 30
echo $dt->diffInDays($dt->copy()->addWeek());                          // 7

$dt = Carbon::createMidnightDate(2012, 1, 1);
echo $dt->diffInMinutes($dt->copy()->addSeconds(59));                  // 0
echo $dt->diffInMinutes($dt->copy()->addSeconds(60));                  // 1
echo $dt->diffInMinutes($dt->copy()->addSeconds(119));                 // 1
echo $dt->diffInMinutes($dt->copy()->addSeconds(120));                 // 2

echo $dt->addSeconds(120)->secondsSinceMidnight();                     // 120

$interval = $dt->diffAsCarbonInterval($dt->copy()->subYears(3), false);
// diffAsCarbonInterval use same arguments as diff($other, $absolute)
// (native method from \DateTime)
// except $absolute is true by default for diffAsCarbonInterval and false for diff
// $absolute parameter allow to get signed value if false, or always positive if true
echo ($interval->invert ? 'minus ' : 'plus ') . $interval->years;      // minus 3
```
These methods have int-truncated results. That means `diffInMinutes` returns 1 for any difference between 1 included and 2 excluded. But same methods are available for float results:

```php
echo Carbon::parse('06:01:23.252987')->floatDiffInSeconds('06:02:34.321450');    // 71.068463
echo Carbon::parse('06:01:23')->floatDiffInMinutes('06:02:34');                  // 1.1833333333333
echo Carbon::parse('06:01:23')->floatDiffInHours('06:02:34');                    // 0.019722222222222
// Those methods are absolute by default but can return negative value
// setting the second argument to false when start date is greater than end date
echo Carbon::parse('12:01:23')->floatDiffInHours('06:02:34', false);             // -5.9802777777778
echo Carbon::parse('2000-01-01 12:00')->floatDiffInDays('2000-02-11 06:00');     // 40.75
echo Carbon::parse('2000-01-15 12:00')->floatDiffInMonths('2000-02-24 06:00');   // 1.301724137931
// floatDiffInMonths count as many full months as possible from the start date, then consider the number
// of days in the months for ending chunks to reach the end date
// So the following result (ending with 24 march is different from previous one with 24 february):
echo Carbon::parse('2000-02-15 12:00')->floatDiffInMonths('2000-03-24 06:00');   // 1.2822580645161
// floatDiffInYears apply the same logic (and so different results with leap years)
echo Carbon::parse('2000-02-15 12:00')->floatDiffInYears('2010-03-24 06:00');    // 10.100684931507
```
Important note about the daylight saving times (DST), by default PHP DateTime does not take DST into account, that means for example that a day with only 23 hours like March the 30th 2014 in London will be counted as 24 hours long.

```php
$date = new DateTime('2014-03-30 00:00:00', new DateTimeZone('Europe/London')); // DST off
echo $date->modify('+25 hours')->format('H:i');                   // 01:00 (DST on, 24 hours only have been actually added)
```
Carbon follow this behavior too for add/sub/diff seconds/minutes/hours. But we provide methods to works with real hours using timestamp:
```
$date = new Carbon('2014-03-30 00:00:00', 'Europe/London');       // DST off
echo $date->addRealHours(25)->format('H:i');                      // 02:00 (DST on)
echo $date->diffInRealHours('2014-03-30 00:00:00');               // 25
echo $date->diffInHours('2014-03-30 00:00:00');                   // 26
echo $date->diffInRealMinutes('2014-03-30 00:00:00');             // 1500
echo $date->diffInMinutes('2014-03-30 00:00:00');                 // 1560
echo $date->diffInRealSeconds('2014-03-30 00:00:00');             // 90000
echo $date->diffInSeconds('2014-03-30 00:00:00');                 // 93600
echo $date->diffInRealMilliseconds('2014-03-30 00:00:00');        // 90000000
echo $date->diffInMilliseconds('2014-03-30 00:00:00');            // 93600000
echo $date->diffInRealMicroseconds('2014-03-30 00:00:00');        // 90000000000
echo $date->diffInMicroseconds('2014-03-30 00:00:00');            // 93600000000
echo $date->subRealHours(25)->format('H:i');                      // 00:00 (DST off)

// with float diff:
$date = new Carbon('2019-10-27 00:00:00', 'Europe/Paris');       
echo $date->floatDiffInRealHours('2019-10-28 12:30:00');          // 37.5
echo $date->floatDiffInHours('2019-10-28 12:30:00');              // 36.5
echo $date->floatDiffInRealMinutes('2019-10-28 12:00:30');        // 2220.5
echo $date->floatDiffInMinutes('2019-10-28 12:00:30');            // 2160.5
echo $date->floatDiffInRealSeconds('2019-10-28 12:00:00.5');      // 133200.5
echo $date->floatDiffInSeconds('2019-10-28 12:00:00.5');          // 129600.5
// above day unit, "real" will affect the decimal part based on hours and smaller units
echo $date->floatDiffInRealDays('2019-10-28 12:30:00');           // 1.5625
echo $date->floatDiffInDays('2019-10-28 12:30:00');               // 1.5208333333333
echo $date->floatDiffInRealMonths('2019-10-28 12:30:00');         // 0.050403225806452
echo $date->floatDiffInMonths('2019-10-28 12:30:00');             // 0.049059139784946
echo $date->floatDiffInRealYears('2019-10-28 12:30:00');          // 0.0042808219178082
echo $date->floatDiffInYears('2019-10-28 12:30:00');              // 0.0041666666666667
```
The same way you can use `addRealX()` and `subRealX()` on any unit.

There are also special filter functions `diffInDaysFiltered()`, `diffInHoursFiltered()` and `diffFiltered()`, to help you filter the difference by days, hours or a custom interval. For example to count the weekend days between two instances:

```php
$dt = Carbon::create(2014, 1, 1);
$dt2 = Carbon::create(2014, 12, 31);
$daysForExtraCoding = $dt->diffInDaysFiltered(function(Carbon $date) {
   return $date->isWeekend();
}, $dt2);

echo $daysForExtraCoding;      // 104

$dt = Carbon::create(2014, 1, 1)->endOfDay();
$dt2 = $dt->copy()->startOfDay();
$littleHandRotations = $dt->diffFiltered(CarbonInterval::minute(), function(Carbon $date) {
   return $date->minute === 0;
}, $dt2, true); // true as last parameter returns absolute value

echo $littleHandRotations;     // 24

$date = Carbon::now()->addSeconds(3666);

echo $date->diffInSeconds();                       // 3665
echo $date->diffInMinutes();                       // 61
echo $date->diffInHours();                         // 1
echo $date->diffInDays();                          // 0

$date = Carbon::create(2016, 1, 5, 22, 40, 32);

echo $date->secondsSinceMidnight();                // 81632
echo $date->secondsUntilEndOfDay();                // 4767

$date1 = Carbon::createMidnightDate(2016, 1, 5);
$date2 = Carbon::createMidnightDate(2017, 3, 15);

echo $date1->diffInDays($date2);                   // 435
echo $date1->diffInWeekdays($date2);               // 311
echo $date1->diffInWeekendDays($date2);            // 124
echo $date1->diffInWeeks($date2);                  // 62
echo $date1->diffInMonths($date2);                 // 14
echo $date1->diffInQuarters($date2);               // 4
echo $date1->diffInYears($date2);                  // 1
```

All diffIn*Filtered method take 1 callable filter as required parameter and a date object as optional second parameter, if missing, now is used. You may also pass true as third parameter to get absolute values.

For advanced handle of the week/weekend days, use the following tools:
```php
echo implode(', ', Carbon::getDays());                       // Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday

$saturday = new Carbon('first saturday of 2019');
$sunday = new Carbon('first sunday of 2019');
$monday = new Carbon('first monday of 2019');

echo implode(', ', Carbon::getWeekendDays());                // 6, 0
var_dump($saturday->isWeekend());                            // bool(true)
var_dump($sunday->isWeekend());                              // bool(true)
var_dump($monday->isWeekend());                              // bool(false)

Carbon::setWeekendDays([
    Carbon::SUNDAY,
    Carbon::MONDAY,
]);

echo implode(', ', Carbon::getWeekendDays());                // 0, 1
var_dump($saturday->isWeekend());                            // bool(false)
var_dump($sunday->isWeekend());                              // bool(true)
var_dump($monday->isWeekend());                              // bool(true)

Carbon::setWeekendDays([
    Carbon::SATURDAY,
    Carbon::SUNDAY,
]);
// weekend days and start/end of week or not linked
Carbon::setWeekStartsAt(Carbon::FRIDAY);
Carbon::setWeekEndsAt(Carbon::WEDNESDAY); // and it does not need neither to precede the start

var_dump(Carbon::getWeekStartsAt() === Carbon::FRIDAY);      // bool(true)
var_dump(Carbon::getWeekEndsAt() === Carbon::WEDNESDAY);     // bool(true)
echo $saturday->copy()->startOfWeek()->toRfc850String();     // Friday, 31-Jan-20 00:00:00 UTC
echo $saturday->copy()->endOfWeek()->toRfc850String();       // Wednesday, 05-Feb-20 23:59:59 UTC

// Be very careful with those global setters, remember, some other
// code or third-party library in your app may expect initial weekend
// days to work properly.
Carbon::setWeekStartsAt(Carbon::MONDAY);
Carbon::setWeekEndsAt(Carbon::SUNDAY);

echo $saturday->copy()->startOfWeek()->toRfc850String();     // Monday, 27-Jan-20 00:00:00 UTC
echo $saturday->copy()->endOfWeek()->toRfc850String();       // Sunday, 02-Feb-20 23:59:59 UTC
```
