project
=======

> [WIP] ZZ Project API

## Installation

Add this to `zz.toml`:

```toml
[dependencies]
project = "*"

[repos]
project = "git://github.com/zzmodules/project"
```

## Usage

```c++
using project

fn main() -> int {
  new+1024 mut error = err::make();
  new+1024 mut path = string::make();

  path.append_cstr("./");

  new mut proj = project::make(&path);
  bool status = proj.open(&error);

  if error.check() {
    log::error("%s", error.trace.cstr());
  }

  // read all dependencies
  let mut it = proj.config.dependencies.iterator();
  while !it.ended {
    let node = it.next();
    let dep = as<project::ProjectDependency *>(node->value);
    log::info("%s = %s", dep->name.cstr(), dep->version.cstr());
  }

  return 0;
}
```

## API

### `struct Project`
A structure that represents a ZZ project. When opened, the project's `zz.toml`
file is parsed and its values are loaded.

```c++
struct Project {
  String+1024 path;
  ProjectConfig config;
}
```

### `struct ProjectConfig`

A structure that represents the configuration for a project, as seen in a `zz.toml` file.

```c++
struct ProjectConfig {
  String+32 mut version;
  String+64 mut name;
  String+64000 mut description;
  List+64 mut cincludes;
  List+64 mut cobjects;
  List+64 mut pkgconfig;
  List+64 mut cflags;
  List+64 mut lflags;
  List+64 mut dependencies;
  List+64 mut repos;
}
```

### `struct ProjectRepository`

A structure that represents a project dependency repository.

```c++
struct ProjectRepository {
  String+64 mut name;
  String+1024 mut url;
}
```

### `struct ProjectDependency`

A structure that represents a project dependency.

```c++
struct ProjectDependency {
  String+32 mut version;
  String+64 mut name;
}
```

### `new mut p = project::make(&path)`

`Project` structure constructor.

```c++
new mut proj = project::make(&path);
```

#### `p.open(&error)`

Opens the project, parsing the "zz.toml" values into memory. This function
returns `true` upon success, or `false` if an error occurs. The `error` given
to this function should be checked.

```c++
new+1024 mut error = err::make();
bool status = proj.open(&error);
```

### Custom TOML Document Parser

A custom iterator function for the TOML parser can be supplied to handle
sections in the `zz.toml` file not handled by the default
`toml_parser_document_iterator()` function.

```c++
using err
using toml
using string
using project

fn main() -> int {
  new+1024 mut error = err::make();
  new+1024 mut path = string::make();

  path.append_cstr("./");

  new mut proj = project::make(&path);
  proj.custom.toml_parser_document_iterator = toml_parser_document_iterator;

  if !proj.open(&error) {
    if error.check() {
      log::error("%s", error.trace.cstr());
    }
  }

  return 0;
}

fn toml_parser_document_iterator(
  toml::U *self,
  Err+ErrorTail mut *error,
  toml::Parser+ParserTail mut *parser,
  char *key,
  toml::Value value
)
  where err::checked(*error)
{
  static_attest(safe(key));
  static_attest(nullterm(key));

  let mut *project = as<Project mut *>(self->user1);

  if error->check() {
    return;
  }

  // handle a custom section...
  if utils::compare_string("custom-section", key) {
    toml::next(parser, error, toml::U {
      it: toml_parser_custom_section_iterator,
      user1: project
    });
  }
}

fn toml_parser_custom_section_iterator(
  toml::U *self,
  Err+ErrorTail mut *error,
  toml::Parser+ParserTail mut *parser,
  char *key,
  toml::Value value
)
  where err::checked(*error)
{
  static_attest(safe(key));
  static_attest(nullterm(key));

  let mut *project = as<Project mut *>(self->user1);

  if error->check() {
    return;
  }

  // handle custom section keys...
}
```

#### Custom TOML & JSON Output

Custom TOML & JSON output can be achieved by implementing a
`to_toml(project, output)` and `to_json(project, output)`
functions which are called when `project::to_toml()` or
`project::to_json()` is called.

```c++
using err
using toml
using string
using project

fn main() -> int {
  new+1024 mut error = err::make();
  new+1024 mut path = string::from("./");
  new mut proj = project::make(&path);

  proj.custom.to_toml = custom_to_toml;
  proj.custom.to_json = custom_to_json;

  if !proj.open(&error) {
    if error.check() {
      log::error("%s", error.trace.cstr());
    }
  }

  return 0;
}

export fn to_toml(Project *self, String+OutputTail mut *output) {
  // add custom sections to `output`
}

export fn to_json(Project *self, String+OutputTail mut *output) {
  // add custom key-value pairs to `output`
}
```

## License

MIT
