# A modest type annotation proposal for ECMAScript

A type annotation proposal for ECMAScript that doesn't try to specify the world. It captures the essence of what we need, while leaving the door open for future extensions and 

-----

So far, here's three things I've noticed:

> 1) Optionally statically typed languages are starting to *seriously* catch on, even in the JavaScript world.

I know it sounds like I'm repeating the hype train, but [no, seriously](https://insights.stackoverflow.com/survey/2018#most-popular-technologies). JavaScript may have unseated Java on Stack Overflow, but TypeScript is *solidly* ahead of even Ruby, Objective-C, and Swift, closing in on C. Angular famously switched to TypeScript for version 4, encouraging all its users to do the same (after many already had on their own), and React has lately started promoting [BuckleScript](https://bucklescript.github.io/) with the [Reason](https://reasonml.github.io/) frontend for easier development as many of its users were already adopting TypeScript and Flow (mostly the latter, but that's starting to change).

It's safe to say that static types within JS and compile-to-JS languages is starting to catch on.

*It's been catching on elsewhere in dynamic language communities, too, like [PHP 7 adding optional type checking](https://secure.php.net/manual/en/migration70.new-features.php) after [Hack](https://hacklang.org/), Python going from [function annotations](https://www.python.org/dev/peps/pep-3107/) to [actual runtime type annotations](https://www.python.org/dev/peps/pep-0484/), Clojure working on [core.typed](https://github.com/clojure/core.typed), and Matz [considering adding type annotations to Ruby at RubyConf 2014](http://500errors.com/omniref/2014-11-17-matz-at-rubyconf-2014-will-ruby-3-dot-0-be-statically-typed.html) after [issue #9999](https://bugs.ruby-lang.org/issues/9999).*

> 2) Most type annotation proposals I've seen are bloated, complicated, and otherwise generally suck in some way.

So far, most of the type annotation proposals I've seen have tried to do more than necessary:

**Static type constraints:** https://github.com/achun/proposal-static-type-constraints-features

This one proposes a static type constraints system, leveraging `void` with various forms. Here's some of the issues it has:

