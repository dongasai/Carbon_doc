## Migrate to Carbon 2

If you plan to migrate from Carbon 1 to Carbon 2, please note the following breaking changes you should take care of.

* Default values (when parameters are omitted) for `$month` and `$day` in the `::create()` method are now `1` (were values from current date in Carbon 1). And default values for `$hour`, `$minute` and `$second` are now `0`, this goes for omitted values, but you still can pass explicitly `null` to get the current value from now (similar behavior as in Carbon 1).
* Now you get microsecond precision everywhere, it also means 2 dates in the same second but not in the same microsecond are no longer equal.
* `$date->jsonSerialize()` and `json_encode($date)` no longer returns arrays but simple strings: `"2017-06-27T13:14:15.000000Z"`. This allows to create dates from it easier in JavaScript. You still can get the previous behavior using:
```php
    Carbon::serializeUsing(function ($date) {
        return [
            'date' => $date->toDateTimeString(),
        ] + (array) $date->tz;
    });
```
* `$date->setToStringFormat()` with a closure no longer return a format but a final string. So you can return any string and the following in Carbon 1:
```php
    Carbon::setToStringFormat(function ($date) {
        return $date->year === 1976 ?
            'jS \o\f F g:i:s a' :
            'jS \o\f F, Y g:i:s a';
    });
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;would become in Carbon 2:
```php
Carbon::setToStringFormat(function ($date) {
    return $date->formatLocalized($date->year === 1976 ?
        'jS \o\f F g:i:s a' :
        'jS \o\f F, Y g:i:s a'
    );
});
```
* `setWeekStartsAt` and `setWeekEndsAt` no longer throw exceptions on values out of ranges, but they are also deprecated.

* `isSameMonth` and `isCurrentMonth` now returns false for same month in different year but you can pass false as a second parameter of `isSameMonth` or first parameter of `isCurrentMonth` to compare ignoring the year.
* `::compareYearWithMonth()` and `::compareYearWithMonth()` have been removed. Strict comparisons are now the default. And you can use the next parameter of isSame/isCurrent set to false to get month-only comparisons.
* As we dropped PHP 5, `$self` is no longer needed in mixins you should just use `$this` instead.
* As PHP 7.1+ perfectly supports microseconds, `useMicrosecondsFallback` and `isMicrosecondsFallbackEnabled` are no longer needed and so have been removed.
* In Carbon 1, calls of an unknown method on `CarbonInterval` (ex: `CarbonInterval::anything())` just returned null. Now they throw an exception.
* In Carbon 1, `dayOfYear` started from 0. Now it starts from 1.
* That's all folks! Every other methods you used in Carbon 1 should continue to work just the same with Carbon 2.
