title: memory usage

# **Memory usage profiling**

You can find out what function allocated what amount of memory.

## **tcmalloc**

### Build with [tcmalloc](https://gperftools.github.io/gperftools/heapprofile.html)

cmake example:
```cmake
find_library(TCMALLOC_LIB NAMES tcmalloc)
if(TCMALLOC_LIB)
   message("Found TCMALLOC_LIB: ${TCMALLOC_LIB}")
else()
   message(FATAL_ERROR "${TCMALLOC_LIB} library not found")
endif()
link_libraries(${TCMALLOC_LIB})
```


### Run your app

Start your application with the environment variable set to a filename
(to which `tcmalloc` will dump the graph of allocations):

```bash
HEAPPROFILE=mybin.hprof ./google_ngram
```


### View report

The command above will produce files like: `mybin.hprof.0132.heap`, it then can be viewed with:

```bash
google-pprof --gv google_ngram mybin.hprof.0132.heap
```
