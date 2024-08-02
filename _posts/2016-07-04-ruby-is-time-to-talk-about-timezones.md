---
layout: post
title:  "Ruby: is Time to talk about Time Zones"
date:   2016-07-04 12:00:00
categories: ruby datetime timezone
excerpt: "Everything you always wanted to know about Ruby's date, time and time zone issues but were afraid to ask"
disqus: true
archive: false
redirect_from: /2016/07/04/ruby-is-time-to-talk-about-timezones/
---

> Any sufficiently advanced bug is indistinguishable from a feature.<br/>
> ―  Rich Kulawiec

## TL;DR

Always use the `Time` class and correctly stores the time zone information.

## Time

* Based on floating-point second intervals from unix epoch (1970-01-01);
* Has date and time attributes (year, month, day, hour, min, sec, subsec);
* Natively works in either UTC or "local" (aka support to time zones);
* Can handle negative times before unix epoch;
* Can handle time arithmetic in units of seconds;
* Is faster than `Date` or `DateTime` implementation;

```ruby
utc   = Time.utc(2016, 10, 15)   # => 2016-10-15 00:00:00 UTC
utc.zone                         # "UTC"
utc.utc_offset                   # 0

local = Time.local(2016, 10, 15) # => 2016-10-15 00:00:00 -0300
local.zone                       # "BRST"
local.utc_offset                 # -10800

time  = Time.new(2016,10,15,14,35,42,"-03:00") # 2016-10-15 14:35:42 -0300
time.zone                                      # nil
time.utc_offset                                # -10800

now   = Time.now             # 2016-05-16 04:10:23 -0300
unix  = now.to_i             # 146338262
time  = Time.at(unix)        # 2016-05-16 04:10:23 -0300
day   = (24 * 60 * 60)       # 86400
time  = Time.at(unix + day)  # 2016-05-17 04:10:23 -0300
```

<strong>Note:</strong> there's a [Time class one that is part of core Ruby](https://ruby-doc.org/core-2.2.0/Time.html) and there is an additional [Time class that is part of the standard library](https://apidock.com/ruby/files/lib/time.rb). The standard library Time class extends the core Time class by adding some methods. See the documentation of both classes for more details.

## Date

* Based on integer whole-day intervals from an arbitrary "day zero" (-4712-01-01);
* Has date attributes only (year, month, day);
* Can handle date arithmetic in units of whole days;
* Can convert between dates in the ancient Julian calendar to modern Gregorian;
* Don't have support to time zones;
* Is slower than `Time` implementation;

```ruby
require "date"

today = Date.today    # #<Date: 2016-06-19 ((2457559j,0s,0n),+0s,2299161j)>
today.to_time         # 2016-06-19 00:00:00 -0300

time = "2016-06-19 23:11:07 -0300"
date = Date.parse(time) # #<Date: 2016-06-19 ((2457559j,0s,0n),+0s,2299161j)>
date.to_time            # 2016-06-19 00:00:00 -0300
```

## DateTime

* Subclass of `Date`, so whatever you can do with `Date` can be done with `DateTime`;
* Based on fractions of whole-day intervals from an arbitrary "day zero" (-4712-01-01);
* Has date and time attributes (year, month, day, hour, min, sec);
* Can handle date arithmetic in units of whole days or fractions;
* Is slower than `Time` and `Date` implementation;

```ruby
require "date"

now = DateTime.now
=> #<DateTime: 2016-06-19T23:11:07-03:00 ((2457560j,7867s,653070000n),-10800s,2299161j)>
now.to_time        # 2016-06-19 23:11:07 -0300

time = "2016-06-19 23:11:07 -0300"
date = DateTime.parse(time)
=> #<DateTime: 2016-06-19T23:11:07-03:00 ((2457560j,7867s,0n),-10800s,2299161j)>
date.to_time       # 2016-06-19 23:11:07 -0300
```

Further, it has a meaningless "zone" attribute and a hidden `usec` after turning it into a `Time` instance:

```ruby
dt = DateTime.new(2012, 12, 6, 1, 0, 0, "-07:00", 123456)
dt.zone       # => "-07:00"
dt.utc?       # => NoMethodError: undefined method `utc?'
dt.utc_offset # => NoMethodError: undefined method `utc_offset'

