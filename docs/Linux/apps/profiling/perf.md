title: perf

# **perf**

[Brendan Gregg's blog](https://www.brendangregg.com/perf.html) about perf.


## **Prerequisites**

If you want to see kernel symbols: `#!bash sudo sysctl -w kernel.kptr_restrict=0`

Make sure you use these flags during compilation: `-g2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer`


## **Counting events** --- `perf stat -e`

It can count various (primarily cpu) events.

* List all events: `#!bash perf list`.
* Example: `#!bash perf stat -e cycles,instructions,cache-references,cache-misses,bus-cycles -a sleep 10`.


## **Profiling / Sampling** --- `perf top` & `perf record`

### perf top

You can use `perf` without recording `perf.data` (similarly to `top`/`htop`):

```bash
sudo perf top -t 739201 -d 5 --call-graph fp
```
where:

* `-t` --- thread id, or `#!fish -p (pgrep -f "^/home/.*/bin/worker ")` --- process id.
* `-d` --- update frequency in seconds.
* `--call-graph dwarf` --- collect and show backtraces.


### Record `perf.data`

Alternatively, you can record `perf.data` and then investigate it.

##### Automatically start and finish

Run the command below in one terminal. It will wait for "signal" to start recording, and then for another "signal" to
stop recording).

=== "Bash"

    ```bash
    nc -l 12345; echo "Starting"; perf record -F 2987 --call-graph fp -p `pgrep -f "ybd/worker"` -g & PID=$!; echo "Waiting for cmd to finish $PID..."; nc -l 12345; kill -15 $PID
    ```

=== "Fish"

    ```bash
    nc -l 12345; echo "Starting"; sudo perf record -F 2987 --call-graph fp -p (pgrep -f "^/home/dimanne/devel/build/.*/bin/worker") -g & set PID (jobs -l -p | head -n 2); echo "Waiting for cmd to finish $PID..."; nc -l 12345; sudo kill -15 $PID
    ```

In a second terminal, run the command wrapped in `nc` (the `nc` will signal to the first command to start and stop recording):

=== "nc without `-q`"

    ```bash
    echo "" | nc 192.168.10.11 12345; ybsql -d yellowbrick_test -c "..." > /dev/null; echo "" | nc 192.168.10.11 12345
    ```

=== "nc with `-q`"

    ```bash
    echo "" | nc -q 0 172.17.0.1 12345; ybsql -d yellowbrick_test -c "..." > /dev/null; echo "" | nc -q 0 172.17.0.1 12345
    ```

After it finishes, `perf record` started on the first terminal should produce `perf.data` file.

##### CPU counters

Alternatively, it is possible to sample/record a specific counter:
```bash
# Sample CPU stack traces, once every 10,000 Level 1 data cache misses, for 5 seconds:
perf record -e L1-dcache-load-misses -c 10000 -ag -- sleep 5
```

### View `perf.data`

##### Flamegraph

=== "Bash"

    ```bash
    bn="gen2-worker5-F2987"; mw=0.01; suffix="$mw"; perf script | ./FlameGraph/stackcollapse-perf.pl > $bn.perf.data.folded; \
    cat $bn.perf.data.folded | ./FlameGraph/flamegraph.pl --minwidth $mw --width 2250 --title "$bn $suffix" --reverse --inverted > ./$bn-$suffix.perf-icicle.svg && \
    cat $bn.perf.data.folded | ./FlameGraph/flamegraph.pl --minwidth $mw --width 2250 --title "$bn $suffix" > ./$bn-$suffix.perf-flame.svg
    ```

=== "Fish"

    ```bash
    set bn "gen2-F2987.1"; set mw 0.01; set suffix "$mw"; perf script | ~/devel/FlameGraph/stackcollapse-perf.pl > $bn.perf.data.folded; \
    cat $bn.perf.data.folded | ~/devel/FlameGraph/flamegraph.pl --minwidth $mw --width 2250 --title "$bn $suffix" --reverse --inverted > ./$bn-$suffix.perf-icicle.svg && \
    cat $bn.perf.data.folded | ~/devel/FlameGraph/flamegraph.pl --minwidth $mw --width 2250 --title "$bn $suffix" > ./$bn-$suffix.perf-flame.svg
    ```

