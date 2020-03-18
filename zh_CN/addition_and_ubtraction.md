# 增加和减少

默认的DateTime提供了几种不同的方法来轻松地增加和减少时间。 有`modify()`，`add()`和`sub()`。 `modify()`采用一个标准的日期/时间格式字符串，即`下个月的最后一天`，它解析并应用修改，而`add()`和`sub()`期望一个`DateInterval`实例 （例如，`new \DateInterval('P6YT5M')`意味着6年5分钟）。 希望在几个星期没看到代码后，使用这些流利的函数会更清晰，更容易阅读。 但是，我当然不会让您选择，因为基类函数仍然可用。
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

// 对于任何单位，依此类推：千年，世纪，十年，年，季度，月，周，日，工作日,
// 时，分，秒，微秒.

// Generic methods add/sub (or subtract alias) can take many different arguments:
echo $dt->add(61, 'seconds');                      // 2012-02-03 00:01:01
echo $dt->sub('1 day');                            // 2012-02-02 00:01:01
echo $dt->add(CarbonInterval::months(2));          // 2012-04-02 00:01:01
echo $dt->subtract(new DateInterval('PT1H'));      // 2012-04-01 23:01:01
```
为了优雅，您还可以将负值传递给`addXXX()`，实际上，这也是实现`subXXX()`的方式。

钉:  如果您忘记了并使用`addDay(5)`或`subYear(3)`，请不要担心，我有您的支持；）

默认情况下，Carbon依赖于基础父类PHP DateTime行为。 结果，增加或减少月份可能会溢出，例如：

```php
$dt = CarbonImmutable::create(2017, 1, 31, 0);

echo $dt->addMonth();                    // 2017-03-03 00:00:00
echo "\n";
echo $dt->subMonths(2);                  // 2016-12-01 00:00:00
```
从Carbon 2开始，您可以为每个实例设置本地溢出行为：

```php
$dt = CarbonImmutable::create(2017, 1, 31, 0);
$dt->settings([
    'monthOverflow' => false,
]);

echo $dt->addMonth();                    // 2017-02-28 00:00:00
echo "\n";
echo $dt->subMonths(2);                  // 2016-11-30 00:00:00
```
这将适用于方法`addMonth(s)`,`subMonth(s)`,`add($x,'month')`,`sub($x， 'month')`和等效的季度方法。但它不适用于interval对象或字符串，如`add(CarbonInterval::month())`或`add('1 month')`。

静态助手可用，但已弃用。 如果您确定需要应用全局设置或使用Carbon的版本1，请[检查溢出静态帮助器部分](https://carbon.nesbot.com/docs/#overflow-static-helpers)

您还可以使用`->addMonthsNoOverflow`，`->subMonthsNoOverflow`，`-> addMonthsWithOverflow`和`->subMonthsWithOverflow`（或不带“ s”到“ month”的单数方法）来显式添加/删除月份，无论是否有溢出 无论当前模式如何，对于任何更大的单位（季度，年份，十年，世纪，千年）结果都相同。

```php
$dt = Carbon::createMidnightDate(2017, 1, 31)->settings([
    'monthOverflow' => false,
]);

echo $dt->copy()->addMonthWithOverflow();          // 2017-03-03 00:00:00
// 也可以使用复数addMonthsWithOverflow()方法
echo $dt->copy()->subMonthsWithOverflow(2);        // 2016-12-01 00:00:00
// 也可以使用单数方法subMonthWithOverflow()
echo $dt->copy()->addMonthNoOverflow();            // 2017-02-28 00:00:00
// 也可以使用复数方法addMonthsNoOverflow() 
echo $dt->copy()->subMonthsNoOverflow(2);          // 2016-11-30 00:00:00
//也可是使用单数方法 subMonthNoOverflow() 

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

多年来也是如此。

使用未知输入时，您还可以控制任何单元的溢出：
```php
$dt = CarbonImmutable::create(2018, 8, 30, 12, 00, 00);

// 增加时数而不增加日
echo $dt->addUnitNoOverflow('hour', 7, 'day');     // 2018-08-30 19:00:00
echo "\n";
echo $dt->addUnitNoOverflow('hour', 14, 'day');    // 2018-08-30 23:59:59
echo "\n";
echo $dt->addUnitNoOverflow('hour', 48, 'day');    // 2018-08-30 23:59:59

echo "\n-------\n";

// 减少小时二不增加天数
echo $dt->subUnitNoOverflow('hour', 7, 'day');     // 2018-08-30 05:00:00
echo "\n";
echo $dt->subUnitNoOverflow('hour', 14, 'day');    // 2018-08-30 00:00:00
echo "\n";
echo $dt->subUnitNoOverflow('hour', 48, 'day');    // 2018-08-30 00:00:00

echo "\n-------\n";

// 设置小时数而不增加天数
echo $dt->setUnitNoOverflow('hour', -7, 'day');    // 2018-08-30 00:00:00
echo "\n";
echo $dt->setUnitNoOverflow('hour', 14, 'day');    // 2018-08-30 14:00:00
echo "\n";
echo $dt->setUnitNoOverflow('hour', 25, 'day');    // 2018-08-30 23:59:59

echo "\n-------\n";

// 增加小时数而不增加月数
echo $dt->addUnitNoOverflow('hour', 7, 'month');   // 2018-08-30 19:00:00
echo "\n";
echo $dt->addUnitNoOverflow('hour', 14, 'month');  // 2018-08-31 02:00:00
echo "\n";
echo $dt->addUnitNoOverflow('hour', 48, 'month');  // 2018-08-31 23:59:59

```
任何可修改的单位都可以作为这些方法的参数传递：

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