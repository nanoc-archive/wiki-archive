# Roadmap

This gives a rough overview of the nanoc pre-releases leading up to nanoc 4.0.

For context around the work for nanoc 4, see [this thread](https://groups.google.com/forum/#!topic/nanoc/H0qS6mis6kw).

## 4.0.0 final

Required:

* [x] Release updated plugins that depend on Nanoc 4 rather than 3
  * [x] `guard-nanoc`
  * [x] `nanoc-asciidoctor`
  * [x] `nanoc-external`
  * [ ] `nanoc-git` — need to get hold of nanoc-git permissions first
  * [x] `nanoc-lftp`
  * [x] `nanoc-typohero`
  * [x] `nanoc-erector` — never released
* [x] Release Nanoc 4 (same “Releasing Nanoc” steps as before)
* [x] Redirect v4.nanoc.ws to nanoc.ws
  * [x] Remove “for beta” indicator
  * [x] Set up redirects from old to new URL structure

Optional:

* [ ] Create v3.nanoc.ws
* [ ] Add outdatedness warning to v3.nanoc.ws

## 4.0.0b2 and beyond

Fix more bugs!

## 4.0.0b1 (released)

Done:

- Add more identifier convenience methods (`#without_ext`)
- Allow `item.identifier =~ "/stuff.*"`
- Allow finding layouts by glob
- Remove private methods on views
- Document new classes/methods
- Go through the API and make sure no return values expose internal objects. For instance, `items.create` unintentionally exposes `Nanoc::Int::Item`.

## 4.0.0a2 (released)

Done:

- Support glob patterns
- Use glob patterns and new-style identifiers by default
- Support new-style identifiers
- Add convenience methods (#570, #571)
- Fix any issues encountered on the nanoc web site

## 4.0.0a1 (released)

Done:

- Remove deprecated code
- Mark non-essential parts of the API as private
- Create view classes
- Move internals into `Nanoc::Int`, and mangle monkeypatched method names
- Create `Nanoc::Identifier` class
- Allow data sources to create items without instantiating `Nanoc::Int::*`.
- Allow preprocessor to modify data
- Have public `Nanoc::Error` class from which internal errors inherit
- Mangle `Nanoc::Extra::TimeExtensions` names

## Miscellaneous

### Ideas

- `ItemRepCollection#[]` to access item reps by name **[DONE]**

- `Nanoc.compile` convenience method (because Site is private now) **[REJECTED]** - Pointless without CLI integration, and I’d like to spend proper time (re)designing this API.

- `Nanoc::Identifier#===` so that it can be used with `Enumerable#grep` and in case comparisons

### Questions

* Does `ItemCollectionView#[]` make sense? Should it instead expose `#find` and `#find_all`? `#select`? `#all`? … **[IMPLEMENTED]**

* Should the nanoc binary set the default encoding to UTF-8? Would it be a good citizen if it did? (I’m assuming this is not OK for libraries, but for applications it is.) **[REJECTED]** - default in new sites is UTF-8 though.

* What should the naming of the identifier and pattern syntax be like? Currently, it is `identifier_style` (`full` or `stripped`) and `pattern_syntax` (`glob` or nil). Syntax vs. style is confusing. **[DONE]** - `string_pattern_type` and `identifier_type`

* Does it make sense to wrap Array and Hash obtained from attributes with functionality for supporting indifferent (symbol/string) access? This can probably wait for later. - **[PENDING]** - Trouble getting Mustache to behave properly

* Does it make sense to make representation names strings instead of symbols? Symbols don’t make that much sense to me. Changing them in a way that doesn’t strangely break existing code might be difficult though. **[PENDING]** - Maybe later; low prio.