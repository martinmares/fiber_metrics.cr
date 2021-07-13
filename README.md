# Fiber metrics (Experimental)
[![Crystal CI](https://github.com/didactic-drunk/fiber_metrics.cr/actions/workflows/crystal.yml/badge.svg)](https://github.com/didactic-drunk/fiber_metrics.cr/actions/workflows/crystal.yml)
[![GitHub release](https://img.shields.io/github/release/didactic-drunk/fiber_metrics.cr.svg)](https://github.com/didactic-drunk/fiber_metrics.cr/releases)
![GitHub commits since latest release (by date) for a branch](https://img.shields.io/github/commits-since/didactic-drunk/fiber_metrics.cr/latest)
[![Docs](https://img.shields.io/badge/docs-available-brightgreen.svg)](https://didactic-drunk.github.io/fiber_metrics.cr/main)

Track run time, wait time or memory allocations per `Fiber`, method or block.

Designed for low overhead use in production.

NOTE: This shard is modular.
* `require "fiber_metrics"` Requires the minimum necessary to run with the lowest overhead
* `require "fiber_metrics/all"` Batteries included
* * Crystal is monkey patched to record idle/blocking time

## TODO
- [x] `Channel.send,recv`
- [ ] `sleep`
- [x] `Mutex`
- [ ] `IO`
- [ ] Hook in to `Scheduler`
- [ ] ENV to control printing


## Installation

1. Add the dependency to your `shard.yml`:

   ```yaml
   dependencies:
     fiber_metrics:
       github: didactic-drunk/fiber_metrics.cr
   ```

2. Run `shards install`

## Usage


```crystal
require "fiber_metrics/all"

class Example
  # Must be included in each class with an annotation **including subclasses**
  include Fiber::Metrics

  @[Measure]
  def run
    sleep 0.2
  end
end

e = Example.new
e.run

Fiber.print_stats
```

## Output
```
Example.run tt:   0.203 rt:   0.200                           calls:      1    mem: 0k
```

## How it works

`@[Measure]` wraps each method in 2 `Time.monotonic` calls and records the difference.

The collected metrics use Fiber local storage (no locking) until the first measured call returns
after which the local metrics are merged with a global set of metrics wrapped in a Mutex.

This means metrics are unavailable via `Fiber.stats` until the first measured method `foo` returns.
This is ideal for
* Applications that show/record stats before exit
* Request/response (HTTP, message queue, etc) type servers that want periodic snapshots of activity

### @[Measure] Logic
```crystal
@[Measure]
def foo
  # Time.monotonic
  bar
  # Time.monotonic
  # Store metrics in fiber local storage
  # Merge fiber local metrics with global metrics
  # Reset fiber local metrics
end

@[Measure]
def bar
  # Time.monotonic
  baz
  # Time.monotonic
  # Store metrics in fiber local storage
end

@[Measure]
def baz
  # Time.monotonic
  ...
  # Time.monotonic
  # Store metrics in fiber local storage
end
```


More options to control the merging process may appear after use cases are discussed.
Open a discussion if the current behavior doesn't fit your needs.


## Contributing

1. Fork it (<https://github.com/didactic-drunk/fiber_metrics.cr/fork>)
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## Contributors

- [Didactic Drunk](https://github.com/didactic-drunk) - creator and maintainer
