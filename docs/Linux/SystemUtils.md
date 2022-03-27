title: System utils

# **System utils**

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

## **Hash of a directory**

```bash
find ./ -type f -exec sha256sum {} + | sha256sum
```
