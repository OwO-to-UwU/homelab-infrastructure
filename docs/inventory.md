# Infrastructure inventory

## Systems

| System | OS | Hardware and role |
| --- | --- | --- |
| MikroTik hAP ax3 | RouterOS | LAN router, WAN uplinks, DNS, routing policies and NAT |
| Arch Linux host | Arch Linux | ASUS PRIME X570-P; AMD Ryzen 7 5700G (8 cores / 16 threads); 64 GB DDR4-3200; KVM/libvirt |
| `storage` VM | Debian 13 | 4 vCPU, 8 GiB RAM; Docker Compose, Caddy and physical storage management |
| Proxybox (`m900`) | Debian 12 | Intel Core i5-8500T, 6 cores; 8 GB RAM; nftables, B4, sing-box, WireGuard |

## Storage devices

Four 4 TB HDDs are used across the physical hypervisor and the `storage` VM. The VM receives three of them and two SSDs through a passed-through LSI SAS2308 controller. Each physical storage disk has its own filesystem.

| Location | Device | Role / mount point |
| --- | --- | --- |
| Arch host | CUSU CV3500Q 512 GB NVMe | System disk, Btrfs |
| Arch host | Kingston SNVS500G 500 GB NVMe | VM image datastore, ext4 at `/srv/vmstore` |
| Arch host | Seagate 4 TB HDD (ST4000VN006 family) | Personal archives, Btrfs at `/srv/backup-a` |
| VM / LSI HBA | 480 GB SSD-A | Forgejo backups, `/srv/storage/ssd-a` |
| VM / LSI HBA | 480 GB SSD-B | Persistent application data, `/srv/storage/ssd-b` |
| VM / LSI HBA | 4 TB HDD-A | Primary files and media, `/srv/storage/hdd-a` |
| VM / LSI HBA | 4 TB HDD-B | File archive copy, `/srv/storage/hdd-b` |
| VM / LSI HBA | 4 TB HDD-C | Additional storage, `/srv/storage/hdd-c` |
| VM / libvirt | 64 GiB `vda` | Guest system disk, ext4 |
| VM / libvirt | 160 GiB `vdb` | Service configuration and Compose, Btrfs at `/srv/services` |

For storage design see [Storage and backups](storage-architecture.md). For device attachment and PCI passthrough see [Virtualization](virtualization.md).
