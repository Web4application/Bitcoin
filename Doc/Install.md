CONTENTS OF THIS FILE
---------------------
* Introduction
* Prerequisites
* Building the Library
* Alternate Build Systems
* Installing the Library
* Makefile Targets
* Dynamic Analysis
* Acceptance Testing
* Reporting problems


INTRODUCTION
------------

Crypto++ Library is a free C++ class library of cryptographic algorithms and schemes. The library was originally written and placed in public domain by Wei Dai, but it is now maintained by the community. The library homepage is at http://www.cryptopp.com/. The latest library source code can be found at http://github.com/weidai11/cryptopp. For licensing and copyright information, please see License.txt.

These are general instructions for AIX, BSDs, Linux, OS X, Solaris and Unix. The library uses GNU Make and a GNUmakefile to avoid anemic make. On AIX, BSD and Solaris you will likely have to use `gmake` to build the library. On Linux and OS X, the system's make should be OK. On Windows, Crypto++ provides Visual Studio solutions.

You should look through the GNUmakefile and config.h to ensure settings look reasonable before building. There are two wiki pages that help explain them at http://www.cryptopp.com/wiki/GNUmakefile and http://www.cryptopp.com/wiki/Config.h.

Wiki pages are available for some platforms with specific build instructions. The pages include Android, ARM, iOS, MSBuild and Solaris. Solaris users should visit the wiki for important information on compiling the library with different versions of SunCC and options, and information on improving library performance and features.

Crypto++ does not depend upon other tools or libraries. The library only needs GNU Make 3.80 on Unix & Linux; or Visual Studio 2010 and above build tools on Windows. The library does not use Autotools, does not use CMake, and does not use Boost.

Autotools and CMake projects are not officially supported. The build systems take too much time and effort. Unofficial projects are available at https://github.com/noloader/cryptopp-autotools and https://github.com/abdes/cryptopp-cmake. The projects provide a central location to support Autotools and CMake. Collaborators for Autotools and CMake are welcomed.


PREREQUISITES
-------------

The library requires a semi-modern C++ compiler and GNU Make 3.81 or above. The compiler must support 64-bit words, C++03, namespaces, RTTI and exceptions.

The library does not depend on other build systems, like Autotools or CMake. The library does not depend on other libraries, like Boost.

BUILDING THE LIBRARY
--------------------

In general, all you should have to do is open a terminal, cd to the cryptopp directory, and then:

    make
    make test
    sudo make install

The command above builds the static library and cryptest.exe program. It also uses a sane default flags, which are usually "-DNDEBUG -g2 -O3 -fPIC".

If you want to build the shared object, then issue:

    make static dynamic cryptest.exe

Or:

    make libcryptopp.a libcryptopp.so cryptest.exe


If you would like to use a different compiler, the set CXX:

    export CXX=/opt/intel/bin/icpc
    make

Or:

    CXX=/opt/solarisstudio12.4/bin/CC make

If you want to build using C++11, then:

    export CXXFLAGS="-DNDEBUG -g2 -O3 -std=c++11"
    make

Or:

    CXXFLAGS="-DNDEBUG -g2 -O3 -std=c++11" make

LLVM's libc++ is also supported, so you can:

    export CXXFLAGS="-std=c++11 -stdlib=libc++"
    make

If you are using the library on OS X with XCode then you should add LLVM's libc++. You can do so by modifying CXXFLAGS, or you can modify the GNUmakefile. To modify the GNUmakefile, open it and find the line for OS X builds around line 150:

    ifneq ($(IS_DARWIN),0)
      CXX ?= c++
      CRYPTOPP_CXXFLAGS += -stdlib=libc++
      AR = libtool
      ARFLAGS = -static -o
    endif

If you target 32-bit IA-32 machines (i386, i586 or i686), then the makefile forgoes -fPIC due to register pressures. You should add -fPIC yourself, if needed:

    CXXFLAGS="-DNDEBUG -g2 -O3 -fPIC" make

You can also override a variable so that only your flags are present. That is, the makefile will not add additional flags. For example, the following builds with only -std=c++11:

    make CXXFLAGS="-std=c++11"

