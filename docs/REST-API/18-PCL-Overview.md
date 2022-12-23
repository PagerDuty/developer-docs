# PCL Overview

The PagerDuty Condition Language (PCL) gives [Event Orchestrations](https://support.pagerduty.com/docs/event-orchestration) the power of boolean expressions, more powerful than are available with Rulesets. For example, with PCL, you can now define an expression(s) like:

```
event.summary matches part 'prod' and (event.location == 'US' or event.location == 'Canada')
```

This document will provide an overview of the PCL expression language.

## Syntax

### Expressions

A PCL expression is made up of a combination of paths, literals, built-in operations, and custom functions. Evaluation of an expression follows the rules of precedence described below. The final PCL expression must evaluate to `true` or `false`. A sub-expression can return a different type of value, such as an integer or string. As long as this non-boolean is used in the proper context, an error will not be returned.

During evaluation, a sub-expression might not be able to evaluate successfully. This will result in an evaluation warning but will _not_ prevent the expression from being evaluated as a whole. See [Non-Boolean Evaluation] for more details.

### Precedence

PCL expressions are made up of boolean sub-expressions and the final expression must return a boolean value. Sub-expressions can be combined with `and`, `or`, and `not` to form more complex boolean expressions. When creating expressions, these precedence rules apply:

1. parenthesized expressions
2. built-in operations
3. custom functions
4. `not`
5. `and`
6. `or`

For example:

```
event.x > 5 and not event.y < 6 or event.z == 2
```

is treated the same as:

```
((event.x > 5) and (not (event.y < 6))) or (event.z == 2)
```

where:

```
event.x > 5 and (not event.y < 6 or event.z == 2)
```

is treated like:

```
(event.x > 5) and ((not (event.y < 6)) or (event.z == 2))
```

### Paths

A PCL expression is evaluated within the context of an Event being evaluated by your Event Orchestration. You use a path in your PCL expression to access nested object and array structures from an Event payload. An `event` path can be used to access [PD-CEF fields](https://support.pagerduty.com/docs/pd-cef#pd-cef-fields) like `event.summary` or `event.custom_details.load`. A `raw_event` path can be used to access the values of the JSON Event payload that was originally sent to PagerDuty.

Object fields can be accessed using *dot notation* (`"."`). Field names accessed using the dot notation must be a valid identifier. Valid identifiers begin with an ASCII letter (`a-zA-Z`) or an underscore (`"_"`). The following characters of the identifier can be any of those plus the numerical digits (`0-9`).

Object field names that do not conform to the identifier description above can be accessed using *bracket notation* by providing the field name as a single-quoted string inside square brackets, e.g. (`object['invalid identifier']`). Also, array elements can only currently be accessed using integers inside square brackets.

When a `path` is used in an expression, then it is evaluated to the value at that path in the data context before being used in the expression. If a final path doesn't exist, then `nil` is returned from the expression. The exception to this is the `exists` operation which checks whether or not the path exists in the context.

For example, if you sent this JSON payload via the V2 Events API:

```json
{
  "payload": {
    "custom_details": {
      "system diagnosis": {
        "important_field": "This is an important value"
      }
    }
  },
  "links": [
    { "href": "https://diag.test/details", "text": "Diagnosis details" }
  ]
}
```

You'd be able to access data from that event using these PCL paths:

```
event.custom_details['system diagnosis'].important_field
  -> "This is an important value"

raw_event.links[0].href
  -> "https://diag.test/details"

raw_event.images[0].src
  -> nil
```



PCL understands these data types:
- strings
- integers and floats
- booleans
- local datetimes
- durations
- schedules
- nil

Values of a certain type cannot currently be converted to another type, but some built-in operations will do conversions or handle multiple types.

#### String

A string is any sequence of utf-8 characters wrapped in single quotes. You can insert a single quote into a string by preceding it with a backslash character to escape it. You can also escape a backslash with a backslash. No other escape sequences are available at this time.

```
'this is a string'
'Mercury'
'the system\'s down'
'this has a single \\ backslash in it'
'こんにちは世界'
```

#### Number

A number can be an integer or a float. A number can be positive or negative.

- Integers are 64-bit signed integers between `-2^63` and `2^63 - 1`
- Floats are double precision (64-bit) IEEE floating point numbers. They must begin with a digit and must have a decimal point. Scientific notation of floats is also supported using a lowercase `e`.
### Types
```
42
0.7
-12
4.5e10
```

#### Boolean

The values `true` or `false`. Written without quotes.

````
true
false
````

#### Local Datetime

A moment in time specified with a date, time, and timezone.
The date format is `YYYY-MM-DD`.
The time uses a 24-hour notation in the format `HH:MM:SS`.
The timezone must be a valid entry from the [tz database][1].

```
2021-12-04 19:00:42 America/Los_Angeles
2022-01-01 08:00:00 Etc/UTC
```

When you want the current datetime, use [`now`](#now)

#### Duration

A duration of time that is represented by a positive integer followed by a time unit. The time unit is one of: `"second"`, `"minute"`, `"hour"`, `"day"` with an optional trailing `"s"`. You can string multiple units together as long as you only have one of each maximum.

Format:
```
[D day(s)] [h hour(s)] [m minute(s)] [s second(s)]
```
`D`, `h`, `m`, `s` all must be integer values.

1. At least one time unit is required.
2. If more than one time unit is included, the units can be in any order as long as there is only one of each i.e. seconds can come before days can come before minutes, e.g. `5 seconds 3 days 12 minutes` is valid, `4 hours 2 days` is also valid.
3. The trailing "s" in the unit name is optional and does not have any semantic meaning, e.g. `5 hour` is the same as `5 hours`, `1 day` equivalent to `1 days`.

```
2 minutes
1 hour
1 hour 30 minutes
```

#### Schedule

A set of weekdays combined with a time range and a timezone to form a weekly schedule.

The day of the week can be one-or-many of: `"Mon"`, `"Tue"`, `"Wed"`, `"Thu"`, `"Fri"`, `"Sat"`, `"Sun"`. Multiple days may be provided by delimiting with commas (no spaces surrounding the commas). These represent the "start" days of a schedule (which becomes more important when the time range spans over multiple days).

The order of the days does not matter, but at least one must be provided.

The time range is made up of a start and end time separated with the word `"to"`. The times use 24-hour notation in the format `HH:MM:SS`.

```
# Week days from 9am through 9pm using New York time
# (EST or EDT depending on the current date)
Mon,Tue,Wed,Thu,Fri 09:00:00 to 17:00:00 America/New_York
```

If the end time is "earlier" than the start time, that indicates the time window ends on the next weekday.

```
# From Wed at 10pm through Thurs at 8am using UTC time. 8am is earlier than 10pm.
# (UTC never observes daylight saving time)
Wed 22:00:00 to 08:00:00 Etc/UTC
```

If the two times are the same, that indicates a 24 hour period.

```
# From Sat noon through Mon noon using Cairo time
# (Currently Cairo is on EGY time year round & does not observe daylight saving time)
Sat,Sun 12:00:00 to 12:00:00 Africa/Cairo
```

The allowable timezones are defined in the [tz database][1].

When doing comparisons of times, PCL compares the times according to what a clock on the wall would say in that timezone. That is, local daylight saving time is factored in. This can cause slightly unexpected behavior during the times when the clock changes from daylight saving time to standard time and vice versa.

For example, when daylight saving time ends, the clocks are changed from 2:00 AM back to 1:00 AM. This means that a wall clock, will repeat the 1:00 AM to 2:00 AM hour that day. Considering a PCL condition that evaluates to `true` from 1:30am through 3:00 AM on Sundays in New York, when the clocks rollback to 1:00 AM, then the half hour between 1:00 AM and 1:30 AM produces a gap in the schedule. The following table shows the evaluation at various points in time around when DST ends:

**PCL Schedule: `Sun 01:30:00 to 03:15:00 America/New_York`**

Wall Clock Time | UTC Time | Matches Schedule? | Notes
-- | -- | -- | --
[01:00 EDT (UTC-0400)](https://www.timeanddate.com/worldclock/converter.html?iso=20211107T050000&p1=179&p2=1440) | 05:00 UTC | ❌ No |
01:30 EDT (UTC-0400) | 05:30 UTC | ✅ Yes |
DST ends: [01:00 EST (UTC-0500)](https://www.timeanddate.com/worldclock/converter.html?iso=20211107T060000&p1=179&p2=1440) | 06:00 UTC | ❌ No | This is the gap
01:15 EST (UTC-0500) | 06:15 UTC | ❌ No | This is the gap
01:30 EST (UTC-0500) | 06:30 UTC | ✅ Yes |
02:00 EST (UTC-0500) | 07:00 UTC | ✅ Yes |
02:30 EST (UTC-0500) | 07:30 UTC | ✅ Yes |
03:00 EST (UTC-0500) | 08:00 UTC | ✅ Yes |
03:30 EST (UTC-0500) | 08:30 UTC | ❌ No |

Another example similar to the above, but when DST starts.

**PCL Schedule: `Sun 01:30:00 to 03:15:00 America/New_York`**

Wall Clock Time | UTC Time | Matches Schedule? | Notes
-- | -- | -- | --
00:00 EST (UTC-0500) | 05:00 UTC | ❌ No |
00:30 EST (UTC-0500) | 05:30 UTC | ❌ No |
01:00 EST (UTC-0500) | 06:00 UTC | ❌ No |
01:30 EST (UTC-0500) | 06:30 UTC | ✅ Yes |
01:59 EST (UTC-0500) | 06:59 UTC | ✅ Yes |
DST begins: 03:00 EDT (UTC-0400) | 07:00 UTC | ✅ Yes | 2:00 AM skipped to 3:00 AM
03:15 EDT (UTC-0400) | 07:15 UTC | ✅ Yes |
03:30 EDT (UTC-0400) | 07:30 UTC | ❌ No |
04:00 EDT (UTC-0400) | 08:00 UTC | ❌ No |
04:30 EDT (UTC-0400) | 08:30 UTC | ❌ No |

#### Nil

A `nil` value represents the absense of any other value type.

<!-- ### Built-in Operations -->

PCL has a variety of built-in operations:
- boolean operations: `and`, `or`, `not`
- numeric comparison operations: `>`, `>=`, `<`, `<=`
- string comparison operations: `matches [exactly]`, `matches part [exactly]`, `matches regex [exactly]`
- threshold conditions: `trigger_count`, `resetting_trigger_count`
- other operations: `==`, `in`, `exists`, `now`

NOTE: `now`, `trigger_count`, and `resetting_trigger_count` are only available to Event Intelligence (EI) customers. If you don't have EI, then trying to create a condition with these functions will return an error.

The operations described below will have an initial codeblock which shows the syntax of the operation. It will look something like:

```
[integer] < [integer] -> [boolean]
```

where to the left of the `->` we show the types of the arguments and the way the operation is used, and to the right of the `->`, we show the type of the return value. In both cases the type will be surrounded by `[]` to distinguish it from the operation name.

#### Boolean Operations

##### `not`

Returns the boolean `not` of the argument, i.e. the opposite boolean value to the one passed in as an argument.

```
not [boolean] -> [boolean]
```

Should be used in combination with the other operators.  Has higher precedence than the other boolean operators - use parentheses if you need to change the order of precdence.

```json
{
  "data": {
    "foo": "code"
  }
}

not data.foo matches 'www' -> true
not data.missing exists and data.foo matches 'www' -> false
not (data.missing exists and data.foo matches 'www') -> true
not (data.foo exists and data.foo matches 'code') -> false
```

A `not` will also treat [Non-Boolean Evaluations][Non-Boolean Evaluation] as `false`.

Expression | Evaluates like | Result | Explanation
-- | -- | -- | --
`raw_event.invalid_path > 2` | `nil > 2` | ❌ `false` | `>` can only compare two `[number]` or `[date]` types but got `[nil] > [number]`
`not raw_event.invalid_path > 2` | `not (nil > 2)` | ✅ `true` | `nil > 2` evaluates as `false` and then `not` inverts the result.

##### `and`

Returns the boolean `and` of the two arguments, i.e. returns `true` if both arguments are `true` and `false` otherwise.

```
[boolean] and [boolean] -> [boolean]
```

No conversions are performed; returns `false` if either arg is not a `boolean` type. Returns `false` if either args is a [Non-Boolean Evaluation]. Can be used with boolean literals or be used for creating nested expressions by joining the other operators. Has higher precedence than `or`, but lower predence than `not`. If the first argument evaluates to `false`, the second argument is not evaluated (short-cicuiting). Note that when short-cicuiting takes place, any errors or warnings that may have occurred in the evaluation of the second argument will not be returned.

 ```json
 {
   "data": {
     "foo": "code"
   }
 }

 data.foo matches 'www' and data.missing matches 'hello' -> false (no warnings)
 data.missing matches 'hello' and data.foo matches 'www' -> false (type mismatch warning)
 data.foo matches 'code' and not data.missing exists -> true (no warnings)
 ```

##### `or`

Returns the boolean `or` of the two arguments, i.e. returns `false` if both arguments are `false` and `true` otherwise.

```
[boolean] or [boolean] -> [boolean]
```

No conversions are performed; returns `false` if either arg is not a `boolean`. Treats [Non-Boolean Evaluation] as `false`. Can be used with boolean literals or be used for creating nested expressions by joining the other operators.  Has lower precedence than `and`. If the first argument evaluates to `true`, the second argument is not evaluated (short-cicuiting). Note that when short-cicuiting takes place, any errors or warnings that may have occurred in the evaluation of the second argument will not be returned.

 ```json
 {
   "data": {
     "foo": "code"
   }
 }

 data.foo matches 'code' or data.missing matches 'hello' -> true (no warnings)
 data.missing matches 'hello' or data.foo matches 'code' -> true (type mismatch warning)
 data.foo matches 'www' or data.missing exists -> false (no warnings)
 ```

#### Numeric Comparison Operations

##### `>`, `>=`, `<`, `<=`

```
[datetime] >  [datetime] -> [boolean]
[datetime] >= [datetime] -> [boolean]
[datetime] <  [datetime] -> [boolean]
[datetime] <= [datetime] -> [boolean]

[number] >  [number] -> [boolean]
[number] >= [number] -> [boolean]
[number] <  [number] -> [boolean]
[number] <= [number] -> [boolean]
```

If both args are of type `datetime`, then evaluate the condition using datetimes. Datetimes further in the past are considered "less than" datetimes further in the future.

If both args are of type `number`, then evaluate the condition using numbers. If both are integers or both are floats, then the numbers will compare as expected. If one number is an integer and the other is a floating point number, then the rule described in [Note About Comparing Integers and Floats] applies.

Otherwise, returns a [Non-Boolean Evaluation], which will be evaluated like a `false` response.

#### String Comparison Operations

For the `matches`, `matches part`, and `matches regex` operations, comparisons are case-insensitive by default. Use the `exactly` modifier for a case-sensitive comparison.

For `matches` and `matches part` the following apply to both arguments. For `matches regex` the following applies to the first argument.

1. If either or both args are `nil`, returns `false`
2. If either argument isn't a `string`, then it is converted to a `string` before the comparison.
   - integers - the digit by digit conversion to string is made
   - floats
     - if the value is smaller than xyz, then the string will be an integer with a decimal part
     - if the value is larger than zzz, then the float will be converted to scientific notation with an `e`
   - objects and lists - the object / list is converted to a JSON string

##### `matches`

If the two strings match, then `true` is returned. Otherwise, returns `false`.

```
[string] matches [string] -> [boolean]
[string] matches exactly [string] -> [boolean]
```

"Matching" means that the arguments (after conversion) are equivalent character by character in their entirety, including any leading and trailing whitespace, taking into acccount the appropriate case-sensitivity.

Examples

```
'this is a test' matches 'This Is A Test' -> true
'trailing whitespace ' matches 'trailing whitespace' -> false
```

##### `matches part`

If the second argument is a part of the first argument, then `true` is returned. Otherwise, returns `false`.

```
[string] matches part [string] -> [boolean]
[string] matches part exactly [string] -> [boolean]
```

Examples

```
'[PROD] Disk space low' matches part 'prod' -> true
'[TEST] CPU usage high' matches part 'cpu' -> true
'[PROD] Network down' matches part 'disk' -> false
```

##### `matches regex`

If the second argument treated as a regex matches the first argument, then `true` is returned. Otherwise, returns `false`.

```
[string] matches regex [string] -> [boolean]
[string] matches regex exactly [string] -> [boolean]
```

The second argument must be a literal string value representing a valid regular expressing using the [RE2 syntax](https://github.com/google/re2/wiki/Syntax). An error will be returned at parse time if this is not the case.

Case-insensitive by default via the `i` flag - removed when the `exactly` modifier is used.  The regex is also run with [the `s` and `m` flags](https://github.com/google/re2/wiki/Syntax#user-content-flags).  The `s` flag allows `.` to match newlines (`\n`) and the `m` flag turns on multi-line mode: `^` and `$` match begin/end line in addition to begin/end text. If you do not want these, they can be turned off by negating them:

```
(?-sm)myregex.*
```

If the `exactly` modifier and the `i` flag are both used, the `i` flag wins; flags in the regex have higher precdence than the `exactly` modifier.

For example, if you sent this JSON payload via the V2 Events API:

```json
{
  "payload": {
    "custom_details": {
      "system diagnosis": {
        "issue": "Low disk"
      }
    }
  },
  "links": [
    { "href": "https://diag.test/details", "text": "Diagnosis details" }
  ],
  "important_field": "This is an important value",
  "another_field": "This has a newline\nin it"
}
```

this is how these regular expressions would evaluate

```
raw_event.important_field matches regex 'this'                -> true
raw_event.important_field matches regex exactly 'this'        -> false
raw_event.important_field matches regex exactly '(?i)this'    -> true
raw_event.important_field matches regex '(?-i)this'           -> false

raw_event.another_field matches regex '.in it'                -> true
raw_event.another_field matches regex '(?-s).in it'           -> false

raw_event.another_field matches regex '^in it'                -> true
raw_event.another_field matches regex '(?-m)^in it'           -> false
```

#### Threshold Conditions

Threshold conditions will evaluate to `true` when that particular condition is evaluated a certain number of times _recently_ within a given time period.

Note that the threshold conditions will never return `0` because the "current" event being evaluated is counted as part of the count.  Thus the lowest possible value is `1` and conditions like `trigger_count over 5 seconds < 1` should be avoided.

##### `trigger_count`

Returns a count of how many trigger events have occurred on this rule _recently_ within the giving time period


```
trigger_count over [duration]
```

Generally will be used in conjunction with a numerical operator like these examples:

```
trigger_count over 10 seconds > 5
trigger_count over 1 hour < 10
```

##### `resetting_trigger_count`

(Legacy Threshold Behavior) Exactly the same as `trigger_count` except that once the condition is evaluated to `true`, the internal counter is reset to `0`.  Because of this, it doesn't really make sense to combine this with either of the "less than" operators (`<`, `<=`) because the count will constantly reset.

```
resetting_trigger_count over [duration]
```

Examples:

```
resetting_trigger_count over 10 seconds > 5
resetting_trigger_count over 1 hour > 50
```

#### Other Operations

##### `==`

Returns `true` if both arguments are equivalent and `false` otherwise.

```
[any type] == [any type] -> [boolean]
```

- When using `==` there will be no conversion of arguments
- Comparisons of strings will be case sensitive
- Floats and Integers are comparable. For example, `3.0 == 3` will result in a `true` evaluation. For more information, see the [Note About Comparing Integers and Floats]
- Comparison of non-comparable types returns `false`.

##### `in`

Returns `true` when the first argument falls within the schedule specified by the second argument. Otherwise, returns `false`.

```
now in [schedule] -> [boolean]
```

The `in` operator is only for creating _scheduled_ conditions.  The first arg must be [`now`](#now). The second arg must be a `schedule`.

For example, if you send an Event to an Event Orchestration on Monday, January 3rd, 2022 at noon in Los Angeles (which is 20:00 UTC):

```
now in Mon,Wed,Fri 01:00:00 to 15:00:00 America/Los_Angeles -> true
now in Mon,Wed,Fri 01:00:00 to 15:00:00 Etc/Utc -> false
```

See [the `schedule` type documentation](#schedule) to learn about the types of recurring schedules you can create in PCL.

##### `exists`

Returns `true` if the path exists in the context, `false` otherwise.

```
[path] exists -> [boolean]
```

Generally, a `path` used in an expression will evaluate to the value of the path in the context. In this case, `exists` checks to see if the path exists rather than retrieving the value at the path.

`exists` will still return `true` if the `path` exists even if the value at that path is set to `nil`.

```json
{
  "a": {
    "b": nil,
    "c": 5
  }
}

a.b exists -> true
a.b -> nil
a.c exists -> true
a.c -> 5
a.d exists -> false
a.d -> nil
```

##### `now`

A built-in PCL function that has no arguments and returns a `datetime` representing the "current" time.  For consistency during the evaluation of a complex exression, a single UTC value is determined before the entire expression is evaluated and that value is returned by this function. Could be used for the `datetime` argument in the one-time or recurring schedule conditions in Event Orchestrations.

```
now > 2020-01-01 00:00:00 Etc/UTC
now in Mon,Fri 09:00:00 to 17:00:00 America/New_York
```

<!-- ### Non-Boolean Evaluation

[Non-Boolean Evaluation]:#non-boolean-evaluation

Evaluation of a PCL sub-expression can produce results are ambiguous / non-sensical. A classic example is an invalid type comparison like `2 > 'two'`.

Since PCL will be used as part of the Event processing pipeline, throwing an error and giving up on the entire event evaluation isn't helpful since there is no user available to handle the error.

So PCL treats all non-boolean evaluation as a `false`, allowing the PCL expression to be evaluated consistently as a whole, without error.

Expression | Evaluates like | Result | Warning
-- | -- | -- | --
`2 > 'two'` | `non-boolean evaluation` | ❌ `false` | Type mismatch: `>` requires a [number] or [datetime] on both sides but got `[number] > [string]`
`not 2 > 'two'` | `not (non-boolean evaluation)` -> `not (false)` -> | ✅ `true` | Type mismatch: `>` requires a [number] or [datetime] on both sides but got `[number] > [string]`
`2 >= 'two' or 2 < 10` | `(non-boolean evaluation) or true` -> `false or true` | ✅ `true` | Type mismatch: `>=` requires a [number] or [datetime] on both sides but got `[number] >= [string]`
`2 <= 'two' and 2 < 10` | `(non-boolean evaluation) and true` -> `false and true` | ❌ `false` | Type mismatch: `<=` requires a [number] or [datetime] on both sides but got `[number] <= [string]` -->

<!-- ## Note About Comparing Integers and Floats

[Note About Comparing Integers and Floats]:#note-about-comparing-integers-and-floats

When comparing an integer with a float, the following rule applies: The number that has less precision is converted to the type of the other number before the comparison occurs.
- If the floating point number is within the range `-2^53` to `2^53`, then the floating point has more precision than the integer, so the integer is converted to a float and the numbers are compared.
- If the floating point number is outside that range, then the floating point representation cannot represent contiguous integers. So, the integer will have more precision, and the floating point number is converted to an integer and the numbers are compared.

Since the floating point representation cannot represent all integers greater than `2^53`, i.e. `9007199254740992`, this leads to some awkward behavior. This is not considered a bug, but a recognized artifact of the inaccuracy of floating point in general and beyond this limit specifically. All programming languages that support floating point numbers have similar issues. Here are some examples with explanations:

```
9007199254740992 == 9007199254740992.0 -> true
```

This is as expected the integer and the corresponding floating point number are equal.

```
9007199254740992 == 9007199254740993.0 -> true
```

What is happening here is that `9007199254740993.0` cannot be represented in the floating point representation. So, the floating point number that is chosen is the closest one that _can_ be represented. That happens to be the original number `9007199254740992.0`. The equality then just compares the two numbers which are now the same.

```
9007199254740992 == 9007199254740994.0 -> false
```

Again, as expected. `9007199254740994.0` is the next number that can be represented in floating point, so PCL can tell that they are not the same. Note that as the floats get larger, the gap between representable numbers also gets larger.

For more information about floating point numbers and their peculiarities:

- [What Every Programmer Should Know About Floating-Point Arithmetic](https://floating-point-gui.de)
- [0.30000000000000004.com](https://0.30000000000000004.com)
- [Floating Point Arithmetic: Issues and Limitations](https://docs.python.org/3/tutorial/floatingpoint.html)


[1]: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones "List of TZ database timezones" -->