??? question "What is FlameGraph?"

    * Read more about FlameGraph [here](https://www.brendangregg.com/flamegraphs.html).    
    * Get flame graph: `git clone git@github.com:brendangregg/FlameGraph.git`
    * You probably want to make this adjustment (otherwise it collapses different template C++ functions, which should not be collapsed):
    
    ```diff
    diff --git a/stackcollapse-perf.pl b/stackcollapse-perf.pl
    index fd3c78e..afc7be9 100755
    --- a/stackcollapse-perf.pl
    +++ b/stackcollapse-perf.pl
    @@ -79,8 +79,8 @@ my $include_pname = 1;        # include process names in stacks
    my $include_pid = 0;   # include process ID with process name
    my $include_tid = 0;   # include process & thread ID with process name
    my $include_addrs = 0; # include raw address where a symbol can't be found
    -my $tidy_java = 1;     # condense Java signatures
    -my $tidy_generic = 1;  # clean up function names a little
    +my $tidy_java = 0;     # condense Java signatures
    +my $tidy_generic = 0;  # clean up function names a little
    my $target_pname;      # target process name from perf invocation
    my $event_filter = "";    # event type filter, defaults to first encountered event
    my $event_defaulted = 0;  # whether we defaulted to an event (none provided)
    ```

##### perf report

Use one of the:
```bash
perf report
perf report --stdio
perf report -g 'graph,0.5,caller'
```

##### perf annotate

```bash
perf annotate
```



## **Tracing / Probes** --- `perf probe --add`

The main idea behind dynamic tracing is that sometimes there is no existing coutners / function, so you can add your
own probes in both Linux kernel and user code, and then use tracing (`perf record -e`). You can add them dynamically
(without recompilation of code) as well as statically (code has to be modified).

See [`perf-probe` man](https://man7.org/linux/man-pages/man1/perf-probe.1.html) for more info.

### Collect `perf.data`

Let's say our binary/library that has a symbol that we are interested in, is:
=== "Bash"
    ```bash
    lib=$HOME/devel/build/archimedes-qtc-Gen2/lib/libkernelLib.so
    ```

=== "Fish"

    ```fish
    set lib $HOME/devel/build/archimedes-qtc-Gen2/lib/libkernelLib.so
    ```

Firstly, find symbol name that we would like to trace. In case of C programs it easy. In the case of C++ we ask perf to
dump all symbols in mangled form:
```bash linenums="1"
$ perf probe --exec $lib --funcs --no-demangle --filter='*' | grep ArenaAllocator | grep allocate
...
_ZN14ArenaAllocator8allocateEm
```

Then we can inspect:

=== "available variables"

    ```bash linenums="1"
    $ perf probe --exec $lib -V _ZN14ArenaAllocator8allocateEm
    Available variables at _ZN14ArenaAllocator8allocateEm
            @<allocate+0>
                     (unknown_type  ptr
                    ArenaAllocator* this
                    size_t  size
    ```

=== "as well as code"

    ```bash linenums="1"
    $ perf probe --exec $lib -L _ZN14ArenaAllocator8allocateEm
    <_ZN14ArenaAllocator8allocateEm@/home/dimanne/devel/ybd/archimedes/kernel/memory/ArenaAllocator.cc:0>
          0  void *ArenaAllocator::allocate(size_t size) {
          1      assert(size != 0);
          2      ...
    ```

And finally add probe:
```bash linenums="1"
$ sudo perf probe --exec $lib --no-demangle --add "_ZN14ArenaAllocator8allocateEm size"
Target program is compiled without optimization. Skipping prologue.
Probe on address 0x2fefa0 to force probing at the function entry.

Added new event:
  probe_libkernelLib:_ZN14ArenaAllocator8allocateEm (on _ZN14ArenaAllocator8allocateEm in /home/dimanne/devel/build/archimedes-qtc-Gen2/lib/libkernelLib.so with size)

You can now use it in all perf tools, such as:
        perf record -e probe_libkernelLib:_ZN14ArenaAllocator8allocateEm -aR sleep 1
```

And then just record data is it suggested in the output above: `#!bash perf record -e probe_libkernelLib:_ZN14ArenaAllocator8allocateEm -aR sleep 1`.

Cleanup:
```bash
sudo perf probe --list
sudo perf probe --del probe_libkernelLib:_ZN14ArenaAllocator8allocateEm
```


### View `perf.data`

=== "Distribution of sizes"
    ```bash linenums="1"
    $ perf script | awk 'match($0, /.*size=(.*)$/, a) {print strtonum(a[1]) }' | sort | uniq -c | sort -rnk 1 | tr -s ' ' | awk '{print $1 " x " $2 " = " strtonum($1) * strtonum($2)}'
    87829 x 160 = 14052640
    18607 x 136 = 2530552
    4640 x 264 = 1224960
    2319 x 128 = 296832
    1600 x 262144 = 419430400
    1569 x 48 = 75312
    ```

=== "Histogram of sizes"
    ```bash linenums="1"
    $ perf script | awk 'match($0, /.*size=(.*)$/, a) {print strtonum(a[1]) }' | sort | uniq -c | sort -rn | head -40 | awk '!max {max=$1} {r=""; i = 80 * $1 / max; while(--i > 0) r=r"#"; printf "%-75s %s %s",$0,r,"\n";}'
    87829 160                                                                 ###############################################################################
    18607 136                                                                 ################
     4640 264                                                                 ####
     2319 128                                                                 ##
     1600 262144                                                              #
     1569 48                                                                  #
     1528 40                                                                  #
     1463 24                                                                  #
    ```

=== "Largest allocations"
    ```bash linenums="1"
    $ perf script | awk 'match($0, /.*size=(.*)$/, a) {print strtonum(a[1]) }' | sort | uniq -c | sort -rnk 1 | tr -s ' ' | awk '{print $1 " x " $2 " = " strtonum($1) * strtonum($2)}' | sort -rnk 5
    1600 x 262144 = 419430400
    87829 x 160 = 14052640
    4 x 3245125 = 12980500
    4 x 1366344 = 5465376
    2 x 2097152 = 4194304
    ```

=== "Total bytes (sum)"
   ```bash linenums="1"
   $ perf script | awk 'BEGIN {total = 0;} match($0, /.*size=(.*)$/, a) { total += strtonum(a[1]); } END {print total}'
   981479078
   ```
