Title: Temporal (Javascript's new Date-Time API) vs java.time (JSJoda on JS runtimes)
Date: 2020-08-02
Tags: clojure,javascript,java,date-time

*UPDATE 2024-03:* Since this blog was published the Temporal API has evolved and addressed many of the issues raised here. Notably, 

* the Absolute entity is now Instant, the same as java.time  
* there is a ZonedDateTime entity
* 'Plain' is the prefix equivalent to java.time's 'Local'
* there are now facilities for truncation/rounding in the API
* there is a new library [Tempo](https://github.com/henryw374/tempo) that targets both java.time and Temporal

Original content from here onwards

If you are involved in the manufacture of computers in Europe then you're probably already aware that your
friendly local 39th technical committee has it's own <a href="https://www.ecma-international.org/publications/standards/Ecma-262.htm" target="_blank">toy scripting language</a> and that language's support for 
dates and times is somewhat lacking (<a href="https://www.youtube.com/watch?v=aVuor-VAWTI" target="_blank">good talk on that</a>). 
Well, work is currently underway to change that 
situation and in this post I'm going to compare the new API, called `Temporal`, to a similar effort from a few years back
that was made for `Java` that resulted in a platform API called `java.time`.

At the time of writing `Temporal` (<a href="https://github.com/tc39/proposal-temporal/tree/2d35fa3ded4d8cef5d9a944365994ae60b5ed663" target="_blank">sha</a>) 
is 'Stage 2' meaning it's still a work in progress.
 The Temporal authors have created a <a href="https://docs.google.com/forms/d/e/1FAIpQLSeYdvnDZZS6tKn18jiomfN7u_rq-_-_BqOevTzAcfgE3J4kHA/viewform" target="_blank">survey</a>
for any feedback you might have.

There is already an <a href="https://github.com/tc39/proposal-temporal/issues/105" target="_blank">open issue in the Temporal github</a> to document comparison
to other date-time libs and since that has been open for a year and a half already, I thought I'd make a start at least for comparison
to one other lib, `java.time` aka (threeten), available to the JS world as <a href="https://js-joda.github.io/js-joda/" target="_blank">JSJoda</a>.

# Motivation

I maintain a date-time library that targets both Javascript and the JVM (Java runtime), <a href="https://github.com/henryw374/cljc.java-time" target="_blank">cljc.java-time</a>.

It has the API of `java.time` and on JS platforms uses <a href="https://js-joda.github.io/js-joda/" target="_blank">JSJoda</a> under the hood. 

So the value proposition is that users only need to know one API for both platforms and that the same date-time logic can be written to target 
either or both.

Great though JSJoda is, it is not the platform API of Javascript of course and is by necessity a pretty chunky lot of JS code. If Temporal
gets implemented on JS platforms then maybe JSJoda could be implemented on top of it, or my lib could drop JSJoda and 
use Temporal directly.

# Comparison

**Note:** *Both `Temporal` and `JSJoda` are included in this page, so you can open your browser's JS console and paste in all of the 
code snippets*.

*tl;dr* IMO Temporal and java.time are very similar overall: they have mostly the same set of entities and there is support for going to and from the majority of 
<a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO8601</a> representations). Temporal is a smaller API overall, but of course
 the gaps could be filled by user libraries if desired.

I start by comparing the main entities and then go on to look at some specific use-cases that I think are interesting. 
IOW this not a full comparison of every entity and method, but if there's something important you think is missing, 
please mention it in the comments at the end. 

## The main entities

Temporal has a subset of the entities of java.time (see table below), but the entities it does have are what I guess the authors consider to be 
the fundamental ones. Some names are different so **in the discussion after, I'll only use the `java.time` entity names for clarity**.

<table>
  <thead>
    <tr>
      <th><a href="https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html" target="_blank">java.time</a></th>
      <th><a href="https://github.com/henryw374/time-literals" target="_blank">Time Literal</a> Example</th>
      <th>Temporal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Instant</td>
      <td><code class="language-plaintext highlighter-rouge">#time/instant "2018-07-25T07:10:05.861Z"</code></td>
      <td>Absolute</td>
    </tr>
    <tr>
      <td>ZoneId</td>
      <td><code class="language-plaintext highlighter-rouge">#time/zone "Europe/London"</code></td>
      <td>TimeZone</td>
    </tr>
    <tr>
      <td>LocalDateTime</td>
      <td><code class="language-plaintext highlighter-rouge">#time/date-time "2018-07-25T08:08:44.026"</code></td>
      <td>DateTime</td>
    </tr>
    <tr>
      <td>LocalDate</td>
      <td><code class="language-plaintext highlighter-rouge">#time/date "2039-01-01"</code></td>
      <td>Date</td>
    </tr>
    <tr>
      <td>LocalTime</td>
      <td><code class="language-plaintext highlighter-rouge">#time/time "08:12:13.366"</code></td>
      <td>Time</td>
    </tr>
    <tr>
      <td>YearMonth</td>
      <td><code class="language-plaintext highlighter-rouge">#time/year-month "3030-01"</code></td>
      <td>YearMonth</td>
    </tr>
    <tr>
      <td>MonthDay</td>
      <td><code class="language-plaintext highlighter-rouge">#time/month-day "12-25"</code></td>
      <td>MonthDay</td>
    </tr>
    <tr>
      <td>Period</td>
      <td><code class="language-plaintext highlighter-rouge">#time/period "P1D"</code></td>
      <td>Duration</td>
    </tr>
    <tr>
      <td>Duration</td>
      <td><code class="language-plaintext highlighter-rouge">#time/duration "PT1S"</code></td>
      <td>Duration</td>
    </tr>
    <tr>
      <td>DateTimeFormatter</td>
      <td>n/a</td>
      <td>(NOT PRESENT)</td>
    </tr>
    <tr>
      <td>Clock</td>
      <td>n/a</td>
      <td>Temporal.now</td>
    </tr>
    <tr>
      <td>Month</td>
      <td><code class="language-plaintext highlighter-rouge">#time/month "JUNE"</code></td>
      <td>(NOT PRESENT)</td>
    </tr>
    <tr>
      <td>Year</td>
      <td><code class="language-plaintext highlighter-rouge">#time/year "3030"</code></td>
      <td>(NOT PRESENT)</td>
    </tr>
    <tr>
      <td>ZonedDateTime</td>
      <td><code class="language-plaintext highlighter-rouge">#time/zoned-date-time "2018-07-25T08:09:11.227+01:00[Europe/London]"</code></td>
      <td>(TBD)</td>
    </tr>
    <tr>
      <td>DayOfWeek</td>
      <td><code class="language-plaintext highlighter-rouge">#time/day-of-week "TUESDAY"</code></td>
      <td>(NOT PRESENT)</td>
    </tr>
    <tr>
      <td>OffsetDateTime</td>
      <td><code class="language-plaintext highlighter-rouge">#time/offset-date-time "2018-07-25T08:11:54.453+01:00"</code></td>
      <td>(TBD)</td>
    </tr>
    <tr>
      <td>OffsetTime</td>
      <td><code class="language-plaintext highlighter-rouge">#time/offset-time "08:12:13.366+01:00"</code></td>
      <td>(NOT PRESENT)</td>
    </tr>
  </tbody>
</table>

<a href="https://github.com/henryw374/time-literals" target="_blank">//]: #  | <a href="https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html" target="_blank">java.time</a>           |  [Time Literal</a> Example      | Temporal    |
[//]: #  |---------------------|---------------|-------------|
[//]: #  | Instant             |  `#time/instant "2018-07-25T07:10:05.861Z"`     |  Absolute |
[//]: #  | ZoneId  | `#time/zone "Europe/London"` | TimeZone |
[//]: #  | LocalDateTime  | `#time/date-time "2018-07-25T08:08:44.026"` | DateTime |
[//]: #  | LocalDate      | `#time/date "2039-01-01"` | Date |
[//]: #  | LocalTime      | `#time/time "08:12:13.366"` | Time |
[//]: #  | YearMonth      | `#time/year-month "3030-01"`  | YearMonth |
[//]: #  | MonthDay       | `#time/month-day "12-25"`  | MonthDay        |
[//]: #  | Period         | `#time/period "P1D"`  | Duration  |
[//]: #  | Duration       | `#time/duration "PT1S"`  | Duration |
[//]: #  | DateTimeFormatter | n/a | (NOT PRESENT)    |
[//]: #  | Clock  | n/a   | Temporal.now |
[//]: #  | Month          | `#time/month "JUNE"`  | (NOT PRESENT)         |
[//]: #  | Year          | `#time/year "3030"`  | (NOT PRESENT)         |
[//]: #  | ZonedDateTime | `#time/zoned-date-time "2018-07-25T08:09:11.227+01:00[Europe/London]"` | (TBD) |
[//]: #  | DayOfWeek      | `#time/day-of-week "TUESDAY"`  | (NOT PRESENT)         |
[//]: #  | OffsetDateTime |  `#time/offset-date-time "2018-07-25T08:11:54.453+01:00"` | (TBD)          | 
[//]: #  | OffsetTime | `#time/offset-time "08:12:13.366+01:00"`  | (NOT PRESENT)          |

## Entity Mismatch

### Period & Duration

These entities represent a span of time which is not attached to a timeline.

java.time allows for negative spans, whereas Temporal does not yet - but [looks like it will](https://github.com/tc39/proposal-temporal/issues/782).

A java.time Duration instance stores time as an amount of seconds, for example 5.999999999 seconds.

A java.time Period instance stores amounts of years, months and days, for example -1 years, 20 months and 100 days

A Period of 1 day, is not equivalent to to a Duration of 86400 seconds (24 hours) of course, because `1 day` is not always 24 hours,
due to things like DST & leap seconds.

Temporal has combined these two entities into one, called Duration. Java has an equivalent entity <a href="https://www.threeten.org/threeten-extra/apidocs/org.threeten.extra/org/threeten/extra/PeriodDuration.html" target="_blank">PeriodDuration</a>
in the official addon lib for java.time.

Here is an example of adding a non-24-hour day in java.time

```
z = JSJoda.ZoneId.of('Europe/Berlin')
zdt = JSJoda.LocalDateTime.parse('2019-03-31T00:00:00').atZone(z)
zdtPlusDay = zdt.plusDays(1)
JSJoda.Duration.between(zdt, zdtPlusDay).toString()
=> "PT23H" (means 23 hours)
```

The equivalent with Temporal operation is done with LocalDateTime, then converting the results to Instants
 
```
dt = Temporal.DateTime.from('2019-03-31T00:00:00')
dt2 = dt.plus({days: 1})
z = Temporal.TimeZone.from('Europe/Berlin') 
z.getAbsoluteFor(dt2).difference(z.getAbsoluteFor(dt), {largestUnit: 'hours'}).toString()
=> "PT23H" (means 23 hours)
```

### Year, Month & DayOfWeek

Temporal just uses numbers where java.time would use these entities. Like java.time though, numbering starts at 1.

Partly this may have been done because you don't get the same compile-time checks with JS, but also I would guess 
this makes Temporal more easily work with other calendar systems. I imagine when working with non-Gregorian calendars
in java.time one could avoid using Month and DayOfWeek.

### <a href="https://docs.oracle.com/javase/8/docs/api/java/time/ZonedDateTime.html" target="_blank">java.time.ZonedDateTime</a>

This entity represents a point on the timeline, in a place, an example being

```
JSJoda.ZonedDateTime.parse("2018-07-25T08:09:11.227+01:00[Pacific/Honolulu]")
```

Temporal can parse that same string to an Instant, but it loses the zone/offset info

```
Temporal.Absolute.from("2018-07-25T08:09:11.227+01:00[Pacific/Honolulu]")
```

In place of this, we can create objects containing a zone and either an Instant or a LocalDateTime.

There is a draft proposal for <a href="https://github.com/tc39/proposal-temporal/pull/700" target="_blank">ZonedDateTime in Temporal</a>

### OffsetTime & OffsetTime 

I've never had a use for OffsetTime, so let's skip over it.

OffsetDateTime functionality is contained within ZonedDateTime, so I'm not considering it separately.
 
## Interesting Examples

I am going to look at how to achieve various use-cases with both APIs, choosing examples I think are interesting.

### Calendar-aware time conversion

Instant is not aware of calendars (e.g. DST, months, leap seconds etc), it's just a straightforward amount of nanos since an arbitrary 
point in time. One 
of the major noob java.time question topics stems from not being aware of this. For example
trying to print the day of the month from an Instant doesn't work unless you provide a zone, or trying to add a year to 
an Instant - that kind of thing.

Java.time refers to this topic as <a href="https://docs.oracle.com/javase/tutorial/datetime/iso/overview.html" target="_blank">human vs machine time</a> whereas 
Temporal refers to entities as being 'Calendar-aware' or not, which seems a more self-explanatory definition. 

Going from calendar-aware to Instant (non-calendar-aware) can involve disambiguation. For example on a DST change,
a wall-clock time can happen twice (the clocks 'go back') or not at all (the clocks 'go forward').


#### Disambiguation 

Here is an example of where we have a wall clock time that doesn't exist in a zone and are converting 
it to a ZonedDateTime. See how we input the hour as '2', but it comes out as '3':   

```
z = JSJoda.ZoneId.of('Europe/Berlin')
JSJoda.LocalDateTime.parse('2019-03-31T02:45:00').atZone(z).toString()
=> "2019-03-31T03:45+02:00[Europe/Berlin]" (which is Instant "2019-03-31T01:45:00Z")
```

<a href="https://tc39.es/proposal-temporal/docs/ambiguity.html" target="_blank">Temporal docs section on resolving ambiguity</a>

Temporal has the same default behaviour as java.time, but you can choose other options:
```
tz = new Temporal.TimeZone('Europe/Berlin');
dt = new Temporal.DateTime(2019, 3, 31, 2, 45);
tz.getAbsoluteFor(dt, { disambiguation: 'earlier' }); // => 2019-03-31T00:45Z
tz.getAbsoluteFor(dt, { disambiguation: 'later' }); // => 2019-03-31T01:45Z
tz.getAbsoluteFor(dt, { disambiguation: 'compatible' }); // => 2019-03-31T01:45Z
tz.getAbsoluteFor(dt, { disambiguation: 'reject' }); // throws
```

A similar example would be finding out the wall clock time of when a day starts - it's not always midnight!

Here we find out that the day starts at 1 a.m.
```
z = JSJoda.ZoneId.of("America/Sao_Paulo")
JSJoda.LocalDate.parse('2015-10-18').atStartOfDay(z).toString()
=> "2015-10-18T01:00-02:00[America/Sao_Paulo]" 
``` 

I'll leave it as an exercise for the reader to do the same in Temporal ;-)

#### Arithmetic 

Since Instant isn't aware of calendars, you can add or take away seconds, but not months or years.

What about days though? As I said, a day is not 24 hours, but wrt Instant, Temporal treats it like it is:

```
Temporal.now.absolute().plus({ months: 5 }); // fail - as expected
Temporal.now.absolute().plus({ days: 5 }); // no fail - days in this context means 24 hours
```

In fairness, so does java.time, so both APIs are consistent on this

``` 
JSJoda.Duration.ofDays(1).toString()
=> "PT24H"
```

### Truncation

Example of truncating a java.time Instant to whole hours

```
JSJoda.Instant.parse("2020-07-30T21:29:54.697Z").truncatedTo(JSJoda.ChronoUnit.HOURS).toString()
=> "2020-07-30T21:00:00Z"
```

Although <a href="https://tc39.es/proposal-temporal/docs/cookbook.html#round-a-time-down-to-whole-hours" target="_blank">Temporal has facilities for Rounding</a>
using the `with` method, this just works on the fields a Temporal object has. Since Instant only has nanos field,
it doesn't have a `with` method, so to achieve the above we go via a calendar aware object:

```
z = Temporal.TimeZone.from('UTC')
dt = z.getDateTimeFor(Temporal.Absolute.from("2020-07-30T21:29:54.697Z"))
rounded = dt.with({minute: 0, second: 0, millisecond: 0, microsecond: 0, nanosecond: 0 })
z.getAbsoluteFor(rounded)
=> 2020-07-30T21:00Z
``` 

### Parsing & Formatting

java.time has <a href="https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html" target="_blank">DateTimeFormatter</a>,
which is used for converting to and from any string representation.

Temporal doesn't have an API for parsing, although there is an <a href="https://github.com/tc39/proposal-temporal/issues/796" target="_blank">issue </a>.

For printing, Temporal provides some facililities via `toString` - see <a href="https://tc39.es/proposal-temporal/docs/cookbook.html#zoned-instant-from-instant-and-time-zone" target="_blank">here</a>,
and Intl.DateTimeFormat, as in <a href="https://tc39.es/proposal-temporal/docs/cookbook.html#zoned-instant-from-instant-and-time-zone" target="_blank">this example</a>

### Accessing Properties

There is no direct equivalent of <a href="https://docs.oracle.com/javase/tutorial/datetime/iso/temporal.html" target="_blank">java.time temporal package</a> 
 in Temporal at present.
 
 You can access the fields of entities though, for example:
 
 ``` 
Temporal.DateTime.from('2019-03-31T00:00:00').day
=> 31 
```


# Conclusion

Temporal and java.time are fundamentally similar. Temporal has a smaller API and possibly that results in something that's
easier to learn, but results in more verbose code. Personally I value ease-of-learning much more. 

Date-time logic is just like math: you can type stuff in and you'll get answers... but are they the right ones? 
Have you got a type system that will let you know that you got it wrong? (please let me know if you do!) But it's not 
like math because it's fundamentals are way more complex... so I just want to know one API and know it really well.
Because of that I created a <a href="https://github.com/henryw374/cljc.java-time" target="_blank">date-time library to target both Java and Javascript</a>.
 
So, of course I would have been happy if proposal-temporal had just decided to copy java.time! Well, they haven't,
but that's not a show-stopper for lib by any means. It's great that Temporal is happening at all and hopefully will
make it's way to our browsers and other JS runtimes in the near future.

<script src="https://tc39.es/proposal-temporal/docs/playground.js" ></script>
<script src="https://cdn.jsdelivr.net/npm/@js-joda/core@3.0.0/dist/js-joda.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@js-joda/timezone@2.3.0/dist/js-joda-timezone-10-year-range.js"></script> 
<script src="https://cdn.jsdelivr.net/npm/@js-joda/locale_en-us@3.1.1/dist/index.min.js"></script>
