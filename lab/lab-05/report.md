# Lab 5: Build Systems

## Part 1: CMake Tutorial

#### 1. 

```C++
// A simple program that computes the square root of a number
#include <cmath>
#include <iostream>
#include <string>

#include "TutorialConfig.h"

int main(int argc, char* argv[])
{
  if (argc < 2) {
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "." << Tutorial_VERSION_MINOR << std::endl;	    
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }

  // convert input to double
  const double inputValue = std::stod(argv[1]);

  // calculate square root
  const double outputValue = sqrt(inputValue);
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}
```
```CMake
cmake_minimum_required(VERSION 3.10)
project(Tutorial VERSION 1.0)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
configure_file(TutorialConfig.h.in TutorialConfig.h)
add_executable(Tutorial tutorial.cxx)
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
```

![Screenshot from 2022-06-17 11-35-57](https://user-images.githubusercontent.com/10250444/174330833-7e42fd20-d269-4aa4-9f7b-047edf858ca6.png)

#### 2. 
```C++
// A simple program that computes the square root of a number
#include <cmath>
#include <iostream>
#include <string>

#include "TutorialConfig.h"

#ifdef USE_MYMATH
#  include "MathFunctions.h"
#endif

int main(int argc, char* argv[])
{
  if (argc < 2) {
    // report version
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }

  // convert input to double
  const double inputValue = std::stod(argv[1]);

  // calculate square root
#ifdef USE_MYMATH
  const double outputValue = mysqrt(inputValue);
#else
  const double outputValue = sqrt(inputValue);
#endif
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}
```
```CMake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

option(USE_MYMATH "Use tutorial provided math implementation" ON)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(TutorialConfig.h.in TutorialConfig.h)


if(USE_MYMATH)
	add_subdirectory(MathFunctions)
	list(APPEND EXTRA_LIBS MathFunctions)
	list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
endif()

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
			   ${EXTRA_INCLUDES}
                           )
```
![Screenshot from 2022-06-17 11-58-28](https://user-images.githubusercontent.com/10250444/174334429-d954625e-b574-4fce-bbd1-d42720bdd5fb.png)

#### 3.
```CMake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(TutorialConfig.h.in TutorialConfig.h)

# add the MathFunctions library
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
endif()

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
```
```CMake
add_library(MathFunctions mysqrt.cxx)
target_include_directories(MathFunctions
			   INTERFACE
			   ${CMAKE_CURRENT_SOURCE_DIR})
```
![Screenshot from 2022-06-17 12-07-29](https://user-images.githubusercontent.com/10250444/174336111-45001acf-b8e0-474a-8f63-6253f436d522.png)

#### 4.
```CMake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(TutorialConfig.h.in TutorialConfig.h)

# add the MathFunctions library
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
endif()

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )

install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
	DESTINATION include
	)

enable_testing()

add_test(NAME Runs COMMAND Tutorial 25)

add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number")

function(do_test target arg result)
	add_test(NAME Comp${arg} COMMAND ${target} ${arg})
	set_tests_properties(Comp${arg} PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endfunction()
```
```CMake
add_library(MathFunctions mysqrt.cxx)

# state that anybody linking to us needs to include the current source dir
# to find MathFunctions.h, while we don't.
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          )

install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```
![Screenshot from 2022-06-17 12-22-33](https://user-images.githubusercontent.com/10250444/174338352-d9f8ed7c-2b80-4cfe-b89b-189eee393754.png)

#### 5.
```CMake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(TutorialConfig.h.in TutorialConfig.h)

# add the MathFunctions library
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
endif()

# add the executable
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )

# add the install targets
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )

# enable testing
enable_testing()

# does the application run
add_test(NAME Runs COMMAND Tutorial 25)

# does the usage message work?
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )

# define a function to simplify adding tests
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction()

# do a bunch of result based tests
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is (-nan|nan|0)")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
```
```CMake
add_library(MathFunctions mysqrt.cxx)

# state that anybody linking to us needs to include the current source dir
# to find MathFunctions.h, while we don't.
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          )

include(CheckCXXSourceCompiles)
check_cxx_source_compiles("
	#include <cmath>
	int main() {
	  std::log(1.0);
	  return 0;
	}
" HAVE_LOG)
check_cxx_source_compiles("
	#include <cmath>
	int main() {
	  std::exp(1.0);
	  return 0;
	}
" HAVE_EXP)

if(HAVE_LOG AND HAVE_EXP)
	target_compile_definitions(MathFunctions
				   PRIVATE "HAVE_LOG" "HAVE_EXP")
endif()
	
  

# install rules
install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```
![Screenshot from 2022-06-17 12-41-16](https://user-images.githubusercontent.com/10250444/174341079-e95bbf0a-1476-4415-ba18-a8e039d90d7d.png)

## Part 2
I didn't save my makefile before CMake overwrote it so it's less pretty (and things are building in the same directory so no `LD_LIBRARY_PATH`, etc. are needed) but the important bit is
`Makefile`
```make
block.o: source/block.c
	gcc -c -fPIC source/block.c -o block.o
	
archive: block.o
	ar rc libblock.a block.o

static: archive program.c
	gcc -o static.out -L. -lblock program.c
	
shared_o: block.o
	gcc -shared -o libblock.so block.o
	
shared: shared_o program.c
	gcc -o shared.out -L. -lblock program.c
```
My CMakeLists.txt
```CMake
cmake_minimum_required(VERSION 3.10)

project(Block VERSION 1.0)

option(BUILD_SHARED_LIBS "Use shared library" ON)

if(BUILD_SHARED_LIBS)
  add_library(block STATIC
              source/block.c
              )
else()
  add_library(block, SHARED
              source/block.c
              )
endif()

add_executable(Block program.c)
target_link_libraries(Block block)

target_include_directories(Block PUBLIC
                           "headers"
                           )
```
Generated Makefile
```make
# CMAKE generated file: DO NOT EDIT!
# Generated by "Unix Makefiles" Generator, CMake Version 3.22

# Default target executed when no arguments are given to make.
default_target: all
.PHONY : default_target

# Allow only one "make -f Makefile2" at a time, but pass parallelism.
.NOTPARALLEL:

#=============================================================================
# Special targets provided by cmake.

# Disable implicit rules so canonical targets will work.
.SUFFIXES:

# Disable VCS-based implicit rules.
% : %,v

# Disable VCS-based implicit rules.
% : RCS/%

# Disable VCS-based implicit rules.
% : RCS/%,v

# Disable VCS-based implicit rules.
% : SCCS/s.%

# Disable VCS-based implicit rules.
% : s.%

.SUFFIXES: .hpux_make_needs_suffix_list

# Command-line flag to silence nested $(MAKE).
$(VERBOSE)MAKESILENT = -s

#Suppress display of executed commands.
$(VERBOSE).SILENT:

# A target that is always out of date.
cmake_force:
.PHONY : cmake_force

#=============================================================================
# Set environment variables for the build.

# The shell in which to execute make rules.
SHELL = /bin/sh

# The CMake executable.
CMAKE_COMMAND = /usr/bin/cmake

# The command to remove a file.
RM = /usr/bin/cmake -E rm -f

# Escaping for special characters.
EQUALS = =

# The top-level source directory on which CMake was run.
CMAKE_SOURCE_DIR = /home/xyrothyl/Documents/osrc/CSCI-4470-OpenSource/Modules/05.BuildSystems/Lab-BuildSystemsExample

# The top-level build directory on which CMake was run.
CMAKE_BINARY_DIR = /home/xyrothyl/Documents/osrc/CSCI-4470-OpenSource/Modules/05.BuildSystems/Lab-BuildSystemsExample

#=============================================================================
# Targets provided globally by CMake.

# Special rule for the target edit_cache
edit_cache:
	@$(CMAKE_COMMAND) -E cmake_echo_color --switch=$(COLOR) --cyan "Running CMake cache editor..."
	/usr/bin/ccmake -S$(CMAKE_SOURCE_DIR) -B$(CMAKE_BINARY_DIR)
.PHONY : edit_cache

# Special rule for the target edit_cache
edit_cache/fast: edit_cache
.PHONY : edit_cache/fast

# Special rule for the target rebuild_cache
rebuild_cache:
	@$(CMAKE_COMMAND) -E cmake_echo_color --switch=$(COLOR) --cyan "Running CMake to regenerate build system..."
	/usr/bin/cmake --regenerate-during-build -S$(CMAKE_SOURCE_DIR) -B$(CMAKE_BINARY_DIR)
.PHONY : rebuild_cache

# Special rule for the target rebuild_cache
rebuild_cache/fast: rebuild_cache
.PHONY : rebuild_cache/fast

# The main all target
all: cmake_check_build_system
	$(CMAKE_COMMAND) -E cmake_progress_start /home/xyrothyl/Documents/osrc/CSCI-4470-OpenSource/Modules/05.BuildSystems/Lab-BuildSystemsExample/CMakeFiles /home/xyrothyl/Documents/osrc/CSCI-4470-OpenSource/Modules/05.BuildSystems/Lab-BuildSystemsExample//CMakeFiles/progress.marks
	$(MAKE) $(MAKESILENT) -f CMakeFiles/Makefile2 all
	$(CMAKE_COMMAND) -E cmake_progress_start /home/xyrothyl/Documents/osrc/CSCI-4470-OpenSource/Modules/05.BuildSystems/Lab-BuildSystemsExample/CMakeFiles 0
.PHONY : all

# The main clean target
clean:
	$(MAKE) $(MAKESILENT) -f CMakeFiles/Makefile2 clean
.PHONY : clean

# The main clean target
clean/fast: clean
.PHONY : clean/fast

# Prepare targets for installation.
preinstall: all
	$(MAKE) $(MAKESILENT) -f CMakeFiles/Makefile2 preinstall
.PHONY : preinstall

# Prepare targets for installation.
preinstall/fast:
	$(MAKE) $(MAKESILENT) -f CMakeFiles/Makefile2 preinstall
.PHONY : preinstall/fast

# clear depends
depend:
	$(CMAKE_COMMAND) -S$(CMAKE_SOURCE_DIR) -B$(CMAKE_BINARY_DIR) --check-build-system CMakeFiles/Makefile.cmake 1
.PHONY : depend

#=============================================================================
# Target rules for targets named block

# Build rule for target.
block: cmake_check_build_system
	$(MAKE) $(MAKESILENT) -f CMakeFiles/Makefile2 block
.PHONY : block

# fast build rule for target.
block/fast:
	$(MAKE) $(MAKESILENT) -f CMakeFiles/block.dir/build.make CMakeFiles/block.dir/build
.PHONY : block/fast

#=============================================================================
# Target rules for targets named Block

# Build rule for target.
Block: cmake_check_build_system
	$(MAKE) $(MAKESILENT) -f CMakeFiles/Makefile2 Block
.PHONY : Block

# fast build rule for target.
Block/fast:
	$(MAKE) $(MAKESILENT) -f CMakeFiles/Block.dir/build.make CMakeFiles/Block.dir/build
.PHONY : Block/fast

program.o: program.c.o
.PHONY : program.o

# target to build an object file
program.c.o:
	$(MAKE) $(MAKESILENT) -f CMakeFiles/Block.dir/build.make CMakeFiles/Block.dir/program.c.o
.PHONY : program.c.o

program.i: program.c.i
.PHONY : program.i

# target to preprocess a source file
program.c.i:
	$(MAKE) $(MAKESILENT) -f CMakeFiles/Block.dir/build.make CMakeFiles/Block.dir/program.c.i
.PHONY : program.c.i

program.s: program.c.s
.PHONY : program.s

# target to generate assembly for a file
program.c.s:
	$(MAKE) $(MAKESILENT) -f CMakeFiles/Block.dir/build.make CMakeFiles/Block.dir/program.c.s
.PHONY : program.c.s

source/block.o: source/block.c.o
.PHONY : source/block.o

# target to build an object file
source/block.c.o:
	$(MAKE) $(MAKESILENT) -f CMakeFiles/block.dir/build.make CMakeFiles/block.dir/source/block.c.o
.PHONY : source/block.c.o

source/block.i: source/block.c.i
.PHONY : source/block.i

# target to preprocess a source file
source/block.c.i:
	$(MAKE) $(MAKESILENT) -f CMakeFiles/block.dir/build.make CMakeFiles/block.dir/source/block.c.i
.PHONY : source/block.c.i

source/block.s: source/block.c.s
.PHONY : source/block.s

# target to generate assembly for a file
source/block.c.s:
	$(MAKE) $(MAKESILENT) -f CMakeFiles/block.dir/build.make CMakeFiles/block.dir/source/block.c.s
.PHONY : source/block.c.s

# Help Target
help:
	@echo "The following are some of the valid targets for this Makefile:"
	@echo "... all (the default if no target is provided)"
	@echo "... clean"
	@echo "... depend"
	@echo "... edit_cache"
	@echo "... rebuild_cache"
	@echo "... Block"
	@echo "... block"
	@echo "... program.o"
	@echo "... program.i"
	@echo "... program.s"
	@echo "... source/block.o"
	@echo "... source/block.i"
	@echo "... source/block.s"
.PHONY : help



#=============================================================================
# Special targets to cleanup operation of make.

# Special rule to run CMake to check the build system integrity.
# No rule that depends on this can have commands that come from listfiles
# because they might be regenerated.
cmake_check_build_system:
	$(CMAKE_COMMAND) -S$(CMAKE_SOURCE_DIR) -B$(CMAKE_BINARY_DIR) --check-build-system CMakeFiles/Makefile.cmake 0
.PHONY : cmake_check_build_system

```
Comparing file sizes
![image](https://user-images.githubusercontent.com/10250444/174713288-a8bf6342-9f4d-4853-b533-174c2b95cc26.png)
Program output
![image](https://user-images.githubusercontent.com/10250444/174713783-be1bc7b3-7b5c-4bff-a7c3-30d5ada68da0.png)

