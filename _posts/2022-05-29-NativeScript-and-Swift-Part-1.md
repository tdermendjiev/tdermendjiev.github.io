#NativeScript and Swift Part 1: Calling Swift functions from C

In 2022 iOS development doesn't make much sense without Swift. The Swift community is getting bigger and bigger and with all these great frameworks out there I wish there was a better way to use them with NativeScript. 

## The state of the Swift interoperability
Currently, in order to access types written in Swift they need to be accessible from the Objective-C runtime, i.e. to be marked as `@objc` or to inherit NSObject. So, if you want to use a Swift framework in your NativeScript project, you may have to create Objective-C wrappers for some of the types. 

##Hello Swift!
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

The logical question here is how to handle different parameters and return types because this approach lacks flexibility - in the snippet above the function signature has to be defined at compile time.
This is where [libffi](https://github.com/libffi/libffi) kicks in. And this is what NativeScript runtime is using too. 

## Using libffi

```
A foreign function interface (FFI) is a mechanism by which a program written in one programming language can call routines or make use of services written in another.
```

Thanks to `libffi` our program can decide at runtime what parameters the function accepts and what value it returns. After adding `libffi` to the project the main function will look like this:

```
#include <dlfcn.h>
#include <ffi.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        
        ffi_cif cif;
        ffi_status status = ffi_prep_cif(&cif, FFI_DEFAULT_ABI, 0, &ffi_type_void,
                                         NULL);
        if (status != FFI_OK) {
            fprintf(stderr, "ffi_prep_cif error: %d\n", status);
            exit(1);
        }
        
        void* fun = dlsym(RTLD_DEFAULT, "$s10SwiftFromC10helloWorldyyF");
        void* result;
        ffi_call(&cif, FFI_FN(fun), &result, NULL);
        
    }
    return 0;
}
```

 First, `ffi_prep_cif` is used to describe the function interface to ffi and then we call `ffi_call` with the function stored as `void*`.
 
 In order to actually use the power of `libffi`, let's update the swift function so it accepts a `Double` and returns an `Int`.
 
 ```
func helloWorld(param: Double) -> Int {
    return Int(param)
}
 ```
 The function's symbol will now be different - `$s10SwiftFromC10helloWorld5paramSiSd_tF`. If we run the program now, the function will still be invoked without any errors. In order to handle the return type and pass the parameter properly though, they should be passed to `ffi_prep_cif` accordingly.
 
 Consider this line again:
 ```
 ffi_status status = ffi_prep_cif(&cif, FFI_DEFAULT_ABI, 0, &ffi_type_void,
                                         NULL);
 ```
 Here we tell libffi that the expected return type is void. So if the function returns `int` it should be `&ffi_type_sint64` (or 32, depending on the architecture). And the `result` type will be `int`.
 
Both `Int` and `Double` are structs. In order the pack the arguments and unpack the return value properly we need to "describe" the structs to libffi. For example, let's say we have the following struct:
 
 ```
 typedef struct {
    int num;
    double dnum;
 } SomeStruct;
 ```
 
 The code for the `ffi_type` would look like this:
 
 ```
 ffi_type* elements[] = {&ffi_type_sint, &ffi_type_double, NULL};
 ffi_type type = {.size = 0, .alignment = 0,
                        .type = FFI_TYPE_STRUCT, .elements = dp_elements};
 ```
 
 Before we start digging into the swift interfaces (they won't be much helpful anyway, because there are computed properties), let's first check the [docs](https://developer.apple.com/documentation/swift/int). 
 ```
 On 32-bit platforms, Int is the same size as Int32, and on 64-bit platforms, Int is the same size as Int64.
 ```
 
 
 Looks like the `Int` is represented the same way as `int64` and if I inspect an `Int` value's memory (`1234` for example), this is what I see:
 
 

![<memory image>](https://raw.githubusercontent.com/tdermendjiev/tdermendjiev.github.io/master/assets/img/Screenshot%202022-05-31%20at%2016.48.02.png)

`1234`'s hex representation is `4D2`, so it looks good. Now, update the `helloWorld` and `main` functions:
```
func helloWorld(param: Double) -> Int {
    return Int(param)
}
```

```
#include <dlfcn.h>
#include <ffi.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        
        ffi_type* args[] = {&ffi_type_double};
        
        ffi_cif cif;
        ffi_status status = ffi_prep_cif(&cif, FFI_DEFAULT_ABI, 1, &ffi_type_sint64,
                                         args);
        if (status != FFI_OK) {
            fprintf(stderr, "ffi_prep_cif error: %d\n", status);
            exit(1);
        }
        
        void* fun = dlsym(RTLD_DEFAULT, "$s10SwiftFromC10helloWorld5paramSiSd_tF");
        int result;
        double param = 124.5;
        void* values[] = {&param};
        ffi_call(&cif, FFI_FN(fun), &result, &values);
        printf("Result is: %d\n", result);
        
    }
    return 0;
}
```

When we run it, we get `Result is: 1234`.


