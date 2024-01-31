---
title: Introduction to Open Source Projects and CMake
date: 2021-06-15 12:00:00 +0500
categories: [Computers and Fun]
tags: [open-source,cmake,make]
---

## Open Source Projects

### Anatomy of Open Source Projects

A typical open source project has the following types of people &rarr;

- Author - The one who creates the project
- Owner - The one who has administrative ownership over the organization or the repository
- Maintainers - Contributors responsible for managing the organizational aspects of the project (may also be authors or owners)
- Contributors - The ones who have contributed something to the project
- Community Members - The ones who use the project and may be active in conversations and expression of opinions

Bigger projects can also have teams working on a particular task such as tooling, community moderation, etc. Documentation for the project has the following structure &rarr;

- License - The Open Source Initiative (OSI) has an approved list of tens of licenses, the most common of which are &rarr;
	- GNU’s General Public License (GNU GPL) - Most popular. Created by Richard Stallman. It is copyleft (Any software written based on any GPL component must be released as open source regardless of the percentage in the entire code).
	- Apache Software Foundation (ASF)’s Apache license - This license allows one to freely use, modify and distribute any Apache licensed product (required to follow the terms of the license).
	- MIT License - This allows one to do anything with the software under this license, provided that a copy of the license is included in the work.

	Other licenses such as BSD licenses, EPL, etc. also exist. All licenses are divided into **permissive** or **copyleft**. MIT, Apache 2.0 and BSD licenses are most popular permissive licenses, while GNU GPLv3 is the most popular copyleft license. Permissive licenses allow developers to include their own copyright statements while copyleft licenses do not (no copyright claims or patents on the original software).

- README - The instruction manual to welcome community members (those who use the project).
- Contributing - Explains what type of contributions are needed and how the process works.
- Code Of Conduct - Sets ground rules for participants’ behavior.

The following are used for an organized discussion &rarr;

- Issue Tracker
- Pull Requests
- Discussion Forums
- Synchronous chat channels

### Contribution Process Flow

To make a contribution to a repository, first fork the repository to a personal repository. Then follow the steps of **fork** &rarr; **clone** &rarr; **push** &rarr; **PR**. These steps basically mean &rarr;

- Fork the repository of interest to create a copy on github personal account.
- Clone the repository (personal copy) to the local machine.
- Change files, add features or any modifications.
- Add the changes and commit them using `git`.
- Push the changes to the required branch in the remote repository (personal).
- (Optional) Merge the branch to personal master using `git merge`.
- "Compare and Pull" or "Pull request" button on the repository link is used to create a pull request.

### Facts

The base repository from which a fork is made is also called the upstream repository. For `git config`, omit the `--global` argument to make config changes to only the current git repository.

### C/C++ Projects

#### Project Structure

A basic structure for a C/C++ project would be as follows &rarr;

```
Project_name
|
|---- CMakeLists.txt
|
|---- include
| |
| |---- Project_name
| |
| |---- public_header(s).h
|
---- src
| |
| |---- private_header(s).h
| |
| |---- code(s).cpp
|
|---- libs
| |
| |---- A
| |
| |---- B
|
|---- tests
```

This could be an example of a library which can be used directly or by a third party. The idea behind the structure is to keep private headers separate from public headers (for functions which anyone using the library can invoke).

#### Structure Explanation

- `include/` &rarr; By convention, this directory is for header files but according to modern practices, it must strictly contain headers that need to be publicly exposed. The directory with the same name as that of the project inside the include directory is used so that it supports generalisation when one imports the public header as &rarr; `#include <Project_Name/public_header.h>` Instead of `#include <public_header.h>`
- `src/` &rarr; This directory contains the source code and header files which are for internal use only. All code that the project consists of goes in here.
- `libs/` &rarr; This directory contains the third party libraries needed by the project. Usually, these follow the same structure as the one used for this project. Libraries in C++ can be used in one of two ways - static and dynamic. Only static libraries are present in the libs directory.
- `test/` &rarr; Directory to store unit tests and other kinds of tests, if any.
- `CMakeList.txt` &rarr; A configuration file to define the function of CMake, which is a build system generator.

## Build System Generation - CMake

CMake is not a build system but a tool that is a build system generator. Usually, a C++ project requires just the following steps after a GitHub clone to run &rarr;

```bash
mkdir build
cd build
cmake
..
make
```

The first step is just to keep the code clean and build everything in a separate directory. The third step i.e., the `cmake` call is to generate a `Makefile`. An example is, say one has the following files - `main.cpp`, `a.cpp`, `a.h`, `b.cpp`, `b.h`. The `main.cpp` file has the `main()` function and depends on the files `a.cpp` and `b.cpp`. Therefore, one must run `g++` compiler on `a.cpp`, then on `b.cpp` and finally on `main.cpp` with the compile flag i.e., `-c`.

> Header files are not compiled. They just tell the compiler about the function declaration.
{: .prompt-info }

