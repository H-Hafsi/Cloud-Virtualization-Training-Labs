# Chapitre 9 : Disques, LVM et systèmes de fichiers / Disks, LVM and File Systems

=== "Français"

    !!! abstract "Objectifs du chapitre"
        - Distinguer table de partitions (MBR, GPT), partition et système de fichiers
        - Comparer ext4, XFS, Btrfs et ZFS et choisir un système de fichiers selon un besoin
        - Décrire le montage d'un volume, le fichier `/etc/fstab` et l'intérêt des UUID
        - Expliquer l'architecture de LVM (PV, VG, LV) et ses opérations : extension à chaud, snapshot
        - Présenter ZFS : pools, jeux de données, sommes de contrôle, snapshots
        - Acquis d'apprentissage visé : **AA3**

        Matériel requis pour ce cours : aucun. Les labs associés indiquent leurs propres ressources.

=== "English"

    !!! abstract "Chapter objectives"
        - Tell partition table (MBR, GPT), partition and file system apart
        - Compare ext4, XFS, Btrfs and ZFS and choose a file system for a need
        - Describe mounting a volume, the `/etc/fstab` file and the benefit of UUIDs
        - Explain LVM's architecture (PV, VG, LV) and its operations: online extension, snapshot
        - Present ZFS: pools, datasets, checksums, snapshots
        - Target learning outcome: **AA3**

        Required equipment for this lesson: none. The associated labs state their own resources.

## 1. Des disques bruts aux partitions / From raw disks to partitions

=== "Français"

    Aux chapitres 7 et 8, vous avez écrit directement sur des disques bruts ou sur des grappes, sans structure. Pour **ranger des fichiers**, il faut ajouter deux niveaux d'organisation :

    ```mermaid
    flowchart LR
      D["Disque<br/>(brut)"] --> P["Table de partitions<br/>et partitions"]
      P --> F["Système de fichiers<br/>(ext4, XFS...)"]
      F --> M["Point de montage<br/>(/mnt/données)"]
    ```

    ### Table de partitions : MBR ou GPT

    | | MBR (ancien) | GPT (actuel) |
    |---|---|---|
    | **Démarrage** | BIOS | UEFI (et BIOS avec une partition dédiée) |
    | **Taille maximale d'un disque** | 2 Tio (secteurs de 512 octets) | Pratiquement illimitée |
    | **Nombre de partitions** | 4 primaires (ou 3 + 1 étendue) | 128 par défaut |
    | **Robustesse** | Table unique, sans somme de contrôle | Sommes de contrôle et copie de la table en fin de disque |

    On crée une table de partitions avec `fdisk` ou `parted`. Une **partition** est une portion contiguë du disque, vue comme un périphérique à part (`/dev/sdb1`). Sous Linux, le disque nu, la partition ou un volume logique peuvent tous recevoir un système de fichiers.

    | Commande | Usage |
    |---|---|
    | `lsblk` | Arbre des disques, partitions et points de montage |
    | `sudo fdisk -l` | Table de partitions de chaque disque |
    | `sudo parted <disque> print` | Table de partitions, avec `parted` |
    | `sudo blkid` | UUID et type des systèmes de fichiers |

=== "English"

    In chapters 7 and 8, you wrote directly on raw disks or on arrays, without any structure. To **store files**, two levels of organization must be added:

    ```mermaid
    flowchart LR
      D["Disk<br/>(raw)"] --> P["Partition table<br/>and partitions"]
      P --> F["File system<br/>(ext4, XFS...)"]
      F --> M["Mount point<br/>(/mnt/data)"]
    ```

    ### Partition table: MBR or GPT

    | | MBR (legacy) | GPT (current) |
    |---|---|---|
    | **Boot** | BIOS | UEFI (and BIOS with a dedicated partition) |
    | **Maximum disk size** | 2 TiB (512-byte sectors) | Practically unlimited |
    | **Number of partitions** | 4 primary (or 3 + 1 extended) | 128 by default |
    | **Robustness** | Single table, no checksum | Checksums and a copy of the table at the end of the disk |

    A partition table is created with `fdisk` or `parted`. A **partition** is a contiguous portion of the disk, seen as a separate device (`/dev/sdb1`). On Linux, a bare disk, a partition or a logical volume can all receive a file system.

    | Command | Use |
    |---|---|
    | `lsblk` | Tree of disks, partitions and mount points |
    | `sudo fdisk -l` | Partition table of each disk |
    | `sudo parted <disk> print` | Partition table, with `parted` |
    | `sudo blkid` | UUIDs and file system types |

