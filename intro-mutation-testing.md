% Mutation Testing for Test Lovers
% Jordan Ryan Reuter
% 17 October 2016

# A test for your tests

## What is it for?

Running 

- Another kind of coverage metric
- Perhaps more interestingly, a _driver_ of implementation

## Assumptions

- Your specs are passing
- Your specs are not flaky
- Your specs are _fast_

## What does it mean?

Mutation testing looks for cases where production code changes could slip past
tests.

The algorithm:

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

<https://github.com/jreut/key-value-store.git>

`mutant-rspec` gem: <https://github.com/mbj/mutant>
