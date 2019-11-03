## Weeks

If you are familiar with momentjs, you will find all week methods working the same. Most of them have an iso{Method} variant. Week methods follow the rules of the current locale (for example with en_US, the default locale, the first day of the week is Sunday, and the first week of the year is the one that contains January 1st). ISO methods follow the ISO 8601 norm, meaning weeks start with Monday and the first week of the year is the one containing January 4th.

```php
$en = CarbonImmutable::now()->locale('en_US');
$ar = CarbonImmutable::now()->locale('ar');

var_dump($en->firstWeekDay);                           // int(0)
var_dump($en->lastWeekDay);                            // int(6)
var_dump($en->startOfWeek()->format('Y-m-d H:i'));     // string(16) "2019-10-13 00:00"
var_dump($en->endOfWeek()->format('Y-m-d H:i'));       // string(16) "2019-10-19 23:59"

echo "-----------\n";

// We still can force to use an other day as start/end of week
$start = $en->startOfWeek(Carbon::TUESDAY);
$end = $en->endOfWeek(Carbon::MONDAY);
var_dump($start->format('Y-m-d H:i'));                 // string(16) "2019-10-08 00:00"
var_dump($end->format('Y-m-d H:i'));                   // string(16) "2019-10-14 23:59"

echo "-----------\n";

var_dump($ar->firstWeekDay);                           // int(6)
var_dump($ar->lastWeekDay);                            // int(5)
var_dump($ar->startOfWeek()->format('Y-m-d H:i'));     // string(16) "2019-10-12 00:00"
var_dump($ar->endOfWeek()->format('Y-m-d H:i'));       // string(16) "2019-10-18 23:59"

$en = CarbonImmutable::parse('2015-02-05'); // use en_US as default locale

echo "-----------\n";

var_dump($en->weeksInYear());                          // int(52)
var_dump($en->isoWeeksInYear());                       // int(53)

$en = CarbonImmutable::parse('2017-02-05');

echo "-----------\n";

var_dump($en->week());                                 // int(6)
var_dump($en->isoWeek());                              // int(5)
var_dump($en->week(1)->format('Y-m-d H:i'));           // string(16) "2017-01-01 00:00"
var_dump($en->isoWeek(1)->format('Y-m-d H:i'));        // string(16) "2017-01-08 00:00"
var_dump($en->weekday());                              // int(0)
var_dump($en->isoWeekday());                           // int(7)
var_dump($en->weekday(3)->format('Y-m-d H:i'));        // string(16) "2017-02-08 00:00"
var_dump($en->isoWeekday(3)->format('Y-m-d H:i'));     // string(16) "2017-02-01 00:00"

$en = CarbonImmutable::parse('2017-01-01');

echo "-----------\n";

var_dump($en->weekYear());                             // int(2017)
var_dump($en->isoWeekYear());                          // int(2016)
var_dump($en->weekYear(2016)->format('Y-m-d H:i'));    // string(16) "2015-12-27 00:00"
var_dump($en->isoWeekYear(2016)->format('Y-m-d H:i')); // string(16) "2017-01-01 00:00"
var_dump($en->weekYear(2015)->format('Y-m-d H:i'));    // string(16) "2014-12-28 00:00"
var_dump($en->isoWeekYear(2015)->format('Y-m-d H:i')); // string(16) "2015-12-27 00:00"

// Note you still can force first day of week and year to use:
$en = CarbonImmutable::parse('2017-01-01');

echo "-----------\n";

var_dump($en->weeksInYear(null, 6, 12));               // int(52)
var_dump($en->isoWeeksInYear(null, 6, 12));            // int(52)
var_dump($en->week(null, 6, 12));                      // int(52)
var_dump($en->isoWeek(null, 6, 12));                   // int(52)
var_dump($en->weekYear(null, 6, 12));                  // int(2016)
var_dump($en->isoWeekYear(null, 6, 12));               // int(2016)
var_dump($en->weekYear(2016, 6, 12)->format('Y-m-d H:i')); // string(16) "2017-01-01 00:00"
var_dump($en->isoWeekYear(2016, 6, 12)->format('Y-m-d H:i')); // string(16) "2017-01-01 00:00"
// Then you can see using a method or its ISO variant return identical results
```