dt = DateTime.parse("2016-06-19 23:11:07.456789 -0300")
dt.usec         # => NoMethodError: undefined method `usec'
dt.to_time.usec # => 456789
```

In short, unless you're dealing with astronomical events in the ancient past and need to convert the Julian date (with time of day) to a modern calendar, you don't need to use `DateTime`.

## Benchmark

```ruby
require 'benchmark'
require 'date'

Benchmark.bm(10) do |x|
  x.report('date')     { 100000.times { Date.today   } }
  x.report('datetime') { 100000.times { DateTime.now } }
  x.report('time')     { 100000.times { Time.now     } }
end
```

Result:

```
                user     system      total        real
date        1.250000   0.270000   1.520000 (  1.799531)
datetime    6.660000   0.360000   7.020000 (  7.690016)
time        0.140000   0.030000   0.170000 (  0.200738)
```

As shown before, the `Time` class are faster and should fits the most usage scenarios.

## Time Zone

Knowing that `Time` has total support to time zones, you just remember this mantra: "Aways handle time with the correct time zone".

### Why time zone is important?

* If you application are deployed in the cloud (which probably was), you can't guarantee the time zone of the server running you application;
* Your application database probably will be located on different server and again, you can't guarantee the server time zone configuration;
* Since you application is on Internet, your users came from all corners of world, that have 24 different time zones;

Getting current time with `Time.now` actually gets the current process time zone, which is defined by the `TZ` variable:

```shell
$ date
Sun Jul  3 18:42:33 BRT 2016

$ ruby -e "puts Time.now"
2016-07-03 18:42:33 -0300

$ TZ="America/Vancouver" ruby -e "puts Time.now"
2016-07-03 14:42:33 -0700
```

### Getting the current time

Your application need to know the "base time zone", which means that all different time zones should be converted to this base time zone to be handled correctly.

Consider that you application time zone is `-03:00` (BRT). In order to ensure that you always are getting the correct time, you could do this:

```ruby
TZ = "-03:00"

server_time = Time.now                   # 2016-06-18 05:30:22 +0200
server_time.utc_offset                   # 7200

current_time = server_time.getlocal(TZ)  # 2016-06-18 00:30:22 -0300
current_time.utc_offset                  # -10800
```

That's it. Don't trust on the time zone of `Time.now` and always convert to your local time zone.

You can ease this repetitive task extending the `Time` class this way:

```ruby
TZ = "-03:00"

module TimeExtensions
  def current
    now.getlocal(TZ)
  end
end
Time.extend(TimeExtensions)

Time.now        # 2016-06-18 05:30:22 +0200
Time.current    # 2016-06-18 00:30:22 -0300
```

Also, is a good idea to always parse time with the correct time zone:

```ruby
TZ = "-03:00"

require "time"

Time.parse("2016-06-18 15:33:44")        # 2016-06-18 15:33:44 +0200
Time.parse("2016-06-18 15:33:44 #{TZ}")  # 2016-06-18 15:33:44 -0300
```

This way, you guarantee that the input time, read from a form or a file, always will be handled with the correct time zone (if it wasn't supplied).

### Serialization

In order to ensure that the time zone information will not be lost on time serialization, is a good practice to use the [ISO 8601 standard](https://en.wikipedia.org/wiki/ISO_8601):

```ruby
require "time"

now = Time.current  # 2016-06-18 20:30:22 -0300
now.iso8601         # "2016-06-18T20:30:22-03:00"
```

For parsing, the `iso8601` also handle the time zone and is a very fast alternative to `Time.parse` method:

```ruby
require "time"

data = "2016-06-18T20:30:22-03:00"

Time.iso8601(data) # 2016-06-18 20:30:22 -0300
Time.parse(data)   # 2016-06-18 20:30:22 -0300
```

On APIs, you [should use the ISO 8601 standard](https://apiux.com/2013/03/20/5-laws-api-dates-and-times/) to exchange date and time values. Period.

## Using TZInfo

TZInfo is a gem that allow to use named time zones in order to obtain and convert time instances, also providing daylight saving information.

In the context of getting the right time zone, you can use TZInfo to get the `utc_offset`, also considering the <strong>dst</strong>, of a time zone without need to hard coded it.

For example, instead of hard code the `-03:00`, you can obtain this offset this way:

```ruby
require "tzinfo"