1. It conflicts very obviously with `void` (or alternatively, `typeof`) throughout the proposal. If you know anything about the `void` expression and `typeof` operator, you should know that they take the form `void expression` and `typeof expression`. Most of the proposal conflicts with either of these, and they do so pretty flagrantly.
2. When you follow [the related es-discuss thread](https://esdiscuss.org/topic/proposal-static-type-constraints-features), it's clear there's an intended runtime component to them implicit in the proposal. It's not really described well, and [it's not certain it even interacts with realms correctly](https://mail.mozilla.org/pipermail/es-discuss/2018-June/050978.html).
3. It assumes types are optional by default, requiring an explicit `!Type` to make it required. [Let's *not* continue to propagate the same mistake made for decades](https://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions) when it comes to static typing.

**Optional Static Typing:** https://github.com/sirisian/ecmascript-types

This one proposes a very complex, non-trivial static and runtime type system with a lightly TypeScript-inspired syntax. Here's some of my objections to it:

1. It's difficult to summarize due to the various cross-cutting concerns it tries to address (which are often orthogonal). This alone makes me [question the entire proposal](https://johnresig.com/blog/ecmascript-harmony/).
2. There's a [very pervasive runtime component](https://github.com/sirisian/ecmascript-types#required-changes-to-the-specification) that even most designed-from-scratch statically-typed languages don't always have. For example:
    - [This](https://github.com/sirisian/ecmascript-types#implicit-casting) has no equivalent in most functional languages
    - Most languages model [multi-dimensional arrays](https://github.com/sirisian/ecmascript-types#multidimensional-and-jagged-array-support-via-user-defined-index-operators) in their type system as simple arrays of arrays (it's more flexible - C++ is the exception here, not the rule).
    - [In-place allocation](https://github.com/sirisian/ecmascript-types#placement-new) is only relevant in high-performance, low-level, object-oriented languages like C++ ([placement `new`/`delete`](https://en.wikipedia.org/wiki/Placement_syntax)) and Scala Native ([zone allocation](http://www.scala-native.org/en/latest/user/interop.html#memory-management)). It doesn't make sense without a binary representation of what an object is supposed to look like, and that's something JS lacks outside typed arrays and array buffers.
    - [Member alignment](https://github.com/sirisian/ecmascript-types#member-memory-alignment-and-offset) only makes sense when you can't actually extend the object, but that proposal makes no such efforts to clarify or restrict that.
3. There's a [blue million new types](https://github.com/sirisian/ecmascript-types#types-proposed) proposed.
4. If you have to acknowledge "it would be potentially years before \[the proposal\] would be implemented", you should be questioning why you spent that much effort in building the proposal in the first place.

**Other ideas:** (various places)

I've seen attempts to standardize both TS and Flow syntax, with and without runtime checking, and I'm not convinced of their efficacy. I don't have any particular places I know of right off, but I know I've seen it both in the TS issue tracker and in the es-discuss mailing list.

> 3) Type checking libraries are already pretty common, even now.

There's various hacks people have used to put in runtime type checking, but engines don't have semantic awareness of these hacks, and they are often pretty inefficient. Here's a few of them:

- React's [prop-types](https://www.npmjs.com/package/prop-types) (not especially elegant outside React)
- @gkz's [type-check](https://gkz.github.io/type-check/) (complicated syntax, makes me think of it as a hacky language extension that isn't)
- [Object Model](http://objectmodel.js.org/) (very verbose with functions, doesn't interact well with polymorphism)
- [is.js](http://is.js.org/) (efficient, but not very declarative or JS-y)
- Node-Machine's [rttc](https://github.com/node-machine/rttc) (schema-driven, imperative - two things idiomatic JS isn't)
- Google's massive [Closure Library](https://developers.google.com/closure/library/) even has its own [`goog.asserts`](https://google.github.io/closure-library/api/goog.asserts.html) namespace for various common type checking and assertions.

**Summary:**

I get that types are necessarily not simple (this stops as soon as you combine parametric polymorphism with subtyping or row polymorphism). I don't want to specify the world, just the smallest subset to enable most things to interoperate with everything else.

### So, what's the issue?

None of them really feel like JS, and they all seem to be taking a concept elsewhere and shoehorning them into something that they claim is JS, but is hardly idiomatic. They feel like their own thing, and often get the job done very inefficiently for the programmer and/or the computer. They are complex enough you find yourself learning and coding a language that isn't JS, that doesn't feel like JS, that doesn't even work like JS. Another issue is that many of these things I've seen created and promoted are by people who don't seem to understand how type systems are designed, nor how runtimes should interpret them. (This seems to be the case with most of these static type language proposals and pushes I've seen, which seem to be backed by type fanatics rather than people concerned about functionality.)

### So, what do we need to do?

First, let's take it incrementally. What makes a type system useful?

> 1) It checks your work.

It's much harder to send the wrong thing to a function when the compiler/runtime yells at you for not sending it the right thing. If the method expects a string, and you send `undefined`, you probably didn't mean to send `undefined`. If the method expects an object with a `foo` property, and you send an object that doesn't, you probably screwed up somewhere.

> 2) It limits how free-form you can treat something.

In smaller projects, it's convenient to have a very lax type system, since you're not likely to screw up repeatedly in a few hundred lines of code, but in a project with 30K+ lines of code (or especially one with 100K+), the subtleties free-form types let you introduce can make code no longer obvious. It might seem like it's magically working, but you could be accidentally setting the wrong thing in it and find symptoms several steps later. If those steps seem unrelated, you've just lost quite a bit of productivity spending the next several hours, if not days, tracing the bug back to its origin, and I've experienced that first-hand. (In fact, that's completely stifled progress in one of my open-source projects.)

> 3) It helps you make sense of what data is being handled.

This is arguably one of the most useful things. If you have an idea what data you're handling, it becomes much easier to work with it. Of course, without explicit annotations, you can still infer this to some extent (especially if the data originates nearby), but static types, you can expand that a little more broadly. Even if the data isn't right here, right now, you can often figure out through a simple cursory search where the data is defined, or if you have your editor hooked up for it, you could just hover over the function or variable in question and immediately *know* what data that variable is getting.

Now, taking that knowledge, let's think about what kind of data we need to describe that data:

> 1) We need something easy to specify.

If there's any mental overhead in adding the type, people are going to be hesitant to add it. It's one of the reasons people still use TypeScript over `propTypes` in React code. To draw an analogue with the type-checking libraries I listed above, it's much easier to use is.js than it is to use rttc.

> 2) We need something easy to comprehend.

