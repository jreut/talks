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
- __No runtime exceptions.__ "If it compiles, it runs"^[I had one runtime exception because I was doing weird stuff.]

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

. . .

From the [Redux docs](redux-principles)

> The __state__ of your whole application is stored in an object tree within a single __store__.
>
> The only way to change the state is to emit an __action__, an object describing what happened.
>
> To specify how the state tree is transformed by actions, you write pure __reducers__.

## The Elm Architecture

![Elm manages effects for you^[https://guide.elm-lang.org/architecture/effects/]](img/elm-diagram.png)

# Journey

## Plan

Let's write some Elm.

> - Hello, world!
> - Bind input to actions
> - Implement tic-tac-toe (we'll see how far we get)

## Hello, world! { .shrink }

```elm
module Main exposing (..)

import Html.App exposing (beginnerProgram)
import Html exposing (h1, text)

main = beginnerProgram
    { model = "Hello, world!"
    , update = (\_ -> identity)
    , view = (\model -> h1 [] [ text model ])
    }
```

## Hello, world!

To note:

- explicit `Main` module
- required `main` function
- alias `Html.App` to `App`
- hash-like thing (it's a _record_)
- compile-time checked, inferred types
- formatting on save via [elm-format](elm-format)

## Binding input

Propagate state changes by sending actions.

### Create the input

```elm
view model =
    div []
        [ h1 [] [ text model ]
        , input [] []
        ]
```

Now we have an `input[type=text]` element, but it doesn't do anything.

## Binding input { .shrink }

### Clarify with type declarations

```elm
module Main exposing (..)

import Html.App exposing (beginnerProgram)
import Html exposing (h1, text)

main = beginnerProgram
    { model = init
    , update = update
    , view = view
    }

type alias Model = String

type alias Msg = NoOp -- we'll come back to this

init : Model
init = "Hello, world!"

update : Msg -> Model -> Model
update msg model = model

view : Model -> Html Msg
view model =
    div []
        [ h1 [] [ text model ]
        , input [] []
        ]
```

## Bind input

### the `beginnerProgram` function

From [the docs](beginner-program):

```elm
beginnerProgram
    : { model : model
    , view : model -> Html msg
    , update : msg -> model -> model
    }
    -> Program Never
```

You provide the initial state, a way to update that state, and a way to render
it. Elm wires up everything else for you.

## Bind input { .shrink }

### Add an action

```elm
import Html.Events exposing (onInput)

type alias Msg = ChangeHeader String

update : Msg -> Model -> Model
update msg model =
    case msg of
        ChangeHeader string ->
            string

view : Model -> Html Msg
view model =
    div []
        [ h1 [] [ text model ]
        , input [ onInput ChangeHeader ] []
        ]
```

. . .

To note:

> - `onInput` _requires_ an action that takes a `String`.^[Its type is `onInput : (String -> msg) -> Html.Attribute msg`]

## Add a reset button {.shrink}

This should be easy now!

### view

```elm
import Html exposing (button)
import Html.Events exposing (onClick)

view : Model -> Html Msg
view model =
    div []
        [ h1 [] [ text model ]
        , input [ onInput ChangeHeader ] []
        , button  [ onClick Reset ] []
        ]
```

### update

```elm
type alias Msg = ChangeHeader String | Reset

update : Msg -> Model -> Model
update msg model =
    case msg of
        ChangeHeader string ->
            string
        Reset ->
            init
```

## Add a reset button

> - trivial features are trivially easy to add
> - `case ... of` is checked at compile time (try deleting a branch)

# Appendix

## Getting started

- Visit the [Elm Web site][elmlang].
- Read about the [Elm Architecture][elm-docs-architecture].
- Let's build something!

---

[typescript]: http://www.typescriptlang.org
[dartlang]: https://www.dartlang.org
[purescript]: http://www.purescript.org
[elmlang]: http://elm-lang.org
[elm-docs-architecture]: https://guide.elm-lang.org/architecture/#the-basic-pattern
[redux-principles]: http://redux.js.org/docs/introduction/ThreePrinciples.html
[elm-format]: https://github.com/avh4/elm-format
[beginner-programm]: http://package.elm-lang.org/packages/elm-lang/html/1.1.0/Html-App#beginnerProgram
