title: cmake

# **CMake**

## **Debuging**

* Debug output: `-LAH --debug-output --trace-expand`
* Variable changes in cmake: `#!cmake variable_watch(CMAKE_INSTALL_LIBDIR)`

### Dependency graph --- [[1](https://stackoverflow.com/a/30803359/758986)]:

Option 1:
```cmake
set_property(GLOBAL PROPERTY GLOBAL_DEPENDS_DEBUG_MODE 1)
```

Option 2:
```bash linenums="1"
cmake --graphviz=test.graph 
dotty test.graph
```




## **Main commands**

* [`add_executable`](https://cmake.org/cmake/help/latest/command/add_executable.html)`(tests testmain.cpp)`
or [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html)`(catch INTERFACE)`
    * [`target_compile_features`](https://cmake.org/cmake/help/latest/command/target_compile_features.html#command:target_compile_features)`(catch INTERFACE cxx_std_11)`
    * [`set_target_properties`](https://crascit.com/2015/03/28/enabling-cxx11-in-cmake/)`(myTarget PROPERTIES CXX_STANDARD 11 CXX_STANDARD_REQUIRED YES CXX_EXTENSIONS NO)`
* [`target_include_directories`](https://cmake.org/cmake/help/latest/command/target_include_directories.html)`(catch INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include)`
* [`target_compile_definitions`](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html)`(tests PRIVATE CATCH_CONFIG_CONSOLE_WIDTH=60)`
* [`target_link_libraries`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#libraries-for-a-target-and-or-its-dependents)`(<target> <PRIVATE|PUBLIC|INTERFACE> <item>... [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)`



## **CMake for monorepos**

### The problem

Sometimes the default CMake approach is too monolithic. This is especially relevant in case of monorepos
(which tend to be large and include sub-project of multiple teams, which you do not want to build at all).

* For example, with the default cmake, given a monorepo, I cannot build a specific (_single_) sub-project.
  I can only build the entire repo.
* Another (even more important) consequence is that the decision as to whether to add `#!cmake add_subdirectory(dependencyA)`
  in the given `CMakeLists.txt` or not, *is not local*. I know that my target depends on `dependencyA`, but
  I do not know (_locally_, in the given `CMakeLists.txt`) whether it has already been included or not by a
  parent `CMakeLists.txt`.
* Finally, with the default cmake approach I have to remember both: (1) target name
  (to specify it in `#!cmake target_link_libraries()`) and (2) target path (to specify it in `#!cmake add_subdirectory()`).
  Alternatively, we could auto-generate target names based on its path, and require to remember only paths.


### The goal

The goal is to be able to write CMakeLists in this way:

* Adding an executable:

    ```cmake linenums="1" title="my/super/project/se2/CMakeLists.txt"
    add_exec(se2 main.cpp)
    depends_upon(nested/util)    # <-- Just list dependencies of the executable, regardless of
    depends_upon(my/super/lib)   #     whether they were included by someone else or not
    ```

* Adding a library:

    ```cmake linenums="1" title="my/super/lib/CMakeLists.txt"
    add_static_library(pci.h pci.cc)   # <-- no need to come up with library name
    depends_upon(nested/util)
    ```

* And configure in this way:

    ```bash
    cmake root/cmake/dir -DDIRS_TO_BUILD="my/super/project/se2"  # <-- specify only one sub-project to build
    ```


### A prototype

Implementation of the functions is pretty straighforward --- we just need:

* to maintain a list of processed directories (`processed_paths`)
* and to introduce our own `#!cmake depends_upon(directory)` that will `#!cmake add_subdirectory(directory)`
  only if it has not yet been added in the `processed_paths`:


```cmake linenums="1" title="root CMakeLists.txt" hl_lines="1 1"
set(processed_paths "" CACHE INTERNAL "")

macro(set_if_not_set variable)
   if(NOT DEFINED ${variable})
      set(${variable} ${ARGN})
   endif()
endmacro()


function(get_default_target_name ret)
   file(RELATIVE_PATH relative_path ${PROJECT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR})
   string(REPLACE "/" "-" dashed_name ${relative_path})
   set(${ret} "${dashed_name}" PARENT_SCOPE)
endfunction()


macro(add_static_library)
   get_default_target_name(default_target_name)
   set_if_not_set(current_target ${default_target_name})
   message("SubProjectBuilder: add_static_library: Adding STATIC library ${current_target}, with files: ${ARGV}")
   add_library(${current_target} STATIC ${ARGV})
endmacro()


macro(add_exec name)
   set(current_target ${name})
   message("SubProjectBuilder: add_exec: Adding executable ${ARGV}")
   add_executable(${ARGV})
endmacro()


function(add_subdirectory_if_not_added name)
   # message("SubProjectBuilder: add_subdirectory_if_not_added: ARGV: ${ARGV}, processed_paths: ${processed_paths}")
   string(REPLACE "/" "-" dashed_name ${name})
   if("${processed_paths}" MATCHES "X${dashed_name}X")
      # message("SubProjectBuilder: add_subdirectory_if_not_added: dashed_name: ${dashed_name} found in the list of processed paths, ignoring it...")
      return()
   endif()

   # message("SubProjectBuilder: add_subdirectory_if_not_added: dashed_name: ${dashed_name} was not found in the list of processed paths, adding it...")
   set(local_processed_paths ${processed_paths})
   list(APPEND local_processed_paths "X${dashed_name}X")
   # message("SubProjectBuilder: add_subdirectory_if_not_added: local_processed_paths: ${local_processed_paths}")
   set(processed_paths "${local_processed_paths}" CACHE INTERNAL "")

   set(backup_current_target ${current_target})
   unset(current_target)
   message("SubProjectBuilder: add_subdirectory_if_not_added: recursing to: ${name}")
   add_subdirectory(${PROJECT_SOURCE_DIR}/${name} ${PROJECT_BINARY_DIR}/${name})
   set(current_target ${backup_current_target})
endfunction()

function(depends_upon paths)
   # message("SubProjectBuilder: depends_upon: name: ${current_target}, path: ${path}")
   foreach(path IN LISTS ARGV)
      add_subdirectory_if_not_added(${path})
      string(REPLACE "/" "-" dashed_name ${path})
      target_link_libraries(${current_target} ${dashed_name})
   endforeach()
endfunction()



# =================================================================================================


include_directories(.)

foreach(target_name IN LISTS DIRS_TO_BUILD)
   add_subdirectory_if_not_added(${target_name})
endforeach()
```
