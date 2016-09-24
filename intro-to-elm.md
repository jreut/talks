% Functional Front End with Elm
% Jordan Ryan Reuter
% 23 Oct 2017

# I hate the front end

# Web history

## the Web circa 2016

- HTML5 is widely implemented
- Immutablity is cool
- jQuery is still alive
- CSS-driven animations are on the rise

## the Web circa 2010

> - ~~HTML5 is widely implemented~~
> - ~~Immutablity is cool~~
> - ~~jQuery is still alive~~
> - ~~CSS-driven animations are on the rise~~

## the Web circa 2010

> - HTML5 is a Working Draft
> - ~~Immutablity is cool~~
> - ~~jQuery is still alive~~
> - ~~CSS-driven animations are on the rise~~

## the Web circa 2010

> - HTML5 is a Working Draft
> - IIFEs and Coffeescript are cool
> - ~~jQuery is still alive~~
> - ~~CSS-driven animations are on the rise~~

## the Web circa 2010

> - HTML5 is a Working Draft
> - IIFEs and Coffeescript are cool
> - jQuery is basically required by default
> - ~~CSS-driven animations are on the rise~~

## the Web circa 2010

> - HTML5 is a Working Draft
> - IIFEs and Coffeescript are cool
> - jQuery is basically required by default
> - CSS gets the box model!

# The Rise of Structure

## JavaScript libraries

Facebook introduced us to immutability

- ImmutableJS
- React
- Flux/Redux

## Transpiled languages

Everyone agreess JavaScript lacks one thing---strong types.

. . .

[Dart](dartlang)

:    Google ca. 2011

:    Java-like object-orientation + Promise-like DOM wrapper

[TypeScript](typescript)

:    Microsoft ca. 2012

:    Java-like object-orientation

[PureScript](purescript)

:    ca. at least 2014

:    ML-flavored type system

[Elm](elmlang)

:    Evan Czaplicki ca. 2012

:    ML-flavored type system + virtual DOM

# Elm

## Type system

- ML-like (meaning Haskell-like)
- Everything is a _value_. Null &rarr; Maybe
- Immutable by default
- Side effects are bundled into monads^[Don't worry, monads aren't scary!]
- __No runtime exceptions.__ "If it compiles, it runs"

## The Elm Architecture

From the [Elm docs](elm-docs-architecture):

> The logic of every Elm program will break up into three cleanly separated parts:
>
> __Model__ — the state of your application
>
> __Update__ — a way to update your state
>
> __View__ — a way to view your state as HTML

. . .

Sound familiar? Redux took inspiration from Elm.

[typescript]: http://www.typescriptlang.org
[dartlang]: https://www.dartlang.org
[purescript]: http://www.purescript.org
[elmlang]: http://elm-lang.org
[elm-docs-architecture]: https://guide.elm-lang.org/architecture/#the-basic-pattern
