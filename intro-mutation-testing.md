% Mutation Testing for Test Lovers
% Jordan Ryan Reuter
% 17 October 2016

# A test for your tests

## What is it for?

Mutation testing looks for cases where production code changes could slip by
without causing test failures.

- Another kind of coverage metric
- Perhaps more interestingly, a _driver_ of implementation

### Assumptions (requirements, really)

- Your specs are passing
- Your specs are not flaky
- Your specs are _fast_

## The algorithm

1. Run a given spec, expecting it to pass
1. Load the code to be tested by a given spec
1. Parse that code into an AST
1. Mutate a single node on that tree, eg. turn `false` to `true`
1. Run the entire spec, expecting it to fail
1. Repeat from (2) for all possible mutations

## Why does it have to be fast?

For a project of trivial complexity (3 classes, 15 examples):

- `rspec` takes less than 0.1 seconds, including startup
- `mutant` takes 3 seconds with 132 mutations, running 8 jobs in parallel

## Technical details

If you care to follow along:

- <https://github.com/jreut/key-value-store.git>

- `mutant-rspec` gem: <https://github.com/mbj/mutant>

# Try it out

## A toy project

I'm writing a key-value store with some initial user stories:

- "I can set an arbitrary key to an arbitrary value"
- "I can retrieve a previously-set value using my key"
- "I can remove my value from the store using my key"
- "Before setting any values, the store is empty"

## Spec {.shrink}

```ruby
context 'getting and setting' do
  it 'starts empty' do
    expect(subject.get('foo')).to be_nil
  end

  it 'works' do
    key = 'foo'
    value = 'bar'
    subject.set key, value
    got = subject.get key
    expect(got).to be value
  end
end

it 'deletes' do
  key = 'foo'
  value = 'bar'
  subject.set key, value
  expect(subject.get(key)).to be value
  subject.delete key
  expect(subject.get(key)).to be_nil
end
```

## First implementation {.shrink}

```ruby
class Store
  attr_reader :h

  def initialize
    @h = {}
  end

  def set(k, v)
    h[k] = v
  end

  def get(k)
    h[k]
  end

  def delete(k)
    h.delete k
  end
end
```

## What does mutant say? {.shrink}

100% coverage, because we're not doing anything. I could have written this:

```ruby
require 'forwardable'

class Store
  extend Forwardable

  def_delegators :@h, :[], :[]=, :delete
  alias set []=
  alias get []

  def initialize
    @h = {}
  end
end
```

A thin wrapper around `Hash`

## New requirements

- "I can configure the store to not overwrite a previously set value"
- "The store will not overwrite by default"

## New spec {.shrink}

```ruby
context 'when trying to clobber' do
  it "won't overwrite a previously set value by default" do
    key = 'foo'
    value = 'bar'
    another_value = 'baz'
    subject.set key, value
    subject.set key, another_value
    expect(subject.get(key)).to be value
  end

  context 'when configured to allow it' do
    subject { described_class.new(clobber: true) }

    it 'will overwrite the value' do
      key = 'foo'
      value = 'bar'
      another_value = 'baz'
      subject.set key, value
      subject.set key, another_value
      expect(subject.get(key)).to be another_value
    end
  end
end
```

## Implementation {.shrink}

```ruby
class Store
  attr_reader :h

  def initialize(clobber: false)
    @h = {}
    @clobber = clobber
  end

  def set(k, v)
    h[k] = v if can_write? k
  end

  def can_write?(k)
    !get(k) || clobberable?
  end

  def clobberable?
    @clobber
  end

  def get(k) # ...
  def delete(k) # ...
end
```

## Running mutant {.shrink}

Testing our `Store` class with `mutant-rspec` looks like this:

    mutant --include lib --use rspec --zombie Store

We have one mutant left "alive".

```
Store#initialize:/lib/store.rb:5
- rspec:0:./spec/store_spec.rb:43/Store deletes
- rspec:1:./spec/store_spec.rb:6/Store getting ...
- rspec:2:./spec/store_spec.rb:10/Store getting...
- rspec:3:./spec/store_spec.rb:19/Store getting...
- rspec:4:./spec/store_spec.rb:31/Store getting...
evil:Store#initialize:/lib/store.rb:5:d8f82
```

```diff
@@ -1,5 +1,5 @@
-def initialize(clobber: false)
+def initialize(clobber: nil)
  @h = {}
  @clobber = clobber
end
```
 
## Killing the survivor

All of our tests passed when Mutant changed our default `false` parameter to
`nil`, which is also falsey.

```diff
@@ -1,5 +1,5 @@
-def initialize(clobber: false)
+def initialize(clobber: nil)
  @h = {}
  @clobber = clobber
end
```

How can we fix this?

- Explicitly disallow `nil` in the initializer?
- Remove the default value?

## Killing the survivor

I decided to type-check the initializer argument.

```ruby
# store_spec.rb
context 'configuration' do
  it 'clobber parameter must be true or false' do
    expect { described_class.new(clobber: nil) }
      .to raise_error(ArgumentError, /nil/)
  end
end
```

```ruby
#store.rb
def initialize(clobber: false)
  raise ArgumentError, 'clobber cannot be nil'
    if clobber.nil?
  @h = {}
  @clobber = clobber
end
```

But that's not really a satisfying solution...

# Mutation-driven Development?

## Alternative to type checking

Type checking---especially Boolean type checking---belies an opportunity for
design improvement. There are two separate behaviors here, and we should write
them down.

## Object-oriented solution {.shrink}

```ruby
class ClobberingStrategy
  def clobber(new:, **)
    new
  end
end

class NoClobberingStrategy
  def clobber(old:, new:)
    old.nil? ? new : old
  end
end

class Store
  attr_reader :h, :storage_strategy

  def initialize(storage_strategy: NoClobberingStrategy)
    @h = {}
    @storage_strategy = storage_strategy.new
  end

  def set(k, v)
    h[k] = storage_strategy.clobber(old: get(k), new: v)
  end

  def get(k) # ...
  def delete(k) # ...
end
```