## 2. Les systèmes de fichiers / File systems

=== "Français"

    Un **système de fichiers** organise les données d'un volume en fichiers et répertoires : il garde la trace de l'emplacement des blocs de chaque fichier (via des **inodes**), de ses attributs (droits, dates) et de l'espace libre. Les systèmes modernes tiennent un **journal** : les modifications des métadonnées y sont d'abord notées, ce qui permet de retrouver un état cohérent rapidement après une coupure de courant, au lieu de parcourir tout le volume.

    | Système | Caractéristiques | Usage typique |
    |---|---|---|
    | **ext4** | Journalisé, très répandu, stable ; extension en ligne, réduction hors ligne | Choix par défaut sous Ubuntu |
    | **XFS** | Journalisé, excellent avec de gros fichiers et de nombreux accès parallèles ; **on peut l'agrandir, pas le réduire** | Serveurs, gros volumes |
    | **Btrfs** | Copy-on-write, snapshots, sommes de contrôle, sous-volumes | Postes et serveurs recherchant ces fonctions |
    | **ZFS** | À la fois gestionnaire de volumes et système de fichiers ; copy-on-write, sommes de contrôle, snapshots (section 5) | Stockage fiable, serveurs de fichiers |
    | **NTFS, exFAT, FAT32** | Systèmes de Windows et d'échange | Disques partagés avec Windows, clés USB |

    | Commande | Usage |
    |---|---|
    | `sudo mkfs.ext4 <périphérique>` | Crée un système de fichiers ext4 (formatage) |
    | `sudo mkfs.xfs <périphérique>` | Crée un système de fichiers XFS |
    | `df -h` | Espace utilisé et libre des systèmes montés |
    | `sudo fsck <périphérique>` | Vérifie et répare un système de fichiers **démonté** |
    | `sudo resize2fs <périphérique>` / `sudo xfs_growfs <point de montage>` | Agrandit ext4 / XFS |

    !!! warning "Formater efface"
        `mkfs` crée un système de fichiers neuf et **efface le contenu** précédent du périphérique : vérifiez toujours le nom du périphérique avec `lsblk` avant de lancer la commande.

=== "English"

    A **file system** organizes a volume's data into files and directories: it keeps track of the location of each file's blocks (through **inodes**), its attributes (permissions, dates) and free space. Modern systems keep a **journal**: metadata changes are noted there first, which allows a consistent state to be recovered quickly after a power cut, instead of scanning the whole volume.

    | System | Characteristics | Typical use |
    |---|---|---|
    | **ext4** | Journaled, very common, stable; online extension, offline shrinking | Default choice on Ubuntu |
    | **XFS** | Journaled, excellent with large files and many parallel accesses; **can be grown, not shrunk** | Servers, large volumes |
    | **Btrfs** | Copy-on-write, snapshots, checksums, subvolumes | Workstations and servers wanting these features |
    | **ZFS** | Both volume manager and file system; copy-on-write, checksums, snapshots (section 5) | Reliable storage, file servers |
    | **NTFS, exFAT, FAT32** | Windows and exchange systems | Disks shared with Windows, USB drives |

    | Command | Use |
    |---|---|
    | `sudo mkfs.ext4 <device>` | Creates an ext4 file system (formatting) |
    | `sudo mkfs.xfs <device>` | Creates an XFS file system |
    | `df -h` | Used and free space of mounted systems |
    | `sudo fsck <device>` | Checks and repairs an **unmounted** file system |
    | `sudo resize2fs <device>` / `sudo xfs_growfs <mount point>` | Grows ext4 / XFS |

    !!! warning "Formatting erases"
        `mkfs` creates a new file system and **erases the previous content** of the device: always check the device name with `lsblk` before running the command.

