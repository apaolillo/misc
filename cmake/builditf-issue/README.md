# Issue CMake build interface

It looks like that the build/install interfaces sometime insert absolute path
when using some syntax.
Read and run the two different `CMakeLists.txt` file to see the difference.

Here are the commands to reproduce the behaviour:

```shell script
antonio@hlab ~/g/m/m/c/builditf-issue> mkdir version1/build
antonio@hlab ~/g/m/m/c/builditf-issue> cd version1/build/
antonio@hlab ~/g/m/m/c/b/v/build> cmake ..
-- The C compiler identification is GNU 7.4.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/antonio/git/me/misc/cmake/builditf-issue/version1/build
antonio@hlab ~/g/m/m/c/b/v/build> make
Scanning dependencies of target testlib
[ 50%] Building C object CMakeFiles/testlib.dir/src/library.c.o
[100%] Linking C static library libtestlib.a
[100%] Built target testlib
antonio@hlab ~/g/m/m/c/b/v/build> make install
[100%] Built target testlib
Install the project...
-- Install configuration: ""
-- Installing: /home/antonio/git/me/misc/cmake/builditf-issue/version1/build/installed/lib/libtestlib.a
-- Installing: /home/antonio/git/me/misc/cmake/builditf-issue/version1/build/installed/cmake/testlib.cmake
-- Installing: /home/antonio/git/me/misc/cmake/builditf-issue/version1/build/installed/cmake/testlib-noconfig.cmake
antonio@hlab ~/g/m/m/c/b/v/build> grep -rn home installed/
installed/cmake/testlib.cmake:55:  INTERFACE_INCLUDE_DIRECTORIES "/home/antonio/git/me/misc/cmake/builditf-issue/version1/;${_IMPORT_PREFIX}/include2"
```

We can see that in the last grep command there is an absolute path that is added
for no apparent reason.
The CMake code responsible for this is the following (see 
`version1/CMakeLists.txt`):

```cmake
target_include_directories(testlib
    PUBLIC
        $<BUILD_INTERFACE:
            ${CMAKE_SOURCE_DIR}/include1
            ${CMAKE_SOURCE_DIR}/include2
        >
        $<INSTALL_INTERFACE:include2>
    PRIVATE
        include
)
```

When replacing it by the following syntax (see `version2/CMakeLists.txt`), the
faulty absolute path does not appear any more:

```cmake
target_include_directories(testlib
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include1>
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include2>
        $<INSTALL_INTERFACE:include2>
    PRIVATE
        include
)
```

Here is an example in the terminal to show how to reproduce:

```shell script
antonio@hlab ~/g/m/m/c/builditf-issue> mkdir version2/build
antonio@hlab ~/g/m/m/c/builditf-issue> cd version2/build/
antonio@hlab ~/g/m/m/c/b/v/build> cmake ..
-- The C compiler identification is GNU 7.4.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/antonio/git/me/misc/cmake/builditf-issue/version2/build
antonio@hlab ~/g/m/m/c/b/v/build> make
Scanning dependencies of target testlib
[ 50%] Building C object CMakeFiles/testlib.dir/src/library.c.o
[100%] Linking C static library libtestlib.a
[100%] Built target testlib
antonio@hlab ~/g/m/m/c/b/v/build> make install
[100%] Built target testlib
Install the project...
-- Install configuration: ""
-- Installing: /home/antonio/git/me/misc/cmake/builditf-issue/version2/build/installed/lib/libtestlib.a
-- Installing: /home/antonio/git/me/misc/cmake/builditf-issue/version2/build/installed/cmake/testlib.cmake
-- Installing: /home/antonio/git/me/misc/cmake/builditf-issue/version2/build/installed/cmake/testlib-noconfig.cmake
antonio@hlab ~/g/m/m/c/b/v/build> grep -rn home installed/
antonio@hlab ~/g/m/m/c/b/v/build>
```