This is why we need it to be declarative, not imperative. Imperative type-checking libraries feel a lot like using constraints with pre- and post-conditions. They also don't always work well with return values, since those so frequently are returned anonymously. To explain this concretely on the library side, it's much easier to comprehend component types declared with prop-types than it is those explicitly checked via type-check preconditions.

> 3) We need something easily statically analyzable.

This is the primary reason people gravitate to TypeScript and Flow over type-checking libraries. You don't need to *run* code to check the basic constraints of what a function can handle and that you're passing the right types along. If it compiles without warnings, you're at least guaranteed to have something that won't blow up short of invalid object states.

### Now, what should we be checking *for*?

This is where it gets interesting, and you're going to find a *lot* of opinions on the subject. There are of course countless ways to do type checking:

- Static class- and interface-based OOP (Java, Scala, C#, etc.)
- Type classes and traits (Haskell, PureScript, Rust, etc.)
- Struct checking (C, Elm, TypeScript, Go, etc.)
- Constraint checking (Clojure's core.typed, Typed Prolog, etc.)
- And of course, there are things that cover more than one category:
    - OCaml: OOP + struct checking
    - C++: OOP + struct checking + constraint checking
    - Scala: OOP + struct checking + traits

These are obviously [not the only ways to approach type checking](https://en.wikipedia.org/wiki/Type_system), and the above list is very obviously incomplete. (It also glosses over things very horribly.) But I think we *can* agree on parts of it, and this proposal attempts to capture much of that intersection, while also providing facilities for type checkers to read certain more complex language idioms (like Knockout's [observables](http://knockoutjs.com/documentation/observables.html) and [observable arrays](http://knockoutjs.com/documentation/observableArrays.html)). It doesn't cover everything, but it at least attempts to cover most things.

## Proposal

So finally, here's my proposal. It's pretty straightforward. My goal here is to define a common subset easy for most checkers to adhere to and build from. I'd rather see them start to gear towards a common interface where each checker exercises their strengths.

We've already started seeing some level of convergence between TypeScript and Flow, with even the Closure Compiler working on accepting a broad subset of TypeScript syntax to check. Also, there's several other platforms and a growing body of utilities reading, writing and/or converting TypeScript definitions to/from their language (like [Kotlin](https://github.com/Kotlin/ts2kt) and [Fable (F#)](https://github.com/fable-compiler/Fable/pull/240)). If we can standardize what a definition file might look like, even at a high level outside the ES spec, that would simplify it quite a bit for them on its own. Such a definition file is out of scope for this proposal, however. (The type syntax would likely mirror this, which is one thing I'm taking into account here.)

For clarity, expressions are always in the form of `let result = (...);` and statements always end in a statement (except class/function declarations) when I introduce them, so they can be trivially discerned.

### Semantics

All annotations should be ignored by engines. Type annotations do not affect the completion value of their operand, although engines *may* choose to accept them as hints. Type declarations are no-op statements. Imported type bindings *do* however cause an unconditional type error to be thrown when used as a value (a very bold exception, but the only one - it's less than what engines have to do for globals anyway).

A checker *could* choose to use module namespace objects as type namespaces, too, but that's not something an engine needs to care about (so it's currently just ignored).

### Annotations

**Basic type assertions:**

```js
let result = (expression: Type);
```

This is the standard way to annotate an expression as having a particular type. The result is an expression, but it cannot be used as the body of an expression statement itself (to not conflict with the already-valid `label: expression`), and it has lower precedence than ternary operators (to avoid breaking them).

```js
let result = (expression as Type);
```

This is provided as the primary way to annotate an expression as being casted to a particular type. There exists a "no *LineTerminator* here" restriction before the `as`, to avoid conflicting with the legitimate statement `expression; as(Type);` in the case of a parenthesized type.

Note: this does *not* perform runtime checks, but a checker *might* choose to introduce one.

**Variable type annotations:**

```js
var foo: VariableType;
var foo: VariableType = ...;
let foo: VariableType;
let foo: VariableType = ...;
const foo: VariableType = ...;
```

These are for marking variables as having a particular type. Note that `let foo: Type = bar` and `let foo = bar: Type` are intended to be semantically equivalent, although checkers aren't required to uphold that invariant.

**Parameter and return type annotations:**

```js
function foo(bar: ParamType, ...baz: RestType): ReturnType { ... }
```

This is the standard way to declare a monomorphic function with particular types.

```js
let result = ((foo: ParamType, ...bar: RestType): ReturnType => ...);
let result = (foo: ParamType => ...);
```

This is the standard way to annotate a monomorphic arrow function. Note that the second form does not provide facilities for the return type to be explicitly listed, so if you wish to annotate the return type, you should just annotate the expression itself instead.

**Generic type annotations:**

```js
function foo<A, B, C>(...params): ReturnType { ... }
function <A, B, C>(...params): ReturnType { ... }
const obj = {
    method<A, B, C>(...params): ReturnType { ... },
};
class Foo {
    constructor<A, B, C>(...params): ReturnType { ... }
    method<A, B, C>(...params): ReturnType { ... }
    get property(): Type { ... }
    set property(value: Type) { ... }
}
```

This is the standard way to declare a polymorphic named functions and methods. Note that a class `constructor` itself can be separately annotated as well (with its own return type), as evidenced above. A checker can mandate that the return type be assignable to the class, but that's not something this mandates.

Note that getters must *not* have parameter type annotations (they can't have parameters), and setters *must not* have return type annotations (the return value is just ignored). Additionally, getter return types and setter parameter types *should* match, but this is up to the checker to enforce, not the parser.

```js
class Foo<A, B, C> { ... }
```

This is the standard way to denote a class itself as polymorphic.

```js
class Foo extends Bar<A, B, C> { ... }
```

This is the standard way to denote a class extending a polymorphic class.

**Generic type constraints:**

```js
let result = (<A, B, C>(...params): ReturnType => ...);
let result = (<A, B, C> foo: ParamType => ...);
```

This is the standard way to annotate a polymorphic arrow function. As there currently exists an arrow lookahead restriction prohibiting `expr \n < A > ( foo ) => bar` (note: newline after `expr`), this is completely safe. (Trust me in that I didn't expect that syntactic real estate to exist. I had to test it [across multiple JS parsers](http://astexplorer.net/#/gist/4233cd4e02a1fe8985b5c5f623078d6a/93d36f696639b19b4463526c7cbbc92e521a0a00) as well as in Firefox before I could believe it existed.)

```js
function foo<A extends B, C super D>(...params): ReturnType { ... }
let result = (<A extends B, C super D>(...params) => ...)
// etc.
```

**`this` type annotations:**

This is the standard way to describe type constraints within a function.

```js
function foo(this: ThisType, ...) { ... }
const obj = {
    method(this: ThisType, ...) { ... },
};
```

This is the standard way to annotate the `this` type in a function or method.

**Property type annotations:**

```js
class Foo {
    property: Type;
    property: Type = value;
    // etc.
}
```

This is the standard way to denote a class instance property.

```js
class Foo {
    static property: Type;
    static property: Type = value;
    // etc.
}
```

This is the standard way to denote a class static property.

### Declarations

Here's a list of the various ways to declare and/or export a named type, with the recommended interpretation for each.

```js
type Foo = ...;
```

Declare a type alias. A checker *should* consider this a simple alias, not opaque or nominal.

```js
type Foo<A, B, C> = ...;
```

Declare a type alias with various parameters. A checker *should* consider this (when specialized) as a simple alias, not opaque or nominal.

```js
interface Foo {
    one: One;
    two: Two;
    three: Three;
}
```

Declare an interface type. A checker *may* consider this as structural, but *should* display the interface's own name when a type mismatch is detected within it.

```js
// Named
import {Foo} from "...";

// Aliased
import {Foo as Bar} from "...";

// Default
import Foo from "...";
import {default as Foo} from "...";
```

Import a type binding. As the sole exception to the zero-interpretation rule, attempts to import a type binding as a value at runtime cause a `ReferenceError` to be thrown unconditionally. It's low-overhead, and it solves the issue of type confusion. (Types target checkers, not engines.)

```js
// Named
export {Foo};

// Alias
export {Foo as Bar};

// Default
export default type Foo = ...;
export {Foo as default};
```

Export a type binding.

### Types

Here's a list of the various types, with the recommended interpretation. Of course, checkers are not required to interpret all of these.

**Basic types:**

```js
Identifier
```

This is just a basic type reference.

```js
Identifier<Type>
```

This is an instantiated generic type reference.

```js
identifier.IdentifierName
```

This is a type located in a namespace. The identifier need not necessarily be defined in this file, as it's the checker's job to ensure its validity, not the parser's or runtime's.

**Primitive types:**

```js
:identifierName
```

These represent primitive types. It's recommended to use the `typeof` names and similar for the various primitive types, but a checker is not required to support any particular identifier.

These exist in a separate namespace deliberately separate from the normal type namespace, to help keep [the addition of new primitive types](https://github.com/Microsoft/TypeScript/pull/24439) from being breaking changes. This is especially useful for other primitive types that could be added in the future, such as [`:decimal`](https://docs.google.com/presentation/d/1jPsw7EGsS6BW59_BDRu9o0o3UwSXQeUhi38QG55ZoPI/edit?pli=1#slide=id.p) or [`:int32x4`](https://github.com/tc39/ecmascript_simd/) (if SIMD gets revived again).

For compatibility, these primitive types are required to be supported:

- `:string` (meant for `typeof value === "string"`)
- `:number` (meant for `typeof value === "number"`)
- `:boolean` (meant for `typeof value === "boolean"`)
- `:object` (meant for `typeof value === "object" && value !== null`)
- `:function` (meant for `typeof value === "function"`)
- `:symbol` (meant for `typeof value === "symbol"`)
- `:bigint` (meant for `typeof value === "bigint"`)
- `:any` (meant for explicitly untyped values, like TypeScript's `any` and Kotlin's JS-only `dynamic`)
- `:unknown` (meant for types of unknown value, like Flow's `mixed` and TypeScript's upcoming `unknown`)

The list of other permitted identifiers for this form is implementation-defined, although those only implementing the grammar (like in a JS parser) *should* provide host-defined hooks for them.

```js
:identifierName<A, B, C, ...>
```

Primitive types may also be made generic. This is not *as* broadly useful, but it offers a nice substitute for checkers like [Flow](https://flow.org/en/docs/types/utilities/) who prefer to rely on magic types rather than syntax to represent things like getting keys, etc. It also would allow TypeScript to fall in line with them using `:keyof<SomeType>` instead of `keyof SomeType` to get the keys of a type. (The list of permitted identifiers for this form is implementation-defined, although those only implementing the grammar *should* provide host-defined hooks for them.)

These exist in a separate namespace deliberately separate from the normal type namespace, to help keep the addition of new primitive types and operators from being breaking changes. This is especially useful for other primitive type operators that might get added as well as incorporating support for [various](https://github.com/mikewest/tc39-proposal-literals) [proposals](https://github.com/tc39-transfer/tc39-module-keys).

For compatibility, these primitive type operators are required to be supported:

- `:keyof<...>` (meant for getting a union of the set of keys an object may have)
- `:async<...>` (meant to alias `Type | Promise<Type>`)
- `:optional<...>` (meant to alias `Type | null | undefined`)

```js
undefined
null
```

These represent those literal types.

```js
void
```

This should represent a function with no return value (separate from `undefined`).

```js
*
```

This should represent an existential type whose value should be inferred. It should be treated as something close to a type-safe `any`, but it also could be used to [allow partially specifying a function's return value](https://flow.org/en/docs/types/utilities/#toc-existential-type). (TypeScript currently does *not* have such an expression, but one could simulate it to a broad extent.)

```js
(Type)
```

This is available to control precedence as necessary.

```js
Type & Type
Type | Type
```

This represents the intersection and union of two particular types.

```js
{foo: Foo, bar: Bar, ...Baz}
```

This should be read as a structual type or an anonymous duck-typed interface.

```js
[Foo, Bar, ...Baz]
```

This should be read as a tuple.

```js
"foo"
1
false
// etc.
```

These should be read as their literal types.

```js
typeof expression
```

These should be read as retrieving the type of the expression's result (not merely the primitive type). The expression is *not* executed, however.

## Potential questions

### Why `:optional<Type>`, not `Type?`

I'm aware it's what Flow uses, and I'm also aware it doesn't even conflict, since [it's only possible to conflict if you use an annotation at the top level](http://astexplorer.net/#/gist/7b3ed71f281b92dbb363b9e98ee2c81b/cf662fc53f2af330978b7a18c7b2ced9c31ce398), which was already expressly banned previously. Third, I know it's gained some traction in other languages (like Kotlin and Swift). This adds up to quite a bit of precedent.

However, this doesn't make it universal, and TypeScript doesn't even provide an equivalent operator either way. In addition, I tried to minimize the syntactic real estate this takes up, so I instead just reused an existing syntax idiom to demonstrate it.