## 3. Monter un volume : `mount` et `/etc/fstab` / Mounting a volume: `mount` and `/etc/fstab`

=== "Français"

    Sous Linux, tous les systèmes de fichiers sont rattachés à **une seule arborescence**. Monter un volume, c'est le rattacher à un répertoire existant, le **point de montage**.

    ```bash
    sudo mkdir -p /mnt/donnees
    sudo mount /dev/sdb1 /mnt/donnees
    df -h /mnt/donnees
    sudo umount /mnt/donnees
    ```

    Un montage fait à la main est perdu au redémarrage. Pour le rendre **permanent**, on déclare le volume dans `/etc/fstab`, une ligne par volume :

    | Champ | Exemple | Rôle |
    |---|---|---|
    | 1. Périphérique | `UUID=0a1b…` | Identifie le volume |
    | 2. Point de montage | `/mnt/donnees` | Répertoire de rattachement |
    | 3. Type | `ext4` | Système de fichiers |
    | 4. Options | `defaults,nofail` | Options de montage |
    | 5. Dump | `0` | Obsolète, laisser 0 |
    | 6. Ordre de vérification | `2` | 0 : pas de vérification ; 1 : racine ; 2 : autres |

    **Pourquoi des UUID ?** Les noms `/dev/sdb`, `/dev/sdc` dépendent de l'ordre de détection des disques et peuvent changer d'un démarrage à l'autre. L'**UUID** est un identifiant propre au système de fichiers : il reste le même (`sudo blkid` le donne).

    !!! danger "Une erreur dans `/etc/fstab` peut empêcher le démarrage"
        Une ligne erronée peut faire échouer le montage au boot et mener à un mode de secours. Testez toujours avec `sudo mount -a` (qui monte tout ce qui est déclaré) **avant de redémarrer**, et utilisez l'option `nofail` pour un volume non essentiel au démarrage.

=== "English"

    On Linux, all file systems are attached to **a single tree**. Mounting a volume means attaching it to an existing directory, the **mount point**.

    ```bash
    sudo mkdir -p /mnt/donnees
    sudo mount /dev/sdb1 /mnt/donnees
    df -h /mnt/donnees
    sudo umount /mnt/donnees
    ```

    A manual mount is lost at restart. To make it **permanent**, the volume is declared in `/etc/fstab`, one line per volume:

    | Field | Example | Role |
    |---|---|---|
    | 1. Device | `UUID=0a1b…` | Identifies the volume |
    | 2. Mount point | `/mnt/donnees` | Directory to attach to |
    | 3. Type | `ext4` | File system |
    | 4. Options | `defaults,nofail` | Mount options |
    | 5. Dump | `0` | Obsolete, leave 0 |
    | 6. Check order | `2` | 0: no check; 1: root; 2: others |

    **Why UUIDs?** The names `/dev/sdb`, `/dev/sdc` depend on the disk detection order and can change from one boot to the next. The **UUID** is an identifier specific to the file system: it stays the same (`sudo blkid` shows it).

    !!! danger "An error in `/etc/fstab` can prevent booting"
        A faulty line can make the mount fail at boot and lead to a rescue mode. Always test with `sudo mount -a` (which mounts everything declared) **before restarting**, and use the `nofail` option for a volume not essential at boot.

## 4. LVM : volumes logiques / LVM: logical volumes