Crypto++ does not engage Specter remediations at this time. You can build with Specter resistance with the following flags:

    CXXFLAGS="-DNDEBUG -g2 -O3 -mfunction-return=thunk -mindirect-branch=thunk" make

The library does not support out-of-tree builds. You must cd to the Crypto++ directory before building. `make distclean` will return the Crypto++ directory to a pristine state.


BUILDING WITH VCPKG
-------------------

You can download and install cryptopp using the [vcpkg](https://github.com/Microsoft/vcpkg/) dependency manager:

    git clone https://github.com/Microsoft/vcpkg.git
    cd vcpkg
    ./bootstrap-vcpkg.sh
    ./vcpkg integrate install
    ./vcpkg install cryptopp

The cryptopp port in vcpkg is kept up to date by Microsoft team members and community contributors.
If the version is out of date, please [create an issue or pull request](https://github.com/Microsoft/vcpkg) on the vcpkg repository.


ALTERNATE BUILD SYSTEMS
-----------------------

The Crypto++ library is Make based and uses GNU Make by default. The makefile uses '-DNDEBUG -g2 -O2' CXXFLAGS by default. If you use an alternate build system, like Autotools or CMake, then ensure the build system includes '-DNDEBUG' for production or release builds. The Crypto++ library uses asserts for debugging and diagnostics during development; it does not rely on them to crash a program at runtime.

If an assert triggers in production software, then unprotected sensitive information could be written to the filesystem and could be egressed to the platform's error reporting program, like Apport on Ubuntu, CrashReporter on Apple and Windows Error Reporting on Microsoft OSes.

The makefile orders object files to help remediate problems associated with C++ static initialization order. The library does not use custom linker scripts. If you use an alternate build system, like Autotools or CMake, and collect source files into a list, then ensure these three are at the head of the list: 'cryptlib.cpp cpu.cpp integer.cpp <other sources>'. They should be linked in the same order: 'cryptlib.o cpu.o integer.o <other objects>'.

If your linker supports initialization attributes, like init_priority, then you can define CRYPTOPP_INIT_PRIORITY to control object initialization order. Set it to a value like 250. User programs can use CRYPTOPP_USER_PRIORITY to avoid conflicts with library values. Initialization attributes are more reliable than object file ordering, but its not ubiquitously supported by linkers.

The makefile links to the static version of the Crypto++ library to avoid missing dynamic libraries, incorrect dynamic libraries, binary planting and other LD_PRELOAD tricks. You should use the static version of the library in your programs to help avoid unwanted redirections.


INSTALLING THE LIBRARY
----------------------

To install the library into a user selected directory, perform:

    make install PREFIX=/usr/local

If you are going to run `make install PREFIX=/usr/local`, then you should build with '-DCRYPTOPP_DATA_DIR='\"$PREFIX/share/cryptopp/\"' to ensure cryptest.exe can locate the test data files and test vectors after installation. The trailing slash in the path is needed because simple preprocessor concatenation is used.

During install, the makefile copies cryptest.exe into $PREFIX/bin, copies headers into $PREFIX/include/cryptopp, and copies libraries into $PREFIX/lib. If you only built a static or dynamic version of the library, then only one library is copied. The install recipe does not fail if the static library or shared object is not built.

PREFIX is non-standard, but its retained for historical purposes. The makefile also responds to `prefix=<path>`.


MAKEFILE TARGETS
----------------

The following are some of the targets provided by the GNU makefile.

`make` invokes the default rule, which builds the Crypto++ static library and test harness. They are called `libcryptopp.a` and `cryptest.exe`, respectively. `cryptest.exe` links against `libcryptopp.a`, so the static library is a prerequisite for the target.

`make libcryptopp.a` and `make static` build the static version of the library.

`make libcryptopp.so` and `make dynamic` build the dynamic version of the library. On Mac OS X, the recipe builds `libcryptopp.dylib` instead.

`make cryptest.exe` builds the library test harness.

`make test` and `make check` are the same recipe and invoke the test harness with the validation option. That is, it executes `cryptest.exe v`.

`make install` installs the library. By default, the makefile copies into `/usr/local` by default.

`make clean` cleans most transient and temporary objects.

`make disclean` cleans most objects that are not part of the original distribution.

`make dist` and `make zip` builds ZIP file that is suitable for distribution.

`make iso` builds an ISO on Linux or OS X that is suitable for alternate distribution.

`make ubsan` and `make asan` builds the library with the respective sanitizer.


DYNAMIC ANALYSIS
----------------

The Crypto++ embraces tools like Undefined Behavior sanitizer (UBsan), Address sanitizer (Asan), Bounds sanitizer (Bsan), Coverity Scan and Valgrind. Both Clang 3.2 and above and GCC 4.8 and above provide sanitizers. Please check with your distribution on how to install the compiler with its sanitizer libraries (they are sometimes a separate install item).

UBsan and Asan are mutually exclusive options, so you can perform only one of these at a time:

    make ubsan
    ./cryptest.exe v 2>&1 | grep -E "(error:|FAILED)"
    ./cryptest.exe tv all 2>&1 | grep -E "(error:|FAILED)"

Or:

    make asan
    ./cryptest.exe v 2>&1 | grep -E "(error:|FAILED)"
    ./cryptest.exe tv all 2>&1 | grep -E "(error:|FAILED)"

If you experience failures under Asan, then gather more information with asan_symbolize. You may not need asan_symbolize nowadays:

    ./cryptest.exe v 2>&1 | asan_symbolize

If you moved Crypto++ such that the paths have changed, then perform:

    ./cryptest.exe v 2>&1 | sed "s/<old path>/<new path>/g" | asan_symbolize


ACCEPTANCE TESTING
------------------

Crypto++ uses five security gates in its engineering process. The library must maintain the quality provided by the review system and integrity of the test suites. You can use the information to decide if the Crypto++ library suits your needs and provides a compatible security posture.

The first gate is code review and discussion of proposed changes. Git commits often cross reference a User Group discussions.

Second is the compiler warning system. The code must clean compile under the equivalent of GCC's -Wall -Wextra (modulo -Wno-type-limits -Wno-unknown-pragmas). This is a moving target as compiler analysis improves.

Third, the code must pass cleanly though GCC and Clang's Undefined Behavior sanitizer (UBsan), Address sanitizer (Asan) and Bounds sanitizer (Bsan). See DYNAMIC ANALYSIS above on how to execute them.

Fourth, the code must pass cleanly though Coverity Scan and Valgrind. Valgrind testing is performed at -O1 in accordance with Valgrind recommendations.

Fifth, the test harness provides a "validation" option which performs basic system checks (like endianness and word sizes) and exercises algorithms (like AES and SHA). You run the validation suite as shown below. The tail of the output should indicate 0 failed tests.

    ./cryptest.exe v
    ...

    Seed used was 1612313449
    Test started at Tue Feb 2 19:50:49 2021
    Test ended at Tue Feb 2 19:50:52 2021

Sixth, the test harness provides a "test vector" option which uses many known test vectors, even those published by other people (like Brian Gladman for AES). You run the test vectors as shown below. The tail of the output should indicate 0 failed tests.

    ./cryptest.exe tv all
    ...

    Testing SymmetricCipher algorithm AES/XTS.
    .....................
    Tests complete. Total tests = 11260. Failed tests = 0.

The library also offers its test script for those who want to use it. The test script is names cryptest.sh, and it repeatedly builds the library and exectues the tests under various configurations. It takes about 4 hours to run on a semi-modern desktop or server; and several days to run on an IoT gadget. Also see http://github.com/weidai11/cryptopp/blob/master/cryptest.sh and http://cryptopp.com/wiki/Cryptest.sh.


REPORTING PROBLEMS
------------------

Build failures, dirty compiles and failures in the validation suite or test vectors should be reported at the Crypto++ User Group. The User Group is located at http://groups.google.com/forum/#!forum/cryptopp-users.

The library uses Wei Dai's GitHub to track issues. The tracker is located at http://github.com/weidai11/cryptopp/issues. Please do not ask questions in the bug tracker; ask questions on the mailing list instead. Also see http://www.cryptopp.com/wiki/Bug_Report.
