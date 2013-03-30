A very basic skeleton for building C++11 libraries with SCons and g++. I add
stuff as I need it.

A certain directory structure is assumed:

    + root
    |- include: C++ header files
    |- src: C++ source files
    |- test: test source files
    |- lib: third party libraries

The following targets are defined:

- `debug`: builds a debug version of the library;
- `release`: builds a release version of the library;
- `test-debug`: builds and runs tests based on the debug build;
- `test`: builds and runs tests based on the release build.

Some configuration options are available.

- `projectName` is a string used for the build targets and for macro prefixes
(sanitized as uppercase).

- `language` specifies the language level used with GCC's `-std=` setting.

- `ignoredWarnings` is a list of GCC warnings to ignore (without any prefixes).

- `noRebuild` is a list of GCC flags that should not cause a rebuild when they
change.

- `systemLibs` is a list of libraries to be passed to `pkg-config` for
obtaining compilation flags.

- `otherLibs` is a list of libraries to link directly.

