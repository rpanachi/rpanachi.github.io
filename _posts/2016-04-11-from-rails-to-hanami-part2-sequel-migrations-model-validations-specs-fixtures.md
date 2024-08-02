---
layout: post
title:  "From Rails to Hanami (Lotus) Part 2: Sequel Migrations, Model Validations, Specs and Fixtures"
date:   2016-04-11 00:00:00
categories: ruby hanami rails sequel rspec tutorial
excerpt: "The 'real world problems' driven guide to use Hanami on production"
disqus: true
archive: false
redirect_from: /2016/04/11/from-rails-to-hanami-part2-sequel-migrations-model-validations-specs-fixtures/
---

> Do or do not. There is no try.<br/>
> ―  Master Yoda

## Recap

In previous post [From Rails to Hanami (Lotus) Part 1: Container Architecture, Models, Views and Assets](/2016/03/28/from-rails-to-hanami-part1-container-architecture-model-views-assets), I show how was the migration of two Rails applications to a single Hanami container, mounting each application individually, reading from Postgres database and doing all write operation throught the Rails "core" application via REST api.

Now it's time to **migrate the API business logic** (models, validations, services, etc) from the Rails application to Hanami. And to ensure that everything works as expected, the specs need to be migrated first. And for the specs run, the database migrations need to run too. So, let the fun begin!

## Migrations

