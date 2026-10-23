# Forgejo: Btrfs backup and isolated restore

## Design

Forgejo runs in Docker Compose on the Debian `storage` VM. It uses a rootless application image and SQLite, with persistent repositories, database and configuration under `/srv/storage/ssd-b/forgejo`. A different physical SSD is mounted at `/srv/storage/ssd-a` for backup storage.

The backup workflow uses an **application-consistent snapshot** rather than copying an active SQLite database. Its script and systemd units are included in [`examples/backup/forgejo`](../../examples/backup/forgejo/README.md).

## Backup sequence

1. Take a `flock` lock and verify source/target mounts and the Forgejo container.
2. Find the latest snapshot already replicated to the destination.
3. Briefly stop Forgejo, create a **read-only Btrfs snapshot** on SSD-B, and restart the application.
4. Send the snapshot to SSD-A using `btrfs send` for a full copy or `btrfs send -p` for an incremental copy.
5. Copy the Compose configuration alongside the backup, compare the received files with the source snapshot and record `BACKUP_OK`.
6. Run the script through a systemd oneshot service and daily timer.

The script has an exit trap that attempts to restart Forgejo if a failure occurs during the stop/snapshot section.

## Isolated recovery exercise

A received snapshot on SSD-A was cloned as a writable test snapshot. A separate Forgejo container started using that data with `--network none`, no published ports, and mailer, SSH and actions disabled in its test configuration.

| Check | Result |
| --- | --- |
| File comparison | Source snapshot and received copy matched with `diff -qr` |
| Production application after backup | Container running; HTTP 200 |
| Recovered web interface | HTTP 200 inside the isolated container's network namespace |
| Recovered SQLite database | `PRAGMA integrity_check` returned `ok` |
| Repository metadata | Six repository records available in recovered database |
| Incremental backups | Manual run and a timer-triggered run completed with `BACKUP_OK` |

## Operational checks

```bash
systemctl list-timers --all forgejo-btrfs-backup.timer
sudo journalctl -u forgejo-btrfs-backup.service --no-pager -n 30
sudo docker inspect -f '{{.State.Status}}' forgejo
```

The timer should be active; successful backup jobs log `BACKUP_OK`; the production container should report `running`.
