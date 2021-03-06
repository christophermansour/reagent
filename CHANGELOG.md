# Changelog

## 0.8.0-alpha1 (2017-07-31)

**[compare](https://github.com/reagent-project/reagent/compare/v0.7.0...v0.8.0-alpha1)**

**BREAKING**: Requires ClojureScript version 1.9.854

This version changes how Reagent depends on React. New ClojureScript
improves support for [npm packages](https://clojurescript.org/news/2017-07-12-clojurescript-is-not-an-island-integrating-node-modules)
and also improves the way code can refer to objects from foreign-libs,
making the transition from foreign libs, like Cljsjs packages, to npm easy: [global exports](https://clojurescript.org/news/2017-07-30-global-exports)

Previously Reagent required foreign-lib namespace `cljsjs.react` and a few others.
This worked well when using Cljsjs React package, but in other environments,
like Node, React-native and when using npm packages, users had to
exclude Cljsjs packages and create empty files providing these `cljsjs.*` namespaces.

With global-exports, foreign-libs can be used like they were real namespaces:

```cljs
(ns ... (:require [react-dom :as react-dom]))

(react-dom/render ...)
```

The same code will in all the environments, and is compiled different based on
compile target and on how the dependency is provided. When targeting
browser and using foreign libs, ClojureScript compiler uses the `:global-exports`
definition to resolve the function from global JS var:

```js
var a = window.ReactDOM;
a.render(...)
```

When targeting browser but using node_modules with Closure module processing,
the CommonJS (or ES6) module is converted to a Closure module, named
by `module$` and the path of the file, and the generated code is same as
if this was a Cljs or Closure module:

```js
module$foo$bar$react$react-dom.render(...)
```

Then targeting NodeJS the object is retrieved using `require` call:

```js
var a = require("react-dom");
a.render(...)
```

This change requires use of ClojureScript 1.9.854, using the latest Cljsjs
React packages (15.6.1-1), and it is not yet sure how well other React
libraries work with these changes, or how this will work with React-native.
Currently it looks like all the Cljsjs React libraries need to be updated
to use require `react` instead of `cljsjs.react`, as the foreign-lib
namespace was renamed to match the npm package.

Read [0.8 upgrade guide](./docs/0.8-upgrade.md) for more information.

## 0.7.0 (2017-06-27)

**[compare](https://github.com/reagent-project/reagent/compare/v0.6.2...v0.7.0)**

- Fixed a warning with recent ClojureScript (1.9.660+) versions about
a variadic method signature in `reagent/impl/util.cljs`.
    - `reagent.core/partial` and `wrap` used a bad deftype ([#303](https://github.com/reagent-project/reagent/pull/303))
- React updated to 15.5.4 ([#292](https://github.com/reagent-project/reagent/issues/292))
    - Uses [create-react-class](https://www.npmjs.com/package/create-react-class) instead of
    deprecated `React.createClass`
    - Reagent has now dependency on `cljsjs/create-react-class`, if you are using other
    methods to provide React, you need to exclude this Cljsjs dependency and provide the library yourself.
- Self-host compatibility ([#283](https://github.com/reagent-project/reagent/pull/283))
    - Removed deprecated `reagent.interop/.'` and `reagent.interop/.!` macros
- Improved assert messages all around ([#301](https://github.com/reagent-project/reagent/pull/301)).

## 0.6.2 (2017-05-19)

**[compare](https://github.com/reagent-project/reagent/compare/v0.6.1...v0.6.2)**

- React updated to 15.4.2
    - Fixes a problem with `number` inputs, ([#289](https://github.com/reagent-project/reagent/issues/289), [facebook/react#8717](https://github.com/facebook/react/issues/8717))

## 0.6.1 (2017-03-10)

**[compare](https://github.com/reagent-project/reagent/compare/v0.6.0...v0.6.1)**

- Fix :ref on inputs ([#259](https://github.com/reagent-project/reagent/issues/259))
- React updated to 15.4.0 ([#275](https://github.com/reagent-project/reagent/issues/275), [#276](https://github.com/reagent-project/reagent/issues/276))
- **BREAKING:** `reagent.core` no longer provides `render-to-string` or `render-to-static-markup` functions
    - `reagent.dom.server` includes the same functions
    - This is due to change in React packaging, including React-dom-server would increase the file size considerably, so now it is only included when `reagent.dom.server` is used

## 0.6.0

- React updated to 15.2.1

- Fix input on-change events in IE11


## 0.6.0-rc

- React updated to 15.1.0

- Symbols and keywords are now allowed in Hiccup content. They are converted using `name`.

- Any object hat satisfies IPrintWithWriter is also allowed, and is converted using `pr-str`.

- Bug fixes, improved error handling/reporting, and lots of testing.


## 0.6.0-alpha

### Breaking changes

- Reagent now depends on `cljsjs/react-dom` and `cljsjs/react-dom-server`, rather than on `cljsjs/react` directly.

- Reactions are now asynchronous, just like Reagent components. `flush` forces outstanding reactions to run.

- Reactions now only trigger updates of dependent components if their value change, as reported by `=` (previously, `identical?` was used).

- The macros `.'` and `.!` in `reagent.interop` have been renamed to `$` and `$!` respectively.


### News

- React updated to 0.14.3

- Added `reagent.dom` and `reagent.dom.server` namespaces, corresponding to new React packages.

- `create-class` now returns a normal React class, that can be used directly from javascript.

- Add `track`: turns a function call into a reactive value.

- Add `track!`: eager version of `track`.

- Add `dispose!`: stop the derefable returned by `track!` from updating.

- Add `with-let` macro: simpler handling of lifecycle in components and reactions.

- Add `rswap!`: works like `swap!`, except that recursive calls are allowed, and they always return nil.

- `cursor` now shares state between all instances corresponding to a given set of parameters.

- `next-tick` now evokes its argument function at a clearly defined time (immediately before rendering, which is in turn triggered using requestAnimationFrame).

- `after-update` is a new function, similar to `next-tick`, except that the function is evoked immediately after rendering.

- Support `[:> nativeComp {:foo "bar"}]`

- Reagent now falls back to using `require` if global `React` is undefined, to simplify use with e.g webpack and node.js.



## 0.5.1

- React updated to 0.13.3

- Deprecate calling the result of `create-class` as a function (i.e always use hiccup forms to call Reagent components).

- Hiccup syntax has been extended to allow nested elements to be defined using '>' as part of the keyword name.

- Add `force-update` for completeness.

- Try harder to maintain cursor position in inputs.

- Simplify examples, taking advantage of new figwheel.

- Better warnings and error messages.


## 0.5.0

- React updated to 0.12.2

- Reagent no longer bundles React. Instead it uses cljsjs/react as a dependency. This means that you should no longer specify React in `:preamble` in your project.clj.

- ClojureScript 0.0-2816 or later is required.

- `adapt-react-class` makes it easier to use "native" React components with Reagent.

- `reactify-component` makes it easier to use Reagent components in JSX.

- `cursor` is re-written, to be more efficient and flexible.

- `render` now forces a deep update of all components, to make it more convenient to use with e.g. figwheel.

- Renamed `as-component` to `as-element`, to match React's new terminology better (old name still works, though, for backward compatiblity).

- Stop wrapping native components. This reduces the number of components created a lot, and can speed up some things substantially (especially render-to-string, that is not bound by browser performance). This is made possible by a new way of keeping track of which order to re-render dirty components.

- Added `create-element` to make it easier to embed native React
components in Reagent ones.

- Arguments to components are now compared using simple `=`, instead of the old, rather complicated heuristics. **NOTE**: This means all arguments to a component function must be comparable with `=` (which means that they cannot be for example infinite `seq`s).

- Reagent now creates all React components using `React.createElement` (required for React 0.12).

- `render-component` is now `render`, and `render-component-to-string` is `render-to-string`, in order to match React 0.12 (but the old names still work).

- Add `render-to-static-markup`. This works exactly like `render-to-string`, except that it doesn't produce `data-react-id` etc.

- `create-class` now takes a Reagent-style render function (i.e with the same arguments you pass to the component), called `:reagent-render`.



## 0.4.3

- React updated to 0.11.2

- Add reagent.core/cursor

- Add javascript interop macros .' and .!

- Add force-update-all to make LightTable integration easier

- Some performance optimizations


## 0.4.2

- Allow multi-methods as component functions.

- Tweak performance by avoiding `clojure.core/memoize`.

- Bugfix: Allow on-change handler on controlled inputs to keep value unchanged.


## 0.4.1

- Made Reagent compatible with ClojureScript 0.0-2173. `reagent.core/atom` now implements the necessary IAtom, ISwap and IReset protocols. Reagent should still be compatible with older ClojureScript versions, but you will get a lot of compilation warnings.


## 0.4.0

- Breaking change: Component functions can get arbitrary arguments, and not just vectors and maps. This is a breaking change, but behaviour is unchanged if you pass a map as the first argument (as in all the examples in the old documentation).

- React updated to 0.9.0.

- You can now use any object that satisfies `ifn?` as a component function, and not just plain functions. That includes functions defined with deftype, defrecord, etc, as well as collections like maps.

- `reagent.core/set-state` and `reagent.core/replace-state` are now implemented using a `reagent.core/atom`, and are consequently async.

- Keys associated with items in a seq (e.g ”dynamic children” in React parlance) can now be specified with meta-data, as well as with a `:key` item in the first parameter as before. In other words, these two forms are now equivalent: `^{:key foo} [:li bar]` and `[:li {:key foo} bar]`.

- Performance has been improved. For example, there is now practically no overhead for tracking derefs in components that don’t use atoms. Allocations and memory use have also been reduced.

- Intro and examples have been tweaked a little to take advantage of the new calling conventions.


## 0.3.0

- Changes in application state are now rendered asynchronously, using requestAnimationFrame.

- Reagent now does proper batching of updates corresponding to changed atoms, i.e parents are rendered before children, and children are only re-rendered once.

- Add `reagent.core/flush` to render changes immediately.

- Bugfix: Allow dynamic id with hiccup-style class names.


## 0.2.1

- Bugfix: allow data-* and aria-* attributes to be passed through unchanged.


## 0.2.0

- Rename Cloact to Reagent, due to popular disgust with the old name...
