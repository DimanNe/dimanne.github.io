title: systemd

# **systemd**

## **Before= vs After=**

These directives establish an order in which units are started or stopped. If you think of units as nodes in a graph, `Before=` and `After=`
specify direction of an edge between nodes (units). Edge itself corresponds to activation (start) or deactivation (stop).

If we say `UnitA.service` has `After=UnitB.service` in its unit file, this creates a directed edge from `UnitB` to `UnitA`, meaning `UnitA` should
be started after `UnitB`.

It's important to note that these directives only set up ordering relationships and don't cause the units mentioned to be activated or deactivated,
which is why, there are:

## **Wants= and Requires=**

If `UnitA.service` `Wants=UnitB.service`, it means that if `UnitA` is activated, systemd will also attempt to activate `UnitB` (if it isn't already active).
However, if `UnitB` fails to activate, `UnitA` will still be started. In other words, this sets up a "weak" dependency edge in the graph from `UnitA` to `UnitB`.

If `UnitA.service` `Requires=UnitB.service`, it's a stronger version of `Wants=`. Now if `UnitB` fails to start, `UnitA` will not be started either.

## **Summary**

* `Wants= and Requires=` introduces an **undirected edge** between units. It means, that if one of them is activated, the other will be too, but the order is
  unspecified.
* `Before= vs After=` creates a **directed edge** between units, but it does so, without forcing systemd into activating its dependency.
  If unit `A` specified `After=X`, **without using Wants=/Requires=**, and then then we start the unit A, then systemd will not start unit X.
  If unit X happens to start due to some reason, then unit `A` will be started too.

In other words:

* `Wants=` and `Requires=` can be used without `Before=` and `After=`, and this would imply that the order in which these units start is not determined.
  They will be started in parallel, if possible.
* `Before=` and `After=` can be used without `Wants=` or `Requires=` as well. This might be useful in cases where you know that another service may start
  a certain unit, and you want to ensure that, **if that happens**, your unit starts before or after it. But without a `Wants=` or `Requires=` directive,
  the `Before=` and `After=` directives by themselves won't cause the other unit to start.


## **PartOf=**

This directive creates a relationship where if the unit specified in `PartOf=` is stopped or restarted, then the unit with the `PartOf=` directive is
also stopped or restarted.
