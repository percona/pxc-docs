# GCache encryption and Write-Set cache encryption

These features are [tech preview](../glossary.md#tech-preview). Before using these features in production, we recommend that you test restoring production from physical backups in your environment, and also use the alternative backup method for redundancy.


## GCache and Write-Set cache encryption

Enabling this feature encrypts the Galera GCache and Write-Set cache files with a File Key.  
 
  GCache has a RingBuffer on-disk file to manage write-sets. The keyring only stores the Master Key which is used to encrypt the File Key used by the Ring Buffer file. The encrypted File Key is stored in the Ring Bugger's preamble. The Ring Buffer file of GCache is non-volatile, which means this file survives a restart. The File Key is not stored for GCache off-pages and Write-Set cache files.
 

!!! admonition "See also"

    For more information, see [Understanding GCache and Record-set Cache](../manual/gcache_record-set_cache_difference.md), and the Percona Database Performance Blog: [All you need to know about GCache](https://www.percona.com/blog/2016/11/16/all-you-need-to-know-about-gcache-galera-cache/)


??? example "Sample preamble key-value pairs"

    ```text

    Version: 2
    GID: 3afaa71d-6665-11ed-98de-2aba4aabc65e
    synced: 0
    enc_version: 1
    enc_encrypted: 1
    enc_mk_id: 3
    enc_mk_const_id: 3ad045a2-6665-11ed-a49d-cb7b9d88753f
    enc_mk_uuid: 3ad04c8e-6665-11ed-a947-c7e346da147f
    enc_fk_id: S4hRiibUje4v5GSQ7a+uuS6NBBX9+230nsPHeAXH43k=
    enc_crc: 279433530

    ```

### Key descriptions

The following table describes the encryption keys defined in the preamble. All other keys in the preamble are not related to encryption.

| Key | Description |
|---|---|
| `enc_version` | The encryption version |
| `enc_encrypted` | If the GCache is encrypted or not |
| `enc_mk_id` | A part of the Master Key ID. Rotating the Master Key increments the sequence number. |
| `enc_mk_const_id` | A part of the Master Key ID, a constant Universally unique identifier (UUID). This option remains constant for the duration of the `galera.gcache` file and simplifies matching the Masater Key inside the keyring to the instance that generated the keys. Deleting the `galera.gcache` changes the value of this key. |
| `enc_mk_uuid` | The first Master Key or if Galera detects that the preamble is inconsistent, which causes a full GCache reset and a new Master Key is required, generates this UUID. |
| `enc_fk_id` | The File Key ID encrypted with the Master Key. |
| `enc_crc` | The cyclic redundancy check (CRC) calculated from all encryption-related keys. |

### Controlling encryption

Encryption is controlled using the wsrep_provider_options. 

| Variable name | Default value | Allowed values |
|---|---|---|
| [`gcache.encryption`](#gcacheencryption) | off | on/off |
| [`gcache.encryption_cache_page_size`](#gcacheencryption_cache_page_size) | 32KB | 2-512 |
| [`gcache.encryption_cache_size`](#gcacheencryption_cache_size) | 16MB | 2 - 512  |
| [`allocator.disk_pages_encryption`](#allocatordisk_pages_encryption) | off | on/off |
| [`allocator.encryption_cache_page_size`](#allocatorencryption_cache_page_size) | 32KB |  |
| [`allocator.encryption_cache_size`](#allocatorencryption_cache_size) | 16MB |  |

## Rotate the GCache Master Key 

GCache and Write-Set cache encryption uses either a keyring plugin or a keyring component. This plugin or component must be loaded.

Store the keyring file outside the data directory when using a keyring plugin or a keyring component.

```sql

mysql> ALTER INSTANCE ROTATE GCACHE MASTER KEY;
```

## Variable descriptions

### GCache encryption

The following sections describe the variables related to GCache encryption.
All variables are read-only.

#### gcache.encryption

Enable or disable GCache cache encryption.

#### gcache.encryption_cache_page_size

The size of the GCache encryption page. The value must be multiple of the CPU page size (typically 4kB). If the value is not, the server reports an error and stops.

#### gcache.encryption_cache_size

Every encrypted file has an encryption.cache, which consists of pages. Use `gcache.encryption_cache_size` to configure the encryption.cache size. 

Configure the page size in the cache with [`gcache.encryption_cache_page_size`](#gcacheencryption_cache_page_size). 

The maximum size for the encryption.cache is 512 pages. This value is a hint. If the value is larger than the maximum, the value is rounded down to 512 x gcache.encryption_cache_page_size.

The minimum size for the encryption.cache is 2 pages. If the value is smaller, the value is rounded up.

### Write-Set cache encryption

The following sections describe the variables related to Write-Set cache encryption.
All variables are read-only.

#### allocator.disk_pages_encryption

Enable or disable the Write-Set cache encryption.

#### allocator.encryption_cache_page_size

The size of the encryption cache for Write-Set pages. The value must be multiple of the CPU page size (typically 4kB).  If the value is not, the server reports an error and stops.

#### allocator.encryption_cache_size

Every Write-Set encrypted file has an encryption.cache, which consists of pages. Use `allocator.encryption_cache_size` to configure the size of the `encryption.cache`. 

Configure the page size in the cache with [`allocator.encryption_cache_page_size`](#allocatorencryption_cache_page_size). 

The maximum size for the encryption.cache is 512 pages. This value is a hint. If the value is larger than the maximum, the value is rounded down to 512 x gcache.encryption_cache_page_size.

The minimum size for the encryption.cache is 2 pages. If the value is smaller, the value is rounded up.