tzinfo = TZInfo::Timezone.get("America/Sao_Paulo")
period = info.current_period

offset = period.offset.utc_total_offset
=> -10800   # 3 hours
```

And if you are in daylight saving time, this will be considered:

```ruby
require "tzinfo"

tzinfo = TZInfo::Timezone.get("America/Sao_Paulo")
period = info.current_period

period.dst?
=> true

offset = period.offset.utc_total_offset
=> -7200    # 2 hours
```

This way, just need to encapsulate this on a `TimeExtensions` class to ease its usage:

```ruby
module TimeExtensions
  require "tzinfo"

  def tzinfo
    TZInfo::Timezone.get(TZ)
  end

  def utc_offset
    period = tzinfo.current_period
    period.offset.utc_total_offset
  end

  def current
    now.getlocal(utc_offset)
  end
end
Time.extend(TimeExtensions)

Time.tzinfo     # #<TZInfo::DataTimezone: America/Sao_Paulo>
Time.utc_offset # -10800
Time.current    # 2016-06-18 00:30:22 -0300
```

## Storing the time zone on database

Many guides and best practices articles say to "always store the time in UTC time zone into database". I think that this is partially true. And I say this: "always store the time on the same time zone with a data type that supports time zone".

### With time zone support

If you are using PostgreSQL, for example, just use the `timestamptz` or `timetz` data types to store time values and you are good to go.

Your database adapter should be able to handle this data type and made the conversions to the correct time zone.

For example, consider the following `posts` table with the `modified_at` column as `timestamp` and the `published_at` column as `timestamptz`:

```
    Column    |            Type             | Modifiers
--------------+-----------------------------+-----------
 title        | text                        |
 modified_at  | timestamp without time zone |
 published_at | timestamp with time zone    |
```

And the Sequel model `Post`, with the databased configured to `UTC` as default time zone:

```ruby
Sequel.database_timezone    = :utc
Sequel.application_timezone = :local

class Post < Sequel::Model
end
```

When a model was created, Sequel convert the time zone to match the time zone defined on table schema:

```ruby
post_time = Time.parse("2016-07-03 20:33:44 -03:00")

Post.create(
  title:        "testing tz is cool",
  modified_at:  post_time,
  published_at: post_time
)

=> #<Post:0x007fb8d0808f78> {
         :title => "testing tz is cool",
   :modified_at => 2016-07-03 20:33:44 -0300,
  :published_at => 2016-07-03 20:33:44 -0300
}
```

Note that both time values have the right time zone offset, but on the database, they was stored this way:

```
    title           |     modified_at     |      published_at
--------------------+---------------------+------------------------
 testing tz is cool | 2016-07-03 23:33:44 | 2016-07-03 20:33:44-03
```

On the column `modified_at`, which type is `timestamp`, the value was converted to UTC and stored without time zone information. But the column `published_at`, which type is `timestamptz`, the original time zone was stored without conversion.

Only the application know that that the `modified_at` column should be converted to the local time and the `published_at` column have the right time zone information.

If other services need to access this database, like a data analytic tool (BI), this information should be shared between consumers. I consider this a data loss since the time zone was lost on the storage. Using data type that supports time zone ensure the consistency of the data.

### Without time zone support

If your database don't have a data type with time zone support storage, you have two alternatives: store time values in UTC or store the time zone offset on a different column.

MySQL, for example, don't have a specific type to store time zone information. The `TIMESTAMP` data type can converts the time to UTC for storage and back from UTC to current time zone for retrieval using the time zone setted in the connection, but don't writes the time zone information on column.

## The Rails way

If you are using Ruby on Rails, you don't need to concern with all this time and time zone related issues. You just need to remember this:

* Always configure your application time zone on `config/application.rb`:

```ruby
config.time_zone = 'Brasilia'
```

* Always get the current time via `current` method:

```ruby
Time.current
=> Sun, 03 Jul 2016 22:46:35 BRT -03:00
```

* Always parse time using the configured time zone:

```ruby
Time.zone.parse("2016-07-03 22:33:45")
=> Sun, 03 Jul 2016 22:33:45 BRT -03:00
```

* And if you are using a database with time zone supports, instead of the classic `t.timestamps`, do this on database migrations:

```ruby
create_table :posts do |t|
  # columns definition
