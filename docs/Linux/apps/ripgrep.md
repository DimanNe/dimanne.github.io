title: ripgrep

# **ripgrep**

[guide](https://github.com/BurntSushi/ripgrep/blob/master/GUIDE.md)

* **Example 1**:

    ```fish
    #   (1)  (2)
    rg -uu -tcpp -tc -tcmake -tmake  "IImpl::topoffBuffers" .
    ```

    1. Do not ignore gitignore, do not ignore hidden files
    2. We are interested in all files related to cpp, c, make, and CMake

* **Print capture group only**: `rg '.*large1x1---(.*)---97' -or '$1'`

* **Pattern is not a regexp**: `-F`
