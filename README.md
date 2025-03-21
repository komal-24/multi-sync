#### multi-sync : Enhanced Bandwidth Utilization with Multi-Stream Rsync

### Overview
Multi-Sync is an advanced Python wrapper around rsync, designed to maximize bandwidth usage during data transfers. Building on the foundation of msrsync, it adds support for handling remote target directories.

The tool efficiently splits transfers into multiple buckets during the source scan, enabling parallel processing and optimal bandwidth utilization. It provides two modes to make the best use of system resources:
- **One-to-One Multi-Sync:** Ideal for synchronizing data between a single source and a remote destination.
- **Many-to-One Multi-Sync:** Designed for scenarios where multiple source machines share access to the same data and buckets (metadata) via NFS mount points or similar configurations. This mode performs parallel rsync operations from all source machines to a single destination, maximizing bandwidth utilization and efficiently leveraging system resources.

The tool is lightweight, requiring only Python (version ≥ 2.6) and rsync, and supports both local and remote target directories, unlike the original msrsync which was limited to local disk or mounted directories (NFS/other).

### Installation

multi-sync is a single Python file that you can easily download. Alternatively, clone the repository and use the provided Makefile to install it:

```bash
git clone https://github.com/jbd/multi-sync
cd multi-sync/usr/bin
sudo make install
```

### Usage

```bash
multi-sync -m [one-to-one|many-to-one] [options] [--rsync 'rsync-options-string']
```

#### Modes:
- **-m, --mode ...**   Specify the synchronization mode [one-to-one, many-to-one]

#### multi-sync Options (same as msrsync):
- **-p, --processes ...**   Number of rsync processes to use [1]
- **-f, --files ...**       Limit buckets to <files> files number [1000]
- **-s, --size ...**        Limit partitions to BYTES size (e.g., 1G, 10M) [1G]
- **-b, --buckets ...**     Directory for bucket files (auto temporary directory by default)
- **-k, --keep**            Do not remove the bucket directory after sync
- **-j, --show**            Display the bucket directory
- **-P, --progress**        Show progress during synchronization
- **--stats**               Display additional statistics
- **-d, --dry-run**         Perform a trial run without actual data transfer
- **-v, --version**         Print version information

#### Rsync Options:
- **-r, --rsync ...**       Specify rsync options as a quoted string (e.g., '-aS --numeric-ids')

#### Self-Test Options:
- **-t, --selftest**        Run integrated unit and functional tests
- **-e, --bench**           Run benchmarks
- **-g, --benchshm**        Run benchmarks in /dev/shm or the directory in $SHM environment variable

### Notes

Performance may degrade if the source or destination cannot handle parallel I/O. To optimize performance:
- Use high-speed storage (SSDs or NVMe)
- Optimize network settings
- Balance parallel processes according to available CPU cores and bandwidth

Monitor performance with tools like `iotop`, `nload`, or `iftop` to identify bottlenecks and make necessary adjustments. Ensuring sufficient memory and CPU resources also helps maintain stable synchronization.