[Sequel migrations](https://sequel.jeremyevans.net/rdoc/files/doc/migration_rdoc.html) is very similar to [Hanami migrations](https://hanamirb.org/guides/migrations/overview/) (almost the same, I guess). To create a migration, I manually created the file `db/migrations/20160327165432_create_users.rb` with the following:

```ruby
Sequel.migration do
  change do
    create_table :users do
      primary_key :id
      column :email, String, size: 255, null: false, unique: true
      column :name,  String, size: 255, null: false
    end
  end
end
```

And to run the migration, I used the Sequel command line passing the connection string and the directory which contains the migrations as argument to option `-m`. Also supplied the `-E` option to print all SQL output:

```bash
$ sequel postgres://localhost/bookshelf_development -m ./db/migrations -E
(0.000288s) SET standard_conforming_strings = ON
(0.000138s) SET client_min_messages = 'WARNING'
(0.000182s) SET DateStyle = 'ISO'
(0.000769s) SELECT NULL AS "nil" FROM "schema_migrations" LIMIT 1
(0.000221s) SELECT * FROM "schema_migrations" LIMIT 1
(0.000515s) SELECT "filename" FROM "schema_migrations" ORDER BY "filename"
Begin applying migration 20160327165432_create_users.rb, direction: up
(0.000120s) BEGIN
(0.004716s) CREATE TABLE "users" ("id" serial PRIMARY KEY, "email" varchar(255) NOT NULL UNIQUE, "name" varchar(255) NOT NULL)
(0.001198s) SELECT pg_attribute.attname AS pk FROM pg_class, pg_attribute, pg_index, pg_namespace WHERE pg_class.oid = pg_attribute.attrelid AND pg_class.relnamespace = pg_namespace.oid AND pg_class.oid = pg_index.indrelid AND pg_index.indkey[0] = pg_attribute.attnum AND pg_index.indisprimary = 't' AND pg_class.oid = CAST(CAST('"schema_migrations"' AS regclass) AS oid)
(0.000238s) INSERT INTO "schema_migrations" ("filename") VALUES ('20160327165432_create_users.rb') RETURNING NULL
(0.000478s) COMMIT
Finished applying migration 20160327165432_create_users.rb, direction: up, took 0.007772 seconds
```

**Note**: Sequel use the column `filename` on table `schema_migrations` to control the schema version. If it don't exists, this error occours:

```bash
$ sequel postgres://localhost/bookshelf_development -m ./db/migrations
Error: Sequel::Migrator::Error: Migrator table schema_migrations does not contain column filename
```

As the idea here is run the migrations on the schema previouslly migrated by ActiveRecord, I need to add this column manually before run the Sequel migrations:

```
$ psql -dbookshelf_development
bookshelf_development=# alter table schema_migrations add column filename varchar not null;
ALTER TABLE
```

This also will be needed on first release to production, since the schema should be prepared to run the Sequel migrations.

In order to help with these database operations, I created a Rakefile with the common operations like drop, create, migrate, rollback, version, etc:

```shell
$ rake -T db
rake db:console              # Start a database console on environment
rake db:create               # Creates database
rake db:drop                 # Drops database
rake db:migrate[version]     # Perform migration up to latest migration available
rake db:reset                # Perform migration reset (full rollback and migration) only on local environment
rake db:rollback[version]    # Perform rollback to specified target or previous version as default
rake db:seed                 # Seed the database with application required data
rake db:structure:dump       # Dump database structure to db/schema.sql
rake db:structure:load       # Load db/schema.sql database structure
rake db:structure:to_rails   # Setup database structure from Sequel to Rails Migrations
rake db:structure:to_sequel  # Setup database structure from Rails to Sequel Migrations
rake db:version              # Prints current schema version
```

The entire [Rakefile is available on this Gist](https://gist.github.com/rpanachi/f16447f8b03fa5edcf935863fcdffb95). Just need to load the task and manage the database with rake commands (like we did with Rails 2).

Now, with the migrations setting up to run, I just need to copy all migration files from Rails project and made the necessary changes to Sequel migration.

For example, the `db/migrate/20150210151518_create_users.rb` ActiveRecord migration:

```ruby
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.references :account, index: true
      t.string     :email,   limit: 255, null: false
      t.string     :name,    limit: 255, null: false
      t.string     :role,    null: false, default: 'owner'
      t.timestamps null: false
    end
    add_foreign_key :users, :accounts
    add_index :users, :email, unique: true
  end
end
```

Becomes `db/migrations/20150210151518_create_users.rb` Sequel migration:

```ruby
Sequel.migration do
  change do
    create_table :users do |t|
      primary_key :id
      foreign_key :account_id, :accounts, null: false

      column :email,       String, size: 255, null: false, unique: true
      column :name,        String, size: 255, null: false
      column :role,        String, null: false, default: 'owner'
      column :created_at,  DateTime, null: false
      column :updated_at,  DateTime, null: false
    end
  end
end
```

Made this, I was able to migrate the development and test databases simply running `rake db:migrate` and `HANAMI_ENV=test rake db:migrate` respectively.

## Models

The models was previously copied from ActiveRecord model with the necessary change to work with Sequel. As it was used only to read from database, the validations and write methods was entirely removed in this process.

Now, I was manually adding the [validations](https://sequel.jeremyevans.net/rdoc/files/doc/validations_rdoc.html) on each model and doing the necessary changes.

For example, the `app/models/account.rb`:

```ruby
class Account < ActiveRecord::Base
  has_many   :users

  validates :document, presence: true, uniqueness: true
  validates :name,     presence: true
  validates :guid,     presence: true
  validates :kind,     inclusion: {in: %w(business personal)}
  validate  :validate_document

  before_validation on: :create do
    self.guid ||= SecureRandom.uuid
  end

  def validate_document
    errors.add(:document, 'is invalid') unless document_valid?
  end
end
```

Becomes `lib/bookshelf/entities/account.rb`:

```ruby
class Account < Sequel::Model
  plugin :validation_helpers

  one_to_many :users

  def validate
    super
    validates_presence [:name, :document, :guid]
    validates_unique   :document
    validates_includes %w(business personal), :kind
    validate_document
  end

  def before_validation
    super
    self.guid ||= SecureRandom.uuid
  end

  def validate_document
    errors.add(:document, 'is invalid') unless document_valid?
  end
end
```

Note: I am using the Sequel `validation_helpers` plugin here, that will be detailed ahead.

## Specs

Considering that at this point the database schema is not a concern, I started to work on specs migration.

The `spec_helper.rb` was correctly generated by Hanami and looks like this:

```ruby
ENV['HANAMI_ENV'] ||= 'test'

require_relative '../config/environment'
Hanami::Application.preload!

Dir[__dir__ + '/support/**/*.rb'].each { |f| require f }
```

And runnig the `rspec` command, everything seems to work:

```shell
$ rspec
No examples found.

Finished in 0.00036 seconds (files took 1.15 seconds to load)
0 examples, 0 failures
```

Now it's time to copy all specs from the models and lib of Rails project and made the necessary changes.

For example, the `spec/models/account_spec.rb`:

```ruby
require 'rails_helper'

RSpec.describe Account, type: :model do
  context "validations" do
    it { is_expected.to validate_presence_of   :name }
    it { is_expected.to validate_presence_of   :document }
    it { is_expected.to validate_uniqueness_of :document }
    it { is_expected.to validate_inclusion_of  :kind, in: %w(business personal) }

    context "document" do
      it "should be validaded" do
        account = described_class.new(document: '12312312312')
        account.validate

        expect(account.errors[:document]).to_not be_empty
      end
    end
  end
end
```

Becomes `spec/bookshelf/entities/account_spec.rb`:

```ruby
require 'spec_helper'

RSpec.describe Account, type: :model do
  context "validations" do
    it { is_expected.to validate_presence :name }
    it { is_expected.to validate_presence :document }
    it { is_expected.to validate_unique   :document }
    it { is_expected.to validate_inclusion :kind, %w(business personal) }

    context "document" do
      it "should be validaded" do
        account = described_class.new(document: '12312312312')
        account.validate

        expect(account.errors.on(:document)).to_not be_nil
      end
    end
  end
```

These Rspec validation macros comes from `rspec_sequel_matchers` gem. Just added to `Gemfile` and worked like a charm.

At this point and after made all necessary changes, I was able to run the specs for models and lib classes:

```shell
$ rspec spec/bookshelf
...........................................................................................................

Finished in 50.27 seconds (files took 3.37 seconds to load)
201 examples, 0 failures, 0 pending
```

## Feature specs

The models and lib classes are covered at this point. Now it's time to migrate the specs for the main API, which deals practically with all business logic operations in the application.

In the previous project, I was already using `Rack::Test`. So, just added the gem `rack-test` to `Gemfile` and changed the `spec/features_helper.rb` to:

```ruby
require_relative './spec_helper'

RSpec.configure do |config|
  config.include RSpec::RackApplication, type: :feature
end
```

And added the `spec/support/rack_application.rb` file with the content:

```ruby
require 'rack/test'

module RSpec
  module RackApplication
    include Rack::Test::Methods

    private

    def app
      @@app ||= Hanami::Container.new
    end
  end
end
```

Now, I was able to define a feature spec like:

```ruby
require "features_helper"

describe "POST /accounts", type: :feature do
  let(:payload) do
    File.read("spec/samples/create_account_valid.json")
  end
  context "with valid data" do
    it "should create account" do
      post "/api/accounts", payload

      json_body = JSON.load(last_response.body)
      expect(json_body["id"]).to_not be_nil
    end
  end
  context "with duplicated data" do
    before(:each) do
      Account.create(name: payload["name"], payload["document"])
    end
    it "should not create account" do
      post "/api/accounts", payload

      json_body = JSON.load(last_response.body)
      expect(json_body["message"]).to eq("Duplicated document")
    end
  end
end
```

Almost the same spec defined on Rails application. Just copied all specs files from previous project and made the necessary changes, which was basically the `require` on first line.

## Fixtures replacement

Now the specs are almost migrated, except for one detail: fixtures. The previous project used a lot of ActiveRecord fixtures for build test scenarios. If I can solve this dependency, all specs will pass.

The best replacement for AC fixtures was the [fixture_dependencies](https://github.com/jeremyevans/fixture_dependencies) gem. Just added to `Gemfile` and configured it on `spec/support/fixtures.rb` file:

```ruby
require 'fixture_dependencies/rspec/sequel'

FixtureDependencies.fixture_path = Hanami.root.join('spec/fixtures')

RSpec::Core::ExampleGroup.class_eval do
  alias :fixtures :load
end
```

And just needed to copy all fixtures from Rails project to `spec/fixtures`. On files using some ERB code, just appended a `.erb` on filename. That's it!

Here some usage example. Given the file `spec/fixtures/account.yml.erb`:

```yaml
acme:
  name: "Acme Inc"
  document: "12345678"
  guid: "<%= SecureRandom.uuid %>"
```

And the `spec/fixtures/users.yml`:

```yaml
owner:
  name: "The Owner"
  email: "owner@acme.com"
  account: acme

employee:
  name: "The Worker"
  email: "employee@acme.com"
  account: acme
```

To load all users on a spec, just call:

```ruby
  let(:users) do
    fixtures(:users)
  end
```

Or to load a specific `User`, just need to call:

```ruby
  let(:employee) do
    fixtures(:user__employee)
  end
```

If the fixture naming was correcly, all associations will be resolved by `fixture_dependencies` in runtime (works more like a factory).

And to ensure that all fixtures are correctly defined, I added a spec just for it on `spec/bookshelf/fixtures_spec.rb`:

```ruby
require "spec_helper"

describe "fixtures" do
  Dir[Hanami.root.join("spec/fixtures/*.{yml,erb}")].each do |fixture_file|
    fixture_name = File.basename(fixture_file.split(".").first)
    it "#{fixture_name} should be successful loaded" do
      fixtures(fixture_name)
    end
  end
end
```

Now, just need to change all occurrences of `fixtures :all` (that is also a code smell) and change to the correct fixture used in the spec. And now, all my specs are running successfully.

## Wrapping up

With the specs running, we was able to migrate all write related code from Rails project. And once all specs are green, the second step of this migration to Hanami was concluded. For now, only the Sidekiq workers still on Rails project (for a little while).

To validate if this Hanami application behave correctly, we deployed it as a mirror of the running Rails application on a separated database. All requests are replicated from Rails to Hanami and we was comparing and checking it for several days, until that it was migrated "for real".

And was another successful! \o/

On next post, I will detail the final topics and answer some questions about the migration process:

* Timezone issues
* Ruby core extensions
* Sidekiq Workers
* I18n
* FAQ

Consider to share this post with your friends or co-workers. For questions, comments or suggestions, use the comments below. Code hard and success!
