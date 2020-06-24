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

  path.append_cstr("zz.toml");

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

> TODO

## License

MIT
