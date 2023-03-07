# How to use Gurobi on Windows using CLion and MinGW

A quick tutorial how to use cmake and MinGW for compiling Gurobi project on Windows with
Clion.

## Prerequisites
- Make sure you have Gurobi installed on your computer.
There should be set `GUROBI_HOME` variable, in my case `GUROBI_HOME=C:\gurobi950\win64`.


- Make sure you have installed MinGW and set `mingw64/bin` in your `%PATH%`.

- Make sure you have installed CMake and clion.

- Clion -> File -> Settings -> Build, Execution, Deployment -> ToolChains -> Check that MinGW is selected  

## Set CMake

- Create a new project in Clion
- Open `CMakeList.txt`
- Change your `CMakeList.txt` to look something like this

```cmake
cmake_minimum_required(VERSION 3.13)
project(Example)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/modules/")

set(SRC
  ./main.cpp
  )
find_package(GUROBI REQUIRED)
include_directories(${GUROBI_INCLUDE_DIRS})

add_executable(${PROJECT_NAME} ${SRC})

target_link_libraries(${PROJECT_NAME} ${GUROBI_LIBRARIES})
```

## Create FindGUROBI.cmake
- Create folder `modules` and file `FindGUROBI.cmake` in it
- Paste this code there (`modules/FindGUROBI.cmake`):

```cmake
if (GUROBI_INCLUDE_DIR)
    set(GUROBI_FOUND TRUE)
    set(GUROBI_INCLUDE_DIRS "${GUROBI_INCLUDE_DIR}")
    set(GUROBI_LIBRARIES "${GUROBI_CXX_LIBRARY};${GUROBI_LIBRARY}")
else (GUROBI_INCLUDE_DIR)

    find_path(GUROBI_INCLUDE_DIR
            NAMES gurobi_c++.h
            PATHS "$ENV{GUROBI_HOME}/include"
            )

    find_library(GUROBI_LIBRARY
            NAMES gurobi gurobi45 gurobi46 gurobi50 gurobi51 gurobi52 gurobi55 gurobi56 gurobi60 gurobi65 gurobi70 gurobi75 gurobi80 gurobi81 gurobi90 gurobi95
            PATHS "$ENV{GUROBI_HOME}/lib"
            )

    find_library(GUROBI_CXX_LIBRARY
            NAMES gurobi_c++
            PATHS "$ENV{GUROBI_HOME}/lib"
            )

    set(GUROBI_INCLUDE_DIRS "${GUROBI_INCLUDE_DIR}")
    set(GUROBI_LIBRARIES "${GUROBI_CXX_LIBRARY};${GUROBI_LIBRARY}")

    set(GUROBI_COMPILER_FLAGS "-DIL_STD" CACHE STRING "Gurobi Compiler Flags")

    include(FindPackageHandleStandardArgs)
    find_package_handle_standard_args(GUROBI DEFAULT_MSG
            GUROBI_LIBRARY GUROBI_CXX_LIBRARY GUROBI_INCLUDE_DIR)
    mark_as_advanced(GUROBI_INCLUDE_DIR GUROBI_LIBRARY GUROBI_CXX_LIBRARY)

endif (GUROBI_INCLUDE_DIR)
```

- Note: I am using Gurobi v. 95, if you are using a newer version, make sure 
that a line starting with `NAMES gurobi gurobi45 gurobi46... ` contains your version



## Build Gurobi library using minGW
- Go to `$GUROBI HOME/src/build/`
- Make copy of `Makefile` ane name it `Makefile_old` in case of something goes wrong 
- Change whole `Makefile` to:

```make
C++ 		= g++
C++FLAGS 	= -m64 -O
C++OBJS 	=  Env.o Model.o attrprivate.o Var.o Constr.o LinExpr.o QuadExpr.o Exception.o Callback.o Column.o SOS.o QConstr.o GenConstr.o TempConstr.o Batch.o

c++: libgurobi_c++.a

%.o: ../cpp/%.cpp ../cpp/%.h
	$(C++) $(C++FLAGS) -I../../include -c $<

libgurobi_c++.a: $(C++OBJS)
	ar rv libgurobi_c++.a $(C++OBJS)

clean:
	rm -f *.o libgurobi_c++.a
```
Note: I am using gurobi v. 95 if you are a different version please open the original `Makefile_old` 
and check that all files from `CPPMTOBJS` are also in `C++OBJS`. 
In the new makefile suffix of the files is changed form `*.OBJ` to `*.o`

- open cmd and type `cd $GUROBI HOME/src/build/` 
- `mingw32-make` - Build files
- `copy "libgurobi_c++.a" ..\..\lib`

- Clear the cmake cache in CLion using: Tools -> CMake -> Reset Cache and Reload
- Build and run your project 


