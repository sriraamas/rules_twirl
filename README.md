# Twirl Template Rules for Bazel

| Travis | Doc
| --- | --- |
| [![Build Status](https://travis-ci.org/lucidsoftware/rules_twirl.svg)](https://travis-ci.org/lucidsoftware/rules_twirl) | [Stardoc](http://lucidsoftware.github.io/rules_twirl/) |

## Overview
`rules_twirl` compiles [Twirl](https://github.com/playframework/twirl) templates to [Scala](http://www.scala-lang.org/), so they can be used with [`rules_scala`](https://github.com/bazelbuild/rules_scala).

For more information about Twirl templates, see [the Play Twirl documentation](https://www.playframework.com/documentation/latest/ScalaTemplates#the-template-engine).

## Installation
Create a file called at the top of your repository named `WORKSPACE` and add the following snippet to it.

```python
# update version as needed
rules_twirl_version = "bf8bb5cea8bb14c968ac29a722ecbdb16d3a974a"
http_archive(
  name = "io_bazel_rules_twirl",
  sha256 = "5abc53fc6a7d42de5e068e35ea01652fd9e2cfe07c1c92c5f8854b0e26ab8cdf",
  strip_prefix = "rules_twirl-{}".format(rules_twirl_version),
  type = "zip",
  url = "https://github.com/lucidsoftware/rules_twirl/archive/{}.zip".format(rules_twirl_version),
)

RULES_JVM_EXTERNAL_TAG = "2.1"
http_archive(
    name = "rules_jvm_external",
    sha256 = "515ee5265387b88e4547b34a57393d2bcb1101314bcc5360ec7a482792556f42",
    strip_prefix = "rules_jvm_external-{}".format(RULES_JVM_EXTERNAL_TAG),
    type = "zip",
    url = "https://github.com/bazelbuild/rules_jvm_external/archive/{}.zip".format(RULES_JVM_EXTERNAL_TAG),
)

load("@io_bazel_rules_twirl//:workspace.bzl", "twirl_repositories")
twirl_repositories()
```

This installs `rules_twirl` to your `WORKSPACE` at the specified commit. Update the commit as needed.

## Stardoc Documentation
http://lucidsoftware.github.io/rules_twirl/

Stardoc is replacing Skydoc and is currently under development. Doc is likely going to look funny for a while.

### Updating Stardoc
Stardoc is automatically updated on build merged into the master branch. To update the documentation, please submit a pull request. The doc will be updated when it is merged.

### Deploying documentation
The Stardoc site for `rules_twirl` is deployed from the `gh-pages` branch. That branch is deployed with each build of the master branch.

## Usage
The `twirl_templates` rule compiles Twirl templates to a source jar that can be used with the `rules_scala` rules. For example,

```python
twirl_templates(
  name = "twirl-templates",
  source_directory = "app",
  srcs = glob(["app/**/*.scala.html"])
    + glob(["app/**/*.scala.xml"])
    + glob(["app/**/*.scala.js"])
    + glob(["app/**/*.scala.txt"]),
)

scala_binary(
  name = "foo-service",
  srcs = glob(["app/**/*.scala"]) + [":twirl-templates"],
  main_class = "foo.server.RunServer",
  deps = [...]
  )
)
```

See the [Stardoc documentation](https://lucidsoftware.github.io/rules_twirl/twirl/twirl.html#twirl_templates) for the full list of options for `twirl_templates`.

### Use with the Play Framework
`twirl_templates` can be used with the [`rules_play_routes`](https://github.com/lucidsoftware/rules_play_routes) to run a Play Framework Service. For example

```python
twirl_templates(
  name = "twirl-templates",
  source_directory = "app",
  include_play_imports = True,
  srcs = glob(["app/**/*.scala.html"])
    + glob(["app/**/*.scala.xml"])
    + glob(["app/**/*.scala.js"])
    + glob(["app/**/*.scala.txt"]),
  additional_imports = [...],
)

play_routes(
  name = "play-routes",
  srcs = ["conf/routes"] + glob(["conf/*.routes"]),
  include_play_imports = True,
  generate_reverse_router = True,
  routes_imports = [...],
)

scala_binary(
  name = "foo-service",
  srcs = glob(["app/**/*.scala"])  + [":twirl-templates", ":play-routes"],
  visibility = ["//visibility:public"],
  main_class = "play.core.server.ProdServerStart",
  resources = ["conf/logback.xml"] + glob(["conf/resources/**/*"]),
  resource_strip_prefix = native.package_name(),
  classpath_resources = ["conf/application.conf"],
  jvm_flags = [
  	"-Dhttp.port=9000",
  	"-Dapplication.name=foo-service",
  ],
  deps = [...],
)
```

## Development
### Command Line Twirl Compiler
This project consists of the Twirl Bazel rules and a command line Twirl compiler. The command line compiler can be built with
```bash
bazel build //twirl-compiler
```

It can be run with
```bash
bazel run //twirl-compiler
```

### Testing
All tests can be run using

```bash
test/run_all_tests.sh
```

They can also be run using
```bash
bazel test //test/...
```

### CI
The CI config in `tools/bazel.rc` and other options in `.bazelrc.travis` are used during CI builds.

#### Skylint
[Skylint](https://github.com/bazelbuild/bazel/blob/master/site/docs/skylark/skylint.md) is run during CI builds. To run it locally use
```bash
tools/skylint.sh
```