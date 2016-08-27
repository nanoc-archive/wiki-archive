**THIS PAGE HAS BEEN ARCHIVED.** The wishlist is no longer maintained on the nanoc wiki; see the [Trello board](https://trello.com/b/dlEWOOBW/nanoc-4-0) instead.

This page collects resolved and obsolete wishlist items.

Item Templates
--------------

**Status**: Resolved

I looked at the old nanoc2 source, and thought of a better way to handle templates. Two main things I didn't like about the old method were:

* There was no way to generate the item attributes dynamically.
* It used yet another data type in addition to pages, assets, layouts, and code snippets.

So, the idea I came up with was having the templates in code snippets. This solves both of the problems, and it lets you define multiple templates in each code snippet. A `define_template` method would be available in the code snippet namespace, and would work like this:

```ruby
define_template :template_name do |path|
  return [ { :meta => :attributes }, content ]
end
```

When `nanoc3 create_item --template=template_name /item/path/` was run, the block would be called with the item's path as its first argument. It wouldn't be allowed to change the item's path, though. In addition, if any extra arguments were passed (like `nanoc3 ci -t template_name /item/path/ foo bar baz`), they would also be passed to the block (in this case, the block's arguments would be `[ '/item/path/', 'foo', 'bar', 'baz' ]`). The block should return an array, the first item of which is the item's metadata and the second item is the actual content. nanoc3 will then go ahead and create the item. On the other hand, if a `Nanoc3::TemplateError` is raised (as in, `raise Nanoc3::TemplateError.new("Must specify an argument for the title")`), its message will be displayed to the user and the item will not be created.

A blog site could use this template:

```ruby
define_template :article do |path, title|
  raise Nanoc3::TemplateError.new("Please specify a post title after the item path") if title.nil?
  return [ {
    :kind => 'article',
    :created_at => DateTime.now,
    :title => title
  }, "A new post." ]
end
```

~ _LeafStorm_

It is somewhat easy to do this with Rake, although not terribly so. Passing commandline arguments (such as the page title) is possible using environment variables. Something like …

	rake create_item title="Hello World" path="/blog/2010/hello-world/"

… is possible using a rake task like this:

```ruby
task :create_item do
  site = Nanoc3::Site.new('.')
  site.load_data

  site.data_sources[0].create_item(
    "default content goes here",
    { :title => ENV['title'] },
    ENV['path']
  )
end
```

This can be improved a bit: for example, templates could then be defined using a `define_template` method that generates tasks like the one above.

