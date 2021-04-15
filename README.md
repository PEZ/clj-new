# clj-new

Generate new projects from Leiningen or Boot templates, or `clj-template` projects, using just the `clojure` command-line installation of Clojure!

For support, help, general questions, use the [#clj-new channel on the Clojurians Slack](https://app.slack.com/client/T03RZGPFR/C019ZQSPYG6).

## Getting Started

> Note: these instructions assume you are using the Clojure CLI version 1.10.1.727 or later! See [Clojure Tools Releases](https://clojure.org/releases/tools) for details about the functionality in recent CLI releases.

The easiest way to use `clj-new` is by adding an alias to your `~/.clojure/deps.edn` file (or `~/.config/clojure/deps.edn` file) like this:

```clj
    ;; add this inside your :aliases map:
    :new {:extra-deps {com.github.seancorfield/clj-new
                         {:mvn/version "1.1.297"}}
            :exec-fn clj-new/create
            :exec-args {:template "app"}}}
```

A minimal, complete `deps.edn` file with just this `:new` alias would look like this:

```clj
{:aliases
 {:new {:extra-deps {com.github.seancorfield/clj-new {:mvn/version "1.1.297"}}
        :exec-fn clj-new/create
        :exec-args {:template "app"}}}}
```

Now you can create a basic application:

```bash
    clojure -X:new :name myname/myapp
    cd myapp
    clojure -M -m myname.myapp
```

Run the tests:

```bash
    clojure -M:test:runner
```

or you can create a basic library:

```bash
    clojure -X:new :template lib :name myname/mylib
    cd mylib
```

Run the tests:

```bash
    clojure -M:test:runner
```

If you think you are going to be creating more libraries than applications, you could specify `:template "lib"` in the `:exec-args` hash map, to specify the default. Or you could provide different aliases, such as:

```clj
      ;; add these into your :aliases map:
      :new-app {:extra-deps {com.github.seancorfield/clj-new
                             {:mvn/version "1.1.297"}}
                :exec-fn clj-new/create
                :exec-args {:template "app"}}
      :new-lib {:extra-deps {com.github.seancorfield/clj-new
                             {:mvn/version "1.1.297"}}
                :exec-fn clj-new/create
                :exec-args {:template "lib"}}}
```

Now you can use those as follows:

```bash
    clojure -X:new-app :name myname/myapp
    clojure -X:new-lib :name myname/mylib
```

The following `:exec-args` can be provided for `clj-new/create`:

* `:name` -- the name of the project (as a symbol or a string); required; must be a qualified project name or a multi-segment dotted project name
* `:template` -- the name of the template to use (as a symbol or a string); required
* `:args` -- an optional vector of strings (or symbols) to pass to the template itself as command-line argument strings
* `:edn-args` -- an optional EDN expression to pass to the template itself as the arguments for the template; takes precedence over `:args`; nearly all templates expect a sequence of strings so `:args` is going to be the easiest way to pass arguments
* `:env` -- a hash map of additional variable substitutions in templates (see [The Generated `pom.xml` File](#the-generated-pomxml-file) below for a list of "built-in" variables that can be overridden)
* `:force` -- if `true`, will force overwrite the target directory if it exists
* `:help` -- if `true`, will provide a summary of these options as help
* `:output` -- specify the project directory to create (the default is to use the project name as the directory)
* `:query` -- if `true`, instead of actually looking up the template and generating the project, output an explanation of what `clj-new` will try to do
* `:snapshot` -- if `true`, look for -SNAPSHOT version of the template (not just a release version)
* `:verbose` -- 1, 2, or 3, indicating the level of debugging in increasing detail
* `:version` -- use this specific version of the template

Unlike Leiningen, `clj-new` requires that you use either a qualified
name for your project, such as `<username>/<project-name>` or
`<org-name>/<project-name>`, or a dotted name, such as `my.project`.

If you are going to publish a library, it will have a group ID and an artifact ID (e.g.,
`com.github.seancorfield/clj-new`) and the group ID should be something unique to you or your
organization -- most people use their GitHub username or their company name (i.e., their domain
name in reverse, e.g., `com.stuartsierra/component`). The qualified name you provide to
`clj-new` is effectively `group/artifact` (but keep reading!). `clj-new` uses that to create the main namespace:
`src/group/artifact.clj` containing `(ns group.artifact ...)` -- this ensures that when someone
uses your library, it's not going to clash with other code because the first portion of the
namespace should be something unique to you or your organization.

If you plan on publishing your library to [clojars.org](https://clojars.org) your project
should have a group ID that follows the [Clojars Verified Group Names policy](https://github.com/clojars/clojars-web/wiki/Verified-Group-Names).
If you use `myname/mylib` as your project name, `clj-new` will generate a `pom.xml` file
with a group ID of `net.clojars.myname` and assume the library source will live at
`https://github.com/myname/mylib` (so that `clojure -X:depstar` and `clojure -X:deps-deploy`
will "do the right thing" by default). The main namespace will be `myname.mylib`,
in `src/myname/mylib.clj`. See [The Generated `pom.xml` File](#the-generated-pomxml-file)
below for more details about group and artifact IDs.

If you use `com.github.myname/mylib` as your project name, `clj-new` will use `com.github.myname`
as the group ID and `mylib` as the artifact ID but will use `myname.mylib` as the primary
namespace, in `src/myname/mylib.clj`, rather than `src/com/github/myname/mylib.clj`
(and `com.github.myname.mylib`). You should think about whether `myname.mylib` is unique
enough that users of your library will not encounter conflicts with namespaces in other
libraries. `clj-new` also understands `io.github.myname`, `com.gitlab.myname`, and
`io.gitlab.myname` in project names.

It's good practice to follow this convention even if you are creating an application, or a
library that you don't plan to publish, because it will mean that your code is much less
likely to clash with any libraries your code uses.

If you're unsure about how `clj-new` will compute the group, artifact, main namespace, and so on,
you can use the `:query true` option, and `clj-new` will print out what it will do:

```clj
$ clojure -X:new :query true :name myname/myproj
Will create the folder: myproj
From the template: app
The following substitutions will be used:
{:date "2021-03-02",
 :group "net.clojars.myname",
 :name "myproj",
 :sanitized "myproj",
 :year 2021,
 :scm-domain "github.com",
 :template-nested-dirs "{{nested-dirs}}",
 :artifact "myproj",
 :developer "Seanc",
 :nested-dirs "myname/myproj",
 :version "0.1.0-SNAPSHOT",
 :namespace "myname.myproj",
 :user "seanc",
 :scm-user "myname",
 :raw-name "myname/myproj"}
```

You can use the `:env` option to pass in a hash map of substitutions to override any of these.

> Note: `lein new myapp` will treat `myapp` as both the group ID and the artifact ID, which is why a lot of older Clojure libs just have an unqualified lib name, like `ring` -- but really it's `ring/ring` and recent versions of the Clojure CLI display a deprecation warning on just `ring`: see the **Deprecated unqualified lib names** section near the end of this [Inside Clojure post about the `-X` option](https://insideclojure.org/2020/07/28/clj-exec/). Leiningen also stuck `.core` onto your project name to create the main namespace, which is why a lot of older Clojure libs have a `something.core` namespace: just because Leiningen did that by default. Both default behaviors here are bad because they're likely to lead to conflicts with other libraries.

You can get something close to Leiningen's default behavior by specifying a dotted project
name that ends in `.core`, e.g., `clojure -X:new :template lib :name foo.core`. That will create a folder
called `foo.core` and the project will have a group ID of `net.clojars.foo` and an artifact ID of
`foo.core` -- which is similar behavior to running `lein new foo.core` but complies with Clojars'
Verified Group Names policy -- with a main namespace of `foo.core` in `src/foo/core.clj`.
_[`lein new foo` creates a folder called `foo` and the project has a group ID of `foo` and an artifact ID of `foo`, even though the main namespace will be `foo.core`]_.

### Templates

Built-in templates are:

* `app` -- A minimal Hello World! application with `deps.edn`. Can run it via `clojure -M -m` and can test it with `clojure -M:test:runner`.
* `lib` -- A minimal library with `deps.edn`. Can test it with `clojure -M:test:runner`.
* `polylith` -- A minimal [Polylith](https://polylith.gitbook.io/) workspace with a minimal application project and a minimal library project (new in 1.1.293).
* `template` -- A minimal `clj-new` template.

> Note: you can currently find third-party templates on Clojars using these searches [`<template-name>/clj-template`](https://clojars.org/search?q=artifact-id:clj-template%2A), [`<template-name>/lein-template`](https://clojars.org/search?q=artifact-id:lein-template%2A) or [`<template-name>/boot-template`](https://clojars.org/search?q=artifact-id:boot-template%2A).

As noted above, the project name should be a qualified Clojure symbol, where the first part is typically your GitHub account name or your organization's domain reversed, e.g., `com.acme`, and the second part is the "local" name for your project (and is used as the name of the folder in which the project is created), e.g., `com.acme/my-cool-project`. This will create a folder called `my-cool-project` and the main namespace for the new project will be `com.acme.my-cool-project`, so the file will be `src/com/acme/my_cool_project.clj`. In the generated `pom.xml` file, the group ID will be `com.acme` and the artifact ID will be `my-cool-project` -- following this pattern means you are already set up for publishing to Clojars (or some other Maven-like repository).

An alternative is to use a multi-segment project name, such as `com.acme.another-project`. This will create a folder called `com.acme.another-project` (compared to above, which just uses the portion after the `/`). The main namespace will be `com.acme.another-project` in `src/com/acme/another_project.clj`, similar to the qualified project name above. In the generated `pom.xml` file, the group ID will be the "stem" of the project name (`com.acme`) and the artifact ID will be the full project name (`com.acme.another-project`) -- again, you'll be set up for publishing to Clojars etc, but be aware of the difference between how dotted names and qualified names affect the generated project.
As noted above, you can override any of these subsitutions using the `:env` option, if you need to.

```clj
$ clojure -X:new :query true :name com.acme.another-project
Will create the folder: com.acme.another-project
From the template: app
The following substitutions will be used:
{:date "2021-03-02",
 :group "com.acme",
 :name "com.acme.another-project",
 :sanitized "com.acme.another_project",
 :year 2021,
 :scm-domain "github.com",
 :template-nested-dirs "{{nested-dirs}}",
 :artifact "com.acme.another-project",
 :developer "Seanc",
 :nested-dirs "com/acme/another_project",
 :version "0.1.0-SNAPSHOT",
 :namespace "com.acme.another-project",
 :user "seanc",
 :scm-user "com.acme",
 :raw-name "com.acme.another-project"}
 ```

You can, of course, modify the generated `pom.xml` file to have whatever group and artifact ID you want, if you don't like these defaults.

#### The `app` Template

The generated project is an application. It has a `-main` function in the main project
namespace, with a `(:gen-class)` class in the `ns` form. In addition to being able to
run the project directly (with `clojure -M -m myname.myapp`) and run the tests, you can
also build an uberjar for the project with `clojure -X:uberjar`, which you can then
run with `java -jar myapp`.

The generated project includes a `pom.xml` file purely for "good hygiene". It will be
kept in sync with `deps.edn` automatically whenever you run `clojure -X:uberjar` to build
the application and it will be added to the JAR file, along with a generated `pom.properties`
file. If you delete `pom.xml`, you will also need to remove `:sync-pom true` from the
`:exec-args` for `depstar` in the `deps.edn` file.

#### The `lib` Template

The generated project is a library. It has no `-main` function. In addition to
being able to run the tests, you can also build a jar file for deployment
with `clojure -X:jar`. You will probably need to adjust some of the information
inside the generated `pom.xml` file before deploying the jar file.

The generated project includes a `pom.xml` file on the assumption that you will be deploying
the library to Clojars or a similar repository. It will be kept in sync with `deps.edn`
automatically whenever you run `clojure -X:jar` to build the library and it will be added
to the JAR file, along with a generated `pom.properties` file. If you do not intend to
deploy the library and you want to delete the `pom.xml` file, you will also need to
remove `:sync-pom true` from the `:exec-args` for `depstar` in the `deps.edn` file.

If you are going to deploy the library, you'll probably want to review and adjust some
of the fields in the `pom.xml` (developer information, group/artifact, version, SCM,
licensing etc) -- although the defaults should mostly be suitable out of the box.

Once you've updated the `pom.xml` file, you can install it locally with
`clojure -X:install` or deploy it to Clojars with `clojure -X:deploy`. For
that you need these environment variables set:

* `CLOJARS_USERNAME` -- your Clojars username
* `CLOJARS_PASSWORD` -- your Clojars password

#### The `polylith` Template

Whilst you can create a new Polylith workspace with the `poly create workspace` command,
that produces a completely empty workspace skeleton. This `clj-new` template produces
a workspace that has some example code in it:

* `bases` -- contains a command-line API (`cli`)
* `components` -- contains a simple component (`greeter` interface and implementation)
* `projects` -- contains a simple application, based on `cli` and `greeter`, and a simple library, based on `greeter`

The generated README shows how you can run tests, build an uberjar, and build a library JAR.

See the [Polylith documentation](https://polylith.gitbook.io/) for more details.
Generated projects currently track the [issue-66 branch of the `poly` tool](https://github.com/polyfy/polylith/tree/issue-66). **That means you must use `clojure -M:poly` instead of the native `poly` command inside the generated project!**

#### The `template` Template

The generated project is a very minimal `clj-template`. It has no `-main`
function and has no tests. You can however build a jar file for deployment
with `clojure -X:jar`. You will probably need to adjust some of the information
inside the generated `pom.xml` file before deploying the jar file.

> Note: when you create a template project called myname/mytemplate, you will get a folder called `mytemplate` and the `pom.xml` file will specify the group/artifact as `net.clojars.myname/clj-template.mytemplate` which is a convention supported by `clj-new`.

As with the `lib` template, this template includes a `pom.xml` to make it easier
to deploy the template as a library. Once you have reviewed and possibly updated
the `pom.xml` file, you can install it locally or deploy it to Clojars, via the
appropriate aliases.

#### The Generated `pom.xml` File

Each of the built-in templates produces a project that contains a `pom.xml`
file, which is used to build the uberjar (`app`) or jar file (`lib` and `template`),
as well as guide the deployment of the latter two. If you don't plan to deploy the
library or template, or you just don't want a `pom.xml` lying around for your application,
you can delete it -- but you will also need to remove `:sync-pom true` from the `:exec-args`
for `depstar` in the generated `deps.edn` file.

The goal is such that if you used an appropriate `myname/myapp` style name for the
project that you asked `clj-new` to create, then most of the fields in the
`pom.xml` file should be usable as-is.

You can override the default value of several fields in the `pom.xml` file
using the `:env` exec-arg to `clj-new/create` as a hash map:

* `:group` -- defaults to the `myname` portion of `myname/myapp` (but see below),
* `:artifact` -- defaults to the `myapp` portion of `myname/myapp`,
* `:version` -- defaults to `"0.1.0-SNAPSHOT"`,
* `:description` -- defaults to `"FIXME: my new ..."` (`application`, `library`, or `template`),
* `:developer` -- defaults to a capitalized version of your computer's logged in username.
* `:scm-domain` -- defaults to `github.com` (but see below); used in all the SCM links in the generated projects: `https://{{scm-domain}}/{{scm-user}}/{{artifact}}`
* `:scm-user` -- defaults to (part of) the group name (but see below); used in all the SCM links in the generated projects: `https://{{scm-domain}}/{{scm-user}}/{{artifact}}`

> Note: `clj-new` tries to conform to the [Clojars Verified Group Names policy](https://github.com/clojars/clojars-web/wiki/Verified-Group-Names) -- which is similar to Maven Central's policy about group IDs -- by setting the default for `:group` to be something that seems to be a reverse domain name. If you use `myname/myapp` for your project name, the default for `:group` will be `net.clojars.myname`, `:artifact` will be `myapp`, `:scm-domain` will be `github.com`, and `:scm-user` will be `myname`. If you use `com.github.myname/myapp` for your project name, the default for `:group` will be `com.github.myname`, `:artifact` will be `myapp`, `:scm-domain` will be `github.com`, and `:scm-user` will be `myname`. `clj-new` also recognizes `io.github`, `com.gitlab`, and `io.gitlab` prefixes. The latter two will cause `:scm-domain` to default to `gitlab.com`. If your project name seems to have a group name that could be a reverse domain name, then it will be accepted as is, e.g., `com.acme/myapp` would produce `:group "com.acme", :artifact "myapp", :scm-domain "github.com", :scm-user "com.acme"`.

The `:description` field is also used in the generated project's `README.md` file.

Example:

```bash
    clojure -X:new-app :name myname/myapp \
      :env '{:group "com.acme" :artifact my-cool-app :version "1.2.3" :scm-user myusername}'
```

This creates the same project structure as in the earlier `myname/myapp` example except that the generated `pom.xml` file will contain:

```xml
  <groupId>com.acme</groupId>
  <artifactId>my-cool-app</artifactId>
  <version>1.2.3</version>
  <name>myname/myapp</name>
  <description>FIXME: my new application.</description>
  <url>https://github.com/myusername/my-cool-app</url>
  ...
  <scm>
    <url>https://github.com/myusername/my-cool-app</url>
    <connection>scm:git:git://github.com/myusername/my-cool-app.git</connection>
    <developerConnection>scm:git:ssh://git@github.com/myusername/my-cool-app.git</developerConnection>
    <tag>v1.2.3</tag>
  </scm>
```

Once you have generated the project, running `depstar` to build the JAR file will keep the
`pom.xml` in sync with the dependencies in your `deps.edn` file, and you can update the
version (and SCM tag) automatically using `depstar`'s `:version` exec argument. You can also change the
`groupId` and/or `artifactId` via `depstar`'s `:group-id` and/or `:artifact-id` exec
arguments respectively.

#### The Generated `LICENSE` File

The generated projects (from the built-in `app`, `lib`, and `template` templates) all
contain a `LICENSE` file which is the Eclipse Public License (version 1.0) and that
is also mentioned in the generated `README.md` files. This is a tradition that started
with Leiningen's `lein new` and carried over into `boot new` and now `clj-new`. The
idea is that it's better to ensure any open source projects created have a valid
license of some sort, as a starting point, and historically most Clojure projects use
the EPLv1.0 because Clojure itself and the Contrib libraries have all used this license
for a long time.

**You are not required to open source your generated project!** Just because the projects
are generated with an open source `LICENSE` file and have a **License** section in their
`README.md` files does not mean you need to keep that license in place.

**You are not required to use EPLv1.0 for your project!** If you prefer a different license,
use it! Replace the `LICENSE` file and update the `README.md` file to reflect your personal
preference in licensing (I have tended to use the [Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0.html) in most of my open source projects, prior to working with Clojure).

> Note: if you incorporate any source code from other people's open source projects, be
aware of the legal implications and that you must respect whatever license _they_ have
used for that code (which _may_ require you to release your enhancements under the same
license and will, most likely, require you to include their copyright notices, etc).
_Do not copy other people's code without attribution!_

#### General Template Usage

The general form of the command is:

    clojure -X:new :template template-name :name project-name :args '[arg1 arg2 arg3 ...]'

As noted above, `project-name` should be a qualified symbol, such as `mygithubusername/my-new-project`, or a multi-segment symbol, such as `my.cool.project`. Some templates will not work with the former but it is recommended you try that format first.

If `template-name` is not one of the built-in ones (or is not already on the classpath), `clj-new` will attempt to find it on Clojars or Maven Central (or any other `:mvn/repos` you have configured) in the following manner:
* If `template-name` is a qualified name, `some.group/example`, look for:
  * `some.group/clj-template.example`, then
  * `some.group/boot-template.example`, then
  * `some.group/lein-template.example`,
* Else, for an unqualified name, look for:
  * `template-name/clj-template`, then
  * `template-name/boot-template`, then
  * `template-name/lein-template`.

Currently, Boot and Leiningen only support the second form, with an
unqualified `template-name` (Leiningen is adding support for the qualified form).
Historically, `clj-new` also only
supported the unqualified `template-name` but as of 1.1.264 the
qualified name is also supported so that templates can have group
names that follow the [Clojars Verified Group Names policy](https://github.com/clojars/clojars-web/wiki/Verified-Group-Names)
and artifact names that start with `clj-template.`.

`clj-new` should be able to run any existing Leiningen or Boot templates (if you find one that doesn't work, [please tell me about it](https://github.com/seancorfield/clj-new/issues)!).

`clj-new` will generate a new project folder based on the `project-name` containing files generated from the specified `template-name`. It does that by requiring `clj.new.<template-name>` (or `boot.new.<template-name>` or `leiningen.new.<template-name>`) and invoking the `<template-name>` function inside that namespace, passing in `<project-name>` and those arguments from the command line.

Alternatively, `template-name` can be a `:git/url` and `:sha` like this:

    clojure -X:new :template '"https://github.com/somename/someapp@c1fc0cdf5a21565676003dbc597e380467394a89"' \
      :name project-name :args '[arg1 arg2 arg3 ...]'

In this case, `clj.new.someapp` must exist in the template and `clj.new.someapp/someapp` will be invoked to generate the template. A GitHub repository may include multiple templates, so you can also use this form:

    clojure -X:new :template '"https://github.com/somename/somerepo/someapp@c1fc0cdf5a21565676003dbc597e380467394a89"' \
      :name project-name :args '[arg1 arg2 arg3 ...]'

`somename/somerepo` here contains templates in subdirectories, including `someapp`. Again, `clj.new.someapp` must exist in the template in that subdirectory and `clj.new.someapp/someapp` will be invoked to generate the template.

Or, `template-name` can be a `:local/root` and template name like this:

    clojure -X:new :template '"/path/to/clj-template::new-app"' \
      :name project-name :args '[arg1 arg2 arg3 ...]'

In this case, `clj.new.new-app` must exist in the template and `clj.new.new-app/new-app` will be invoked to generate the template.

> Note: since the `:git/url` and `:local/root` forms of `:template` cannot be provided as Clojure symbols, they must be provided as Clojure strings, with `"..."`, and those must be quoted for the shell correctly, with `'...'` around the string.

If the folder for `project-name` already exists, `clj-new` will not overwrite it unless you specify the `:force` option.

#### Example Usage

Here are some examples, generating projects from existing templates:

```bash
    clojure -X:new :template luminus :name yourname/example.webapp \
      :output mywebapp :args '[+http-kit +h2 +reagent +auth]'
```

This creates a folder called `mywebapp` with a Luminus web application that will use `http-kit`, the H2 database, the Reagent ClojureScript library, and the Buddy library for authentication. The `-main` function is in `yourname.example.webapp.core`, which is in the  `mywebapp/src/clj/yourname/example/webapp/core.clj` file. Note that the [Luminus template](https://github.com/luminus-framework/luminus-template) produces a Leiningen-based project, not a CLI/`deps.edn` one, but you can also tell it to produce a Boot-based project (with `+boot`).

```
    clojure -X:new :template re-frame :name yourname/spa \
      :output front-end :args '[+garden +10x +routes]'
```

This creates a folder called `front-end` with a ClojureScript Single Page Application that uses Garden for CSS, `re-frame-10x` for debugging, and Secretary for routing. The entry point is in the `yourname.spa.core` namespace which is in the `front-end/src/cljs/yourname/spa/core.cljs` file. As with Luminus, the [`re-frame` template](https://github.com/day8/re-frame-template) produces a Leiningen-based project, not a CLI/`deps.edn` one.

```
    clojure -X:new :template electron-app :name yourname/example
```

This creates a folder called `example` with a skeleton Electron application, using Figwheel and Reagent. The entry point is in the `example.main.core` namespace which is in the `example/src/main/example/main/core.cljs` file. This [Electron template](https://github.com/paulbutcher/electron-app) produces a CLI/`deps.edn`-based project.

#### `clj` Templates

`clj` templates are very similar to Leiningen and Boot templates but have an artifact name based on `clj-template` instead of `lein-template` or `boot-template` and use `clj` instead of `leiningen` or `boot` in all the namespace names. In particular the `clj.new.templates` namespace provides functions such as `renderer` and `->files` that are the equivalent of the ones found in `leiningen.new.templates` when writing a Leiningen Template (or `boot.new.templates` when writing a Boot Template). The built-in templates are `clj` templates, that produce `clj` projects with `deps.edn` files.

If your template project name is `myname/foo-bar`, then you should have `clj.new.foo-bar` as the main namespace and it should contain a `foo-bar` function that will render the template:

```clj
;; src/clj/new/foo_bar.clj:
(ns clj.new.foo-bar ,,,)

(defn foo-bar
  "Generate a cool new foo bar project!"
  [name & args]
  ,,,)
```

When you publish it to Clojars, it should have an appropriate
(reverse domain name) group ID and the artifact ID should match
the template name preceded by `clj-template.`:
`net.clojars.myname/clj-template.foo-bar`. If you expect people
to depend on the template via GitHub, you should also name the
repo `foo-bar` so that `https://github.com/<username>/foo-bar`
is the `:git/url` people will use.

A minimal example, using the default bare bones template:

```
$ clojure -X:new :template template :name myname/mytemplate
Generating a project called mytemplate that is a 'clj-new' template
```

You will now have a folder called `mytemplate` that is a very minimal template.

To create a new project based on that template, you need to have it on the classpath (just as if it were a library) and you also need `clj-new` on the classpath since you are using it to generate a project from that template:

```
$ clojure -Sdeps '{:deps {myname/mytemplate {:local/root "mytemplate"}}}' \
  -X:new :template mytemplate :name myname/myproject
Generating fresh 'clj new' mytemplate project.
$ tree myproject
myproject
|____deps.edn
|____src
| |____myname
| | |____myproject
| | | |____foo.clj
```

This example uses a local template project structure, which is probably a good idea when you are developing your template, because the only real way to test a template is by trying to use it to generate a new project.

Once you have it working, you can publish it to GitHub or Clojars just like a regular library.

#### Arguments

Previous sections have revealed that it is possible to pass arguments to templates. For example:

```
    clojure -X:new :template custom-template :name project-name \
      :args '[arg1 arg2 arg3]'
```

These arguments are accessible in the `custom-template` function as a second argument.

```clj
(ns clj.new.custom-template ,,,)

(defn custom-template
  [name & args]
  (println name " has the following arguments: " args))
```

Nearly all templates will expect these to be strings but you can use symbols and `clj-new` will coerce them to strings for you:

```bash
    clojure -X:new :template custom-template :name project-name \
      :args '["arg1" "arg2" "arg3"]'
    # can usually be written as:
    clojure -X:new :template custom-template :name project-name \
      :args '[arg1 arg2 arg3]'
    # unless the arguments cannot be represented as Clojure symbols
```

> Note: conversion of `:args` (and `:output`) from symbols to strings was added in `clj-new` 1.1.next.

## clj Generators

Whereas clj templates will generate an entire new project in a new directory, clj generators are intended to add / modify code in an existing project.

You can either say `clojure -X:new clj-new/generate ...` or add an alias for it:

```clj
    ;; add this inside your :aliases map:
    :generate {:extra-deps {com.github.seancorfield/clj-new
                            {:mvn/version "1.1.297"}}
               :exec-fn clj-new/generate}}
```

Given the alias above, you can say `clojure -X:generate` to run one or more generators, based on a `:generate` vector argument that you provide. Each generator in the vector is a string -- either `"type"` or `"type=name"`. The `type` specifies the type of generator to use. The `name` is the main argument that is passed to the generator.

A clj generator can be part of a project or a template. A generator `foo`, has a `clj.generate.foo/generate` function that accepts at least two arguments, `prefix` and the `name` specified as the main argument. `prefix` specifies the directory in which to perform the code generation and defaults to `src` (it cannot currently be overridden). In addition, any additional arguments are passed as additional arguments to the generator.

There are currently a few built-in generators:
- `file`
- `ns`
- `def`
- `defn`
- `edn`

The `file` generator creates files relative to the prefix. It optionally accepts a body, and file extension, supplied via an `:args` vector of strings. Those default to `nil` and `"clj"` respectively.
```bash
# Inside project folder, relying on the clj-new dependency.
clojure -X:generate :generate '["file=foo.bar"]' :args '["(ns foo.bar)" "clj"]'
```

The `ns` generator creates a clojure namespace by using the `file` generator and providing a few defaults.
```bash
clojure -X:generate :generate '["ns=foo.bar"]'
```

This will generate `src/foo/bar.clj` containing `(ns foo.bar)` (and a placeholder docstring). It will not replace an existing file.
```bash
clojure -X:generate :generate '["defn=foo.bar/my-func"]'
```

If `src/foo/bar.clj` does not exist, it will be generated as a namespace first (using the `ns` generator above), then a definition for `my-func` will be appended to that file (with a placeholder docstring and a dummy argument vector of `[args]`). The generator does not check whether that `defn` already exists so it always appends a new `defn`.

Both the `def` and `defn` generators create files using the `ns` generator above.

The `edn` generator uses the `file` generator internally, with a default extension of `"edn"`.
```bash
clojure -X:generate :generate '["edn=foo.bar"]' :args '["(ns foo.bar)"]'
```

You can provide as many generators as you want in the `:generate` vector, but if you provide an `:args` vector then those arguments will be passed into each of the generator functions, so you may still need to run multiple `clojure -X:generate` commands.

The exec-args available for the `generate` function are:

* `:generate` -- a (non-empty) vector of generator strings to use
* `:args` -- an optional vector of string to pass to the generator itself as command-line arguments
* `:edn-args` -- an optional EDN expression to pass to the generator itself as the arguments for the generator; takes precedence over `:args`; nearly all generators expect a sequence of strings so `:args` is going to be the easiest way to pass arguments
* `:force` -- if `true`, will force overwrite the target directory/file if it exists
* `:help` -- if `true`, will provide a summary of these options as help
* `:prefix` -- specify the project directory in which to run the generator (the default is `src` but `:p '"."'` will allow a generator to modify files in the root of your project)
* `:snapshot` -- if `true`, look for -SNAPSHOT version of the template (not just a release version)
* `:template` -- load this template (using the same rules as for `clj-new/create` above) and then run the specified generator
* `:version` -- use this specific version of the template

# Releases

This project follows the version scheme MAJOR.MINOR.COMMITS where MAJOR and MINOR provide some relative indication of the size of the change, but do not follow semantic versioning. In general, all changes endeavor to be non-breaking (by moving to new names rather than by breaking existing names). COMMITS is an ever-increasing counter of commits since the beginning of this repository.

Latest stable release: 1.1.297

## Roadmap

* Improve the built-in template `template` so that it can be used to seed a new `clj` project.

## License

Copyright © 2016-2021 Sean Corfield and the Leiningen Team for much of the code -- thank you!

Distributed under the Eclipse Public License version 1.0.
