## 升级到Carbon 2

如果您打算从Carbon 1迁移到Carbon 2，请注意以下重要更改，您应该注意。

* 现在:`::create()`方法中`$month`和`$day`的默认值（省略参数时）现在为`1`(Carbon 1中的当前日期的值). `$hour`，`$minute`和`$second`的默认值现在为`0`，省略的值也可以使用，但是您仍然可以显式传递`null`来获取当前值（类似于 在Carbon 1中）。
* 现在，您可以在任何地方获得微秒精度，这也意味着在同一秒内但不在同一微秒内的2个日期不再相等。
* `$date->jsonSerialize()`和`json_encode($date)`不再返回数组而是返回简单的字符串：`"2017-06-27T13:14:15.000000Z"`。这允许使用JavaScript更容易创建日期。您仍然可以使用以下方法获取以前的行为：
```php
    Carbon::serializeUsing(function ($date) {
        return [
            'date' => $date->toDateTimeString(),
        ] + (array) $date->tz;
    });
```
* 带有闭包的 `$date->setToStringFormat()`不再返回格式而是返回最终字符串。因此，您可以在Carbon 1中返回任何字符串和以下内容：
```php
    Carbon::setToStringFormat(function ($date) {
        return $date->year === 1976 ?
            'jS \o\f F g:i:s a' :
            'jS \o\f F, Y g:i:s a';
    });
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Carbon 2中的样子:
```php
Carbon::setToStringFormat(function ($date) {
    return $date->formatLocalized($date->year === 1976 ?
        'jS \o\f F g:i:s a' :
        'jS \o\f F, Y g:i:s a'
    );
});
```
* `setWeekStartsAt`和`setWeekEndsAt`不再对超出范围的值抛出异常，但它们也被拒绝使用。

* `isSameMonth`和`isCurrentMonth`现在在不同年份的同一个月返回false，但您可以将false作为`isSameMonth`的第二个参数或`isCurrentMonth`的第一个参数传递给忽略年份的比较。
* `::CompareYarWithMonth()`和`::CompareYarWithMonth()`。现在默认是严格比较的。并且您可以使用isSame/isCurrent设置为false的下一个参数来获得仅月份的比较。
* 当我们不再兼容PHP5时，mixins中不再需要`$self`，您应该只使用`$this`。
* 由于PHP 7.1+完全支持微秒，因此不再需要`useMicrosecondsFallback`和`ismicrosofallbackenabled`，因此已被删除。
* 在Carbon 1中，调用`CarbonInterval`上的未知方法（例如：`CarbonInterval::anything())`只是返回null。 现在他们抛出异常。
* 在Carbon 1中，`dayOfYear`从0开始。现在从1开始。
* 这就是所有不同！您在Carbon 1中使用的每种其他方法应继续与Carbon 2相同。
