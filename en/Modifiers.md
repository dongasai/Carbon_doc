# Modifiers

These group of methods perform helpful modifications to the current instance. Most of them are self explanatory from their names... or at least should be. You'll also notice that the startOfXXX(), next() and previous() methods set the time to 00:00:00 and the endOfXXX() methods set the time to 23:59:59 for unit bigger than days.

The only one slightly different is the `average()` function. It moves your instance to the middle date between itself and the provided Carbon argument.

The powerful native [`modify()` method of `DateTime`](https://www.php.net/manual/en/datetime.modify.php) is available untouched. But we also provide an enhanced version of it: `change()` that allows some additional syntax like `"next 3pm"` and that is called internally by `->next()` and `->previous()`.

```php
$dt = new Carbon('2012-01-31 15:32:45.654321');
echo $dt->startOfSecond()->format('s.u');          // 45.000000


$dt = new Carbon('2012-01-31 15:32:45.654321');
echo $dt->endOfSecond()->format('s.u');            // 45.999999

$dt = new Carbon('2012-01-31 15:32:45.654321');
echo $dt->startOf('second')->format('s.u');        // 45.000000

$dt = new Carbon('2012-01-31 15:32:45.654321');
echo $dt->endOf('second')->format('s.u');          // 45.999999
// ->startOf() and ->endOf() are dynamic equivalents to those methods

$dt = Carbon::create(2012, 1, 31, 15, 32, 45);
echo $dt->startOfMinute();                         // 2012-01-31 15:32:00

$dt = Carbon::create(2012, 1, 31, 15, 32, 45);
echo $dt->endOfMinute();                           // 2012-01-31 15:32:59

$dt = Carbon::create(2012, 1, 31, 15, 32, 45);
echo $dt->startOfHour();                           // 2012-01-31 15:00:00

$dt = Carbon::create(2012, 1, 31, 15, 32, 45);
echo $dt->endOfHour();                             // 2012-01-31 15:59:59

$dt = Carbon::create(2012, 1, 31, 15, 32, 45);
echo Carbon::getMidDayAt();                        // 12
echo $dt->midDay();                                // 2012-01-31 12:00:00
Carbon::setMidDayAt(13);
echo Carbon::getMidDayAt();                        // 13
echo $dt->midDay();                                // 2012-01-31 13:00:00
Carbon::setMidDayAt(12);

$dt = Carbon::create(2012, 1, 31, 12, 0, 0);
echo $dt->startOfDay();                            // 2012-01-31 00:00:00

$dt = Carbon::create(2012, 1, 31, 12, 0, 0);
echo $dt->endOfDay();                              // 2012-01-31 23:59:59

$dt = Carbon::create(2012, 1, 31, 12, 0, 0);
echo $dt->startOfMonth();                          // 2012-01-01 00:00:00

$dt = Carbon::create(2012, 1, 31, 12, 0, 0);
echo $dt->endOfMonth();                            // 2012-01-31 23:59:59

$dt = Carbon::create(2012, 1, 31, 12, 0, 0);
echo $dt->startOfYear();                           // 2012-01-01 00:00:00

$dt = Carbon::create(2012, 1, 31, 12, 0, 0);
echo $dt->endOfYear();                             // 2012-12-31 23:59:59

$dt = Carbon::create(2012, 1, 31, 12, 0, 0);
echo $dt->startOfDecade();                         // 2010-01-01 00:00:00

$dt = Carbon::create(2012, 1, 31, 12, 0, 0);
echo $dt->endOfDecade();                           // 2019-12-31 23:59:59

$dt = Carbon::create(2012, 1, 31, 12, 0, 0);
echo $dt->startOfCentury();                        // 2001-01-01 00:00:00

$dt = Carbon::create(2012, 1, 31, 12, 0, 0);
echo $dt->endOfCentury();                          // 2100-12-31 23:59:59

$dt = Carbon::create(2012, 1, 31, 12, 0, 0);
echo $dt->startOfWeek();                           // 2012-01-30 00:00:00
var_dump($dt->dayOfWeek == Carbon::MONDAY);        // bool(true) : ISO8601 week starts on Monday

$dt = Carbon::create(2012, 1, 31, 12, 0, 0);
echo $dt->endOfWeek();                             // 2012-02-05 23:59:59
var_dump($dt->dayOfWeek == Carbon::SUNDAY);        // bool(true) : ISO8601 week ends on Sunday

$dt = Carbon::create(2012, 1, 31, 12, 0, 0);
echo $dt->next(Carbon::WEDNESDAY);                 // 2012-02-01 00:00:00
var_dump($dt->dayOfWeek == Carbon::WEDNESDAY);     // bool(true)
echo $dt->next('Wednesday');                       // 2012-02-08 00:00:00
echo $dt->next('04:00');                           // 2012-02-08 04:00:00
echo $dt->next('12:00');                           // 2012-02-08 12:00:00
echo $dt->next('04:00');                           // 2012-02-09 04:00:00

$dt = Carbon::create(2012, 1, 1, 12, 0, 0);
echo $dt->next();                                  // 2012-01-08 00:00:00

$dt = Carbon::create(2012, 1, 31, 12, 0, 0);
echo $dt->previous(Carbon::WEDNESDAY);             // 2012-01-25 00:00:00
var_dump($dt->dayOfWeek == Carbon::WEDNESDAY);     // bool(true)

$dt = Carbon::create(2012, 1, 1, 12, 0, 0);
echo $dt->previous();                              // 2011-12-25 00:00:00

$start = Carbon::create(2014, 1, 1, 0, 0, 0);
$end = Carbon::create(2014, 1, 30, 0, 0, 0);
echo $start->average($end);                        // 2014-01-15 12:00:00

echo Carbon::create(2014, 5, 30, 0, 0, 0)->firstOfMonth();                       // 2014-05-01 00:00:00
echo Carbon::create(2014, 5, 30, 0, 0, 0)->firstOfMonth(Carbon::MONDAY);         // 2014-05-05 00:00:00
echo Carbon::create(2014, 5, 30, 0, 0, 0)->lastOfMonth();                        // 2014-05-31 00:00:00
echo Carbon::create(2014, 5, 30, 0, 0, 0)->lastOfMonth(Carbon::TUESDAY);         // 2014-05-27 00:00:00
echo Carbon::create(2014, 5, 30, 0, 0, 0)->nthOfMonth(2, Carbon::SATURDAY);      // 2014-05-10 00:00:00

echo Carbon::create(2014, 5, 30, 0, 0, 0)->firstOfQuarter();                     // 2014-04-01 00:00:00
echo Carbon::create(2014, 5, 30, 0, 0, 0)->firstOfQuarter(Carbon::MONDAY);       // 2014-04-07 00:00:00
echo Carbon::create(2014, 5, 30, 0, 0, 0)->lastOfQuarter();                      // 2014-06-30 00:00:00
echo Carbon::create(2014, 5, 30, 0, 0, 0)->lastOfQuarter(Carbon::TUESDAY);       // 2014-06-24 00:00:00
echo Carbon::create(2014, 5, 30, 0, 0, 0)->nthOfQuarter(2, Carbon::SATURDAY);    // 2014-04-12 00:00:00
echo Carbon::create(2014, 5, 30, 0, 0, 0)->startOfQuarter();                     // 2014-04-01 00:00:00
echo Carbon::create(2014, 5, 30, 0, 0, 0)->endOfQuarter();                       // 2014-06-30 23:59:59

echo Carbon::create(2014, 5, 30, 0, 0, 0)->firstOfYear();                        // 2014-01-01 00:00:00
echo Carbon::create(2014, 5, 30, 0, 0, 0)->firstOfYear(Carbon::MONDAY);          // 2014-01-06 00:00:00
echo Carbon::create(2014, 5, 30, 0, 0, 0)->lastOfYear();                         // 2014-12-31 00:00:00
echo Carbon::create(2014, 5, 30, 0, 0, 0)->lastOfYear(Carbon::TUESDAY);          // 2014-12-30 00:00:00
echo Carbon::create(2014, 5, 30, 0, 0, 0)->nthOfYear(2, Carbon::SATURDAY);       // 2014-01-11 00:00:00

echo Carbon::create(2018, 2, 23, 0, 0, 0)->nextWeekday();                        // 2018-02-26 00:00:00
echo Carbon::create(2018, 2, 23, 0, 0, 0)->previousWeekday();                    // 2018-02-22 00:00:00
echo Carbon::create(2018, 2, 21, 0, 0, 0)->nextWeekendDay();                     // 2018-02-24 00:00:00
echo Carbon::create(2018, 2, 21, 0, 0, 0)->previousWeekendDay();                 // 2018-02-18 00:00:00
```

Rounding is also available for any unit:
```php
$dt = new Carbon('2012-01-31 15:32:45.654321');
echo $dt->roundMillisecond()->format('H:i:s.u');   // 15:32:45.654000

$dt = new Carbon('2012-01-31 15:32:45.654321');
echo $dt->roundSecond()->format('H:i:s.u');        // 15:32:46.000000

$dt = new Carbon('2012-01-31 15:32:45.654321');
echo $dt->floorSecond()->format('H:i:s.u');        // 15:32:45.000000

$dt = new Carbon('2012-01-31 15:32:15');
echo $dt->roundMinute()->format('H:i:s');          // 15:32:00

$dt = new Carbon('2012-01-31 15:32:15');
echo $dt->ceilMinute()->format('H:i:s');           // 15:33:00

// and so on up to millennia!

// precision rounding can be set, example: rounding to ten minutes
$dt = new Carbon('2012-01-31 15:32:15');
echo $dt->roundMinute(10)->format('H:i:s');        // 15:30:00

// and round, floor and ceil methods are shortcut for second rounding:
$dt = new Carbon('2012-01-31 15:32:45.654321');
echo $dt->round()->format('H:i:s.u');              // 15:32:46.000000
$dt = new Carbon('2012-01-31 15:32:45.654321');
echo $dt->floor()->format('H:i:s.u');              // 15:32:45.000000
$dt = new Carbon('2012-01-31 15:32:45.654321');
echo $dt->ceil()->format('H:i:s.u');               // 15:32:46.000000

// you can also pass the unit dynamically (and still precision as second argument):
$dt = new Carbon('2012-01-31');
echo $dt->roundUnit('month', 2)->format('Y-m-d');  // 2012-03-01
$dt = new Carbon('2012-01-31');
echo $dt->floorUnit('month')->format('Y-m-d');     // 2012-03-01
$dt = new Carbon('2012-01-31');
echo $dt->ceilUnit('month', 4)->format('Y-m-d');   // 2012-05-01
```
