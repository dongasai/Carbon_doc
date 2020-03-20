## CarbonInterval

The CarbonInterval class is [inherited](http://php.net/manual/en/keyword.extends.php) from the PHP [DateInterval](http://php.net/manual/en/class.dateinterval.php) class.

```
<?php
class CarbonInterval extends \DateInterval
{
    // code here
}
```
您可以通过以下方式创建实例：

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
// 年，月，周，天，小时，分钟，秒，微秒
echo CarbonInterval :: create（2，0，5，1，1，2，2，7，123）; // 2年5周1天1小时2分钟7秒
echo "\n";
echo CarbonInterval::createFromDateString('3 months'); // 3 months
```

如果发现自己从另一个库继承了一个`\DateInterval`实例，请不要担心！ 您可以通过友好的`instance()`函数创建一个`CarbonInterval`实例。

```php
$di = new \DateInterval('P1Y2M'); // <== instance from another API
$ci = CarbonInterval::instance($di);
echo get_class($ci);                                   // 'Carbon\CarbonInterval'
echo $ci;                                              // 1 year 2 months
```

相反，您可以从`CarbonInterval`中提取原始的`DateInterval`，甚至将其转换为任何继承`DateInterval`的类。

```php
$ci = CarbonInterval::days(2);
$di = $ci->toDateInterval();
echo get_class($di);   // 'DateInterval'
echo $di->d;           // 2

// 您的自定义类还可以继承CarbonInterval
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
CarbonInterval继承了DateInterval，您也可以使用[ISO-8601持续时间格式](https://en.wikipedia.org/wiki/ISO_8601#Durations) 两种方法创建：

```php
$ci = CarbonInterval::create('P1Y2M3D');
var_dump($ci->isEmpty()); // bool(false)
$ci = new CarbonInterval('PT0S');
var_dump($ci->isEmpty()); // bool(true)
```
Carbon借助`fromString()`方法，可以从通俗易懂的字符串创建碳间隔。
```php
CarbonInterval::fromString('2 minutes 15 seconds');
CarbonInterval::fromString('2m 15s'); // or abbreviated
```

或从另一个`DateInterval`/`CarbonInterval`对象创建它。

```php
$ci = new CarbonInterval(new DateInterval('P1Y2M3D'));
var_dump($ci->isEmpty()); // bool(false)
```
请注意，月份缩写为"mo"以区别分钟，并且整个语法不区分大小写。

它还有一个快捷方法的`forHumans()`，它映射为`__toString()`实现，可打印易于阅读的时间间隔。


```php
CarbonInterval::setLocale('fr');
echo CarbonInterval::create(2, 1)->forHumans();        // 2 ans 1 mois
echo CarbonInterval::hour()->seconds(3);               // 1 heure 3 secondes
CarbonInterval::setLocale('en');
```

如您所见，您可以使用来更改字符串的语言环境`CarbonInterval::setLocale('fr')`.

至于Carbon，您可以使用make方法从其他间隔或字符串中返回CarbonInterval的新实例：

```php
$dateInterval = new DateInterval('P2D');
$carbonInterval = CarbonInterval::month();
echo CarbonInterval::make($dateInterval)->forHumans();       // 2 days
echo CarbonInterval::make($carbonInterval)->forHumans();     // 1 month
echo CarbonInterval::make('PT3H')->forHumans();              // 3 hours
echo CarbonInterval::make('1h 15m')->forHumans();            // 1 hour 15 minutes
// forHumans有很多选项，自版本2.9.0起，建议的方式是将它们作为关联数组传递:
echo CarbonInterval::make('1h 15m')->forHumans(['short' => true]); // 1h 15m
$interval = CarbonInterval::make('1h 15m 45s');
echo $interval->forHumans(['join' => true]);                 // 1 hour, 15 minutes and 45 seconds
$esInterval = CarbonInterval::make('1h 15m 45s');
echo $esInterval->forHumans(['join' => true]);               // 1 hour, 15 minutes and 45 seconds
echo $interval->forHumans(['join' => true, 'parts' => 2]);   // 1 hour and 15 minutes
echo $interval->forHumans(['join' => ' - ']);                // 1 hour - 15 minutes - 45 seconds

// 可用的语法模式:
// 以前/现在（在当前语言环境中翻译）
echo $interval->forHumans(['syntax' => CarbonInterface::DIFF_RELATIVE_TO_NOW]);      // 1 hour 15 minutes 45 seconds ago
// 之前/之后（在当前语言环境中翻译）
echo $interval->forHumans(['syntax' => CarbonInterface::DIFF_RELATIVE_TO_OTHER]);    // 1 hour 15 minutes 45 seconds before
// 默认值（无前缀/后缀）：
echo $interval->forHumans(['syntax' => CarbonInterface::DIFF_ABSOLUTE]);             // 1 hour 15 minutes 45 seconds

// 可用选项:
// 将空白转换为“just now”:
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
加，减（或减），时间，份额，乘和除法允许进行间隔计算：

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
您可以从输入单位中获得纯计算。 要将分钟分为几小时，将几小时变为几天等，请使用级联方法：

```php
echo $interval->forHumans();             // 10 hours 34 minutes
echo $interval->cascade()->forHumans();  // 10 hours 34 minutes
```

默认是：

- 1 minute = 60 seconds
- 1 hour = 60 minutes
- 1 day = 24 hour
- 1 week = 7 days
- 1 month = 4 weeks
- 1 year = 12 months

CarbonIntervals不带有上下文，因此它们不能更精确（没有夏令时，没有闰年，没有实际月份长度或年份长度考虑）。 但是您可以完全自定义这些因素。 例如处理工作时间日志：

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

// 您可以使用getFactor查看当前设置的因素:
echo CarbonInterval::getFactor('minutes', /* per */ 'hour');                 // 60
echo CarbonInterval::getFactor('days', 'week');                              // 5

// 公因子可以通过一下快捷方法获取:
echo CarbonInterval::getDaysPerWeek();                                       // 5
echo CarbonInterval::getHoursPerDay();                                       // 8
echo CarbonInterval::getMinutesPerHour();                                    // 60
echo CarbonInterval::getSecondsPerMinute();                                  // 60
echo CarbonInterval::getMillisecondsPerSecond();                             // 1000
echo CarbonInterval::getMicrosecondsPerMillisecond();                        // 1000

CarbonInterval::setCascadeFactors($cascades); // restore original factors
```
是否可以将间隔转换为给定的单位（使用提供的级联系数）。

```php
echo CarbonInterval::days(3)->hours(5)->total('hours');    // 77
echo CarbonInterval::days(3)->hours(5)->totalHours;        // 77
echo CarbonInterval::months(6)->totalWeeks;                // 24
echo CarbonInterval::year()->totalDays;                    // 336
```
`->total`方法和属性需要级联间隔，如果间隔可能溢出，请在调用以下功能之前将它们级联：

```php
echo CarbonInterval::minutes(1200)->cascade()->total('hours'); // 20
echo CarbonInterval::minutes(1200)->cascade()->totalHours; // 20
```

您还可以通过`spec()`获取周期的ISO 8601规范
```php
echo CarbonInterval::days(3)->hours(5)->spec(); // P3DT5H
```
也可以使用静态函数助手从DateInterval对象中获取：
```php
echo CarbonInterval::getDateIntervalSpec(new DateInterval('P3DT6M10S')); // P3DT6M10S
```

可以使用`compare()`和`compareDateIntervals()`方法对日期周期列表进行排序：

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

将周期值转储为具有以下内容的数组：

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


最后，可以通过使用互补参数调用`toPeriod()`将CarbonInterval实例转换为CarbonPeriod实例。

我听到你问什么是CarbonPeriod实例。 哦! 完美过渡到下一章。



