#### Multi-Sync: Enhanced Bandwidth Utilization with Multi-Stream Rsync

### Overview
Multi-Sync is an advanced Python wrapper around rsync, designed to maximize bandwidth usage during data transfers. Building on the foundation of msrsync, it adds support for handling remote target directories.

The tool efficiently splits transfers into multiple buckets during the source scan, enabling parallel processing and optimal bandwidth utilization. It provides two modes to make the best use of system resources:
- **One-to-One Multi-Sync:** Ideal for synchronizing data between a single source and a remote destination.
- **Many-to-One Multi-Sync:** Designed for scenarios where multiple source machines share access to the same data and buckets (metadata) via NFS mount points or similar configurations. This mode performs parallel rsync operations from all source machines to a single destination, maximizing bandwidth utilization and efficiently leveraging system resources.

The tool is lightweight, requiring only Python (version ≥ 2.6) and rsync, and supports both local and remote target directories, unlike the original msrsync which was limited to local disk or mounted directories (NFS/CIFS/other).

---

### Motivation
Why multi-sync if msrsync already does parallel synchronization?

While msrsync optimizes bandwidth by running multiple parallel rsync processes for local transfers, it lacks the ability to handle remote target directories. Multi-Sync addresses this limitation by supporting both local and remote target directories, allowing for more flexible and scalable data transfers.

Multi-Sync also introduces advanced synchronization modes, such as one-to-one and many-to-one, which can further optimize system resource usage and improve performance when dealing with multiple remote destinations. This makes Multi-Sync a more versatile tool for modern data migration needs.

---

### Requirements
- python3
- rsync
- fabric
- invoke


### Installation
multi-sync is available as a Debian package. Simply download and install it using dpkg:

```bash
$ git clone https://github.com/komal-24/multi-sync.git
$ cd multi-sync/
$ sudo dpkg -i multi-sync.deb
```


### Configuration

## One-to-One Multi-Sync
To set up One-to-One synchronization, update the configuration files located in the directory:
```
/etc/multi-sync
```

1. **src_dest_path.txt:** Specify the source and destination directory paths:
   ```
   /path/to/local/directory/
   user@remote_host:/path/to/remote/directory/
   ```

2. **buckets_path.txt (optional):** Specify the absolute path for storing buckets:
   ```
   /mnt/nfs/buckets/
   ```
   - If `buckets_path.txt` is empty, the default bucket path will be used:
    

## Many-to-One Multi-Sync
To set up Many-to-One synchronization, update the following configuration files located in:
```
/etc/multi-sync
```

1. **src_dest_path.txt:** Specify the source and destination directory paths:
   ```
   /path/to/local/directory/
   user@remote_host:/path/to/remote/directory/
   ```

2. **src_ips_hosts.txt:** Contains the IP addresses and usernames of the source machines:
   ```
   ip_address_1,username_1
   ip_address_2,username_2
   ip_address_3,username_3
   ```

3. **buckets_path.txt (mandatory):** Specify the absolute path (like NFS or similar) where the buckets will be stored:
   ```
   /path/to/nfs/directory/
   ```
   - If `buckets_path.txt` is empty, the default bucket path will be used:
    


### Usage
```
Usage: multi-sync -m [one-to-one|many-to-one] [options] [--rsync 'rsync-options-string']
```

## Modes:
```
  -m, --mode ...        specify the synchronization mode [one-to-one, many-to-one]
```
## multi-sync Options (same as msrsync):
```
  -p, --processes ...   number of rsync processes to use [1]  
  -f, --files ...       limit buckets to <files> files number [1000]  
  -s, --size ...        limit partitions to BYTES size (1024 suffixes: K, M, G, T, P, E, Z, Y) [1G]  
  -b, --buckets ...     where to put the buckets files (default: auto temporary directory)  
  -k, --keep            do not remove buckets directory at the end  
  -j, --show            show bucket directory  
  -P, --progress        show progress  
  --stats               show additional stats  
  -d, --dry-run         do not run rsync processes  
  -v, --version         print version  
```

## Rsync Options:
```
  -r, --rsync ...       MUST be last option. rsync options as a quoted string ['-aS --numeric-ids']
```

## Self-Test Options:
```
  -t, --selftest        run the integrated unit and functional tests  
  -e, --bench           run benchmarks  
  -g, --benchshm        run benchmarks in /dev/shm or the directory in $SHM environment variable  
```



### Examples:

## One-to-One Sync:
```
multi-sync -m one-to-one -P -p 4 --show --keep --rsync "--inplace -avzpP -e 'ssh'"
```

## Many-to-One Sync:
```
multi-sync -m many-to-one -P -p 4 --show --keep --rsync "--inplace -avzpP -e 'ssh'"
```

---


### Usage Examples

## One-to-One Multi-Sync
1. To set up One-to-One synchronization, update the `src_dest_path.txt` file with the source and destination directory paths:
```
$ vi /etc/multi-sync/src_dest_path.txt
/home/user/data/local_dir/
user@192.168.1.10:/home/user/data/remote_dir/
```
2. Optionally, specify the bucket path in `buckets_path.txt` (absolute path):
```
$ vi /etc/multi-sync/buckets_path.txt
/mnt/buckets/
```
If `buckets_path.txt` is empty, the default path will be used:

Run the sync operation:
```
$ multi-sync -m one-to-one -P -p 4 --show --keep --rsync "--inplace -avzpP -e 'ssh'"
```

## Many-to-One Multi-Sync
To set up Many-to-One synchronization, update the following configuration files:

1. **src_dest_path.txt:**
```
$ vi /etc/multi-sync/src_dest_path.txt
/home/backup/data/
user@192.168.1.20:/home/central/data/
```

2. **src_ips_hosts.txt:** Contains the IP addresses and usernames of the source machines.
```
$ vi /etc/multi-sync/src_ips_hosts.txt
192.168.1.11,backup_user1
192.168.1.12,backup_user2
192.168.1.13,backup_user3
```

3. **buckets_path.txt (mandatory):** Specifies the absolute path (like NFS or similar) where the buckets will be stored.
```
$ vi /etc/multi-sync/buckets_path.txt
/mnt/nfs/buckets/
```
If `buckets_path.txt` is empty, the synchronization will proceed but use the default bucket path:

Run the sync operation:
```
$ multi-sync -m many-to-one -P -p 4 --show --keep --rsync "--inplace -avzpP -e 'ssh'"
```

---


### Notes

Performance may degrade if the source or destination cannot handle parallel I/O. To optimize performance:

Monitor performance with tools like `iotop`, `nload`, or `iftop` to identify bottlenecks and make necessary adjustments. Ensuring sufficient memory and CPU resources also helps maintain stable synchronization.


