**THIS PAGE HAS BEEN ARCHIVED.**

nanoc’s documentation is pretty good, but there are several areas that could certainly be improved. This wiki page is meant to collect feedback about the current documentation and ideas for new, improved documentation.

Please do add your own thoughts. If you add something or vote for something, be sure to add your GitHub username.

## Current documentation

### Good

  * Everything is documented ~ @ddfreyne

### Bad

  * Not everything is documented in the same place. Some parts are documented in the API documentation (filters and helpers) and some on the commandline (commands). ~ @ddfreyne

## Ideas

  * Add site-wide search
    * +1 @ddfreyne
    * +1 @bzerangue
  * Add screencasts
    * +1 @ddfreyne
    * +1 @bzerangue
  * Document filters and helpers on the site, not in the API documentation
    * +1 @ddfreyne (not sure how easy this is for helpers)
    * +1 @bzerangue
  * Document commands on the site, not just on the commandline
    * +1 @ddfreyne
    * +1 @bzerangue
  * Split the documentation in a book and in a reference. The book should be readable from start to finish, while the reference is very detailed information about one specific subject.
    * +1 @ddfreyne
    * that's related to the point above about filters: the book should link to the reference (@cdlm)
    * +1 @bzerangue
  * Add a small 'cookbook' section that shows how to do common tasks such as having drafts, adding compass, etc.
    * +1 @zmanji
    * This already exists in the form of guides on the nanoc site, but it could (should) certainly be expanded. @ddfreyne
  * Make the documentation as a book, which should be available in different formats (HTML and PDF)
    * +1 @ddfreyne
  * Include all documentation (= the web site + code documentation) into the nanoc executable, so it can be consulted using a new “nanoc doc” command. This “nanoc doc” command can either start a web server that serves the documentation site, or it can be used to quickly retrieve information on a specific item (e.g. “nanoc doc filter erb” for viewing the erb filter documentation).
    * +1 @ddfreyne

## New documentation hierarchy

### Book

1. **Preface**
    1. What nanoc is, how it came to be, what makes it special
    2. How is this book structured, how it should be read, what conventions are used
2. **Getting Started**
    1. **Installing nanoc** - ruby, rubygems, nanoc, bundler, ...
    2. **What you need** - Knowledge of the commandline, knowledge of Ruby
    3. **Tutorial part I** - the existing one, perhaps trimmed down and with more references to later subjects
    4. **Tutorial part II**
    5. ...
3. **Using nanoc**
    1. **What’s in a site?** - The directory structure
    2. Configuration
    3. The command line
    4. Items
    5. Layouts
    6. Rules & the compilation process
    7. **Filters** - what are they, which ones are available + links to their reference documentation
    8. **Helpers** - what are they, which ones are available + links to their reference documentation
4. **Extending nanoc**
    1. Data sources
    2. Custom filters
    3. Custom helpers
    4. Custom commands
5. **Guides**
    1. Deploying sites
    2. Paginating articles
    3. Integrating with Compass
    4. Using filters on file extensions
    5. Using binary items effectively
    6. Creating multilingual sites
6. **Glossary**

### Reference

* Command reference
* Configuration reference
* Filter reference
* Helper reference

## Inspiration

* [git-scm.com](http://git-scm.com/) - They might say that git is not user friendly, but the site is surprisingly nice! I especially like the search functionality, where topics are grouped by section.
* [The Hg book](http://hgbook.red-bean.com/read/)
* [The Sinatra book](http://sinatra-book.gittr.com/)
* [The Padrino guides](http://www.padrinorb.com/guides/home)

## Cool software

* [Swiftype](http://swiftype.com/) - a nice, embeddable search engine

## References

* [Writing great documentation](http://jacobian.org/writing/great-documentation/)