
## Localization
---

With Carbon 2, localization changed a lot, 751 new locales are supported and we now embed locale formats, day names, month names, ordinal suffixes, meridiem, week start and more. While Carbon 1 provided partial support and relied on third-party like IntlDateFormatter class and language packages for advanced translation, you now benefit of a wide internationalization support. You still use Carbon 1? I hope you would consider to upgrade, version 2 has really cool new features. Else you still read the [version 1 documentation of Localization by clicking here](https://carbon.nesbot.com/docs/#localization-v1).

Since 2.9.0, you can easily customize translations:

```php
// we recommend to use custom language name/variant
// rather than overriding an existing language
// to avoid conflict such as "en_Boring" in the example below:
$boringLanguage = 'en_Boring';
$translator = \Carbon\Translator::get($boringLanguage);
$translator->setTranslations([
    'day' => ':count boring day|:count boring days',
]);
// as this language starts with "en_" it will inherit from the locale "en"

$date1 = Carbon::create(2018, 1, 1, 0, 0, 0);
$date2 = Carbon::create(2018, 1, 4, 4, 0, 0);

echo $date1->locale($boringLanguage)->diffForHumans($date2); // 3 boring days before

$translator->setTranslations([
    'before' => function ($time) {
        return '['.strtoupper($time).']';
    },
]);

echo $date1->locale($boringLanguage)->diffForHumans($date2); // [3 BORING DAYS]
```

You can use fallback locales by passing in order multiple ones to `locale()`:

```php
\Carbon\Translator::get('xx')->setTranslations([
    'day' => ':count Xday',
]);
\Carbon\Translator::get('xy')->setTranslations([
    'day' => ':count Yday',
    'hour' => ':count Yhour',
]);

$date = Carbon::now()->locale('xx', 'xy', 'es')->sub('3 days 6 hours 40 minutes');

echo $date->ago(['parts' => 3]); // hace 3 Xday 6 Yhour 40 minutos
```

In the example above, it will try to find translations in "xx" in priority, then in "xy" if missing, then in "es", so here, you get "Xday" from "xx", "Yday" from "xy", and "hace" and "minutos" from "es".

Note that you can also use an other translator with `Carbon::setTranslator($custom)` as long as the given translator implements `Symfony\Component\Translation\TranslatorInterface`. And you can get the global default translator using `Carbon::getTranslator()` (and `Carbon::setFallbackLocale($custom)` and `Carbon::getFallbackLocale()` for the fallback locale, setFallbackLocale can be called multiple times to get multiple fallback locales) but as those method will change the behavior globally (including third-party libraries you may have in your app), it might cause unexpected results. You should rather customize translation using custom locales as in the example above.

Carbon embed a default translator that extends [Symfony\Component\Translation\Translator](https://api.symfony.com/2.7/Symfony/Component/Translation/Translator.html) You can [check here the methods we added to it](https://carbon.nesbot.com/docs/#translator-details).

So the support of a locale for `formatLocalized`, getters such as `localeMonth`, `localeDayOfWeek` and short variants is driven by locales installed in your operating system. For other translations, it's supported internally thanks to Carbon community. You can check what's supported with the following methods:

```php
echo implode(', ', array_slice(Carbon::getAvailableLocales(), 0, 3)).'...';      // aa, aa_DJ, aa_ER...

// Support diff syntax (before, after, from now, ago)
var_dump(Carbon::localeHasDiffSyntax('en'));                                     // bool(true)
var_dump(Carbon::localeHasDiffSyntax('zh_TW'));                                  // bool(true)
// Support 1-day diff words (just now, yesterday, tomorrow)
var_dump(Carbon::localeHasDiffOneDayWords('en'));                                // bool(true)
var_dump(Carbon::localeHasDiffOneDayWords('zh_TW'));                             // bool(false)
// Support 2-days diff words (before yesterday, after tomorrow)
var_dump(Carbon::localeHasDiffTwoDayWords('en'));                                // bool(true)
var_dump(Carbon::localeHasDiffTwoDayWords('zh_TW'));                             // bool(false)
// Support short units (1y = 1 year, 1mo = 1 month, etc.)
var_dump(Carbon::localeHasShortUnits('en'));                                     // bool(true)
var_dump(Carbon::localeHasShortUnits('zh_TW'));                                  // bool(false)
// Support period syntax (X times, every X, from X, to X)
var_dump(Carbon::localeHasPeriodSyntax('en'));                                   // bool(true)
var_dump(Carbon::localeHasPeriodSyntax('zh_TW'));                                // bool(false)
```

So, here is the new recommended way to handle internationalization with Carbon.

```php
$date = Carbon::now()->locale('fr_FR');

echo $date->locale();            // fr_FR
echo "\n";
echo $date->diffForHumans();     // il y a 1 seconde
echo "\n";
echo $date->monthName;           // octobre
echo "\n";
echo $date->isoFormat('LLLL');   // lundi 14 octobre 2019 14:49
```

`The ->locale()` method only change the language for the current instance and has precedence over global settings. We recommend you this approach so you can't have conflict with other places or third-party libraries that could use Carbon. Nevertheless, to avoid calling `->locale()` each time, you can use factories.

```php
// Let say Martin from Paris and John from Chicago play chess
$martinDateFactory = new Factory([
    'locale' => 'fr_FR',
    'timezone' => 'Europe/Paris',
]);
$johnDateFactory = new Factory([
    'locale' => 'en_US',
    'timezone' => 'America/Chicago',
]);
// Each one will see date in his own language and timezone

// When Martin moves, we display things in French, but we notify John in English:
$gameStart = Carbon::parse('2018-06-15 12:34:00', 'UTC');
$move = Carbon::now('UTC');
$toDisplay = $martinDateFactory->make($gameStart)->isoFormat('lll')."\n".
    $martinDateFactory->make($move)->calendar()."\n";
$notificationForJohn = $johnDateFactory->make($gameStart)->isoFormat('lll')."\n".
    $johnDateFactory->make($move)->calendar()."\n";
echo $toDisplay;
/*
15 juin 2018 12:34
Aujourd’hui à 14:49
*/

echo $notificationForJohn;
/*
Jun 15, 2018 12:34 PM
Today at 2:49 PM
*/
```

You can call any static Carbon method on a factory (make, now, yesterday, tomorrow, parse, create, etc.) Factory (and FactoryImmutable that generates CarbonImmutable instances) are the best way to keep things organized and isolated. As often as possible we recommend you to work with UTC dates, then apply locally (or with a factory) the timezone and the language before displaying dates to the user.

What factory actually do is using the method name as static constructor then call `settings()` method which is a way to group in one call settings of locale, timezone, months/year overflow, etc. ([See references for complete list](https://carbon.nesbot.com/docs/#doc-method-Carbon-settings).)

```php
$factory = new Factory([
    'locale' => 'fr_FR',
    'timezone' => 'Europe/Paris',
]);
$factory->now(); // You can recall $factory as needed to generate new instances with same settings
// is equivalent to:
Carbon::now()->settings([
    'locale' => 'fr_FR',
    'timezone' => 'Europe/Paris',
]);
// Important note: timezone setting calls ->shiftTimezone() and not ->setTimezone(),
// It means it does not just set the timezone, but shift the time too:
echo Carbon::today()->setTimezone('Asia/Tokyo')->format('d/m G\h e');        // 14/10 9h Asia/Tokyo
echo "\n";
echo Carbon::today()->shiftTimezone('Asia/Tokyo')->format('d/m G\h e');      // 14/10 0h Asia/Tokyo
```

Factory settings can be changed afterward with `setSettings(array $settings) `or to merge new settings with existing ones `mergeSettings(array $settings)`and the class to generate can be initialized as the second argument of the construct then changed later with `setClassName(string $className)`.

```php
$factory = new Factory(['locale' => 'ja'], CarbonImmutable::class);
var_dump($factory->now()->locale);                                           // string(2) "ja"
var_dump(get_class($factory->now()));                                        // string(22) "Carbon\CarbonImmutable"

class MyCustomCarbonSubClass extends Carbon { /* ... */ }
$factory
    ->setSettings(['locale' => 'zh_CN'])
    ->setClassName(MyCustomCarbonSubClass::class);
var_dump($factory->now()->locale);                                           // string(5) "zh_CN"
var_dump(get_class($factory->now()));                                        // string(22) "MyCustomCarbonSubClass"
```

Previously there was `Carbon::setLocale` that set globally the locale. But as for our other static setters, we highly discourage you to use it. It breaks the principle of isolation because the configuration will apply for every class that uses Carbon.

You also may know `formatLocalized()` method from Carbon 1. This method still works the same in Carbon 2 but you should better use `isoFormat()` instead.

`->isoFormat(string $format)`: string use ISO format rather than PHP-specific format and use inner translations rather than language packages you need to install on every machine where you deploy your application. isoFormat method is compatible with [momentjs format method](https://momentjs.com/), it means you can use same format strings as you may have used in moment from front-end or node.js. Here are some examples:

```php
$date = Carbon::parse('2018-06-15 17:34:15.984512', 'UTC');
echo $date->isoFormat('MMMM Do YYYY, h:mm:ss a'); // June 15th 2018, 5:34:15 pm
echo "\n";
echo $date->isoFormat('dddd');           // Friday
echo "\n";
echo $date->isoFormat('MMM Do YY');      // Jun 15th 18
echo "\n";
echo $date->isoFormat('YYYY [escaped] YYYY'); // 2018 escaped 2018
```
You can also create date from ISO formatted strings:

```php
$date = Carbon::createFromIsoFormat('!YYYY-MMMM-D h:mm:ss a', '2019-January-3 6:33:24 pm', 'UTC');
echo $date->isoFormat('M/D/YY HH:mm'); // 1/3/19 18:33
```
`->isoFormat` use contextualized methods for day names and month names as they can have multiple forms in some languages, see the following examples:

```php
$date = Carbon::parse('2018-03-16')->locale('uk');
echo $date->getTranslatedDayName('[в] dddd'); // п’ятницю
// By providing a context, we're saying translate day name like in a format such as [в] dddd
// So the context itself has to be translated first consistently.
echo "\n";
echo $date->getTranslatedDayName('[наступної] dddd'); // п’ятниці
echo "\n";
echo $date->getTranslatedDayName('dddd, MMM'); // п’ятниця
echo "\n";
// The same goes for short/minified variants:
echo $date->getTranslatedShortDayName('[наступної] dd'); // пт
echo "\n";
echo $date->getTranslatedMinDayName('[наступної] ddd'); // пт
echo "\n";

// And the same goes for months
$date->locale('ru');
echo $date->getTranslatedMonthName('Do MMMM'); // марта
echo "\n";
echo $date->getTranslatedMonthName('MMMM YYYY'); // март
echo "\n";
// Short variant
echo $date->getTranslatedShortMonthName('Do MMM'); // мар
echo "\n";
echo $date->getTranslatedShortMonthName('MMM YYYY'); // мар
echo "\n";

// And so you can force a different context to get those variants:
echo $date->isoFormat('Do MMMM');        // 16-го марта
echo "\n";
echo $date->isoFormat('MMMM YYYY');      // март 2018
echo "\n";
echo $date->isoFormat('Do MMMM', 'MMMM YYYY'); // 16-го март
echo "\n";
echo $date->isoFormat('MMMM YYYY', 'Do MMMM'); // марта 2018
echo "\n";
```
Here is the complete list of available replacements (examples given with `$date = Carbon::parse('2017-01-05 17:04:05.084512')`;):


|Code|	Example|	Description|
|-|-|-|
|OD	|5	|Day number with alternative numbers such as 三 for 3 if locale is ja_JP|
|OM	|1|	Month number with alternative numbers such as ၀၂ for 2 if locale is my_MM|
|OY	|2017	|Year number with alternative numbers such as ۱۹۹۸ for 1998 if locale is fa|
|OH	|17	|24-hours number with alternative numbers such as ႑႓ for 13 if locale is shn_MM|
|Oh	|5	|12-hours number with alternative numbers such as 十一 for 11 if locale is lzh_TW|
|Om|	4|	Minute number with alternative numbers such as ୫୭ for 57 if locale is or|
|Os	|5	|Second number with alternative numbers such as 十五 for 15 if locale is ja_JP|
|D|	5|	Day of month number (from 1 to 31)|
|DD	|05	|Day of month number with trailing zero (from 01 to 31)|
|Do	|5th	|Day of month with ordinal suffix (from 1st to 31th), translatable|
|d	|4|	Day of week number (from 0 (Sunday) to 6 (Saturday))|
|dd	|Th|	Minified day name (from Su to Sa), transatable|
|ddd	|Thu	|Short day name (from Sun to Sat), transatable|
|dddd|	Thursday|	Day name (from Sunday to Saturday), transatable|
|DDD	|5	|Day of year number (from 1 to 366)|
|DDDD	|005|	Day of year number with trailing zeros (3 digits, from 001 to 366)|
|DDDo|	5th	|Day of year number with ordinal suffix (from 1st to 366th), translatable|
|e|	4	|Day of week number (from 0 (Sunday) to 6 (Saturday)), similar to "d" but this one is translatable (takes first day of week of the current locale)|
|E	|4	|Day of week number (from 1 (Monday) to 7 (Sunday))|
|H|	17	|Hour from 0 to 23|
|HH|	17|	Hour with trailing zero from 00 to 23|
|h	|5	|Hour from 0 to 12|
|hh	|05	|Hour with trailing zero from 00 to 12|
|k	|17	|Hour from 1 to 24|
|kk	|17	|Hour with trailing zero from 01 to 24|
|m	|4	|Minute from 0 to 59|
|mm|	04	|Minute with trailing zero from 00 to 59|
|a|	pm|	Meridiem am/pm|
|A	|PM|	Meridiem AM/PM|
|s|	5	|Second from 0 to 59|
|ss	|05	|Second with trailing zero from 00 to 59|
|S	|0	|Second tenth|
|SS	|08	|Second hundredth (on 2 digits with trailing zero)|
|SSS|	084	|Millisecond (on 3 digits with trailing zeros)|
|SSSS	|0845	|Second ten thousandth (on 4 digits with trailing zeros)|
|SSSSS	|08451	|Second hundred thousandth (on 5 digits with trailing zeros)|
|SSSSSS	|084512	|Microsecond (on 6 digits with trailing zeros)|
|SSSSSSS	|0845120|	Second ten millionth (on 7 digits with trailing zeros)|
|SSSSSSSS	|08451200|	Second hundred millionth (on 8 digits with trailing zeros)|
|SSSSSSSSS	|084512000|	Nanosecond (on 9 digits with trailing zeros)|
|M	|1	|Month from 1 to 12|
|MM	|01	|Month with trailing zero from 01 to 12|
|MMM	|Jan	|Short month name, translatable|
|MMMM	|January	|Month name, translatable|
|Mo	|1st	|Month with ordinal suffix from 1st to 12th, translatable|
|Q	|1	|Quarter from 1 to 4|
|Qo	|1st	|Quarter with ordinal suffix from 1st to 4th, translatable|
|G	|2017	|ISO week year (see ISO week date)|
|GG	|2017	|ISO week year (on 2 digits with trailing zero)|
|GGG	|2017	|ISO week year (on 3 digits with trailing zeros)|
|GGGG	|2017	|ISO week year (on 4 digits with trailing zeros)|
|GGGGG	|02017	|ISO week year (on 5 digits with trailing zeros)|
|g	|2017|	Week year according to locale settings, translatable|
|gg	|2017|	Week year according to locale settings (on 2 digits with trailing zero), translatable|
|ggg	|2017	|Week year according to locale settings (on 3 digits with trailing zeros), translatable|
|gggg	|2017	|Week year according to locale settings (on 4 digits with trailing zeros), translatable|
|ggggg	|02017	|Week year according to locale settings (on 5 digits with |trailing zeros), translatable|
|W	|1	|ISO week number in the year (see ISO week date)|
|WW	|01	| ISO week number in the year (on 2 digits with trailing zero)
|Wo	|1st	| ISO week number in the year with ordinal suffix, translatable|
|w	|1	|Week number in the year according to locale settings, translatable|
|ww	|01	|Week number in the year according to locale settings (on 2 digits with trailing zero)|
|wo	|1st|	Week number in the year according to locale settings with ordinal suffix, translatable|
|x	|1483635845085|	Millisecond-precision timestamp (same as date.getTime() in JavaScript)|
|X	|1483635845	|Timestamp (number of seconds since 1970-01-01)|
|Y	|2017|	Full year from -9999 to 9999|
|YY	|17	|Year on 2 digits from 00 to 99|
|YYYY	|2017|	Year on 4 digits from 0000 to 9999|
|YYYYY	|02017	| Year on 5 digits from 00000 to 09999|
|YYYYYY	|+002017 |	Year on 5 digits with sign from -09999 to +09999|
|z	|utc	| Abbreviated time zone name|
|zz	|UTC	| Time zone name|
|Z	|+00:00	| Time zone offset HH:mm|
|ZZ	|+0000	| Time zone offset HHmm|

Some macro-formats are also available. Here are examples of each in some languages:


<table class="table table-bordered table-striped table-condensed">
    <tr>
        <td>
            <code>LT</code><br>
        </td>
        <td>
            <span class="doc-comment">h:mm A</span><br>
            5:04 PM
        </td>
        <td>
            <span class="doc-comment">HH:mm</span><br>
            17:04
        </td>
        <td>
            <span class="doc-comment">HH:mm</span><br>
            17:04
        </td>
        <td>
            <span class="doc-comment">H:mm</span><br>
            17:04
        </td>
    </tr>
    <tr>
        <td>
            <code>LTS</code><br>
        </td>
        <td>
            <span class="doc-comment">h:mm:ss A</span> <br>
            5:04:05 PM
        </td>
        <td>
            <span class="doc-comment">HH:mm:ss</span><br>
            17:04:05
        </td>
        <td>
            <span class="doc-comment">HH:mm:ss</span><br>
            17:04:05
        </td>
        <td>
            <span class="doc-comment">H:mm:ss</span><br>
            17:04:05
        </td>
    </tr>
    <tr>
        <td>
            <code>L</code><br>
            <br><code>l</code>
        </td>
        <td>
            <span class="doc-comment">MM/DD/YYYY</span><br>
            01/05/2017
            <br>1/5/2017
        </td>
        <td>
            <span class="doc-comment">DD/MM/YYYY</span><br>
            05/01/2017
            <br>5/1/2017
        </td>
        <td>
            <span class="doc-comment">YYYY/MM/DD</span><br>
            2017/01/05
            <br>2017/1/5
        </td>
        <td>
            <span class="doc-comment">DD.MM.YYYY</span><br>
            05.01.2017
            <br>5.1.2017
        </td>
    </tr>
    <tr>
        <td>
            <code>LL</code><br>
            <br><code>ll</code>
        </td>
        <td>
            <span class="doc-comment">MMMM D, YYYY</span><br>
            January 5, 2017
            <br>Jan 5, 2017
        </td>
        <td>
            <span class="doc-comment">D MMMM YYYY</span><br>
            5 janvier 2017
            <br>5 janv. 2017
        </td>
        <td>
            <span class="doc-comment">YYYY年M月D日</span><br>
            2017年1月5日
            <br>2017年1月5日
        </td>
        <td>
            <span class="doc-comment">D. MMMM YYYY</span><br>
            5. siječanj 2017
            <br>5. sij. 2017
        </td>
    </tr>
    <tr>
        <td>
            <code>LLL</code><br>
            <br><code>lll</code>
        </td>
        <td>
            <span class="doc-comment">MMMM D, YYYY h:mm A</span><br>
            January 5, 2017 5:04 PM
            <br>Jan 5, 2017 5:04 PM
        </td>
        <td>
            <span class="doc-comment">D MMMM YYYY HH:mm</span><br>
            5 janvier 2017 17:04
            <br>5 janv. 2017 17:04
        </td>
        <td>
            <span class="doc-comment">YYYY年M月D日 HH:mm</span><br>
            2017年1月5日 17:04
            <br>2017年1月5日 17:04
        </td>
        <td>
            <span class="doc-comment">D. MMMM YYYY H:mm</span><br>
            5. siječanj 2017 17:04
            <br>5. sij. 2017 17:04
        </td>
    </tr>
    <tr>
        <td>
            <code>LLLL</code><br>
            <br><code>llll</code>
        </td>
        <td>
            <span class="doc-comment">dddd, MMMM D, YYYY h:mm A</span><br>
            Thursday, January 5, 2017 5:04 PM
            <br>Thu, Jan 5, 2017 5:04 PM
        </td>
        <td>
            <span class="doc-comment">dddd D MMMM YYYY HH:mm</span><br>
            jeudi 5 janvier 2017 17:04
            <br>jeu. 5 janv. 2017 17:04
        </td>
        <td>
            <span class="doc-comment">YYYY年M月D日 dddd HH:mm</span><br>
            2017年1月5日 木曜日 17:04
            <br>2017年1月5日 木 17:04
        </td>
        <td>
            <span class="doc-comment">dddd, D. MMMM YYYY H:mm</span><br>
            četvrtak, 5. siječanj 2017 17:04
            <br>čet., 5. sij. 2017 17:04
        </td>
    </tr>
</table>

When you use macro-formats with `createFromIsoFormat` you can specify a locale to select which language the macro-format should be searched in.
```php
$date = Carbon::createFromIsoFormat('LLLL', 'Monday 11 March 2019 16:28', null, 'fr');
echo $date->isoFormat('M/D/YY HH:mm'); // 3/11/19 16:28
```
An other usefull translated method is `calendar($referenceTime = null, array $formats = []): string:`

```php
$date = CarbonImmutable::now();
echo $date->calendar();                                      // Today at 2:49 PM
echo "\n";
echo $date->sub('1 day 3 hours')->calendar();                // Yesterday at 11:49 AM
echo "\n";
echo $date->sub('3 days 10 hours 23 minutes')->calendar();   // Last Friday at 4:26 AM
echo "\n";
echo $date->sub('8 days')->calendar();                       // 10/06/2019
echo "\n";
echo $date->add('1 day 3 hours')->calendar();                // Tomorrow at 5:49 PM
echo "\n";
echo $date->add('3 days 10 hours 23 minutes')->calendar();   // Friday at 1:12 AM
echo "\n";
echo $date->add('8 days')->calendar();                       // 10/22/2019
echo "\n";
echo $date->locale('fr')->calendar();                        // Aujourd’hui à 14:49
```
If you know momentjs, then it works the same way. You can pass a reference date as second argument, else now is used. And you can customize one or more formats using the second argument (formats to pass as array keys are: sameDay, nextDay, nextWeek, lastDay, lastWeek and sameElse):

```php
$date1 = CarbonImmutable::parse('2018-01-01 12:00:00');
$date2 = CarbonImmutable::parse('2018-01-02 8:00:00');

echo $date1->calendar($date2, [
    'lastDay' => '[Previous day at] LT',
]);
// Previous day at 12:00 PM
```
[Click here](https://carbon.nesbot.com/docs/#supported-locales)is an overview of the 281 locales (and 824 regional variants) supported by the last Carbon version:

If you can add missing translations or missing languages, [please go to translation tool](https://carbon.nesbot.com/contribute/translate/), your help is welcome.

Note that if you use Laravel 5.5+, the locale will be automatically set according to current last `App:setLocale` execution. So `diffForHumans`, `isoFormat`, `translatedFormat` and localized properties such as `->dayName` or `->monthName` will be localized transparently.

Each Carbon, CarbonImmutable, CarbonInterval or CarbonPeriod instance is linked by default to a `Carbon\Translator` instance according to its locale set. You can get and/or change it using `getLocalTranslator()/setLocalTranslator(Translator $translator)`.

If you prefer the `date()` [pattern](https://php.net/manual/en/function.date.php), you can use `translatedFormat()` which works like `format()` but translate the string using the current locale.

```php
$date = Carbon::parse('2018-03-16 15:45')->locale('uk');

echo $date->translatedFormat('g:i a l jS F Y'); // 3:45 дня п’ятниця 16-го березня 2018
```

Be warned that some letters like `W` are not supported because they are not safely `translatable` and translatedFormat offers shorter syntax but less possibilities than `isoFormat()`.

You can customize the behavior of the `format()` method to use any other method or a custom one instead of the native method from the PHP DateTime class:

```php
$date = Carbon::parse('2018-03-16 15:45')->locale('ja');

echo $date->format('g:i a l jS F Y');    // 3:45 pm Friday 16th March 2018
echo "\n";

$date->settings(['formatFunction' => 'translatedFormat']);

echo $date->format('g:i a l jS F Y');    // 3:45 午後 金曜日 16日 3月 2018
echo "\n";

$date->settings(['formatFunction' => 'isoFormat']);

echo $date->format('LL');                // 2018年3月16日
echo "\n";

// When you set a custom format() method you still can access the native method using rawFormat()
echo $date->rawFormat('D');              // Fri
```

You can translate a string from a language to an other using dates translations available in Carbon:

```php
echo Carbon::translateTimeString('mercredi 8 juillet', 'fr', 'nl');
// woensdag 8 juli
echo "\n";

// You can select translations to use among available constants:
// - CarbonInterface::TRANSLATE_MONTHS
// - CarbonInterface::TRANSLATE_DAYS
// - CarbonInterface::TRANSLATE_UNITS
// - CarbonInterface::TRANSLATE_MERIDIEM
// - CarbonInterface::TRANSLATE_ALL (all above)
// You can combine them with pipes: like below (translate units and days but not months and meridiem):
echo Carbon::translateTimeString('mercredi 8 juillet + 3 jours', 'fr', 'nl', CarbonInterface::TRANSLATE_DAYS | CarbonInterface::TRANSLATE_UNITS);
// woensdag 8 juillet + 3 dagen
```

If input locale is not specified, `Carbon::getLocale()` is used instead. If output locale is not specified, `"en"` is used instead. You also can translate using the locale of the instance with:

```php
echo Carbon::now()->locale('fr')->translateTimeStringTo('mercredi 8 juillet + 3 jours', 'nl');
// woensdag 8 juli + 3 dagen
```

You can use strings in any language directly to create a date object with `parseFromLocale`:

```php
$date = Carbon::parseFromLocale('mercredi 6 mars 2019 + 3 jours', 'fr', 'UTC'); // timezone is optional

echo $date->isoFormat('LLLL'); // Saturday, March 9, 2019 12:00 AM
```
Or with custom format using `createFromLocaleFormat` (use the `date()` pattern for replacements):

```php
$date = Carbon::createFromLocaleIsoFormat('!DD/MMMM/YY', 'fr', '25/Août/19', 'Europe/Paris'); // timezone is optional

echo $date->isoFormat('LLLL'); // Sunday, August 25, 2019 12:00 AM
```

To get some interesting info about languages (such as complete ISO name or native name, region (for example to be displayed in a languages selector), you can use `getAvailableLocalesInfo`.

```php
$zhTwInfo = Carbon::getAvailableLocalesInfo()['zh_TW'];
$srCyrlInfo = Carbon::getAvailableLocalesInfo()['sr_Cyrl'];
$caInfo = Carbon::getAvailableLocalesInfo()['ca'];

var_dump($zhTwInfo->getId());                      // string(5) "zh_TW"
var_dump($zhTwInfo->getNames());                  
/*
array(2) {
  ["isoName"]=>
  string(7) "Chinese"
  ["nativeName"]=>
  string(38) "中文 (Zhōngwén), 汉语, 漢語"
}
*/
var_dump($zhTwInfo->getCode());                    // string(2) "zh"
var_dump($zhTwInfo->getVariant());                 // NULL
var_dump($srCyrlInfo->getVariant());               // string(4) "Cyrl"
var_dump($zhTwInfo->getVariantName());             // NULL
var_dump($srCyrlInfo->getVariantName());           // string(8) "Cyrillic"
var_dump($zhTwInfo->getRegion());                  // string(2) "TW"
var_dump($srCyrlInfo->getRegion());                // NULL
var_dump($zhTwInfo->getRegionName());              // string(25) "Taiwan, Province of China"
var_dump($srCyrlInfo->getRegionName());            // NULL
var_dump($zhTwInfo->getFullIsoName());             // string(7) "Chinese"
var_dump($caInfo->getFullIsoName());               // string(18) "Catalan, Valencian"
var_dump($zhTwInfo->getFullNativeName());          // string(38) "中文 (Zhōngwén), 汉语, 漢語"
var_dump($caInfo->getFullNativeName());            // string(18) "català, valencià"
var_dump($zhTwInfo->getIsoName());                 // string(7) "Chinese"
var_dump($caInfo->getIsoName());                   // string(7) "Catalan"
var_dump($zhTwInfo->getNativeName());              // string(20) "中文 (Zhōngwén)"
var_dump($caInfo->getNativeName());                // string(7) "català"
var_dump($zhTwInfo->getIsoDescription());          // string(35) "Chinese (Taiwan, Province of China)"
var_dump($srCyrlInfo->getIsoDescription());        // string(18) "Serbian (Cyrillic)"
var_dump($caInfo->getIsoDescription());            // string(7) "Catalan"
var_dump($zhTwInfo->getNativeDescription());       // string(48) "中文 (Zhōngwén) (Taiwan, Province of China)"
var_dump($srCyrlInfo->getNativeDescription());     // string(34) "српски језик (Cyrillic)"
var_dump($caInfo->getNativeDescription());         // string(7) "català"
var_dump($zhTwInfo->getFullIsoDescription());      // string(35) "Chinese (Taiwan, Province of China)"
var_dump($srCyrlInfo->getFullIsoDescription());    // string(18) "Serbian (Cyrillic)"
var_dump($caInfo->getFullIsoDescription());        // string(18) "Catalan, Valencian"
var_dump($zhTwInfo->getFullNativeDescription());   // string(66) "中文 (Zhōngwén), 汉语, 漢語 (Taiwan, Province of China)"
var_dump($srCyrlInfo->getFullNativeDescription()); // string(34) "српски језик (Cyrillic)"
var_dump($caInfo->getFullNativeDescription());     // string(18) "català, valencià"

$srCyrlInfo->setIsoName('foo, bar')->setNativeName('biz, baz');
var_dump($srCyrlInfo->getIsoName());               // string(3) "foo"
var_dump($srCyrlInfo->getFullIsoName());           // string(8) "foo, bar"
var_dump($srCyrlInfo->getFullIsoDescription());    // string(19) "foo, bar (Cyrillic)"
var_dump($srCyrlInfo->getNativeName());            // string(3) "biz"
var_dump($srCyrlInfo->getFullNativeName());        // string(8) "biz, baz"
var_dump($srCyrlInfo->getFullNativeDescription()); // string(19) "biz, baz (Cyrillic)"

// You can also access directly regions/languages lists:
var_dump(\Carbon\Language::all()['zh']);          
/*
array(2) {
  ["isoName"]=>
  string(7) "Chinese"
  ["nativeName"]=>
  string(38) "中文 (Zhōngwén), 汉语, 漢語"
}
*/
var_dump(\Carbon\Language::regions()['TW']);      
/*
string(25) "Taiwan, Province of China"
*/

```

Please let me thank some projects that helped us a lot to support more locales, and internationalization features:

* [jenssegers/date](https://github.com/jenssegers/date): many features were in this project that extends Carbon before being in Carbon itself.
* [momentjs](https://momentjs.com/): many features are inspired by momentjs and made to be compatible with this front-side pair project.
* [glibc](https://www.gnu.org/software/libc/) was a strong base for adding and checking languages.
* [svenfuchs/rails-i18n](https://github.com/svenfuchs/rails-i18n) also helped to add and check languages.
* We used [glosbe.com](https://glosbe.com/) a lot to check translations and fill blanks.
