title: Brendan Gregg

# [**Brendan Gregg**](https://www.brendangregg.com/)


??? note "vmstat output description"

    **Procs (Processes)**

    * `r`: The number of runnable processes (running or waiting for run time).
    * `b`: The number of processes in uninterruptible sleep. These are processes that are waiting for some I/O access,
      like disk or network activity.

    **Memory**

    * `swpd`: The amount of virtual memory used (in KB).
    * `free`: The amount of idle memory (in KB).
    * `buff`: The amount of memory used as buffers (in KB).
    * `cache`: The amount of memory used as cache (in KB).

    **Swap**

    * `si`: Amount of memory swapped in from disk (in KB per second).
    * `so`: Amount of memory swapped out to disk (in KB per second).

    **I/O**

    * `bi`: Blocks received from a block device (blocks input per second).
    * `bo`: Blocks sent to a block device (blocks output per second).

    **System**

    * `in`: The number of interrupts per second, including the clock.
    * `cs`: The number of context switches per second.

    **CPU**

    * `us`: User time. The time the CPU has spent running users' processes that are not niced.
    * `sy`: System time. The time the CPU has spent running the kernel and its processes.
    * `id`: Idle time. The time the CPU has spent doing nothing.
    * `wa`: Wait time. The time the CPU has spent waiting for I/O to complete.
    * `st`: Steal time. The amount of CPU 'stolen' from this virtual machine by the hypervisor for other tasks
      (such as running another virtual machine).



??? note "sar -q output description"

    * `runq-sz`: This is the run queue size (the number of tasks waiting for run time, the number of processes that are
      currently executing or waiting to execute).
    * `plist-sz`: This is the number of tasks in the task list: the total number of tasks managed by the Linux scheduler.
      This includes running tasks, sleeping tasks, stopped tasks, and zombies.


--------------------------------------------------------------------------------------------------------------
## [**USE Method**](https://www.brendangregg.com/usemethod.html) & [**Checklist**](https://www.brendangregg.com/USEmethod/use-linux.html)

Terminology definitions:

* resource: all physical server functional components (CPUs, disks, busses, ...) [1]
* utilization: the average time that the resource was busy servicing work [2]
* saturation: the degree to which the resource has extra work which it can't service, often queued
* errors: the count of error events


### **CPU**

=== "utilization"

    * **system-wide**:
        * `vmstat 1`, sum: `us` + `sy` + `st`
        * `sar -u`, sum fields except `%idle` and `%iowait`
        * `dstat -c`, sum fields except `idl` and `wai`;
    * **per-cpu**:
        * `mpstat -P ALL 1`, sum fields except `%idle` and `%iowait`
        * `sar -P ALL`, same as mpstat
    * **per-process**:
        * `top`, check: `%CPU`
        * `htop`, check: `CPU%`
        * `ps -o pcpu`
        * `pidstat 1`, check: `%CPU`
    * **per-kernel-thread**:
        * `top`/`htop` (`K` to toggle), where `VIRT == 0` (heuristic)


=== "saturation"

    * **system-wide**:
        * `vmstat 1`, where `r` > CPU count
        * `sar -q`, where `runq-sz` > CPU count
        * `dstat -p`, where `run` > CPU count
    * **per-process**:
        * `/proc/PID/schedstat` 2nd field (sched_info.run_delay);
        * `perf sched latency` (shows "Average" and "Maximum" delay per-schedule)
        * dynamic tracing, eg, SystemTap schedtimes.stp "queued(us)

=== "errors"

    * perf (LPE) if processor specific error events (CPC) are available eg, AMD64's "04Ah Single-bit ECC Errors Recorded by Scrubber"



### **Memory**

=== "utilization"

    * **system-wide**:
        * `free -m`, check: `Mem:` (main memory), `Swap:` (virtual memory)
        * `vmstat 1`, check: `free` (main memory), `swap` (virtual memory)
        * `sar -r`, check: `%memused`
        * `dstat -m`, check: `free`
        * `slabtop -s c` for kmem slab usage
    * **per-process**:
        * `top`/`htop`, `RES` (resident main memory), `VIRT` (virtual memory), `Mem` for system-wide summary

=== "saturation"

    * **system-wide**:
        * `vmstat 1`, `si`/`so` (swapping)
        * `sar -B`, `pgscank` + `pgscand` (scanning)
        * `sar -W`
    * **per-process**:
        * 10th field (`min_flt`) from `/proc/PID/stat` for minor-fault rate, or dynamic tracing [5]
        * OOM killer: `dmesg | grep killed`

=== "errors"

    * `dmesg` for physical failures
    * dynamic tracing, eg, SystemTap uprobes for failed malloc()s


### **Network**


=== "utilization"

    * `sar -n DEV 1`, `rxKB/s`/max `txKB/s`/max
    * `ip -s link`, RX/TX tput / max bandwidth
    * `/proc/net/dev`, `bytes` RX/TX tput/max
    * `nicstat`, `%Util` [6]

=== "saturation"

    * `ifconfig`, `overruns`, `dropped`
    * `netstat -s`, `segments retransmited`
    * `sar -n EDEV`, drop and fifo metrics
    * `/proc/net/dev`, RX/TX `drop`
    * `nicstat` `Sat` [6]
    * dynamic tracing for other TCP/IP stack queueing [7]

=== "errors"

    * `ifconfig`, `errors`, `dropped`
    * `netstat -i`, `RX-ERR`/`TX-ERR`
    * `ip -s link`, `errors`
    * `sar -n EDEV`, `rxerr/s` `txerr/s`
    * `/proc/net/dev`, `errs`, `drop`
    * extra counters may be under /sys/class/net/...
    * dynamic tracing of driver function returns 76]




### **Storage device I/O**

=== "utilization"

    * **system-wide**:
        * `iostat -xz 1`, `%util`
        * `sar -d`, `%util`
    * **per-process**:
        * `iotop`
        * `pidstat -d`
        * `/proc/PID/sched` `se.statistics.iowait_sum`

=== "saturation"

    * `iostat -xnz 1`, `avgqu-sz` > 1, or high `await`
    * `sar -d` same
    * LPE block probes for queue length/latency
    * dynamic/static tracing of I/O subsystem (incl. LPE block probes)


=== "errors"

    * smartctl
    * dynamic/static tracing of I/O subsystem response codes [8]