Then to link them all together, one must call &rarr; `g++ a.o b.o main.o -o binary`. Thus, one must compile and link the code in two different orderly steps to generate a binary/executable. To make this easy for a number of files, a tool or a build system named `Make` was developed. This reduced the steps to writing a `Makefile` and then running the `make` tool by pointing it to the location of the `Makefile`.

Though after a point, writing `Makefile`s too became tedious, the solution to which was a tool called `cmake` which generates `Makefile`s using a relatively simpler and easy to maintain `CMakeLists.txt` file. This is why it’s called a build system generator. Thus, running the `cmake` tool and pointing it to the location of the `CMakeLists.txt` file creates a `Makefile` in the present working directory. After this, just calling the `make` tool to work on the `Makefile` will build the project (by default it looks under the build directory unless a specific location is specified).

### CMakeLists details and file types

The main types of files that are dealt with are &rarr; archive files (`.a`), shared objects (`.so`), header files (`.h`) and objects (`.o`). To create object files, the `.c` or `.cpp` file is compiled with the `-c` flag. Files with suffix `.a` are called archived files which are used as a statically linked binary i.e., when it is linked with the program, all code defined in the archive is included in the binary.

To create the archive, the following command is used &rarr; 

```bash
gcc -c source1.c
gcc -c source2.c
ar -cvq libtest.a source1.o source2.o
gcc main.c libtest.a -o final
```

The last command is where the `main.c` file is compiled with the archive file. The files with the suffix `.so` are shared objects which are not included in the final executable. To create a shared library, an object file must be compiled as position independent code or PIC. This is done as follows &rarr; 

```bash
gcc -fPIC -c source1.c source2.c gcc -shared -o libtest.so source1.o source2.o
```

To link the code of either archived files or shared objects, the `-l` flag is used as follows &rarr;

```bash
gcc main.c -ltest -o final
```

The `-ltest` argument looks for either `libtest.so` or `libtest.a`. Some projects provide both a dynamic shared library and a statically linked library to let the user choose the required method. The other way for using a statically linked library is as shown earlier while creating the archive. Header files (`.h`) are not compiled. They are included in `.c` files to inform the compiler about functions. Once the compiler cross-references everything, their information is discarded and they do not affect the final executable size. The capital i flag, `-I`, is used to specify which directories contain header files. The flag `-L` is used to tell the compiler which directories contain the static libraries (`.a`). The lowercase l flag, `-l`, is used to specify the name of the library to be linked i.e., `-l test` means that the compiler looks for `libtest.a` or `testlib.a` or a similar shared object.

### CMake

A standard compile command needs to have the following basic things &rarr;

- Path to files that need to be compiled.
- Path to header files including those which are used from third party libraries.
- Paths to third party libraries' `.a` files.
- Names of the .a files that the compiler should link with the code.

This is automated by CMake, which has 5 major sections - flags, files, include, targets, external libraries, unit testing. In order, a `CMakeLists.txt` file will have the following (prefixed with `#`) &rarr;

- Version &rarr; This is to tell `CMake` which version of the tool to use. Cannot use a version lower than that specified.
- Project Name
- Flags &rarr; Tells `CMake` which compiler and version of compiler to use to build the project with. With an empty value, it sets the best fit on its own.
- Files &rarr; Specify all files and club them into variable names like source, include, etc. But this can be skipped and the filenames can be used directly.
- Include &rarr; This is used to specify the path of include directories that the compiler must look into while searching for header files. This also includes header files from third party libraries.
- Target &rarr; The name of the output file is specified first. All names after that are the source files that need to be compiled. `<output>`
- External Library &rarr; The linking of libraries. In previous steps the location of header files for third party libraries were informed to the compiler. This step is used to inform it about the location of the libraries. By convention, library files are named with `lib` as a prefix or suffix.

    - If a `.a` file has already been compiled and `CMake` isn’t required to do it, then the following is written &rarr;

        ```
        add_library(<name from libname.a> STATIC IMPORTED)
        set_property(TARGET <name> PROPERTY IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/<path to .a file from there>)
        target_link_libraries(<output> <name>)
        ```

    - If the library is not compiled, but has a CMakeLists.txt file of its own then the following can be written &rarr;
    
        ```
        add_subdirectory(<path to library directory with CMakeLists.txt>/<library_project_name>)
        target_link_libraries(<output> <filename of .a that results from internal CMakeLists.txt>)
        ```

This concludes the `CMakeLists.txt` file. An example `CMakeLists.txt` file is as follows &rarr;

```
cmake_minimum_required( VERSION 3.0 )
project( sample_cmake )

# flags
# include files
include_directories( ./include ./src ./libs/Logger/include ./libs/Randomize/include )

# target
add_executable( binary ./src/main.cpp ./src/game_engine.cpp ./src/game_interface.cpp )

# 3rd party libs
add_subdirectory( ./libs/Logger )
target_link_libraries( binary logger )

add_library(randomize STATIC IMPORTED)
set_property(TARGET randomize PROPERTY IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/libs/Randomize/librandomize.a)
target_link_libraries( binary randomize )
```