Something like [Thor](http://yehudakatz.com/2008/05/12/by-thors-hammer/) may also be quite useful here, or perhaps even nanoc3’s built-in command line interface (with commands that inherit from `Cri::Command`. In fact, I believe that nanoc shouldn’t necessarily need to provide support for templates, but it should provide an API and a CLI that allow such features to be added.

~ _Denis Defreyne_

Easy way to render one item inside another
------------------------------------------

**Status**: Resolved

That is, items as partials. The following seems unwieldy (although of course it can be wrapped in a helper)

```ruby
someitem.reps.find { |rep| rep.name == :default }.content_at_snapshot(:last)
```

Maybe have Item should have `#to_s`: `def to_s(rep = :default)`. User still need a suitable compile rule to ensure no layout is applied, and need to set `is_hidden` property.

Finding a child by name is also awkward:

```ruby
child = @item.children.find { |c| c.identifier == @item.identifier + name + '/' }
```

Perhaps want Item#child method. If we have identifiers as paths, then we might also want to find sibling, e.g. /foo/about.html -> /foo/bar.css

~ _(unknown)_

This will be partially resolved in nanoc 3.1. Getting the content of an item rep will be easier. Compare the old and the new ways:

```ruby
# old
someitem.reps.find { |rep| rep.name == :default }.content_at_snapshot(:pre)
someitem.reps.find { |rep| rep.name == :default }.content_at_snapshot(:last)
someitem.reps.find { |rep| rep.name == :moo }.content_at_snapshot(:pre)

# new
someitem.compiled_content
someitem.compiled_content(:snapshot => :last)
someitem.compiled_content(:rep => :moo)
```

Finding a child by name could indeed be a bit cleaner. Perhaps a `Nanoc3::Identifier` class could be useful here. Such a class would make the code look more natural, e.g.:

```ruby
child = @item.children.find { |c| c.identifier.components[-1] == name }
```

~ _Denis Defreyne_

create_code_snippet DataSource method
---------------------------------------

**Status**: Resolved

This method could be implemented by DataSources, and it would create a new code snippet. The signature would be `create_code_snippet(content, filename)` - for example,

```ruby
data_source.create_code_snippet(
  "\# All files in the lib/ directory will automatically " +
  "\# be loaded by nanoc when compiling the site.",
  'default.rb'
)
```

~ _LeafStorm_

In 3.0.x, all data sources (at least the built-in ones) load code from `.rb` files in the `lib/` directory. In nanoc 3.1.x, code snippets will no longer be managed by the data source, but will be loaded directly from `lib/` by the `Nanoc3::Site` class. It is therefore probably easier to manually create code snippets:

```ruby
File.open('lib/default.rb', 'w') do |io|
  io.write '# All files in the lib/ directory will automatically'
  io.write '# be loaded by nanoc when compiling the site.'
end
```

~ _Denis Defreyne_

Make compile the default command
--------------------------------

**Status**: Resolved

Makes it slightly easier to compile. ~ _ddfreyne_, _VitamineD_

Convenience helpers for default route idioms
--------------------------------------------

**Status**: Resolved (nanoc 4.0)

Often, routes just set the extension based on the kind of item :

```ruby
route '/stylesheets/*/' do
  item.identifier.chop + '.css'
end
```

I suppose there are a few other cases like that, but here a quick extension: 'css' rather than chopping and appending to the identifier explicitly would be quite nice.

~ _cDlm_

The routing rules could indeed use some cleanup. Do you mean something along the lines of

```ruby
route '/stylesheets/*/' do
  extension 'css'
  in_directory false
end

route '*' do
  extension 'html'
  in_directory true
end
```
The former would cause the file extension to be “css” and it would also prevent a directory from being generated (so no /style/index.css but just /style.css). The latter would set the extension to “html” and the compiled files would be put in a separate directory (so /about/index.html instead of /about.html). What do you think?

~ _Denis Defreyne_

By the way, you can also use item[:extension] to get the extension of the original item. The following rule would be more general:

```ruby
route '/stylesheets/*/' do
  item.identifier.chop + item[:extension]
end
```

~ _Denis Defreyne_

Yes indeed. In fact I have this in default.rb:

```ruby
# Route by setting the extension
def extension(ext=nil)
  e = ext || item[:extension] || ''
  e = ".#{e}" unless e.empty?
  id = item.identifier.chop
  id = "/index" if id.empty?
  id + e
end
```

My point is rather to provide a convenient method to do that out of the box and make routing rules more abstract and readable.

~ _cDlm_

With merged compilation and routing rules (which will arrive in a future version some time), this probably needs to look like this:

```ruby
compile '/foo/*/' do
  # Write /foo/abc/ to /foo/abc.ext
  write :in_directory => false
end

compile '/bar/*/' do
  # Same as above
  write item.identifier.chop + (item[:extension] ? '.' + item[:extension] : '')
end
```

~ _Denis Defreyne_

Rule passing
------------

**Status:** Rejected (deemed unnecessary)

It would be nice to have a method in the compile/route rule DSL, nominally #pass, which aborts processing the current rule body and resumes matching at the next rule. An example of using this is routing binary items:

```ruby
route '*' do
  pass unless item.binary?
  item.identifier.chop
end

route '/...' do
end
```

~ _coderanger_

I know Sinatra has such a feature for their rules (also called `#pass`) but I have never found a good use case for it in Sinatra nor nanoc. Can you provide an example where this would be useful?

~ @ddfreyne

As discussed on IRC, I believe this approach has two disadvantages:

1. This would only be syntactic sugar and would therefore add unnecessary complexity; and
2. it will make routing rules harder to understand, because there could be more than one matching routing rule, meaning you’d have to read them all to know which one handles a given item.

An alternative would be to provide a conditional block to rules, like this:

```ruby
route '*', :if => ->{ item[:kind] == 'article' } do
  # ... code here ...
end
```

~ @ddfreyne

Wordpress data source
---------------------

**Status:** Rejected (deemed not useful)

Probably hard to accomplish, but here we go anyway: write a data source that reads Wordpress items and layouts so that a Wordpress site can be compiled with nanoc. This would require some way of executing PHP code in Ruby, perhaps using [Phuby](https://github.com/tenderlove/phuby).

~ @ddfreyne

Importers
---------

**Status:** Rejected (deemed not useful enough to implement now)

Importers from WordPress, Jekyll, … would be quite useful.

~ @ddfreyne