=== "Français"

    **LVM** (*Logical Volume Manager*) ajoute une couche de **souplesse** entre les disques et les systèmes de fichiers : au lieu de partitions de taille fixe, on gère des volumes que l'on peut agrandir, déplacer ou « photographier ».

    ```mermaid
    flowchart TB
      D1["Disque 1"] --> PV1["PV 1<br/>(volume physique)"]
      D2["Disque 2"] --> PV2["PV 2<br/>(volume physique)"]
      PV1 --> VG["VG<br/>(groupe de volumes)"]
      PV2 --> VG
      VG --> LV1["LV 1<br/>(volume logique)"]
      VG --> LV2["LV 2<br/>(volume logique)"]
      LV1 --> FS1["Système de fichiers"]
      LV2 --> FS2["Système de fichiers"]
    ```

    | Couche | Rôle | Commandes |
    |---|---|---|
    | **PV** (*physical volume*) | Disque ou partition initialisé pour LVM | `pvcreate`, `pvs`, `pvdisplay` |
    | **VG** (*volume group*) | Réserve d'espace formée d'un ou plusieurs PV | `vgcreate`, `vgextend`, `vgs` |
    | **LV** (*logical volume*) | Volume découpé dans un VG, qui reçoit un système de fichiers | `lvcreate`, `lvextend`, `lvs` |

    L'espace d'un VG est géré par **extents** (blocs de 4 Mio par défaut). Un LV peut s'étendre sur **plusieurs disques**.

    ### Les opérations qui font l'intérêt de LVM

    | Opération | Principe | Commande type |
    |---|---|---|
    | **Extension à chaud** | Ajouter de l'espace à un LV **sans le démonter**, puis agrandir le système de fichiers | `lvextend -r -L +1G /dev/vg/lv` (l'option `-r` agrandit aussi le système de fichiers) |
    | **Ajout d'un disque** | Agrandir le VG avec un nouveau PV | `vgextend vg /dev/sdX` |
    | **Déplacement de données** | Vider un PV vers les autres, par exemple avant de le retirer | `pvmove` |
    | **Snapshot** | Copie figée d'un LV à un instant donné (copy-on-write) | `lvcreate -s -n snap -L 1G /dev/vg/lv` |

    **Snapshot LVM.** Le snapshot ne recopie pas tout : il stocke seulement les **blocs qui changent** après sa création. Son espace réservé doit donc être suffisant : s'il est plein, le snapshot devient **invalide**. Comme tout snapshot, ce n'est **pas une sauvegarde** (chapitre 2).

=== "English"

    **LVM** (*Logical Volume Manager*) adds a layer of **flexibility** between disks and file systems: instead of fixed-size partitions, you manage volumes that can be grown, moved or "photographed".

    ```mermaid
    flowchart TB
      D1["Disk 1"] --> PV1["PV 1<br/>(physical volume)"]
      D2["Disk 2"] --> PV2["PV 2<br/>(physical volume)"]
      PV1 --> VG["VG<br/>(volume group)"]
      PV2 --> VG
      VG --> LV1["LV 1<br/>(logical volume)"]
      VG --> LV2["LV 2<br/>(logical volume)"]
      LV1 --> FS1["File system"]
      LV2 --> FS2["File system"]
    ```

    | Layer | Role | Commands |
    |---|---|---|
    | **PV** (*physical volume*) | Disk or partition initialized for LVM | `pvcreate`, `pvs`, `pvdisplay` |
    | **VG** (*volume group*) | Space pool made of one or more PVs | `vgcreate`, `vgextend`, `vgs` |
    | **LV** (*logical volume*) | Volume carved from a VG, which receives a file system | `lvcreate`, `lvextend`, `lvs` |

    A VG's space is managed by **extents** (4 MiB blocks by default). An LV can span **several disks**.

    ### The operations that make LVM worthwhile

    | Operation | Principle | Typical command |
    |---|---|---|
    | **Online extension** | Add space to an LV **without unmounting it**, then grow the file system | `lvextend -r -L +1G /dev/vg/lv` (the `-r` option also grows the file system) |
    | **Adding a disk** | Grow the VG with a new PV | `vgextend vg /dev/sdX` |
    | **Moving data** | Empty a PV onto the others, for example before removing it | `pvmove` |
    | **Snapshot** | Frozen copy of an LV at a given moment (copy-on-write) | `lvcreate -s -n snap -L 1G /dev/vg/lv` |

    **LVM snapshot.** The snapshot does not copy everything: it only stores the **blocks that change** after its creation. Its reserved space must therefore be sufficient: if it fills up, the snapshot becomes **invalid**. Like any snapshot, it is **not a backup** (chapter 2).

