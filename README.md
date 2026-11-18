# Foundation

## Build Instructions

Obtain access to an arm64 Linux machine, preferably Debian. Other distributions will require small edits to the Makefile. The amd64-specific code paths have only been lightly tested, and were merely included to satisfy a friendly "bet", of sorts. Install Clang >= 22, LLVM ld, LLVM unwind, and BSD Make from your package manager. Optionally, install libffi to run comparator tests against NSInvocation's dynamic invocation code. Compile the [Blocks runtime](https://github.com/swiftlang/swift-corelibs-blocksruntime) and [GNUstep Objective-C runtime](https://github.com/gnustep/libobjc2) from source with Clang. Install the Blocks runtime first, then configure libobjc2 to disable its internal Blocks runtime. CMake should automatically detect and link libobjc2 against the external Blocks runtime.

(Shameless plug: Although untested and rather old, [my Blocks runtime](https://github.com/cmpwi/BlocksRuntime) should work but makes use of C23 features, which of course requires more changes to libobjc2's CMakeLists. On second thought, just use Apple's.)

If you compile the GNUstep runtime as `libobjc.so`, its name will conflict with the GCC runtime and Clang will attempt to link against the old runtime. This will cause linking errors as Foundation depends on many modern features like ARC autorelease pools that are not implemented in the GCC runtime. You may either manually delete the GCC runtime or install the GNUstep runtime under a different name like `libobjc2.so` (which requires updating `PROJECT_LDLIBS`).

From within the project root, you may run (with the usual option to append `-j<threads>`):

```
bmake debug
bmake san
bmake release
```

These modes will build Foundation for various use cases. `debug` embeds debugging information; `san` adds `-fsanitize=address,undefined,integer` to trap on common programming errors; `release` builds a fully optimized library at `-O3` with full link-time optimization.

```
bmake [mode] test-run
bmake [mode] test-run-NSData
```

The mode for both can be one of `debug`, `san`, or `release`, as above. The first builds and runs all tests. The second runs tests only for `NSData`. You may swap out `NSData` for any other class name present in `Testing/`. 

```
bmake clean
sudo bmake install
```

The first cleans out all object files. The second installs `libFoundation.so` in `/usr/lib` and headers in `/usr/include`.
