---
layout: post
title:  "Ruby: recursion, stack size and tail call optimization"
date:   2016-05-30 12:00:00
categories: ruby
excerpt: "To understand recursion, you must understand recursion"
disqus: true
archive: false
redirect_from:
  - /2016/05/30/ruby-recursion-stack-size-tail-call-optimization/
  - /blog/ruby-recursion-stack-size-tail-call-optimization/
---

> To understand recursion, you must understand recursion.<br/>
> ―  Author Unknown

## TL;DR

Recursion is a tricky programming technique. It could be very useful or very harmful, depending of its use.

The default Ruby VM (MRI) has a heap size limit to handle recursions. This may cause the catastrophic error `SystemStackError` for big recursion calls.

There are two ways to avoid this: increase the stack size limit (without change the code) or enable tail call optimization (changing the code).

## Stack level too deep

The most famous example of recursion usage is the factorial implementation:

```ruby
def factorial(n)
  raise InvalidArgument, "negative input given" if n < 0

  return 1 if n == 0
  return factorial(n - 1) * n
end
```

Works like a charm for small numbers:

```ruby
irb> factorial(1_000).to_s.size
=> 2568
```

Yes, the factorial of `1000` returns a number of `2568` digits! But, let's push the limits a little:

```ruby
irb> factorial(100_000).to_s.size
SystemStackError: stack level too deep
```

Fail! Take that `SystemStackError: stack level too deep` on your face :(

## Increasing Stack Size

A factorial of `100000` is sufficient to cause a `SystemStackError`. The easiest way to fix this is increasing the Stack Size of Ruby VM, setting the `RUBY_THREAD_VM_STACK_SIZE` environment variable:

```shell
export RUBY_THREAD_VM_STACK_SIZE=50000000
```

The stack size was setted to 50MB. And now, running the `factorial` again:

```ruby
irb> factorial(100_000).to_s.size
=> 456574
```

Works like a charm. But let's try something different...

## Tail call optimization

The [tail call optimization](https://en.wikipedia.org/wiki/Tail_call) (TCO) is a programming technique available in many languages where the compiler transform a recursive function into loops.

Let's refactor the `factorial` method to be tail call optimizable:

```ruby
def factorial(n, accumulator = 1)
  raise InvalidArgument, "negative input given" if n < 0

  return accumulator if n == 0
  return factorial(n - 1, accumulator * n)
end
```

But TCO is not enabled by default on RubyVM. This should be made on runtime:

```ruby
RubyVM::InstructionSequence.compile_option = {
  tailcall_optimization: true,
  trace_instruction: false
}
```

And now, running the `factorial` again:

```ruby
irb> factorial(100_000).to_s.size
=> 456574
```

Works... but do you really need to activate TCO to use recursion with safety?

## Recursionless

Recursion is great, but in most scenarios, you don't need to use it. See this factorial implementation without recursion:

```ruby
def factorial(n)
  raise InvalidArgument, "negative input given" if n < 0

  result = 1
  for i in (1..n) do
    result *= i
  end
  result
end

```

And running the `factorial` for the last time:

```ruby
irb> factorial(100_000).to_s.size
=> 456574
```

That's it! The same behavior without recursion, TCO or any VM aditional configuration :)

## Conclusion

* The stack size could be configured via `RUBY_THREAD_VM_STACK_SIZE` environment variable;
* TCO could be enabled in runtime via `RubyVM` compile options;
* TCO should be avoided since it messes with stack trace, making harder to debug;
* If you can, don't use recursion! Period.

## References

* [Recursion on Wikipedia](https://en.wikipedia.org/wiki/Recursion)
* [Tail Call Optimisation in Ruby (MRI)](https://www.reinteractive.net/posts/214-tail-call-optimisation-in-ruby-mri)
* [Tail Call Optimization in Ruby](https://nithinbekal.com/posts/ruby-tco/)
* [Increase max stack size for Ruby 2.0](https://clearcove.ca/2013/10/how-to-increase-max-stack-size-for-ruby-2-0-when-experiencing-systemstackerror-stack-level-too-deep/)

Learn something new about recursion, TCO or RubyVM? Consider to share this post with your friends or co-workers. For questions, comments or suggestions, use the comments below. Code hard and success!
