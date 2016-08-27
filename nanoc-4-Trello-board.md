# Pluggable colorisers

**Status:** Ideas

**Labels:** Changed in nanoc 4.0

* * *

# Selective helpers

**Status:** Ideas

Being able to define what helpers you exactly need could be useful for optimising. This way, a changed helper would only impact items that use this helper.

The helpers would need to follow a more strict naming pattern. Each helper needs its own name, defined by the filename. For instance:

```
helpers/
  blogging.rb
  link_to.rb
lib/
  somecode.rb
```

To use a filter, enable it in `Rules`, like this:

```ruby
compile '*' do
  filter :erb
  filter :kramdown
  using_helper :link_to do
    layout 'default'
  end
end
```

Explicitly calling helpers by name before using them may also solve the issue, and additionally solve namespace collision issues. For example:

```ruby
helper(:link_to).link_to('Homepage!', @item['/'])
```

* * *

# Efficient outdatedness checking

**Status:** Ideas

The outdatedness checker evaluates every single item and layout to determine whether it is outdated or not.

Instead of doing that, it should look at what has changed and infer the collection of changed items.

One problem with the current outdatedness checking approach is that it is linear in the number of items: it receives a collection of items as input and returns the items that need to be recompiled, and it does so by evaluating every item.

This has a negative impact for very large sites, because a single change requires all items to be loaded into memory and evaluated.

An alternative approach is to only take changes into account, and build a list of items to recompile that way. This works fairly well in quite a few cases:

* If the **configuration** is modified, all items need to be recompiled (this is still less than optimal)
* If some **items** are changed, recompile only those items
* If any **code snippet** is modified, all items need to be recompiled
* If any **rule** is modified, items matching the changed rules need to be recompiled (this is trickier than it sounds, because rules can change position without changing)
* Last but not least, any **items depending on changed items** need to be recompiled as well

Note how none of these steps require looping through all items.

### Rules

The original approach using recording proxies does not scale, as this approach is linear in the number of items.

The experimental implementation (see below) parses the Rules file and generates a summary, which is an ordered list of tuples containing the type (`compile`, `layout`, `preprocess`), the glob (e.g. `/**/*.css`) and a checksum of the rule content. To figure out the globs that match items that need to be recompiled, the current list of tuples is compared with the previous list of tuples, and anything not common is treated as outdated (to be more precise, the common prefix is removed from both lists).

### Dependencies

If the experimental Rules parsing implementation is used, then the dependency tracking gets a tad complicated because the item globs will need to be concretised (converted into an item identifier collection), after which the items depending on these items are added to the collection of items to recompile. This is potentially inefficient because the list of items matched by a glob could be very large.

An alternative approach is to never concretised globs, but let the dependency tracker find items by globs.

It may also be possible to delay the dependency analysis until after all directly outdated items are recompiled. Or, similarly (but less memory-intensive): after compiling an item, recompile all items that depend on the item that was just compiled.

### See also

