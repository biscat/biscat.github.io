# Basic Definitions

## library
Library is a packaged collection of[object files](http://nickdesaulniers.github.io/blog/2016/08/13/object-files-and-symbols/)that program can[link against](http://dn.embarcadero.com/article/29930).
So a library is a bunch of compiled code. You can create 2 types of libs:
- static 
- dynamic

###  Mach-O file format 
When you're compiling the source files you are basically making object files, using the[Mach-O](https://lowlevelbits.org/parsing-mach-o-files/) (Mach Object)  file format.
The object files are then packaged into executable code or static libraries.


## framework
A framework is a hierarchical directory that encapsulates shared resources, such as a dynamic shared library, nib files, image files, localized strings, header files, and reference documentation in a single package.
So let's make this simple: frameworks are[static or dynamic](https://speakerdeck.com/marius/static-vs-dynamic-linking)libraries packed into a bundle with some extra assets, meta description for versioning and more.

## package
A package consists of Swift source files and a manifest file.
You can also check out the open sourced[switf-corelibs-foundation](https://github.com/apple/swift-corelibs-foundation)package by Apple, which is used to build the Foundation framework for Swift.


## module
Swift organizes code into[modules](https://gist.github.com/briancroom/5d0f1b966fa9ef0ae4950e97f9d76f77). 
Each module specifies a namespace and enforces access controls on which parts of that code can be used outside of the module.
With the[import keyword](https://stackoverflow.com/questions/18947516/import-vs-import-ios-7)you are literally importing external modules into your sorce. In Swift you are always using frameworks as modules

Before modules you had to import framework headers directly into your code and you also had to link manually the framework's binary within Xcode.
The`#import`macro literally copy-pasted the whole resolved dependency structure into your code, and the compiler did the work on that huge source file.
It was a fragile system, things could go wrong with macro definitions, you could easily break other frameworks. That was the reason for defining prefixed uppercased very long macro names like: NS_MYSUPERLONGMACRONAME… 
There was an other issue: the copy-pasing resulted in non-scalable compile times. 
In order to solve this,[precompiled header (PCH) files](https://useyourloaf.com/blog/modules-and-precompiled-headers/)were born, but that was only a partial solution, because they polluted the namespace (you know if you import UIKit in a PCH file it gets available in everywhere), and no one really maintained them.

### modules and module maps
module maps defines what kind of headers are part of a module and what's the binary that has the implementation.
Header files are defining the interface (API), and the (automatically) linked dylib file contains the implementation. There's no need to parse framework headers during compilation time (scalability), so local macro definitions won't break anything.
Modules can contain submodules (inheritance), and you don't have to link them explicitly inside your (Xcode) project, because the .modulemap file has all the information that the build system needs.

# Linking libraries
Linking refers to the creation of a single executable file from multiple object files.
Linking is just combining all your object files into an executable and resolving all the externals, so the system will be able to call all the functions inside the binary.

## static linking
The source code of the library is literally going to be copied into the application's source. This will result in a big executable, it'll take more time to load, so the binary will have a slower startup time.
if you are trying to link the same library more than once, the process will fail because of duplicated symbols?
This method has advantages as well, for example the executable will 
- always contain the correct version of the library, 
- and only those parts will be copied into the main application that are really used, so you don't have to load the whole stuff
![static_linking.png](static_linking.png)

## dynamic linking
[Dynamic libraries](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/OverviewOfDynamicLibraries.html#//apple_ref/doc/uid/TP40001873-SW1) are not embedded into the source of the binary, they are loaded at runtime. 
This means that apps can be smaller and startup time can significantly be faster because of the lightweight binary files.
As a gratis dynamic libraries can be shared with multiple executables so they can have lower memory footprints. That's why sometimes they're being referred as shared libraries.
Of course if the dynamic library is not available - or it's available but the version is incompatible - your application won't run or it'll crash.
On the other hand this can be an advantage, because the author of the dynamic library can ship fixes and  your app can benefit from these, without recompilation.

![dynamic_linking.png](dynamic_linking.png)


# references
https://theswiftdev.com/deep-dive-into-swift-frameworks/
