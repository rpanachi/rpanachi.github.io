---
layout: post
title: What 6 months of Laravel taught me as a Rails developer
date: 2026-01-05 00:00:00
categories: [php, laravel, ruby, rails, career, thoughts]
excerpt: Different languages, surprisingly similar developer experience
disqus: true
archive: true
---

> “Good programmers use their brains, but good guidelines save us having to think out every case.”
> — Francis Glassborow

After many years working with **Ruby on Rails**, I’ve spent the last **six months working full time with PHP and Laravel**.
What surprised me the most was not learning a new syntax, but **how little friction I felt switching stacks**.

Not because the languages are the same (they aren't) but because **Rails and Laravel share a very similar philosophy**: convention over configuration, developer happiness, and code that reads like intention.

Below are the main similarities and differences I noticed from a very practical, day-to-day perspective.

## Eloquent Models feel like ActiveRecord (and that’s a good thing)

If you’ve used Rails’ ActiveRecord for years, Eloquent will feel instantly familiar.

### Model definition

**Rails (ActiveRecord):**

```ruby
class Post < ApplicationRecord
  belongs_to :user
  has_many :comments

  scope :published, -> { where(published: true) }
end
````

**Laravel (Eloquent):**

```php
class Post extends Model
{
    protected $fillable = ['title', 'body', 'published'];

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function comments()
    {
        return $this->hasMany(Comment::class);
    }

    public function scopePublished($query)
    {
        return $query->where('published', true);
    }
}
```

### Querying feels almost identical

This is where Rails muscle memory really kicks in.

**Rails:**

```ruby
Post.published
    .where(user_id: 1)
    .order(created_at: :desc)
    .limit(10)
```

**Laravel:**

```php
Post::published()
    ->where('user_id', 1)
    ->orderBy('created_at', 'desc')
    ->limit(10)
    ->get();
```

Chaining, scopes, lazy evaluation - the flow and intent are basically the same.

## Collections API: Ruby vibes everywhere

Laravel’s `Collection` API is one of its strongest features — and clearly inspired by Ruby’s Enumerable.

**Ruby:**

```ruby
users
  .select { |u| u.active? }
  .map(&:email)
  .sort
```

**Laravel:**

```php
$users
    ->filter(fn ($u) => $u->active)
    ->map(fn ($u) => $u->email)
    ->sort()
    ->values();
```

Once you stop thinking in raw arrays and start thinking in **collections**, PHP becomes much more expressive.

## Hash vs Array: different names, same idea

This is one of the first mental adjustments when moving from Ruby to PHP.

In Ruby, **Hash** is the natural structure for key-value data.
In PHP, **arrays do everything** - lists, maps, hashes - which feels odd at first but works well in practice.

### Defining key-value data

**Ruby (Hash):**

```ruby
user = {
  name: "User",
  email: "user@example.com",
  active: true
}
```

**PHP (array as hash):**

```php
$user = [
    'name' => 'User',
    'email' => 'user@example.com',
    'active' => true,
];
```

### Accessing elements

**Ruby:**

```ruby
user[:email]
user.fetch(:email)
```

**PHP:**

```php
$user['email'];
$user['email'] ?? null;
```

The null-coalescing operator makes safe access very straightforward.

### Adding or updating values

**Ruby:**

```ruby
user[:role] = "admin"
user.merge!(last_login: Time.now)
```

**PHP:**

```php
$user['role'] = 'admin';

$user = array_merge($user, [
    'last_login' => now(),
]);
```

With Laravel Collections:

```php
$user = collect($user)
    ->merge(['last_login' => now()])
    ->toArray();
```

### Transforming data

**Ruby:**

```ruby
prices = { apple: 10, banana: 5 }
prices.transform_values { |v| v * 2 }
```

**PHP (Collection):**

```php
$prices = collect(['apple' => 10, 'banana' => 5])
    ->map(fn ($v) => $v * 2)
    ->toArray();
```

### Nested access

**Ruby:**

```ruby
user.dig(:profile, :address, :city)
```

**PHP (Laravel helper):**

```php
data_get($user, 'profile.address.city');
```

Different syntax, same intent.

## Request validation is as simple as Rails strong params

Validation in Laravel feels very close to Rails - simple, readable, and explicit.

**Rails:**

```ruby
params.require(:user).permit(:name, :email)
validates :email, presence: true, uniqueness: true
```

**Laravel:**

```php
$request->validate([
    'name'  => 'required|string',
    'email' => 'required|email|unique:users',
]);
```

In both frameworks, validation lives close to where data enters the system.

## Routes: familiar DSL, different syntax

Routing is another area where the philosophies strongly align.

**Rails:**

```ruby
Rails.application.routes.draw do
  resources :posts
  get "/health", to: "health#check"
end
```

**Laravel:**

```php
Route::resource('posts', PostController::class);
Route::get('/health', [HealthController::class, 'check']);
```

RESTful by default, expressive, and easy to reason about.

## Artisan feels like Rails generators on steroids

If you like `rails generate`, you’ll feel at home with Artisan.

**Rails:**

```bash
rails generate model Post title:string body:text
rails routes
```

**Laravel:**

```bash
php artisan make:model Post -m
php artisan make:controller PostController --resource
php artisan route:list
```

Artisan goes beyond generators:

* queue workers
* cache management
* app optimization
* custom CLI commands

It’s a very strong part of the Laravel ecosystem.

## PHP is simple, explicit, and predictable

Modern PHP is very different from what many of us remember.

### Less magic, more clarity

**Ruby:**

```ruby
user.find_by_email("test@test.com")
```

**PHP:**

```php
User::where('email', 'test@test.com')->first();
```

No hidden meta-programming - what you see is what runs.

### Interfaces and typing help a lot

```php
interface PaymentGateway
{
    public function charge(int $amount): bool;
}
```

Clear contracts reduce ambiguity, especially in larger teams.

## The not-so-great parts

No stack is perfect.

* Laravel caching can be annoying and inconsistent across environments
* Finding libraries for newer or niche services can be harder than in Ruby
* PHP still requires more boilerplate in some areas
* The ecosystem is more fragmented (versions, extensions, hosting)

None of these are deal breakers - just trade-offs.

## Final thoughts

After six months working daily with Laravel, one thing is very clear to me:

**PHP is not dead. Not even close.**

Laravel delivers:

* a modern developer experience
* a familiar MVC philosophy
* excellent productivity
* and a surprisingly smooth transition for Rails developers

Rails still shines in elegance and ecosystem maturity, but Laravel proves that **PHP has evolved - a lot - and in the right direction**.

At the end of the day, good software is less about the language and more about:

* clear code
* good tests
* solid architecture
* and teams that know what they’re doing

And in that sense, **both Rails and Laravel get the job done - really well**.
