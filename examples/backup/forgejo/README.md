# Forgejo: backup script and systemd timer

This directory contains the script and unit files used for Btrfs replication of a self-hosted Forgejo installation with SQLite. The source is on SSD-B and received snapshots are kept on SSD-A, a different physical drive.

| File | Purpose |
| --- | --- |
| `forgejo-btrfs-backup` | Mount checks, locking, consistent read-only snapshots, full/incremental Btrfs send/receive, comparison |
| `forgejo-btrfs-backup.service` | systemd oneshot invoking the script |
| `forgejo-btrfs-backup.timer` | Daily 04:30 schedule in the server's local time zone |

## Execution

The script checks that both Btrfs filesystems are mounted and that Forgejo is running. It briefly stops the container to snapshot its SQLite and repository data consistently, restarts the container, then replicates the snapshot to the second SSD. Subsequent runs choose the latest successfully received snapshot as the incremental parent. A `flock` lock avoids concurrent backup jobs.

A successful run logs `BACKUP_OK` after replication and a file comparison. The timer has completed a scheduled run; the received data was also used for a separate isolated restore test.

```bash
systemctl list-timers --all forgejo-btrfs-backup.timer
sudo journalctl -u forgejo-btrfs-backup.service -n 40 --no-pager
sudo btrfs subvolume list -r /srv/storage/ssd-a
```

Expected: the timer appears in the systemd list, the journal contains `BACKUP_OK` for a successful run and the destination contains read-only received snapshots. Paths and container names are specific to this homelab; review them before reusing the script elsewhere.

[Backup design and recovery results](../../../docs/services/forgejo-backup-and-restore.md)
