---
ID: 38898
post_title: >
  Using nice date and time functions for
  your language
author: Kathy Reid
post_excerpt: "If you're building Skills that support multiple languages, use this documentation to help you ensure that spoken phrases associated with numbers, times, and dates are customized for your language. "
layout: page
permalink: >
  http://mycroft.ai/documentation/language-support/using-nice-date-and-time-functions-for-your-language/
published: true
post_date: 2018-06-15 14:03:04
---
# Configuring some of the “nice” functions for a language

- [Configuring some of the “nice” functions for a language](#configuring-some-of-the-nice-functions-for-a-language)
  * [Configuration files in the file system](#configuration-files-in-the-file-system)
  * [The configuration file](#the-configuration-file)
    + [The building blocks](#the-building-blocks)
      - [The decade_format](#the-decade_format)
        * [Components in the decade format](#components-in-the-decade-format)
    + [The hundreds format](#the-hundreds-format)
        * [Components in the hundreds format](#components-in-the-hundreds-format)
    + [The thousands_format](#the-thousands_format)
        * [Components in the thousand format](#components-in-the-thousand-format)
    + [The year_format](#the-year_format)
        * [Components in the year template](#components-in-the-year-template)
    + [The date_format](#the-date_format)
        * [Components in the date template](#components-in-the-date-template)
    + [The date_time_format](#the-date_time_format)
      - [Arguments to format templates](#arguments-to-format-templates)
  * [Maps](#maps)
    + [The weekday section](#the-weekday-section)
    + [The date section](#the-date-section)
    + [The month section](#the-month-section)
  * [Unit testing](#unit-testing)
  * [Configuration file for en-us](#configuration-file-for-en-us)
  * [Test file for en-us](#test-file-for-en-us)
  

The following functions are used to convert a Python datetime to a pronounceable string, for a given language [please refer to the Mycroft API documentation](http://mycroft-core.readthedocs.io/en/stable/))

* ```nice_date(dt, lang='en-us', now=None)```
* ```nice_date_time(dt, lang='en-us', now=None, use_24hour=False, use_ampm=False)```
* ```nice_year(dt, lang='en-us', bc=False)```

Adding a new language and unit testing it can be added using configuration only.

This page describe how to add a new language, or change configuration for an existing language. Unit testing is covered as well.

## Configuration files in the file system

Each language has a configuration file and a unit test file. The configuration files are written in json. The configurations are:
* ```mycroft/res/text//date_time.json```
* ```mycroft/res/text//date_time_test.json```

For example: ```mycroft/res/text/en-us/date_time.json```

The `mycroft.util.nice_time()` function is not configurable using the approach described in this document (historical reasons). The `nice_date_and_time()` depends on `nice_time()`, which must be able to support the selected language, if you want nice results from `nice_date_time()`. Without support for a language in `nice_time()`, `nice_date_and_time(`) will still return a string, but the time part not be appropriate for your language.

## The configuration file

The section [Configuration file for en-us](# Configuration file for en-us) contains an example configuration file. The configuration file contains several main sections, some of which contains maps from numbers to words to be spoken, others containing the format template to use when the pronounceable string is created.

The template sections are:

1. decade_format
2. hundreds_format
3. thousands_format
4. year_format
5. date_format
6. date_time_format

The map sections are:

* weekday
* date
* month
* number

The format templates operate on a number of components, like a ```{formatted_year}```. The components are substituted by a value calculated by the `nice_*()` function. In general, a format created by a template at the top of the list above, can be used in a template lower on the list. The exact possibilities are described later in this document.

### The building blocks

Before the template sections are considered, the input year is decomposed into components, and each component is translated to a word using the number map. The components can then be used in the template sections. The building of the basic components are not configurable, and it is typically not necessary to map all components.

A list of basic components with examples for year 1969 in en-us are shown below:

* {x} example: 'nine'
* {xx} example: '69' (fall back to number since the map does not contain '69')
* {x0} example: 'sixty'
* {x_in_x0} example: 'six'
* {xxx} example: '969' (fall back to number since the map does not contain '969')
* {x00} example: '969' (fall back to number since the map does not contain '900')
* {x_in_x00} example: 'one'
* {xx00} example: '1969' (fall back number since the map does not contain '1900')
* {xx_in_xx00} example: 'nineteen'
* {x000} example: '1969' (fall back number since the map does not contain '1000')
* {x_in_x000} example: 'one'
* {number} example: '1969' (can be used for fall back by templates, if no other patterns match)

When we look at the list above, it is possible to create 'nineteen sixty nine' if we choose the correct components. That process happens in the templates.

The `decade_format`, `hundreds_format` and `thousand_format` templates create new components, that can be used by the other templates. And so, those new component are configurable. The `year_format` template can assemble decades, hundreds and thousands. The `date_format` will assemble the year_format and dates.

The best practice is to limit the `decade_format` to only produce words for the numbers 0 to 99. Consider the decade format as numbers used for years before year 100. For `en-us` the `decade_format` produces 'seventeen' for 17, and 'four' for 4.

Limit the `hundreds_format` to only produce words for 100, 200,,,900, i.e. do not include decades in the hundreds. Consider the hundreds format as numbers for centuries before year 1000. For `en-us` the `hundreds_format` produces 'five hundred' for 524.

The `thousands_format` should produce words for thousands, and if needed in your language, hundreds above 999. Consider the `thousands_format` as numbers for millennium and optionally centuries above 999, and below 10.000. For en-us the `thousands_format` produces 'nineteen hundred' for 1969, and 'two thousand' for 2018.

The `decade_format`, `hundreds_format`, `thousands_format` and the `year_format` are using indexed lists and regular expressions to do their task. Each list entry contains a regular expression to match, and a format to apply. The lists are searched from beginning (index 1) to end. The first match is used, if any. The lists are read from a `json` configuration file, and must start with index 1, and the increment must be 1.

#### The decade_format

The decade format will work on the two rightmost digits of the year. For `en-us` it looks like this:

```json
"decade_format": {
"1": {"match": "^\d$", "format": "{x}"},
"2": {"match": "^1\d$", "format": "{xx}"},
"3": {"match": "^\d0$", "format": "{x0}"},
"4": {"match": "^[2-9]\d$", "format": "{x0} {x}"},
"default": "{number}"
}
```

The format consists of a list of four templates, and a default fall back template. The two rightmost digits of the year are converted to a string, and then matched to the value of the 'match' keyword above, starting from index 1. For instance, the first list entry match `^d$`, which is a single digit. The second list entry are for numbers between 10 and 19, the third matches multiples of ten, the fourth is for the rest, the 'default' entry will never be used. The double `''` in the `json` file is because it acts as an escape character.

Let's use 1969 as an example with language en-us:

The string '1969' is used, and the first match is index 4: `^[2-9]\d`. The components `{x0}` is `'sixty'` and `{x}` is `{nine}`, the end result is `'sixty nine'`. If the string had been 1910, the second list entry would have been used, since that would be the first match, even if the third entry also match '10'.

In some languages, for instance Danish, the order of ones and tens is reversed, so 69 is pronounced 'ni og tres' (nine and sixty). The list entry for Danish would be:
```"4": {"match": "^[2-9]\d$", "format": "{x} og {x0}"}```

The result of the decade format is `{formatted_decade}`, that can be used as a component in other formats.

##### Components in the decade format

The following components can be used in the decade format:

* {x}
* {xx}
* {x0}
* {x_in_x0}
* {number}

### The hundreds format

The hundreds format will work on the three rightmost digits of the year. For `en-us` it looks like this:

```json
"hundreds_format": {
"1": {"match": "^\d{3}$", "format": "{x_in_x00} hundred"},
"default": "{number}"
}
```

Lets use 1969 as an example with language en-us:

The string '969' is used, and matches the first (and only) list entry. `{x_in_x00}` is nine, so the end result is `'nine hundred'`.

The result of the hundreds format is `{formatted_hundreds}`, that can be used as a component in other formats.

##### Components in the hundreds format

The following components can be used in the hundreds format:

* {xxx}
* {x00}
* {x_in_x00}
* {formatted_decade}
* {number}

Even if `{formatted_decade}` can be used as a component in hundreds, it is usually easier to configure, test and read if it is not used. Instead use the `year_format` to put hundreds and decades together.

### The thousands_format

The thousand format will work on the four rightmost digits of the year. For `en-us` it looks like this:

```json
"thousand_format": {
"1": {"match": "^1[1-9]\d{2}$", "format": "{xx_in_xx00} hundred"},
"2": {"match": "^\d{4}$", "format": "{x_in_x000} thousand"},
"default": "{number}"
}
```

The first list entry handles 1100 to 1900, the second handles 1000, and 2000 and up.You may want to check out the 'en-au' for a more complex example, where 2017 is formatted to 'twenty seventeen' but 2007 is formatted to 'two thousand and seven'.

Lets use 1969 as an example with language `en-us`:

The string '1969' is used, it matches the first list entry and is formatted to 'nineteen hundred'

The result of the thousands format is `{formatted_thousands}`, that can be used as a component in other formats.

##### Components in the thousand format 

The following components can be used in the thousand format:

* {x_in_x00}
* {xx00}
* {xx_in_xx00}
* {x000}
* {x_in_x000}
* {formatted_decade}
* {formatted_hundreds}
* {number}

Even if `{formatted_decade}` and `{formatted_hundreds}` can be used as a component in thousands, it is usually easier to configure, test and read if it is not used. Instead use the `year_format` to put thousands, hundreds and decades together.

### The year_format

This section contains information on how to format a pronounceable year. For `en-us` the configuration looks like this:

```json
"year_format": {
"1": {"match": "^\d\d?$", "format": "{formatted_decade} {bc}"},
"2": {"match": "^\d00$", "format": "{formatted_hundreds} {bc}"},
"3": {"match": "^\d{3}$", "format": "{formatted_hundreds} and {formatted_decade} {bc}"},
"4": {"match": "^(1\d00)|([2-9]000)$", "format": "{formatted_thousand} {bc}"},
"5": {"match": "^\d{2}00$", "format": "{formatted_thousand} {formatted_hundreds} {bc}"},
"6": {"match": "^(1\d{3})|(\d0\d{2})$", "format": "{formatted_thousand} and {formatted_decade} {bc}"},
"7": {"match": "^\d{4}$", "format": "{formatted_thousand} {formatted_hundreds} and {formatted_decade} {bc}"},
"default": "{year} {bc}",
"bc": "b.c."
}
```

The first entry handles years below 100. The second entry handles years below 1000, that are multiples of 100. The third entry handles years below 1000 (years below 1000, that are multiples of 100, are caught by the second entry). The fourth entry handles years in the range 1100 to 1900 that are multiples of 100, and years that are multiples of 1000. This allows for instance 'eighteen hundred' and 'three thousand'. The fifth entry handles years multiple of 100, that has not been matched before. For instance 'two thousand four hundred'. The sixth entry handles years int the range 1100 to 1900, and years before the first century, when part of a millennium. This allows for instance 'eighteen hundred and forty one' and 'two thousand and five'. The seventh entry handles the rest of the years, up to 9999. For instance "two thousand one hundred and forty five"

The default entry is used if the year is larger than 9999.

The entry "bc" is the string to be used in the year format, if the year is b.c. Python does not handle years b.c. and a year b.c. is made by creating a datetime of the year (a.d.), and calling nice_year() whit the bc flag set to True. When bc=False, or not provided, the {bc} format is not used when creating the year.

The result of the year format is `{formatted_year}`, that can be used as a component in other formats.

##### Components in the year template

* {formatted_decade}
* {formatted_hundreds}
* {formatted_thousands}
* {century} : The datetime century as digits
* {decade}: The datetime decade as digits
* {year}: The datetime year as digits
* {bc}: A string to apply for years B.C.

### The date_format

This section contains information on how to format a pronounceable date. For `en-us` the configuration looks like this:

```json
"date_format": {
"date_full": "{weekday}, {month} the {day}, {formatted_year}",
"date_full_no_year": "{weekday}, {month} the {day}",
"date_full_no_year_month": "{weekday}, the {day}",
"today": "today",
"tomorrow": "tomorrow",
"yesterday": "yesterday"
}
```

The `nice_date()` function takes an optional now parameter in addition to the `datetime` parameter. If the `now` parameter is not supplied, the `date_full` format is always used. If the `now` parameter is supplied, the other formats may come into play, as described below in the paragraph about keys for the `date_format`.

If your language don't have, for instance, a tradition for saying yesterday, it is of cause possible to use arguments here instead. For example:

```json
"yesterday": "{weekday}, {month} the {day}, {formatted_year}"
```
will create the same output as `date_full` above, in case of yesterday.

The result of the date format is `{formatted_date}`, that can be used as a component in other formats.

The following keys exist for the date_format:

* "date_full": A format containing a full date
* "date_full_no_year": A format without the year, used if the "now" date parameter is set, and if "now" is in the same year and before the date.
* "date_full_no_year_month": A format without the year and month, used if the "now" date parameter is set, and if "now" is before the date and is in the same year and month as the date.
* "today": The word for today, used if the "now" date parameter is set, and if "now" is in the same year and month and day as the date.
* "tomorrow": The word for tomorrow, used if the "now" date parameter is set, and if "now" is in the same year and month, but one day before the day in the date.
* "yesterday": The word for yesterday, used if the "now" date parameter is set, and if "now" is in the same year and month, but one day after the day in the date.

##### Components in the date template

* {formatted_year}: The year formatted like in [The year_format](## The year_format)
* {weekday}: The weekday, formatted according to [The weekday section](## The weekday section)
* {month}: The month, formatted according to [The month section](## The month section)
* {day}: The day of the month, formatted according to [The date section](## The date section)

### The date_time_format

This section contains information on how to format a pronounceable date and time. For `en-us` the configuration looks like this:

```json
"date_time_format": {
"date_time": "{formatted_date} at {formatted_time}"
}
```

The formatted time is obtained using the existing `mycroft.util.nice_time()` function. The output of this function is not configurable like the other formatting functions mentioned in this document. If `mycroft.util.nice_time()` does not support a language, it will still produce a string, but it will not be formatted.

#### Arguments to format templates

* {formatted_date}: Date formatted as specified in the "date_format" section
* {formatted_time}: Time formatted by nice_time(), [please refer to the Mycroft API documentation](http://mycroft-core.readthedocs.io/en/stable/)

## Maps

### The weekday section
A map from the day number in the week to the pronounceable weekday name

### The date section
A map from date as a number to a pronounceable date

### The month section
A map from the month number to a pronounceable month

## Unit testing

A new language requires new unit tests, to ensure that it produce correct results. Unit tests that assert that all years between 1 and 9999 produce a non empty string, and that all dates in a year produce a non empty string, already exists. The unit tests will automatically find and test a new language.

To prove the likelihood that formatting makes sense, a configuration file must be provided, that lists the unit tests. One unit test file exists for each language, it is placed in:
```mycroft/res/text//date_time_test.json```

Section [Test file for en-us](# Test file for en-us) contains the test file for the en-us language. It has three sections:

* test_nice_year
* test_nice_date
* test_nice_date_time

one section for each of the three `nice_*()` functions.

For each section there is a list, the index must start at 1 and continue in increments of 1. The content of a list entry is ```datetime_param```, which contains the parameters that the Python datetime.datetime class take. Then there is ```assertEqual```, which is the expected formatted value when given the datetime. Each `nice_*()` function takes different parameters, the parameters must be specified in the list entry as well, they cannot be omitted even if they are Python 'None'

## Configuration file for en-us

```json
{
"decade_format": {
"1": {"match": "^\d$", "format": "{x}"},
"2": {"match": "^1\d$", "format": "{xx}"},
"3": {"match": "^\d0$", "format": "{x0}"},
"4": {"match": "^[2-9]\d$", "format": "{x0} {x}"},
"default": "{number}"
},
"hundreds_format": {
"1": {"match": "^\d{3}$", "format": "{x_in_x00} hundred"},
"default": "{number}"
},
"thousand_format": {
"1": {"match": "^1[1-9]\d{2}$", "format": "{xx_in_xx00} hundred"},
"2": {"match": "^\d{4}$", "format": "{x_in_x000} thousand"},
"default": "{number}"
},
"year_format": {
"1": {"match": "^\d\d?$", "format": "{formatted_decade} {bc}"},
"2": {"match": "^\d00$", "format": "{formatted_hundreds} {bc}"},
"3": {"match": "^\d{3}$", "format": "{formatted_hundreds} and {formatted_decade} {bc}"},
"4": {"match": "^(1\d00)|([2-9]000)$", "format": "{formatted_thousand} {bc}"},
"5": {"match": "^\d{2}00$", "format": "{formatted_thousand} {formatted_hundreds} {bc}"},
"6": {"match": "^(1\d{3})|(\d0\d{2})$", "format": "{formatted_thousand} and {formatted_decade} {bc}"},
"7": {"match": "^\d{4}$", "format": "{formatted_thousand} {formatted_hundreds} and {formatted_decade} {bc}"},
"default": "{year} {bc}",
"bc": "b.c."
},
"date_format": {
"date_full": "{weekday}, {month} the {day}, {formatted_year}",
"date_full_no_year": "{weekday}, {month} the {day}",
"date_full_no_year_month": "{weekday}, the {day}",
"today": "today",
"tomorrow": "tomorrow",
"yesterday": "yesterday"
},
"date_time_format": {
"date_time": "{formatted_date} at {formatted_time}"
},
"weekday": {
"0": "monday",
"1": "tuesday",
"2": "wednesday",
"3": "thursday",
"4": "friday",
"5": "saturday",
"6": "sunday"
},
"date": {
"1": "first",
"2": "second",
"3": "third",
"4": "fourth",
"5": "fifth",
"6": "sixth",
"7": "seventh",
"8": "eighth",
"9": "ninth",
"10": "tenth",
"11": "eleventh",
"12": "twelfth",
"13": "thirteenth",
"14": "fourteenth",
"15": "fifteenth",
"16": "sixteenth",
"17": "seventeenth",
"18": "eighteenth",
"19": "nineteenth",
"20": "twentieth",
"21": "twenty-first",
"22": "twenty-second",
"23": "twenty-third",
"24": "twenty-fourth",
"25": "twenty-fifth",
"26": "twenty-sixth",
"27": "twenty-seventh",
"28": "twenty-eighth",
"29": "twenty-ninth",
"30": "thirtieth",
"31": "thirty-first"
},
"month": {
"1": "january",
"2": "february",
"3": "march",
"4": "april",
"5": "may",
"6": "june",
"7": "july",
"8": "august",
"9": "september",
"10": "october",
"11": "november",
"12": "december"
},
"number": {
"0": "zero",
"1": "one",
"2": "two",
"3": "three",
"4": "four",
"5": "five",
"6": "six",
"7": "seven",
"8": "eight",
"9": "nine",
"10": "ten",
"11": "eleven",
"12": "twelve",
"13": "thirteen",
"14": "fourteen",
"15": "fifteen",
"16": "sixteen",
"17": "seventeen",
"18": "eighteen",
"19": "nineteen",
"20": "twenty",
"30": "thirty",
"40": "forty",
"50": "fifty",
"60": "sixty",
"70": "seventy",
"80": "eighty",
"90": "ninety",
"100": "hundred",
"1000": "thousand"
}
}

```

## Test file for en-us
```
{
"test_nice_year": {
"1": {"datetime_param": "2017, 1, 31, 13, 22, 3", "bc": "None", "assertEqual": "two thousand and seventeen"},
"2": {"datetime_param": "1984, 1, 31, 13, 22, 3", "bc": "None", "assertEqual": "nineteen hundred and eighty four"},
"3": {"datetime_param": "1906, 1, 31, 13, 22, 3", "bc": "None", "assertEqual": "nineteen hundred and six"},
"4": {"datetime_param": "1802, 1, 31, 13, 22, 3", "bc": "None", "assertEqual": "eighteen hundred and two" },
"5": {"datetime_param": "806, 1, 31, 13, 22, 3", "bc": "None", "assertEqual": "eight hundred and six" },
"6": {"datetime_param": "1800, 1, 31, 13, 22, 3", "bc": "None", "assertEqual": "eighteen hundred" },
"7": {"datetime_param": "1000, 1, 31, 13, 22, 3", "bc": "None", "assertEqual": "one thousand" },
"8": {"datetime_param": "2000, 1, 31, 13, 22, 3", "bc": "None", "assertEqual": "two thousand" },
"9": {"datetime_param": "99, 1, 31, 13, 22, 3", "bc": "True", "assertEqual": "ninety nine b.c." },
"10": {"datetime_param": "5, 1, 31, 13, 22, 3", "bc": "True", "assertEqual": "five b.c." },
"11": {"datetime_param": "3120, 1, 31, 13, 22, 3", "bc": "True", "assertEqual": "three thousand one hundred and twenty b.c." },
"12": {"datetime_param": "3241, 1, 31, 13, 22, 3", "bc": "True", "assertEqual": "three thousand two hundred and forty one b.c." },
"13": {"datetime_param": "1001, 1, 31, 13, 22, 3", "bc": "False", "assertEqual": "one thousand and one" },
"14": {"datetime_param": "2100, 1, 31, 13, 22, 3", "bc": "False", "assertEqual": "two thousand one hundred" },
"15": {"datetime_param": "5200, 1, 31, 13, 22, 3", "bc": "False", "assertEqual": "five thousand two hundred" }
},
"test_nice_date": {
"1": {"datetime_param": "2017, 1, 31, 0, 2, 3", "now": "None", "assertEqual": "tuesday, january the thirty-first, two thousand and seventeen"},
"2": {"datetime_param": "2018, 2, 4, 0, 2, 3", "now": "2017, 1, 1, 0, 2, 3", "assertEqual": "sunday, february the fourth, two thousand and eighteen"},
"3": {"datetime_param": "2018, 2, 4, 0, 2, 3", "now": "2018, 1, 1, 0, 2, 3", "assertEqual": "sunday, february the fourth"},
"4": {"datetime_param": "2018, 2, 4, 0, 2, 3", "now": "2018, 2, 1, 0, 2, 3", "assertEqual": "sunday, the fourth"},
"5": {"datetime_param": "2018, 2, 4, 0, 2, 3", "now": "2018, 2, 3, 0, 2, 3", "assertEqual": "tomorrow"},
"6": {"datetime_param": "2018, 2, 4, 0, 2, 3", "now": "2018, 2, 4, 0, 2, 3", "assertEqual": "today"},
"7": {"datetime_param": "2018, 2, 4, 0, 2, 3", "now": "2018, 2, 5, 0, 2, 3", "assertEqual": "yesterday"},
"8": {"datetime_param": "2018, 2, 4, 0, 2, 3", "now": "2018, 2, 6, 0, 2, 3", "assertEqual": "sunday, february the fourth"},
"9": {"datetime_param": "2018, 2, 4, 0, 2, 3", "now": "2019, 2, 6, 0, 2, 3", "assertEqual": "sunday, february the fourth, two thousand and eighteen"}
},
"test_nice_date_time": {
"1": {"datetime_param": "2017, 1, 31, 13, 22, 3", "now": "None", "use_24hour": "False", "use_ampm": "True", "assertEqual": "tuesday, january the thirty-first, two thousand and seventeen at one twenty two PM"},
"2": {"datetime_param": "2017, 1, 31, 13, 22, 3", "now": "None", "use_24hour": "True", "use_ampm": "False", "assertEqual": "tuesday, january the thirty-first, two thousand and seventeen at thirteen twenty two"}
}
}
```