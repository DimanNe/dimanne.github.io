title: screen

# **screen**


* **Create session or re-attach**: `screen -dRS my_name`
* **Detach**: `Ctrl+A, Ctrl+D`
* **Rename session**: `Ctrl + a`, then: `:sessionname mySessionName<ENTER>`


### Enable scroll

```linenums="1" title="nano ~/.screenrc"
defscrollback 100000
termcapinfo xterm* ti@:te@
```
