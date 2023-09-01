title: Tools & Utils

# **Tools & Utils**

## **Duplicate files**

Find: `find ~/Videos/ -type f -print0 | xargs -0 -P 16 -L 1 md5sum | sort | uniq -D -w 32`


Delete duplicates:

1. Save the output of the above command in a file (db.txt), and then
2. Execute this:

    ```fish title="nano ~/remove-duplicates.fish"
    #!/usr/bin/env fish

    set db_path $argv[1]

    if test -z $db_path
        echo "Please provide the path to the file that contains the MD5 hashes and file paths."
        exit 1
    end

    set prev_seen_hash ""

    function maybe_delete_file --argument-names file
        if test -z $DELETE_DUPLICATES_DRY_RUN
            echo "Deleting \"$file\"..."
            set fish_trace 1
            rm -f "$file"
            set fish_trace 0
        else
            echo "Would delete (dry-run): $argv"
        end
    end

    cat $db_path | while read -l line
        set -l components (string split " " -- $line)
        set curr_hash $components[1]
        set curr_file $components[3..-1]

        if test "$prev_seen_hash" = "$curr_hash"
            maybe_delete_file $curr_file
        end
        set prev_seen_hash $curr_hash
    end
    ```


## **Compression**

=== "zstd"

    A directory:
    ```bash
    # Compress:
    tar -I "zstd --threads 30 --ultra -19 --memory 8192" -cf scripts.tar.zst scripts/

    # Decompress:
    tar --zstd -xf scripts.tar.zst
    ```

    A single file:

    ```bash
    # Compress:
    zstd --compress --threads 30 --ultra -19 --memory 8192 -o file.txt.zstd file.txt

    # Decompress:
    zstd --decompress file.txt.zstd
    ```

A couple of observations:

* zstd uses cores up to the compression level of 19 (including).
* `--ultra -22` gives the best compression



## **Alternatives**

Add clang alternatives:

```bash linenums="1"
sudo update-alternatives                                             \
      --install /usr/bin/clang   clang   /usr/bin/clang-3.8     50   \
      --slave   /usr/bin/clang++ clang++ /usr/bin/clang++-3.8        \
      --slave   /usr/bin/lldb    lldb    /usr/bin/lldb-3.8           \
      --slave   /usr/bin/llvm-symbolizer llvm-symbolizer /usr/bin/llvm-symbolizer-3.8
```

Display info about given alternative:

```bash linenums="1"
update-alternatives --display clang
clang - auto mode
  link currently points to /usr/bin/clang-3.5
/usr/bin/clang-3.5 - priority 50
  slave clang++: /usr/bin/clang++-3.5
  slave lldb: /usr/bin/lldb-3.5
Current 'best' version is '/usr/bin/clang-3.5'.
```



## **Permissions**

Restore default permissions (`x` for dirs, no `x` for files):

=== "chmod with `X`"

    ```bash
    chmod -R u=rwX,g=rwX,o=rwX ${HOME}
    ```

    `X` permission: execute/search only if the file is a directory or already has execute permission for some user.

=== "find + chmod"

    ```bash linenums="1"
    find ${HOME} -type f -exec chmod 0664 {} +
    find ${HOME} -type d -exec chmod 0775 {} +
    ```

    Note, unlike `X`, it will remove `x` permission from files.

=== "ACL"

    ```bash
    setfacl -dm u::rws,g::rwx,o::rx /dir/
    ```



## **initrd.img**

* Extract `initrd.img`: `unmkinitramfs /boot/initrd.img-5.15.0-30-generic ../new-init/`






## **Hash of a directory**

```bash
find ./ -type f -exec sha256sum {} + | sha256sum
```


## **Mouse scroll speed**

See [askubuntu](https://askubuntu.com/questions/285689/increase-mouse-wheel-scroll-speed/621140#621140).
