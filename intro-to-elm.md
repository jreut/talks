% Functional Front End with Elm
% Jordan Ryan Reuter
% 31 Oct 2017

# Web history

## First, some pertinent images {.shrink .c}

. . .

![A spooky tree](https://c1.staticflickr.com/9/8262/8701149466_95d7360127_b.jpg)

## First, some pertinent images {.shrink .c}

![A Jack O'Lantern](https://c1.staticflickr.com/3/2279/1795110611_35b40b26c1_b.jpg)

## First, some pertinent images {.shrink .c}

![A skeleton (fish)](https://upload.wikimedia.org/wikipedia/commons/7/73/BLW_Meyer's_Butterfly_Fish.jpg)

## First, some pertinent images {.shrink .c}

![Some Día de los Muertes confections](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c0/Alfe%C3%B1iques_or_sugar_fugures.jpg/1024px-Alfe%C3%B1iques_or_sugar_fugures.jpg)

## First, some pertinent images {.shrink .c}

![My favorite animal](https://upload.wikimedia.org/wikipedia/commons/c/c9/Bubo_virginianus00.jpg)

## First, some pertinent images {.shrink .c}

![The spookiest of all](https://upload.wikimedia.org/wikipedia/commons/6/6a/JavaScript-logo.png)

# Web history

## Everyone hates JavaScript

But we're stuck with it.

## Transpiled languages

Instead of trying to get every Web browser to implement some other language
interpreter, some folks are writing languages that compile to JavaScript.

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
- Events, actions, etc. are _data_.
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

- explicit `Main` module and `main` function
- hash-like thing (it's a _record_)
- compile-time checked, inferred types
<!-- - formatting on save via [elm-format](elm-format) -->

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

type Msg = NoOp -- we'll come back to this

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

## Bind input

```elm
type Msg = ChangeHeader String

update : Msg -> Model -> Model
update msg model =
    case msg of
        ChangeHeader string -> string

view : Model -> Html Msg
view model =
    div []
        [ h1 [] [ text model ]
        , input [ onInput ChangeHeader ] []
        ]
```

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