* [experimental implementation](https://gist.github.com/ddfreyne/7388847)

* * *

# Streaming data sources

**Status:** Ideas

**Labels:** Changed in nanoc 4.0

Originally, nanoc was designed to handle small sites (100ish items). There are people with thousands, sometimes tens of thousands of items. Huge sites fail to compile because of memory usage, and even if they do fail, garbage collection takes up a lot of time.

The main problem is that nanoc loads all items into memory. This needs to be changed.

One way of solving this is allowing data sources to return a stream of items. This way, only one item needs to be in memory at a time. When a dependency arises, e.g. one item includes attributes from another item, the data source can be requested to give the item for the given identifier.

Since this approach would potentially use significantly more I/O, there would need to be an item cache. Additional optimisations could be done, such as letting the datasources asynchronously pre-fetch the next 10 items in the item stream.

* * *

# Intelligent item queries

**Status:** Ideas

**Labels:** New in nanoc 4.0

When a new item is added to a site, nanoc recompiles the entire site, because it cannot accurately estimate the impact of the addition of an item. Archive pages will need to be recompiled when a new article is added, for instance. Thus, all items have a dependency on the new item.

A similar problem appears when one page filters all items based on attributes. For instance, an archive page may find items to include using `@items.select { |i| i[:kind] == 'article' }`. This, however, creates a dependency from the archive item onto all items.

Intelligent queries can solve this problem. Intelligent queries replace the usage of `#select`, as mentioned in the paragraph above, and look more like this:

```ruby
@items.query(:kind, eq: 'article').and(:author, in: %w( Denis Steve Bill ))
```

Queries such as these can be stored and reasoned about. When a new item is added, nanoc can look at every item and determine which queries it uses. If and only if any of the queries match an item, a dependency is created.

* * *

# Explicit globals YAML file

**Status:** Ideas

**Labels:** Changed in nanoc 4.0

Instead of using the site configuration file for both configuration *and* site globals (such as author name, author e-mail address, public URL, …), nanoc should have a separate file that can contain these globals.

This will probably be necessary in combination with a pure Ruby configuration file.

Comments:

Justin Hileman (bobthecow) on 2013-04-11 22:35:05 UTC:

> In a similar vein, I've been thinking about ENV stuff. I currently store API keys and such in the ENV so that I don't have to commit them to version control, and so that I can compile a couple of different ways without changing my config file. It would be nice for there to be a good way to do that :)

* * *

# Support for metadata in JSON or TOML

**Status:** Ideas

**Labels:** New in nanoc 4.0

Comments:

Justin Hileman (bobthecow) on 2013-04-11 22:27:36 UTC:

> We already support JSON metadata (in frontmatter, not in standalone files) ... I'm happy to extend that support for `foo.md.json` in addition to `foo.md.yaml`. I'm not sure adding another flavor of metadata besides YAML and JSON would be worth the added complexity and support headaches.

Denis Defreyne (denisdefreyne) on 2013-04-12 06:22:26 UTC:

> If the meta filename includes the entire content filename, it would be fairly easy to detect the format of the meta filename, and use it to look up the right parser. If we do it that way, it should be easy to add other parsers.
> 
> Support for different parsers in frontmatter could be done by explicitly giving the language:
> 
> --- json
> { "people": ["bob", "alice"] }
> 
> -- ini
> [nanoc]
> version = 4
> 
> etc. I’m not serious about including an INI parser by default though. ;)

Justin Hileman (bobthecow) on 2013-04-13 00:56:58 UTC:

> But YAML is a strict superset of JSON. If you put *either* JSON or YAML in your frontmatter it'll parse properly. Between those two, I think we've got everything covered.

* * *

# Allow generating assets during compilation

**Status:** Ideas

**Labels:** New in nanoc 4.0

This could be useful when compiling LaTeX to HTML, where formulas are converted to PNGs.

* * *

# Filters that take both text and binary input

**Status:** Ideas

**Labels:** Changed in nanoc 4.0

See original discussion at https://github.com/nanoc/nanoc/issues/263.

* * *

# Symlinks in input should be symlinks in output too

**Status:** Ideas

**Labels:** Changed in nanoc 4.0, Problematic

Comments:

Denis Defreyne (denisdefreyne) on 2013-05-01 10:37:27 UTC:

> I am not sure whether there is a valid use case for this. Maybe this one should be skipped.

Justin Hileman (bobthecow) on 2013-08-11 21:29:33 UTC:

> I feel like symlinks in input should *not* be symlinks in output. That feels contrary to the way symlinks ought to work :)
> 
> Mebbe we could provide a symlink method for the Rules DSL for letting people define things that should be symlinked? That might work...

* * *

# Parse Rules file to find out patterns matching items that need recompiling

**Status:** Ideas

**Labels:** Changed in nanoc 4.0

