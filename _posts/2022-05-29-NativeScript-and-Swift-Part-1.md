---
layout: post
title: NativeScript and Swift Part 1: Calling Swift functions from C
#subtitle: Excerpt from Soulshaping by Jeff Brown
#cover-img: /assets/img/path.jpg
#thumbnail-img: /assets/img/thumb.png
#share-img: /assets/img/path.jpg
tags: [nativescript, swift, runtime, javascript]
---

# NativeScript and Swift Part 1: Calling Swift functions from C

In 2022 iOS development doesn't make much sense without Swift. The Swift community is getting bigger and bigger and with all these great frameworks out there I wish there was a better way to use them with NativeScript. 

## The state of the Swift interoperability
Currently, in order to access types written in Swift they need to be accessible from the Objective-C runtime, i.e. to be marked as `@objc` or to inherit NSObject. So, if you want to use a Swift framework in your NativeScript project, you may have to create Objective-C wrappers for some of the types. 

## Hello Swift!
Let's start with something simple as a proof of concept - our "Hello Swift" application will be able to call a Swift function from javascript. To make it more interesting, the function will take an `Int` parameter and return a `String`:

```
func helloFromSwift(intParam: Int) -> String {
   return "Hello from Swift with parameter: \(intParam)"
}
```

Somewhere in a NativeScript application we will do `console.log(helloFromSwift(5))` and get `"Hello from Swift with parameter: 5"` in the console.

## Foreign functions

When our code is compiled all functions are stored in memory (if not stripped but we will discuss this later). To invoke a function we can get a pointer to it's memory address using [dlsym](https://man7.org/linux/man-pages/man3/dlsym.3.html) and that would be pretty easy if swift symbols weren't [mangled](https://en.wikipedia.org/wiki/Name_mangling). Because *dlsym* needs the *runtime* name of the function, i.e. the *mangled* symbol, we should find a way to get it. There are several options though - by putting the symbol name together by [doing what compiler does](https://github.com/apple/swift/blob/main/docs/ABI/Mangling.rst) or by using a [library](https://github.com/apple/swift/tree/main/tools/SourceKit). 
Here we will take the shortcut by just dumping the symbol table of the compiled binary.

First, create a new project in Xcode by using the Command Line Tool template and choose Objective-C (C++ or C would also do the job) as language. Then create a new swift file and declare the following function:

```
func helloWorld() {
    print("Hello from Swift")
}
```

Now compile the project and see dump the binary's symbol table:
```
nm -a /Users/<username>/Library/Developer/Xcode/DerivedData/<project>/Build/Products/Debug/<product name>
```

The terminal should print something like this:
```
0000000100003b70 - 01 0000   FUN _$s10SwiftFromC10helloWorldyyF
```
If we double check with the mangling docs, the symbol name should be `$s10SwiftFromC10helloWorldyyF` and we are good to go.

Now update the main function and it will successfuly invoke the swift func:

```
#include <dlfcn.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        
        void (*func)(void) = dlsym(RTLD_DEFAULT, "$s10SwiftFromC10helloWorldyyF");
        func();
    }
    return 0;
}
```

The logical question here is how to handle parameters and return types. This is where [libffi](https://github.com/libffi/libffi) kicks in. And this is what NativeScript runtime is using too. We will play with libffi in Part 2 and meanwhile you can try extending what we just wrote with adding parameters and why not a class and a class function?
