Glow
====

Glow is an OpenGL binding generator for Go. Glow parses the [OpenGL XML API registry](https://cvs.khronos.org/svn/repos/ogl/trunk/doc/registry/public/api/) to produce a machine-generated cgo bridge between Go functions and native OpenGL functions. Glow is a fork of [GoGL2](https://github.com/chsc/gogl2).

Features:
- Go functions that mirror the C specification using Go types.
- Support for multiple OpenGL APIs (GL/GLES/EGL/WGL/GLX/EGL), versions, and profiles.
- Support for extensions (including debug callbacks).

See the [open issues](https://github.com/errcw/glow/issues) for caveats about the current state of the implementation.

Usage
-----

Use `go get` to download and install one of the prebuilt packages. The prebuilt packages support OpenGL versions 3.2, 3.3, 4.1, and 4.4 across both the core and compatibility profiles and include all extensions.

    go get github.com/errcw/glow/gl-{core,compatibility}/{3.2,3.3,4.1,4.4}/gl
    go get github.com/errcw/glow/gl-core/3.3/gl

Once the bindings are installed you can use them with the appropriate import statements.

```Go
import "github.com/errcw/glow/gl-core/3.3/gl"

func main() {
  if err := gl.Init(); err != nil {
    panic(err)
  }
}
```

The `gl` package contains the OpenGL functions and enumeration values for the imported version. It also contains helper functions for working with the API. Of note is `gl.Ptr` which takes a Go array or slice or pointer and returns a corresponding `uintptr` to use with functions expecting data pointers. Also of note is `gl.Str` which takes a null-terminated Go string and returns a corresponding `*int8` to use with functions expecting character pointers.

A note about threading and goroutines. The bindings do not expose a mechanism to make an OpenGL context current on a different thread so you must restrict your usage to the thread on which you called `gl.Init()`. To do so you should use [LockOSThread](https://code.google.com/p/go-wiki/wiki/LockOSThread).

Examples
--------

A simple example illustrating how to use the bindings is available in the [examples](https://github.com/errcw/glow/tree/master/examples) directory.

Extensions
----------

If the bindings are generated with extensions included (true by default, or otherwise configured via the `-addext` and `-remext` flags) package-global booleans (e.g., `gl.ARB_cl_event`) indicate whether an extension is available. Availability is determined at `Init` time based on the ability to load the relevant functions (_not_ via `glGetString(GL_EXTENSIONS)`).

Function Loading
----------------

The `procaddr` package contains platform-specific functions for [loading OpenGL functions](https://www.opengl.org/wiki/Load_OpenGL_Functions). Calling `gl.Init()` uses the `auto` subpackage to automatically select an appropriate implementation based on the build environment. If you want to select a specific implementation you can use the `noauto` build tag and the `gl.InitWithProcAddrFunc` initialization function.

Custom Packages
---------------

If the prebuilt, go-gettable packages are not suitable for your needs you can build your own. For example,

    go get github.com/errcw/glow
    cd $GOPATH/src/github.com/errcw/glow
    go build
    ./glow download
    ./glow generate -api=gl -version=3.3 -profile=core -remext=GL_ARB_cl_event
    go install ./gl-core/3.3/gl

A few notes about the flags to `generate`:
- `api`: One of `gl`, `egl`, `wgl`, or `glx`.
- `version`: The API version to generate. The `all` pseudo-version includes all functions and enumerations for the specified API.
- `profile`: For `gl` packages with version 3.2 or higher, `core` or `compatibility` ([explanation](http://www.opengl.org/wiki/Core_And_Compatibility_in_Contexts)).
- `addext`: A regular expression describing which extensions to include. `.*` by default, including everything.
- `remext`: A regular expression describing which extensions to exclude. Empty by default, excluding nothing. Takes precedence over explicitly added regular expressions.
- `lenientInit`: Flag to disable strict function availability checks at `Init` time. By default if any non-extension function pointer cannot be loaded then initialization fails; when this flag is set initialization will succeed with missing functions. The generated package exposes `Has` markers to indicate which functions were successfully loaded. For example if `Init` cannot load the `glAccum` function pointer then `gl.HasAccum` will be false.