This is necessary to go from a outdatedness detection that is O(#items) to one that is O(#changes).

This would also eliminate the need for recorder proxies.

* * *

# External stores

**Status:** Ideas

Data that is generally stored in-memory should be storable elsewhere, to decrease memory pressure on Ruby.

* * *

# DSL for generating items

**Status:** Ideas

```ruby
merge '/assets/style/*.sass' do
  input parts.map { |p| p.compiled_content }.join("\n")
  filter :rainpress
  write '/assets/style.css'
end
```

More flexible:

```ruby
generate do
  input @items['/assets/style/*.sass'].map { |i| i.compiled_content }.join("\n\n")
  filter :rainpress
  write '/assets/style.css'
end
```

* * * * *

See the asset registry here: https://github.com/stbuehler/nanoc/commits/release-3.4.x

* * * * *

nanoc and make have some similarities. They are both build systems (and it is possible to compile C code with nanoc, although it is ugly!).

Defining targets in make which can be compiled from a single file is easy. For instance:

```make
output/blog.html: source/blog.erb.html
  erb blahblah
```

This is also easily possible in nanoc. What is _not_ easily possible in nanoc, is generating files that do not have a source file, but rather consist of many different source files that are aggregated together. For instance:

```make
dist/tetrimentalist: src/foo.c src/bar.c src/qux.c
  gcc ...
```

To emulate this in nanoc, you'd need to generate a fake item in nanoc in the preprocess step.

It would be neat if you could define generated items in nanoc as well. Something like this:

```ruby
years = @items.collect { |i| i[:published_on].year }.uniq
years.each do |year|
  target "/blog/archive/#{year}/" do
    layout :archive
  end
end
```

The 'archive' layout would, in this case, not use <%= yield %> but rather serve as a partial which generates the default content for the archive page.

* * *

# True incremental compilation

**Status:** Ideas


Currently, when nanoc is interrupted during the compilation process, restarting nanoc will recompile the site as if the previous compilation run did not happen. True incremental compilation (TIC) means continuing where nanoc left off in the previous run.

The main issue that is preventing TIC from being implemented is that some outdatedness information is not treated on an item level. For example, nanoc keeps a checksum of the `nanoc.yaml` file and will consider items outdated if that file changes, but there is no way to say that an old `nanoc.yaml` applies to some files (that have not yet been recompiled) and that a new `nanoc.yaml` applies to some other files (that have already been recompiled).

In order to implement TIC, outdatedness information should be stored on a per-item basis. Each item should have an outdatedness record, containing

1. The checksum of its source file
2. The checksum of the configuration file
3. The checksums of any layouts it uses
4. The checksums of all code snippets
5. The checksums of relevant parts of the Rules file

An item should be recompiled if the entire record is different.

* * *

# Multiple content parts

**Status:** Ideas

The capturing helper currently generates (or captures) content into separate buffers. This goes against the established nanoc workflow and requires ugly workarounds. It may be nice to have the concept of separate buffers in nanoc core.

* * *

# Extract output diff generator

**Status:** Ideas

The output diff generator does not belong in `cli`. Maybe as a separate plugin? Could this be part of core?

* * *

# Querying data sources for changed items

**Status:** Ideas

It should be possible to ask a data source for all items that were changed since the last time the site was compiled. This would make it a lot quicker to incrementally build sites.

* * *

# Parallel compilation

**Status:** Ideas

nanoc needs a scheduler that can compile items in parallel.

* * *

# Deploy to multiple destinations

**Status:** Ideas

The deployers are useful, but sometimes one part of the site is deployed to one location (e.g. the site itself) while another part is deployed elsewhere (e.g. assets). Ideally, nanoc’s deployers should be able to cope with that.

* * *

# Amend config on CLI

**Status:** Ideas

Amend site configuration via command line
----------------------------------------

It would be quite useful to be able to amend the site's configuration (`@config`) with a command line option. It should be global like `--debug` or `--warn` thus affecting every nanoc command. The option's value should probably use YAML too.

One use case for this would be compiling a site for different targets like development and production. JS and CSS minification (maybe even image optimization) would only be run when `@config[:target] == 'production'`. So instead of having to edit `config.yaml` every time you change targets you could just run your nanoc command with the appropriate option:

    # Compile the site using a production setup
    nanoc compile --config="target: production"

~ _raphinesse_

Having separate environments would indeed be useful. I don't think that passing commandline configuration options is useful in general; I'd stick to environments.

Supporting multiple environments would require separate `tmp/` directories for each environment, because the `tmp/` directory contains caches that may no longer be valid.

Environments could also be useful for the `deploy` command: when deploying with target `production`, it could recompile the site for production and then upload, so that you don’t accidentally upload a staging version (with different URLs to the assets server for example).

~ @ddfreyne

---------------------------

```
bundle exec jekyll build --config _config.yml,_config_production.yml,_config_whatever.yml
```

Support multi envs by providing multiple yaml files. I think the code above explains.

btw, I think you should deprecate `deploy` subcommand just like `create-item` & `watch`, they are too fancy and non-core.

~ @clc3123

* * *

# Custom content kinds

**Status:** Ideas

nanoc currently has two kinds of content: text and binary. Having more kinds of content would be beneficial for correctness and for speed. For example:

* DOM - contains the parsed HTML as a document object model. Filters that modify HTML could act on the DOM directly, instead of doing a full parse-modify-serialize cycle.

* Kramdown internal representation - Similar to the DOM, but using Kramdown’s internal representation. Again, this would avoid a full parse-modify-serialize cycle.

* Markdown - Since this is a specific kind of text, it would not make sense to run filters on this that expect a different content kind, such as Haml.

* * *

# Filtering parts of items

**Status:** Ideas


The `colorize_syntax` filter is awkward. It should be possible to express the individual colorizers as filters, so that we end up with filters like `:pygments`, `:pygmentize`, … etc.

However, running these filters on an item as a whole is problematic: there would be no way to specify different colorisers for different languages.

It should be possible to split item content into multiple sections, that can each be processed in their own way. Something along these lines:

```ruby
compile '*' do
  filter :kramdown
  filter :detect_code_blocks do |lang, content|
    if lang == 'javascript'
      filter :pygmentize
    else
      filter :coderay
    end
  end
end
```

* * *

# Flexible data source config

**Status:** Ideas

```ruby
source :items,   :from => :filesystem, :path => '/content/**/*', :strip => /^content\//
source :layouts, :from => :filesystem, :path => '/layouts/**/*', :strip => /^layouts\//
```

This could entirely replace the data source configuration, and make the data source implementation more flexible since `layouts` and `content` are no longer hardcoded.

* * *

# Check whether output paths are unique

**Status:** Ideas

Output paths should be unique, but nanoc currently does not enforce it. It would be nice if nanoc did.

* * *

# Overrideable/extendable helpers

**Status:** Ideas

Helpers provide useful basic functionality, but they are intended to be a starting point for other, more sophisticated helpers. Allowing others to reuse the basic helpers and extend them would be useful. A standard way of extending existing functionality is subclassing, but this would require helpers to be classes.

* * *

# Data objects

**Status:** Ideas


All content data in nanoc need to be items. Data that is not part of items cannot be used by the dependency tracker.

This can be quite annoying if you want to incorporate structured data from elsewhere, because that structured data will have to be converted into an item, which can be a lot of work and can cause confusion. For example, documenting the commandline interface for nanoc would require converting commands to items.

It may be worth introducing the concept of “data objects”, which would allow arbitrary objects to be used, while still doing dependency tracking. For example:

    <!-- does not generate a dependency -->
    <%= Nanoc::CLI.root_command.name %>

    <!-- generates a dependency -->
    <%= do(Nanoc::CLI.root_command).name %>

* * *

# DRYly described config file

**Status:** Ideas

The site configuration is described in both `Nanoc::Site` and the `create-site` command. There should be only one place where the site configuration is described.

There should be a way to describe the site configuration options, the default values and the documentation cleanly. Perhaps like this?

```ruby
{
  :text_extensions =>
  {
    :description => 'Blah blah blah…',
    :default => [ 'txt', 'md' ]
  }
}
```

* * *

# Check severity

**Status:** Ideas

I have some checks that fail on some files, but they're not a big enough deal to keep me from deploying. I would like to be able to mark some deploy checks as :warning to prevent them from killing the deploy.

    deploy_check :ilinks, :warning 

... or something like that?

* * *

# Check for non-referenced files

**Status:** Ideas

It would be great to have a check that reports files that are not references anywhere within the site.

More detailed discussion: https://github.com/nanoc/nanoc/issues/227

* * *

# Sass filter with access to content

**Status:** Ideas

Issue #254 describes a custom Sass filter that provides access to the assigns (items, site, layouts, config, …). It would be useful to have this in nanoc’s default Sass filter.

* * *

# Cleaner #render method, #concat

**Status:** Ideas

`#concat` (inspired by the [ActionView method with the same name](http://api.rubyonrails.org/classes/ActionView/Helpers/TextHelper.html#method-i-concat)) concats text to the output stream (i.e. something like `def concat(s) ; _erbout << s ; end`).

As for rendering, nanoc should have one method to return the rendered string, and another method to output it (or an argument to the method to choose between those). `#render`’s behavior is weird when a block is passed.

* * *

# `nanoc doc` command

**Status:** Ideas

**Labels:** New in nanoc 4.0

Include all documentation (= the web site + code documentation) into the nanoc executable, so it can be consulted using a new “nanoc doc” command. This “nanoc doc” command can either start a web server that serves the documentation site, or it can be used to quickly retrieve information on a specific item (e.g. “nanoc doc filter erb” for viewing the erb filter documentation).

* * *

# Catch-all for representations

**Status:** Ideas

`route "...", :rep => "*"` would be really nice to clean things up. Or, in the full syntax discussed here it would be:

```ruby
route "*", :rep => "*", :if => lambda { DATA_TYPES.include?(item[:kind]) } do
  nil
end
```

… just to have one rule that nukes all of my internal date types.

~ @coderanger

* * *

# Make adding items not cause entire site to be recompiled

**Status:** Ideas

Adding dependencies from all items onto the new items is a correct solution, but not optimal. Perhaps only items that access @items should be considered relevant for this. ~ @ddfreyne

* * *

# Command categories

**Status:** Ideas

A “core” category would include commands such as `create_site` and `compile`, while `validate_links` would be parts of an “extra” category.

* * *

# Improved dependency tracking

**Status:** Ideas

nanoc’s dependency tracking mechanism is quite useful in keeping the compilation speed down in most cases. However, it fails in a few distinct cases.

* Cases requiring total recompilation of the site
  * A new item is added
  * A new layout is added
  * The site configuration is updated
  * Code is added to lib/
* Cases requiring significant recompilation
  * An item depends on only a single, virtually never-changing attribute of an item or layout whose content or other attributes change regularly

This page tracks problems with the dependency generation in the current implementation as well as suggestions for improvements. **Woefully incomplete, but I’ll add stuff to this soon!**

* * *

# GUI

**Status:** Ideas

My intention is to build a graphical user interface for nanoc sooner or later. Time is too sparse for me to actively work on it, but that doesn’t prevent me from collecting ideas and brainstorming about the concept.

Goal
----

The GUI would be aimed at people who are responsible for managing content (creating and updating pages) of an existing nanoc web site. It is not aimed at web developers or even designers.

Features
--------

* Interface for creating, updating, deleting, renaming pages
* Item “blueprints” to define how an item should behave (blog article, blog archive, homepage, review, …)
* Version history of everything (likely using git)

### Blueprints example

A site administrator could define a “review” item blueprint, which adds the following constraint to review items:

* The identifier is /reviews/<something>/
* The item needs the name of the product to review
* The item needs a rating (an integer from 1 to 5)
* The item needs an author (default = logged in user)
* The item needs a review date
* The item needs a one-line summary (could be used as a title as well)

Links
-----

Some useful web-based editors:

* [Create](http://createjs.org/)
* [Copycopter](https://github.com/copycopter/copycopter-server)
* [CodeMirror](http://codemirror.net/) - mostly for code, rather than content.  Supports HTML and Markdown.
* [Prose](http://prose.io/) - A hosted editor for github.com content based on *CodeMirror*.  Has support for Jekyll's YAML front matter so it should be possible to patch to support nanoc attributes equally well.

* * *

# Allow calling filtering helper with a string rather than a block

**Status:** Ideas

* * *

# Fine-grained dependency tracking

**Status:** Ideas

**Labels:** Changed in nanoc 4.0

See https://github.com/nanoc/nanoc/issues/541

Related:

* [Efficient outdatedness checking](https://trello.com/c/AVIuz24Z/33-efficient-outdatedness-checking)
* [Improved dependency tracking](https://trello.com/c/2pd0LJ9R/64-improved-dependency-tracking)
* [Make adding items not cause entire site to be recompiled](https://trello.com/c/19zdXx7e/62-make-adding-items-not-cause-entire-site-to-be-recompiled)

* * *

# Ruby configuration file

**Status:** Planned

**Labels:** Changed in nanoc 4.0

We should add nanoc.rb as a configuration option. This would allow more complicated, environment specific configuration.

It would probably look something like this: https://gist.github.com/bobthecow/7432cfd7f2e91a794e5f

It would allow setting third-party authentication configuration via environment variables rather than hardcoding and committing them to revision control. It would allow appending to default (like text_extensions) rather than always with the copypasta. It would allow all sorts of awesome :)

Comments:

Bill Burton (bburton333) on 2013-11-17 02:45:34 UTC:

> A lot of nanoc sites use compass. Right now, compass integration is rather inconvenient as the paths in config.rb have to be in sync with the compile and route in Rules. 
> 
> It would be even better if the config file was named something like config.rb that the compass command also recognizes by default.  This could enable strong integration between nanoc and compass which is not the case now.  
> 
> If the nanoc create-site command generated a correctly configured config.rb that worked with compass it would make things much easier for people.  As a compass config.rb does not actually depend on compass itself, having a config.rb does not make nanoc dependent on compass.
> 
> The end result would be that nanoc would use the config.rb and when running the compass command, it would recognize the settings it needed and make it easy to install compass plugins into a nanoc site.

* * *

# Explicit lib/ directory structure

**Status:** Planned

**Labels:** Changed in nanoc 4.0

The fact that the `lib/` directory structure is freeform, gives rise to problems when optimising: due to the unpredictability of changes in `lib/`, **any change to `lib/` will require the entire site to be recompiled**. For large sites, this is obviously a large drawback, which fortunately should be resolvable in a few ways.

First, the `lib/` directory needs to have an explicit structure. More specifically:

1. `lib/helpers/` for helpers
2. `lib/filters/` for filters
3. `lib/commands/` for commands
4. `lib/data_sources/` for data sources

These restrictions are quite strict, e.g. nanoc will refuse to compile a site if a filter is defined in `lib/helpers/`.

The advantage of this approach is that a filter can be matched to a single source file, so if a single source file changes, its impact is known. For example, if `lib/filters/foo.rb` changes and only one item is filtered using the `foo` filter, only that item will be recompiled.

For helpers, the situation is a bit more difficult, because helpers will need to be activated explicitly on a per-item or per-rule basis. Perhaps something like this:

```ruby
compile '/' do
  helper :home # this would load lib/helpers/home.rb
  filter :erb
end
```

Comments:

Justin Hileman (bobthecow) on 2014-03-23 14:53:10 UTC:

> I don't know that I like requiring rules to define what helpers they're using. That adds more cognitive load than necessary when writing and reading rules. I'd rather have large sites recompile completely if anything in `helpers/` changed, personally.

* * *

# Postprocess block

**Status:** Planned

**Labels:** New in nanoc 4.0

A postprocess block would be useful for triggering further actions, such as feeding a search index.

Comments:

Jean-Denis Vauguet (jeandenisvauguet) on 2014-03-22 16:45:12 UTC:

> Postprocessing would be useful for triggering *external* processing of compiled data, such as feeding a database indexer (SolR, Elasticsearch, Indextank, whatever).

* * *

# Configurable `tmp`, `Rules` and `Checks` paths

**Status:** Planned

**Labels:** Changed in nanoc 4.0

See original discussion at https://github.com/nanoc/nanoc/issues/259

Comments:

Justin Hileman (bobthecow) on 2014-03-23 14:51:27 UTC:

> Along with this, maybe remove the automatic Rules/rules/Rules.rb/rules.rb check and just make users configure their filename if they want something besides Rules?

* * *

# Extract filesystem data source

**Status:** Planned

* * *

# Extract ERB filter

**Status:** Planned

* * *

# Add Coveralls support to all repositories

**Status:** Planned

* * *

# Add Code Climate support to all repositories

**Status:** Planned

* * *

# Add Travis support to all repositories

**Status:** Planned

* * *

# Extract plugins

**Status:** Doing

**Labels:** Changed in nanoc 4.0

(duplicate)

* * *

# Add CodeClimate to all repos

**Status:** Doing

* * *

# Add Travis CI to all repos

**Status:** Doing

* * *

# Extract deployers

**Status:** Done

* * *

# Split nanoc

**Status:** Done

**Labels:** Changed in nanoc 4.0

See https://github.com/nanoc/nanoc/wiki/4.0-Development%3A-Subprojects

Comments:

Justin Hileman (bobthecow) on 2013-08-11 21:29:46 UTC:

> :+1:

Denis Defreyne (denisdefreyne) on 2013-09-28 15:33:16 UTC:

> Checks are combined in a single repo. This will likely be split in different repos later (`nanoc-check-html` etc).

* * *

# Always auto-prune

**Status:** Done

**Labels:** Changed in nanoc 4.0

Comments:

Justin Hileman (bobthecow) on 2013-04-11 22:33:35 UTC:

> My concerns from before still stand.

* * *

# Clean up content accessing methods

**Status:** Done

**Labels:** Changed in nanoc 4.0

`raw_filename`, `raw_path`, `path`, `content`, `compiled_content`, `raw_content`, …

Some of them are for binary items only, some for textual items only, some for both.

* * *

# @items should not be an array

**Status:** Done

**Labels:** Changed in nanoc 4.0

It should be a collection, not an array.

* * *

# Remove static data source

**Status:** Done

**Labels:** Removed from nanoc 3.x

The static data source will be redundant once identifiers act the same as filenames.

* * *

# Remove watch command

**Status:** Done

**Labels:** Removed from nanoc 3.x

* * *

# Meta filename should contain the entire content filename

**Status:** Done

**Labels:** Changed in nanoc 4.0

Currently, when a content file also has a matching meta file, the meta file has the same *basename*, like this:

* `content/foo.md`
* `content/foo.yaml`

nanoc 4.x could require the meta filename to contain the entire filename, *not* just the basename, like this:

* `content/foo.md`
* `content/foo.md.yaml`

This would make the relationship between the items more clear.

Comments:

Justin Hileman (bobthecow) on 2013-04-11 22:25:16 UTC:

> Yeah, I like this a lot.

* * *

# Remove all deprecated code

**Status:** Done

**Labels:** Removed from nanoc 3.x

* * *

# Remove create-item and create-layout

**Status:** Done

**Labels:** Removed from nanoc 3.x

These commands serve no real purpose, because creating items and layouts is trivial anyway. This will also prevent the question about generating files with custom extensions from being asked.

* * *

# Remove autocompile command

**Status:** Done

**Labels:** Removed from nanoc 3.x

* * *

# Remove Bundler support

**Status:** Done

**Labels:** Removed from nanoc 3.x

Letting nanoc load Bundler can lead to annoying issues (such as other gems not being found). It may be better to recommend using `bundle exec nanoc` instead.

Comments:

Justin Hileman (bobthecow) on 2013-04-11 22:31:30 UTC:

> Maybe as a concession we could do what Guard does, and detect when someone is inside a bundle but didn't run via bundle exec or binstubs or whatever:
> 
> 15:31:06 - INFO - Guard here! It looks like your project has a Gemfile, yet you are running
> > [#] `guard` outside of Bundler. If this is your intent, feel free to ignore this
> > [#] message. Otherwise, consider using `bundle exec guard` to ensure your
> > [#] dependencies are loaded correctly.
> > [#] (You can run `guard` with --no-bundler-warning to get rid of this message.)

Denis Defreyne (denisdefreyne) on 2013-04-12 06:12:49 UTC:

> That already happens: https://github.com/nanoc/nanoc/commit/24efb5f5ff492529bbe438f9b5b1c051ac8003e1 :)

* * *

# Do not infer encoding from environment

**Status:** Done

**Labels:** Changed in nanoc 4.0

The input encoding should always be explicitly given, and not inferred from the environment.

* * *

# Remove update command

**Status:** Done

**Labels:** Removed from nanoc 3.x

* * *

# Keep only filesystem_unified data source

**Status:** Done

**Labels:** Removed from nanoc 3.x

Comments:

Justin Hileman (bobthecow) on 2013-04-11 22:32:58 UTC:

> Change it to `filesystem` then? :)

Justin Hileman (bobthecow) on 2013-04-11 23:05:00 UTC:

> Ahh. You already did that. Awesome.

* * *

# Remove Ruby 1.8.x support

**Status:** Done

**Labels:** Removed from nanoc 3.x

* * *

# Globs in rules

**Status:** Done

**Labels:** Changed in nanoc 4.0

Comments:

Justin Hileman (bobthecow) on 2013-04-11 22:29:59 UTC:

> Very yes.
> 
> I'd like to see identifiers change to actual filenames, in the process. The almost-map that currently exists mostly just confuses people. Treating them like filenames, with the obvious exception of metadata files, would play well with globbing.
> 
> My vote is for rules to accept both regexp and glob strings, and match against something like `/foo/bar.md` if the content file is `content/foo/bar.md`.

Denis Defreyne (denisdefreyne) on 2013-04-12 06:19:37 UTC:

> Yeah, I’m probably going for filenames everywhere. Doing so will need a few changes in different places:
> 
> * Finding items by identifiers (@items['/foo/']) no longer directly works, but it should be fairly easy to let it support globs there (e.g. @items['/foo.*']). That means it would return an array instead of a single element, though.
> 
> * Rules matching, obviously

Justin Hileman (bobthecow) on 2013-04-13 00:53:16 UTC:

> I'm not a huge fan of globs inside ['...']
> 
> I realize Dir does that, but it always kinda rubs be wrong :)

Denis Defreyne (denisdefreyne) on 2013-04-14 08:46:57 UTC:

> What do you suggest instead? @items.find('/foo.*')? That would override the existing #find method on Enumerable though.

Stefan Bühler (stbuehler) on 2013-04-15 09:47:28 UTC:

> #glob for finding all matching items, #find to find the first/shortest? match. #find can be overloaded, the Enumerable#find takes only a "ifnone" Proc parameter, not a string.

* * *

# Merged compilation and routing rules

**Status:** Done

**Labels:** New in nanoc 4.0

See original description at https://github.com/nanoc/nanoc/issues/206

* * *

# Do not use #inspect to serialize rules

**Status:** Done

**Labels:** Changed in nanoc 4.0

That said, neither Sass::Importers::Filesystem nor Compass::SpriteImporter implements #inspect and the stock Object#inspect prints the importers' memory address in the result. That precise behavior confuses nanoc which believes the Rules file has changed since the last Sass+Compass item compile; because the "inspect dump differs by the importer's memory address"

(See https://github.com/chriseppstein/compass/issues/1260)

* * *
