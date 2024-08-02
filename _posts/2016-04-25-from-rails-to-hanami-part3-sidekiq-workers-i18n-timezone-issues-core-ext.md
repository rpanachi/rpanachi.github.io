---
layout: post
title:  "From Rails to Hanami (Lotus) Part 3: Sidekiq Workers, Sequel Plugins, I18n, Timezone issues and Core Extensions"
date:   2016-04-25 00:00:00
categories: ruby hanami rails sequel tutorial
excerpt: "The final chapter of the Rails to Hanami migration saga"
disqus: true
archive: false

redirect_from: /2016/04/25/from-rails-to-hanami-part3-sidekiq-workers-i18n-timezone-issues-core-ext/
---

> The most basic question is not what is best, but who shall decide what is best.<br/>
> ―  Thomas Sowell

## Recap

In first part [From Rails to Hanami (Lotus) Part 1: Container Architecture, Models, Views and Assets](/2016/03/28/from-rails-to-hanami-part1-container-architecture-model-views-assets), I show how was the migration of two Rails applications to a single Hanami container, mounting each application individually, reading from Postgres database using Sequel and doing all write operation throught the Rails "core" application via REST api.

In second part [From Rails to Hanami (Lotus) Part 2: Sequel Migrations, Model Validations, Specs and Fixtures](/2016/04/11/from-rails-to-hanami-part2-sequel-migrations-model-validations-specs-fixtures), all API business logic was migrated to Hanami project, as well the database migrations are converted to Sequel that allowed the specs to be performed. From this point, the Hanami core API was deployed in production.

The last chapter of this migration saga will cover the minor (but not less important) details of application infrastructure as async workers and reveal some issues related from the lack of Rails support (and its pampering).

## Sidekiq Workers