## 5. ZFS / ZFS

=== "Français"

    **ZFS** (via **OpenZFS** sous Linux) réunit en un seul système le gestionnaire de volumes et le système de fichiers.

    | Notion | Définition |
    |---|---|
    | **Pool** (`zpool`) | Ensemble de périphériques regroupés ; il fournit l'espace à tous les jeux de données |
    | **vdev** | Groupe de disques au sein d'un pool : disque seul, **miroir** ou **RAID-Z** (RAID-Z1 : une parité, comme un RAID 5 ; RAID-Z2 : double parité, comme un RAID 6) |
    | **Jeu de données** (*dataset*) | Système de fichiers créé dans un pool (`zfs create`), avec ses propres propriétés (compression, quota) |
    | **Somme de contrôle** | Chaque bloc est vérifié à la lecture ; avec de la redondance, ZFS **répare** automatiquement un bloc corrompu (*self-healing*) |
    | **Snapshot** | Copie figée instantanée (copy-on-write) d'un jeu de données (`zfs snapshot`) ; très économe |
    | **Compression** | Par exemple `lz4`, activable par jeu de données |
    | **Scrub** | Parcours de vérification de toutes les données (`zpool scrub`) |

    **Points d'attention :** ZFS utilise la mémoire pour son cache (ARC) ; il demande de la RAM pour de gros volumes, et on le décrit d'abord comme **un volume en pool**, pas comme une partition. Les commandes se résument à `zpool` (pools) et `zfs` (jeux de données).

    !!! tip "À retenir"
        - Disque → table de partitions (MBR ou GPT) → partitions → système de fichiers → point de montage.
        - ext4 : défaut et réduction possible ; XFS : agrandissable seulement ; Btrfs et ZFS : copy-on-write et snapshots.
        - `mkfs` efface le contenu ; vérifier le périphérique avant de formater.
        - `/etc/fstab` : utiliser les UUID, tester avec `mount -a`, `nofail` pour un volume non essentiel.
        - LVM : PV, VG, LV ; extension à chaud avec `lvextend -r` ; un snapshot LVM a besoin d'espace réservé et n'est pas une sauvegarde.
        - ZFS : pool, vdev (miroir, RAID-Z), jeux de données, sommes de contrôle, snapshots, scrub.