end
add_column :posts, :created_at, :timestamptz
add_column :posts, :updated_at, :timestamptz
```

## The non-Rails way

If you are [not using Rails](https://solnic.eu/2016/05/22/my-time-with-rails-is-up.html), eventually you will need to handle time parsing and/or time zones conversions. You could do this by yourself, implementing these operations on a helper class or extending the `Time` behavior. You will learn a lot in the process.

But if you are looking for a ready-to-use solution, you could use the [CoreExt](https://github.com/rpanachi/core_ext) gem and pick only the `Time` extension:

```ruby
require "core_ext/time"

Time.zone = "America/Sao_Paulo"
Time.zone.parse("16:35:42")   # Sun, 03 Jul 2016 16:35:42 BRT -03:00

now = Time.current            # Sun, 03 Jul 2016 20:31:10 BRT -03:00
now + 2.days                  # Tue, 05 Jul 2016 20:31:10 BRT -03:00
now.iso8601                   # "2016-07-03T20:31:10-03:00"

10.days.ago                   # Fri, 23 Jun 2016 20:31:10 BRT -03:00
```

This gem is a fork of ActiveSupport with many changes to make it more modular and focused only on the <em>extensions</em> of Ruby core classes. You could read more about the motivation and usage examples on my previous post about Hanami migration: [From Rails to Hanami (Lotus) Part 3: Sidekiq Workers, Sequel Plugins, I18n, Timezone issues and Core Extensions](https://rpanachi.com/2016/04/25/from-rails-to-hanami-part3-sidekiq-workers-i18n-timezone-issues-core-ext).

## Ruby 2.4 to_time bugfix

Until Ruby 2.3, the methods `to_time` of `Date` and `DateTime` objects did not preserve the time zone:

```ruby
require 'date'

time = DateTime.strptime('2016-06-21 PST', '%Y-%m-%d %Z')
=> #<DateTime: 2016-06-21T00:00:00-08:00 ((2457561j,28800s,0n),-28800s,2299161j)>

time.zone    # "-08:00"
time.to_time # 2016-06-21 05:00:00 -0300
```

On Ruby 2.4 the time zone will be preserved:

```ruby
require 'date'

time = DateTime.strptime('2016-06-21 PST', '%Y-%m-%d %Z')
=> #<DateTime: 2016-06-21T00:00:00-08:00 ((2457561j,28800s,0n),-28800s,2299161j)>

time.zone    # "-08:00"
time.to_time # => 2016-06-21 00:00:00 -0800
```

## Conclusion

Time zones exists in our real world and should be respected on software engineering. Ruby provide a very useful API to handle `Time` but a little deficient one to handle time zones.

Know the Ruby `Time` capabilities and always remember to use and store the correct time zone on database.

## References

* [Are the Date, Time, and DateTime classes necessary?](https://stackoverflow.com/questions/5941613/are-the-date-time-and-datetime-classes-necessary)
* [Calculate the offset, in hours, of a given timezone from UTC in ruby?](https://stackoverflow.com/questions/9962038/how-do-i-calculate-the-offset-in-hours-of-a-given-timezone-from-utc-in-ruby)
* [The 5 laws of API dates and times](https://apiux.com/2013/03/20/5-laws-api-dates-and-times/)
* [TZInfo - Ruby Timezone Library](https://github.com/tzinfo/tzinfo)
* [PostgreSQL: Date/Time Types](https://www.postgresql.org/docs/9.1/static/datatype-datetime.html)
* [MySQL: The DATE, DATETIME, and TIMESTAMP Types](https://dev.mysql.com/doc/refman/5.5/en/datetime.html)
* [Working with time zones in Ruby on Rails](https://www.varvet.com/blog/working-with-time-zones-in-ruby-on-rails/)
* [The Exhaustive Guide to Rails Time Zones](https://danilenko.org/2012/7/6/rails_timezones/)
* [Behavior changes in Ruby 2.4](https://wyeworks.com/blog/2016/6/22/behavior-changes-in-ruby-2.4/)

Learn something new about Ruby time or time zones? Consider to share this post with your friends or co-workers. For questions, comments or suggestions, use the comments below. Code hard and success!
