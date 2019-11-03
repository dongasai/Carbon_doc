## Getter

The getters are implemented via PHP's `__get()` method. This enables you to access the value as if it was a property rather than a function call.

```php
$dt = Carbon::parse('2012-10-5 23:26:11.123789');

// These getters specifically return integers, ie intval()
var_dump($dt->year);                                         // int(2012)
var_dump($dt->month);                                        // int(10)
var_dump($dt->day);                                          // int(5)
var_dump($dt->hour);                                         // int(23)
var_dump($dt->minute);                                       // int(26)
var_dump($dt->second);                                       // int(11)
var_dump($dt->micro);                                        // int(123789)
// dayOfWeek returns a number between 0 (sunday) and 6 (saturday)
var_dump($dt->dayOfWeek);                                    // int(5)
// dayOfWeekIso returns a number between 1 (monday) and 7 (sunday)
var_dump($dt->dayOfWeekIso);                                 // int(5)
var_dump($dt->englishDayOfWeek);                             // string(6) "Friday"
var_dump($dt->shortEnglishDayOfWeek);                        // string(3) "Fri"
var_dump($dt->locale('de')->dayName);                        // string(7) "Freitag"
var_dump($dt->locale('de')->shortDayName);                   // string(3) "Fr."
var_dump($dt->locale('de')->minDayName);                     // string(2) "Fr"
var_dump($dt->englishMonth);                                 // string(7) "October"
var_dump($dt->shortEnglishMonth);                            // string(3) "Oct"
var_dump($dt->locale('de')->monthName);                      // string(7) "Oktober"
var_dump($dt->locale('de')->shortMonthName);                 // string(3) "Okt"

// Following are deprecated, locale* and shortLocale* properties
// are translated using formatLocalized() based on LC_TIME language.
setlocale(LC_TIME, 'German');
var_dump($dt->localeDayOfWeek);                              // string(7) "Freitag"
var_dump($dt->shortLocaleDayOfWeek);                         // string(2) "Fr"
var_dump($dt->localeMonth);                                  // string(7) "Oktober"
var_dump($dt->shortLocaleMonth);                             // string(3) "Okt"
setlocale(LC_TIME, '');

var_dump($dt->dayOfYear);                                    // int(279)
var_dump($dt->weekNumberInMonth);                            // int(1)
// weekNumberInMonth consider weeks from monday to sunday, so the week 1 will
// contain 1 day if the month start with a sunday, and up to 7 if it starts with a monday
var_dump($dt->weekOfMonth);                                  // int(1)
// weekOfMonth will returns 1 for the 7 first days of the month, then 2 from the 8th to
// the 14th, 3 from the 15th to the 21st, 4 from 22nd to 28th and 5 above
var_dump($dt->weekOfYear);                                   // int(40)
var_dump($dt->daysInMonth);                                  // int(31)
var_dump($dt->timestamp);                                    // int(1349479571)
// Millisecond-precise timestamp (useful to pass it to JavaScript)
var_dump($dt->valueOf());                                    // float(1349479571124)
// Custom-precision timestamp
var_dump($dt->getPreciseTimestamp(6));                       // float(1.3494795711238E+15)
var_dump(Carbon::createFromDate(1975, 5, 21)->age);          // int(44) calculated vs now in the same tz
var_dump($dt->quarter);                                      // int(4)

// Returns an int of seconds difference from UTC (+/- sign included)
var_dump(Carbon::createFromTimestampUTC(0)->offset);         // int(0)
var_dump(Carbon::createFromTimestamp(0, 'Europe/Paris')->offset);                // int(3600)
var_dump(Carbon::createFromTimestamp(0, 'Europe/Paris')->getOffset());           // int(3600)

// Returns an int of hours difference from UTC (+/- sign included)
var_dump(Carbon::createFromTimestamp(0, 'Europe/Paris')->offsetMinutes);         // int(60)
var_dump(Carbon::createFromTimestamp(0, 'Europe/Paris')->offsetHours);           // int(1)

// Returns timezone offset as string
var_dump(Carbon::createFromTimestamp(0, 'Europe/Paris')->getOffsetString());     // string(6) "+01:00"

// Returns timezone as CarbonTimeZone
var_dump(Carbon::createFromTimestamp(0, 'Europe/Paris')->getTimezone());
/* object(Carbon\CarbonTimeZone)#3587 (2) {
  ["timezone_type"]=>
  int(3)
  ["timezone"]=>
  string(12) "Europe/Paris"
} */

// Indicates if day light savings time is on
var_dump(Carbon::createFromDate(2012, 1, 1)->dst);           // bool(false)
var_dump(Carbon::createFromDate(2012, 9, 1)->dst);           // bool(false)
var_dump(Carbon::createFromDate(2012, 9, 1)->isDST());       // bool(false)

// Indicates if the instance is in the same timezone as the local timezone
var_dump(Carbon::now()->local);                              // bool(true)
var_dump(Carbon::now('America/Vancouver')->local);           // bool(false)
var_dump(Carbon::now()->isLocal());                          // bool(true)
var_dump(Carbon::now('America/Vancouver')->isLocal());       // bool(false)
var_dump(Carbon::now()->isUtc());                            // bool(true)
var_dump(Carbon::now('America/Vancouver')->isUtc());         // bool(false)
// can also be written ->isUTC()

// Indicates if the instance is in the UTC timezone
var_dump(Carbon::now()->utc);                                // bool(true)
// London is not UTC on summer time
var_dump(Carbon::parse('2018-10-01', 'Europe/London')->utc); // bool(false)
// London is UTC on winter time
var_dump(Carbon::parse('2018-11-01', 'Europe/London')->utc); // bool(true)
var_dump(Carbon::createFromTimestampUTC(0)->utc);            // bool(true)

// Gets the DateTimeZone instance
echo get_class(Carbon::now()->timezone);                     // Carbon\CarbonTimeZone
echo "\n";
echo get_class(Carbon::now()->tz);                           // Carbon\CarbonTimeZone
echo "\n";

// Gets the DateTimeZone instance name, shortcut for ->timezone->getName()
echo Carbon::now()->timezoneName;                            // UTC
echo "\n";
echo Carbon::now()->tzName;                                  // UTC
echo "\n";

// You can get any property dynamically too:
$unit = 'second';
var_dump(Carbon::now()->get($unit));                         // int(8)
// equivalent to:
var_dump(Carbon::now()->$unit);                              // int(8)
// If you have plural unit name, use singularUnit()
$unit = Carbon::singularUnit('seconds');
var_dump(Carbon::now()->get($unit));                         // int(8)
// Prefer using singularUnit() because some plurals are not the word with S:
var_dump(Carbon::pluralUnit('century'));                     // string(9) "centuries"
var_dump(Carbon::pluralUnit('millennium'));                  // string(9) "millennia"

```