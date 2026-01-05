---
layout: post
title: Useful Ruby methods and tips that you might not know (or remember)
date: 2025-05-29 00:00:00
categories: [ruby, ruby on rails]
excerpt: Ruby tricks you forgot you knew (but totally should remember)
disqus: true
archive: true
---

> "Code is like humor. When you have to explain it, it’s bad."<br/>
> — Cory House

## Enumerable Methods

### Enumerable#tally

The tally method groups and counts elements in an enumerable. It's great for frequency analysis.

```ruby
irb> ["apple", "banana", "apple", "orange"].tally
=> {"apple"=>2, "banana"=>1, "orange"=>1}
```

### Enumerable#inject

`inject` accumulates a result by applying a block to each element. It's useful for summing, multiplying, and more.

Using with an accumulator:

```ruby
irb> [1, 2, 3, 4].inject(0) { |sum, num| sum + num }
=> 10
```

Or passing a method proc as argument:


```ruby
irb> [1, 2, 3, 4].inject(:+)
=> 10
```

### Enumerable#chunk

`chunk` groups consecutive elements based on a block condition.

```ruby
irb> [1, 2, 2, 3, 1, 1].chunk(&:even?).to_a
=> [[false, [1]], [true, [2, 2]], [false, [3]], [true, [1, 1]]]
```

### Enumerable#each_with_object

`each_with_object` iterates and builds an object like a hash or array.

```
irb> ["apple", "banana"].each_with_object({}) { |fruit, hash| hash[fruit] = fruit.length }
=> {"apple"=>5, "banana"=>6}
```

### Enumerable#slice_when

`slice_when` splits an enumerable when the block returns true.

```ruby
irb> [1, 3, 6, 10].slice_when { |a, b| (b - a) > 3 }.to_a
=> [[1, 3, 6], [10]]
```

### Enumerable#flat_map

`flat_map` maps and flattens the result in one step.

```ruby
irb> [[1, 2], [3, 4]].flat_map { |arr| arr.map { |x| x * 2 } }
=> [2, 4, 6, 8]
```

## Hash Methods

### Hash#dig

Extracts the nested value specified by the sequence of key objects.

```ruby
irb> h = { foo: {bar: {baz: 1}}}
irb> h.dig(:foo, :bar, :baz)
=> 1
irb> h.dig(:invalid, :key)
=> nil
```

### Hash#invert

`invert` swaps keys and values in a hash.

```ruby
irb> { a: 1, b: 2 }.invert
=> {1=>:a, 2=>:b}
```

### Hash#transform_keys

`transform_keys` lets you change hash keys with a block.

```ruby
irb> { a: 1, b: 2 }.transform_keys { |key| key.to_s.upcase }
=> {"A"=>1, "B"=>2}
```

### Hash#transform_values

`transform_values` creates a new hash with the values transformed by the given block, while keeping the keys unchanged.

```ruby
irb> { a: 1, b: 2 }.transform_values { |value| value * 10 }
=> { a: 10, b: 20 }
```

## Array Methods

### Array#rotate

`rotate` shifts elements to the left by default, or by a given count.
```ruby
irb> [1, 2, 3, 4].rotate
=> [2, 3, 4, 1]
```

### Array#combination

`combination` generates all combinations of a given length.

```ruby
irb> [1, 2, 3].combination(2).to_a
=> [[1, 2], [1, 3], [2, 3]]
```

### Array#permutation

`permutation` returns all possible permutations of a given length.

```ruby
irb> [1, 2, 3].permutation(2).to_a
=> [[1, 2], [1, 3], [2, 1], [2, 3], [3, 1], [3, 2]]
```

## Object Methods

### Object#method(:method_name).source_location

Find where a [method is defined](https://apidock.com/ruby/Method/source_location) in your source code.

```ruby
irb> g = Greeter.new
irb> g.method(:say_it).source_location
=> ["/Users/home/app/models/greeter.rb", 5] 
```

### Object#method(:method_name).arity

Check the [number of arguments](https://apidock.com/ruby/Method/arity) a method accepts.

```ruby
irb> foo = "Hello"
irb> foo.method(:slice).arity
=> -2
```

### Object#methods(false)

List only [methods defined directly on the object](https://apidock.com/ruby/Object/methods), excluding inherited methods.

```ruby
irb> obj = Object.new
def obj.greet; "Hello!"; end
irb> obj.methods(false)
=> [:greet]
```

## Kernel Methods

### callee

__callee__ returns the current method’s name.

```ruby
irb> def example
irb>   __callee__
end
irb> example
=> :example
```

### rescue/retry

`retry` restarts the block after a `rescue` clause.

```ruby
irb> attempts = 0
irb> begin
irb>   raise "Error" if (attempts += 1) < 3
irb> rescue
irb>   retry
irb> end
=> nil
```

### BEGIN / END

`BEGIN` and `END` define blocks executed before and after the script runs.

```ruby
BEGIN { puts "Starting..." }
END { puts "Ending..." }
puts "Hello!"
```

## ActiveSupport Methods

### Enumerable#compact_blank

`compact_blank` removes blank elements from an enumerable.

```ruby
irb> [nil, "", "Hello", [], {}].compact_blank
=> ["Hello"]
```

### Array#index_by

`index_by` creates a hash where keys are determined by the block.

```ruby
irb> ["apple", "banana", "pear"].index_by { |fruit| fruit[0] }
=> {"a"=>"apple", "b"=>"banana", "p"=>"pear"}
```

## Miscellaneous

### Right Assignment

Assign results in reverse order for cleaner syntax.

```ruby
irb> Users.where(active: true).all => users
=> nil
irb> users
=> [#<Contact:>,...]
```
### Destructuring Block Arguments

Pattern matching in blocks lets you destructure array elements.

```ruby
irb> a = [[1, 2], [3, 4]]
irb> a.each do |(first, last)|
irb>   puts "First: #{first}, Last: #{last}"
irb> end
First: 1, Last: 2
First: 3, Last: 4
```

### Calling Procs

You could call procs in different ways:

```ruby
irb> my_proc = -> arg { puts arg }
#<Proc:0x007fb1ebe9c1a0@(irb):1 (lambda)>

irb> my_proc.call('hello')
hello

irb> my_proc.('hello')
hello

irb> my_proc['hello']
hello
```

## Command line Ruby tips

Given the following files in the current directory:

```shell
$ ls -l
-rw-r--r--@ 1 user  group    36 28 Mai 17:57 file1.txt
-rw-r--r--@ 1 user  group     8 28 Mai 17:57 file2.txt
-rw-r--r--@ 1 user  group   613 28 Mai 17:58 file3.txt
-rw-r--r--@ 1 user  group  3143 28 Mai 18:48 photo.jpg
-rw-r--r--@ 1 user  group  1004 28 Mai 18:47 report.ofx
```

### Using -n and -e to Process Lines

`-n` wraps your code in a loop that reads each line from input. Combine with `-e` for quick one-liners.

```shell
$ ls -1 | ruby -n -e 'puts $_ if $_.match(/\.txt/'
file1.txt
file2.txt
file3.txt
```

### Using -p to Modify Lines in Place

`-p` is like `-n` but automatically prints each line. Great for filters and transformations.

```shell
$ ls -1 | ruby -p -e '$_ = $_.capitalize'
File1.txt
File2.txt
File3.txt
Photo.jpg
Report.ofx
```

### Using STDIN

The command below returns the total size of all files returned by `ls -l` command:

```shell
$ ls -l | ruby -e 'puts STDIN.to_a.map { |l| l.split[4].to_i }.sum'
4804
```
