---
layout: post
title:  "From Rails to Hanami (Lotus) Part 1: Container Architecture, Models, Views and Assets"
date:   2016-03-28 00:00:00
categories: ruby hanami rails sequel tutorial
excerpt: "A pragmatic approach to migrate Rails apps into Hanami container"
disqus: true
archive: false
redirect_from: /2016/03/28/from-rails-to-hanami-part1-container-architecture-model-views-assets/
---

> Those who cannot change their minds cannot change anything.<br/>
> ―  George Bernard Shaw

## The problem

A year ago, my company started a big project from scratch. Given the opportunity, we decided to build a Rails API, that would be the system core, centralizing all the business logic, storing data on Postgresql and making calculations with a dozen of Sidekiq workers. And two Rails client applications: one web interface for customers and another for administrators. Both are practically logicless and access all data from the core API.

All three applications was made using Rails 4.2 and a few common gems to handle validation, authentication, AWS, logging, etc. There was deployed on [Docker](https://www.docker.com/) containers managed by [Deis](https://deis.io/), using separated containers for each app and another for workers ([Sidekiq](https://github.com/mperham/sidekiq)).

After 6 months of development, we launched a beta with restricted access and we had an unpleasant surprise: each application are booting up with almost 500MB of RAM and started to grow. We made a benchmark with a very heavy payload, and the API reaches 4GB of RAM, becoming unresponsive :(

Was our first experience deploying Rails on Docker containers. What's the problem? Memory leak? Outdated gems? Containers orchestration? The answer never shows up. After a lot of debugging and a little use of caching, the memory consumption was reduced a little bit, but still be a large number for such little load scenario.

Another point, besides that resources consumption problem, during development we realized how much work is needed to maintaining the client apps syncronized with the API. Was the double of work, or in our case, the triple. Literally, a pain in the ass!

## Rethinking the Strategy

[Rails](https://rubyonrails.org/) is a very fun framework and has facilited my work for almost 8 years. I've been using it on production successfully since version 2.0, which made me to contribute to [several open source projects](https://github.com/rpanachi) as well give me more time to focus on business problems.

And now the game is changing: containers has made the microservices architecture cheaper and easily available to our industry. And Rails is still focusing on the monolith :(

It's time to change the way of thinking: look for an alternative that **make productive the development, maintenance and deployment** of services (or microservices, if you prefer) on containers, and in a way that **each container consumes the minimum of resources possible**.

## The Hanami Way

I wanted to validate if using the [Hanami](https://hanamirb.org/) container architecture, we are able to **develop the applications as a monolith but deploys each one separately**, and of course, its performance.

To do that, I made a simple prove of concept with an application writing data on Postgres database thought a JSON api and another web application reading this data, applying a search and pagination, rendering HTML.

To get real stats, I deployed these applications on our production infrastructure and made a bunch of benchmark tests, and for my surprise, the results was very well: the application responded with 200 requests/second and the memory consumption was lower than 100MB per each process. In this scenario, means that we can handle 5x more load with Hanami than with Rails with the same memory consumption \o/

The performance checklist was validated. The reduced development time was also validated during the POC.

We was ready to put Hanami on proof and decided to migrate first the read-only applications to Hanami and keep all the write operations with the working Rails API.

And here comes the fun :)

## Bootstrap the Hanami Container

Consider that my application is called Bookshelf and has three "apps", being the api, admin and dashboard.

My first goal is to boot a Hanami container mounting each application individually, since in production they will be served in separately endpoins.

My database is Postgresql and my tests are writen in RSpec. So, I created the Hanami and de API app, running:

```shell
$ hanami new bookshelf --application_name=api --db=postgresql --arch=container --test=rspec
      create  .hanamirc
      create  .env
      create  .env.development
      create  .env.test
      create  Gemfile
      create  config.ru
      create  config/environment.rb
      create  lib/bookshelf.rb
      create  lib/config/mapping.rb
      create  public/.gitkeep
      create  config/initializers/.gitkeep
      create  lib/bookshelf/entities/.gitkeep
      create  lib/bookshelf/repositories/.gitkeep
      create  lib/bookshelf/mailers/.gitkeep
      create  lib/bookshelf/mailers/templates/.gitkeep
      create  spec/bookshelf/entities/.gitkeep
      create  spec/bookshelf/repositories/.gitkeep
      create  spec/bookshelf/mailers/.gitkeep
      create  spec/support/.gitkeep
      create  db/migrations/.gitkeep
      create  Rakefile
      create  .rspec
      create  spec/spec_helper.rb
      create  spec/features_helper.rb
      create  spec/support/capybara.rb
      create  db/schema.sql
      create  .gitignore
         run  git init . from "."
      create  apps/api/application.rb
      create  apps/api/config/routes.rb
      create  apps/api/views/application_layout.rb
      create  apps/api/templates/application.html.erb
      create  apps/api/assets/favicon.ico
      create  apps/api/controllers/.gitkeep
      create  apps/api/assets/images/.gitkeep
      create  apps/api/assets/javascripts/.gitkeep
      create  apps/api/assets/stylesheets/.gitkeep
      create  spec/api/features/.gitkeep
      create  spec/api/controllers/.gitkeep
      create  spec/api/views/.gitkeep
      insert  config/environment.rb
      insert  config/environment.rb
      append  .env.development
      append  .env.test
```

To generate the dashboard app, ran:

```shell
$ hanami generate app dashboard
      create  apps/dashboard/application.rb
      create  apps/dashboard/config/routes.rb
      create  apps/dashboard/views/application_layout.rb
      create  apps/dashboard/templates/application.html.erb
      create  apps/dashboard/assets/favicon.ico
      create  apps/dashboard/controllers/.gitkeep
      create  apps/dashboard/assets/images/.gitkeep
      create  apps/dashboard/assets/javascripts/.gitkeep
      create  apps/dashboard/assets/stylesheets/.gitkeep
      create  spec/dashboard/features/.gitkeep
      create  spec/dashboard/controllers/.gitkeep
      create  spec/dashboard/views/.gitkeep
      insert  config/environment.rb
      insert  config/environment.rb
      append  .env.development
      append  .env.test
```

And for the admin app:

```shell
$ hanami generate app admin
      create  apps/admin/application.rb
      create  apps/admin/config/routes.rb
      create  apps/admin/views/application_layout.rb
      create  apps/admin/templates/application.html.erb
      create  apps/admin/assets/favicon.ico
      create  apps/admin/controllers/.gitkeep
      create  apps/admin/assets/images/.gitkeep
      create  apps/admin/assets/javascripts/.gitkeep
      create  apps/admin/assets/stylesheets/.gitkeep
      create  spec/admin/features/.gitkeep
      create  spec/admin/controllers/.gitkeep
      create  spec/admin/views/.gitkeep
      insert  config/environment.rb
      insert  config/environment.rb
      append  .env.development
      append  .env.test
```

At this point, the Hanami container is created with the three applications, as follows:

```shell
$ tree -L 2 -F
.
├── Gemfile
├── Rakefile
├── apps/
│   ├── admin/
│   ├── api/
│   └── dashboard/
├── config/
│   ├── environment.rb
│   └── initializers/
├── config.ru
├── db/
│   ├── migrations/
│   └── schema.sql
├── lib/
│   ├── bookshelf/
│   ├── bookshelf.rb
│   └── config/
├── public/
└── spec/
    ├── admin/
    ├── api/
    ├── bookshelf/
    ├── dashboard/
    ├── features_helper.rb
    ├── spec_helper.rb
    └── support/
```

## Mounting

As previously said, I want to mount each app separately on production. To do this, I changed the `config/environment.rb` to following:

```ruby
require 'rubygems'
require 'bundler/setup'
require 'hanami/setup'
require_relative '../lib/bookshelf'

Hanami::Container.configure do
  if ENV["APP"] == "api"
    require_relative '../apps/api/application'
    mount Api::Application, at: '/'

  elsif ENV["APP"] == "dashboard"
    require_relative '../apps/dashboard/application'
    mount Dashboard::Application, at: '/'

  elsif ENV["APP"] == "admin"
    require_relative '../apps/admin/application'
    mount Admin::Application, at: '/'

  else
    require_relative '../apps/api/application'
    require_relative '../apps/dashboard/application'
    require_relative '../apps/admin/application'
    mount Api::Application, at: '/api'
    mount Dashboard::Application, at: '/dashboard'
    mount Admin::Application, at: '/admin'
  end
end
```

Just need to set the `APP` env and the Hanami container will be configured like my desire. Else, all apps will be mounted as regular way (for test and development environment).

## Database

While developing the POC, I was very frustrated with the [Hanami Model](https://github.com/hanami/model) because it's lack of associations and it's declarative mapping model. TL;DR I decided to use [Sequel](https://github.com/jeremyevans/sequel/) directly.

First, I removed all "hanami model" configuration on `lib/bookshelf.rb`. The file contains only:

```ruby
Dir["#{ __dir__ }/config/**/*.rb"].each { |file| require_relative file }
Dir["#{ __dir__ }/bookshelf/**/*.rb"].each { |file| require_relative file }
```

And configured the Sequel connection on `lib/config/database.rb`:

```ruby
require 'sequel'

DB = Sequel.connect(
  ENV["BOOKSHELF_DATABASE_URL"],
  max_connections: 10,
  logger: Logger.new("log/sequel.log")
)
```

At this point, I was able to boot the application and access the database using `hanami console`:

```ruby
irb(main):001:0> DB["select * from users"]
=> #<Sequel::Postgres::Dataset: "select * from users">
```

## Models

Since my goal is to connect to a previous migrated database schema from Rails, the migrations are not concern for now. There will be shown on next post.

Right now, just need to declare the models and associations using [Sequel Model](https://sequel.jeremyevans.net/rdoc/files/doc/association_basics_rdoc.html).

By example, consider the `Company` and `User` models, with the following database schema:

```sql
CREATE TABLE companies (
    id integer NOT NULL,
    name character varying(100) NOT NULL
);
CREATE TABLE users (
    id integer NOT NULL,
    company_id integer NOT NULL,
    name character varying(100) NOT NULL,
    email character varying(255) NOT NULL
);
```

Just need to create the model files, being `lib/bookshelf/entities/company.rb`:

```ruby
class Company < Sequel::Model
  one_to_many :users
end
```

And the `lib/bookshelf/entities/user.rb` with the content:

```ruby
class User < Sequel::Model
  many_to_one :company
end
```

And to validate, with a previous seed data, ran `hanami console`:

```ruby
irb(main):001:0> company = Company.first
=> #<Company @values={:id=>1, :name=>"Acme Inc."}>

irb(main):002:0> company.users
=> [#<User @values={:id=>1, :name=>"User from Acme Inc.", :email=>"address@server.com", :company_id=>1}>]
```

At this point, the application are configured to read from Postgres with the previous migrated database schema. I just need to copy the models from Rails project and make the necessary changes to Sequel.

For example, the `app/models/company.rb`:

```ruby
class Company < ActiveRecord::Base
  belongs_to :account
  has_many   :users
end
```

Becomes `lib/bookshelf/entities/company.rb`:

```ruby
class Company < Sequel::Model
  many_to_one :account
  one_to_many :users
end
```

## Views/Templates

This was the most easily migrated feature. Just copied and pasted the controllers and ERB views from Rails application to Hanami structure and made the [necessary changes](https://hanamirb.org/guides/views/templates/), like move the controller actions from methods to action classes, template structures, partials and everything worked like a charm.

For example, the Rails controller `app/controllers/admin/users_controller.rb`:

```ruby
class Admin::UsersController < ActionController::Base
  def index
    @users = User.all
  end
  def show
    @user = User.find_by(id: params[:id])
  end
end
```

Becoming the Hanami action `apps/admin/controllers/users/index.rb`:

```ruby
module Admin::Controllers::Users
  class Index
    include Admin::Action
    expose :users
    def call(params)
      @users = User.all
    end
  end
end
```

And `apps/admin/controllers/users/show.rb`:

```ruby
module Admin::Controllers::Users
  class Show
    include Admin::Action
    expose :user
    def call(params)
      @user = User.find(id: params[:id])
    end
  end
end
```

Fortunately we was on first release and the web features was very simple (don't have any form or validation, for example). No problems here, but the next topic was a pain in the ass!

## Assets

The **assets pipeline is a complexity and/or an overhead** that I decided to not deal for now (and probably never will on this project). As it was on previous Rails application, assets are plain (javascripts are plain js and stylesheets are plain css), centralized on directory `public/assets` separated by application name, like:

```shell
/public/assets/admin/javascripts
/public/assets/admin/stylesheets
/public/assets/dashboard/javascripts
/public/assets/dashboard/stylesheets
```

And I decided to serve them with `Rack::Static`, declaring the middleware on `config.ru` file:

```ruby
use Rack::Static, root: "public", urls: %w(/assets)
```

On templates, just reference the `href` starting from `/assets`, like:

```html
<link rel="stylesheet" href="/assets/admin/stylesheets/application.css" />
```

Thats it. Quick and dirty static assets serving.

## Wrapping up

Our goal was to migrate from Rails to Hanami in two steps: first replacing the web apps that are basically read-only and keep the API with Rails. And after, migrate all write operations, which is basically the entire API, but to do that, we need that the test suite be migrated too. This will be shown on next post.

For now, we was able to deploy the new Hanami applications (dashboard and admin), with read-operations directly to Postgres using Sequel/Model, serving assets with Rack::Static and the write-operations, like create transactions, calculations and async operations, delegated to the Rails API over HTTP client.

**And was successfully made!** The first step was concluded with the two Hanami apps deployed on production **running on its own mounted "app"**. The **memory comsumption was between 80MB and 100MB per process**, and the response time between 40ms and 200ms, depending of operation.

Yes, it's a win \o/

## Next steps

On next post, I will detail how was solved the topics:

* Sequel plugins
* Timezone issues
* Test environment / RSpec
* DB Fixtures
* Ruby core extensions
* Sidekiq Workers
* I18n
* And the great Hanami migration (API)

If you had any question, do not hesitate to comment below. Code hard and success!