Since [Sidekiq](https://github.com/mperham/sidekiq) is framework-agnostic, the workers classes was simply copied from Rails project and added to application stack. To configure Sidekiq client and server, I added the file `config/sidekiq.rb` with the following:

```ruby
require 'sidekiq'
redis_config = {url: "redis://localhost:6347"}

Sidekiq.configure_server do |config|
  config.redis = redis_config
end

Sidekiq.configure_client do |config|
  config.redis = redis_config
end
```

And added the worker process to `Procfile` requiring the application environment on initializer:

```procfile
worker: bundle exec sidekiq -r ./config/environment.rb
```

That's it. Sidekiq will run as worker process on our PaaS infrastructure.

## I18n

Thanks to `i18n` gem, this task was easily solved.

All core locales, which is basically the models attributes and enums, was declared in `config/locales` directory. To initialize them, I added the file `config/locale.rb` with the following:

```ruby
require 'i18n'

I18n.load_path      = Dir[Hanami.root.join("config/locales/**/*.yml")]
I18n.default_locale = "pt-BR"
I18n.backend.load_translations
```

To lookup a translation, just need to call:

```ruby
I18n.t "kind", scope: "models.account/kind"
```

The locales specific for each application was declared in `apps/application/config/locales` directory. Each application need to initialize its own locale just adding this inside the `configure` block on `appliation.rb`:

```ruby
configure do
  I18n.load_path << Dir[root.join("config/locales/**/*.yml")]
  I18n.backend.load_translations
end
```

To lookup a translation, declared nested to the application namespace, just need to call:

```ruby
I18n.t "title", scope: "admin.users"
```

To help with this repetitive task, I add this helper:

```ruby
module LocaleHelper
  def t(key, options = {})
    if key.to_s[0] == '.'
      app, _, controller, action = self.class.name.split("::").map(&:downcase)
      scope    = "#{app}.#{controller}"
      defaults = [:"#{scope}#{key}"]
      scope    << ".#{action}"
      defaults << [:"#{scope}#{key}"]
      defaults << options[:default] if options[:default]

      options[:default] = defaults
      key = "#{scope}#{key}"
    end

    ::I18n.t key, options
  end
end
```

And included it on view configuration block inside `application.rb`:

```ruby
configure do
  view.prepare do
    include LocaleHelper
  end
end
```

That's it. All applicaion's views and templates were able to lookup translations with no efforts simply calling:

```erb
<h1><%= t ".title" %></h1>
```

## Sequel plugins and extensions

[Sequel](https://github.com/jeremyevans/sequel) is a slim framework and its architecture allow to be easily [extended with plugins and extensions](https://sequel.jeremyevans.net/plugins.html). They could be enabled individually on each model:

```ruby
class User < Sequel::Model
  plugin :validation_helpers
end
```

Or globally (on database configuration):

```ruby
Sequel::Model.plugin :validation_helpers
```

To make the Sequel models behave like ActiveModel, the following plugins and extensions were used:

- [timestamps](https://sequel.jeremyevans.net/rdoc-plugins/classes/Sequel/Plugins/Timestamps.html): automatically sets the `created_at` and `updated_at` attributes.

- [update_or_create](https://sequel.jeremyevans.net/rdoc-plugins/classes/Sequel/Plugins/UpdateOrCreate.html): add the method `update_or_create` on model, similar to `find_or_create` of ActiveRecord:

```ruby
Album.update_or_create(artist: "Lionel", name: "Hello")
```

- [validation_helpers](https://sequel.jeremyevans.net/rdoc-plugins/classes/Sequel/Plugins/ValidationHelpers.html): add validations to models:

```ruby
class Album < Sequel::Model
  plugin :validation_helpers
  def validate
    super
    validates_min_length 1, :num_tracks
  end
end
```

- [boolean_readers](https://sequel.jeremyevans.net/rdoc-plugins/classes/Sequel/Plugins/BooleanReaders.html): define `attribute?` methods for boolean columns.

- [defaults_setter](https://sequel.jeremyevans.net/rdoc-plugins/classes/Sequel/Plugins/DefaultsSetter.html): makes the column getter methods return the default values for new objects, if the values have not already been set.

- [dirty](https://sequel.jeremyevans.net/rdoc-plugins/classes/Sequel/Plugins/Dirty.html): traces changes on model's attributes:

```ruby
artist.name           # => 'Foo'
artist.name = 'Bar'
artist.column_changes # {:name => ['Foo', 'Bar']}
```

- [delay_add_association](https://sequel.jeremyevans.net/rdoc-plugins/classes/Sequel/Plugins/DelayAddAssociation.html): allow adding of associated objects to a new (unsaved) object. New associated objects will be saved after saving the parent object, allowing this:

```ruby
artist = Artist.new(name: "Lionel")
artist.albums << Album.new(name: "Hello")
artist.save
```

- [polymorphic](https://github.com/jackdempsey/sequel_polymorphic): available as individual gem, allow to use polymorphic associations.

Check the [Sequel for ActiveRecord users](https://sequel.jeremyevans.net/rdoc/files/doc/active_record_rdoc.html) for more plugins and complete info.

## Timezone issues

Since all data written in database was on UTC, the datetime attributes must be converted to application's timezone for correct use.

In Rails, this timezone conversion is transparently made by ActiveRecord (using ActiveSupport), since you configure it. For our luck, [Sequel has this support too](https://sequel.jeremyevans.net/rdoc/classes/Sequel/Timezones.html).

Add the gem `tzinfo` to `Gemfile` and enable the [named_timezone](https://github.com/jeremyevans/sequel/blob/master/lib/sequel/extensions/named_timezones.rb) extension adding this on Sequel configuration file:

```ruby
Sequel.extension(:named_timezone)

Sequel.database_timezone    = :utc
Sequel.application_timezone = "America/Sao_Paulo"
Sequel.tzinfo_disambiguator = proc { |datetime, periods| periods.first }
```

The `tzinfo_disambiguator` must be configured in case of the parsing detect more than one timezone. Just return the first period and you will be ok.

By default, all timestamp attributes will be parsed in `DateTime` instances, with the correct `utc_offset` setted, example:

```ruby
(irb):0 > a = Account.last
=> <Account:0x007f91e8ff0330>
(irb):0 > a.created_at
=> Fri, 08 Apr 2016 18:49:24 -0300
(irb):0 > a.created_at.utc_offset
=> -10800
(irb):0 > a.created_at.class
=> DateTime < Date
```

As the same way, all time and datetime instances will be converted into `utc` by Sequel:

```ruby
(irb):0 > now = DateTime.now
=> Fri, 22 Apr 2016 18:40:08 -0300
(irb):0 > Account.find("created_at < ?", now)
# (0.000572s) SELECT * FROM "accounts" WHERE (created_at < '2016-04-22 21:40:08.802735+0000') LIMIT 1
=> <Account:0x007fd1902ff210>
```

The object/database timezone issues were solved by Sequel, but was still missing helpers to handle datetime parsing and/or get the current time with correct timezone. The quick and dirty solution was to extend the Time class:

```ruby
module TimeZone
  TZ = "-03:00" # America/Sao_Paulo

  def self.included(base)
    base.extend(ClassMethods)
  end

  module ClassMethods
    def current
      now.getlocal(TZ)
    end
  end

  def with_timezone
    self.getlocal(TimeZone::TZ)
  end
end

Time.send(:include, TimeZone)
```

Solved, but we still have issues like daylight saving time and calculations. We need a better and more robust solution, an implementation of ActiveSupport but just for these core extensions...

## Core Extensions (aka ActiveSupport replacement)

For example, ActiveSupport bring us the following pampering:

```ruby
Time.current + 1.day
due_on = 5.days.ago
Worker.perform_in(10.seconds)
```

Aiming on the lack of these features, I decide to fork only the "core extensions" from ActiveSupport into a new gem. Yes, I did it :)

Introducing **[core_ext](https://github.com/rpanachi/core_ext)** gem, bringing the ActiveSupport's core extensions, isolated in a `CoreExt` namespace. Also, you can pick only the extension that you need, like:

```ruby
irb(main):001:0> require "core_ext/hash"
=> true
irb(main):002:0> {a: 1}.stringify_keys
=> {"a"=>1}
irb(main):003:0> require "core_ext/time"
=> true
irb(main):004:0> Time.zone = "America/Sao_Paulo"
=> "America/Sao_Paulo"
irb(main):005:0> Time.zone.parse "2015-09-28 15:44:33"
=> Mon, 28 Sep 2015 15:44:33 BRT -03:00
irb(main):006:0> 1.day.ago
=> 2016-04-21 18:55:50 -0300
```

That's it. No more lack of ActiveSupport features without its external "active" dependencies :)

## Conclusion

Was a lot of work, research, adaptation, refactoring and a little of faith.

But we reached our goals:

- The development of the applications was integrated, sharing the same "core";
- The performance was a little better;
- The memory comsumption was much less;
- We have fun during the migration process :)

As consequence of giving up from Rails:

- The development process was a little more complex due to non-standard framework;
- A slight learn curve when new members join the team;
- Lesser options of gems and frameworks that offer support to Hanami;

At end of day, looks like we win more than loose.

Hanami proved to be a great framework, performed very well on production, but lack some helpful features like a more flexible object-relational model, a better extensible architecture like Railtie and a more detailed documentation of advanced features.

Without the Rails convenience and magic, we was forced to change some implementations and throw away many unnecessary code that was inserted on application just by "follow the standard", which gave us much more control and knowledge about the code, making it better and more testable.

My final thoughts about this experience:

- Choose the best framework to fit your needs. Rails is great, but you don't need to use it just because "everybody is using too";

- Don't have fear of change. Try new tools and frameworks. Experiment new approaches and concepts to solve problems;

- Measure your application performance. You need metrics to tell if you are on the right or wrong way;

- Remember: every web application is a request/response processor. You could achieve your goals using pure Rack.

Consider to share this post with your friends or co-workers. For questions, comments or suggestions, use the comments below. Code hard and success!
