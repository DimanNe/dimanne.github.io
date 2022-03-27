title: Generic

# **Generic**

## Compression

=== "zstd"

    A directory:
    ```bash
    # Compress:
    tar -I "zstd -T16 --ultra -22" -cf scripts.tar.zst scripts/
    
    # Decompress:
    tar --zstd -xf scripts.tar.zst
    ```

    A single file:
    ```bash
    # Compress:
    zstd --compress --threads 16 --ultra -22 -o file.txt.zstd file.txt

    # Decompress:
    zstd --decompress file.txt.zstd
    ```