=== "English"

    **ZFS** (through **OpenZFS** on Linux) brings the volume manager and the file system together in a single system.

    | Notion | Definition |
    |---|---|
    | **Pool** (`zpool`) | Set of grouped devices; it provides space to all datasets |
    | **vdev** | Group of disks within a pool: single disk, **mirror** or **RAID-Z** (RAID-Z1: one parity, like a RAID 5; RAID-Z2: double parity, like a RAID 6) |
    | **Dataset** | File system created in a pool (`zfs create`), with its own properties (compression, quota) |
    | **Checksum** | Each block is verified on read; with redundancy, ZFS automatically **repairs** a corrupted block (*self-healing*) |
    | **Snapshot** | Instant frozen copy (copy-on-write) of a dataset (`zfs snapshot`); very space-efficient |
    | **Compression** | For example `lz4`, enabled per dataset |
    | **Scrub** | Verification pass over all data (`zpool scrub`) |

    **Points of attention:** ZFS uses memory for its cache (ARC); it needs RAM for large volumes, and it is first described as **a pooled volume**, not as a partition. Commands boil down to `zpool` (pools) and `zfs` (datasets).

    !!! tip "Key takeaways"
        - Disk → partition table (MBR or GPT) → partitions → file system → mount point.
        - ext4: default and can shrink; XFS: grow only; Btrfs and ZFS: copy-on-write and snapshots.
        - `mkfs` erases content; check the device before formatting.
        - `/etc/fstab`: use UUIDs, test with `mount -a`, `nofail` for a non-essential volume.
        - LVM: PV, VG, LV; online extension with `lvextend -r`; an LVM snapshot needs reserved space and is not a backup.
        - ZFS: pool, vdev (mirror, RAID-Z), datasets, checksums, snapshots, scrub.

## Pour vérifier votre compréhension / Check your understanding

=== "Français"

    ??? question "Trois disques de 10 Go forment un groupe de volumes LVM. Peut-on créer un volume logique de 25 Go ?"
        Oui : le VG regroupe l'espace des trois PV (30 Go au total), et un LV peut s'étendre sur plusieurs disques. Un volume de 25 Go est donc possible, avec 5 Go restant libres dans le VG.

    ??? question "Pourquoi utiliser l'UUID plutôt que `/dev/sdb1` dans `/etc/fstab` ?"
        Le nom `/dev/sdb1` dépend de l'ordre de détection des disques, qui peut changer d'un démarrage à l'autre (ajout d'un disque, autre contrôleur) : le montage pourrait viser le mauvais volume. L'UUID est propre au système de fichiers et reste stable.

    ??? question "Un snapshot LVM de 1 Go est créé sur un volume de 20 Go, puis 3 Go de données sont modifiés. Que se passe-t-il ?"
        Le snapshot ne stocke que les blocs modifiés depuis sa création. Si plus de blocs sont modifiés que son espace réservé ne peut en contenir, il se remplit et devient **invalide**. Il faut donc dimensionner le snapshot selon le volume de modifications attendu pendant sa durée de vie.

=== "English"

    ??? question "Three 10 GB disks form an LVM volume group. Can a 25 GB logical volume be created?"
        Yes: the VG pools the space of the three PVs (30 GB in total), and an LV can span several disks. A 25 GB volume is therefore possible, with 5 GB remaining free in the VG.

    ??? question "Why use the UUID rather than `/dev/sdb1` in `/etc/fstab`?"
        The name `/dev/sdb1` depends on the disk detection order, which can change from one boot to the next (added disk, other controller): the mount could target the wrong volume. The UUID is specific to the file system and stays stable.

    ??? question "A 1 GB LVM snapshot is created on a 20 GB volume, then 3 GB of data is modified. What happens?"
        The snapshot only stores blocks modified since its creation. If more blocks are modified than its reserved space can hold, it fills up and becomes **invalid**. The snapshot must therefore be sized according to the amount of change expected during its lifetime.

## Ressources du chapitre / Chapter resources

- [Lab 9.1 : Partitions et systèmes de fichiers / Partitions and file systems](lab-9-1-partitions-et-systemes-de-fichiers.md)
- [Lab 9.2 : LVM, extension à chaud et snapshot / LVM: online extension and snapshot](lab-9-2-lvm-extension-a-chaud-et-snapshot.md)
- [Lab 9.3 : ZFS, pools et snapshots / ZFS: pools and snapshots](lab-9-3-zfs-pools-et-snapshots.md)
- [Quiz du chapitre / Chapter quiz](quiz.md)
- [Glossaire / Glossary](../../annexes/glossaire.md)
- [Environnement de TP / Lab environment](../../annexes/environnement.md)