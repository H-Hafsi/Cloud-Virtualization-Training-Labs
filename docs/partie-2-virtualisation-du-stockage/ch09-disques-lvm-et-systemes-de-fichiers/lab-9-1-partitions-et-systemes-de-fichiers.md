# Lab 9.1 : Partitions et systèmes de fichiers / Partitions and File Systems

| | |
|---|---|
| **Durée / Duration** | 75 à 90 min / 75 to 90 min |
| **Niveau / Level** | Guidé / Guided |
| **Prérequis / Prerequisites** | [Chapitre 9 : cours](cours.md) ; [Lab 7.1](../ch07-technologies-et-interfaces-de-stockage/lab-7-1-disques-virtuels-ajouter-convertir-comparer.md) réalisé (snapshot `etat-disques` avec trois disques vides) |
| **Ressources minimales / Minimum resources** | **VM** : `srv1-vmware` (2 vCPU, 2 Go de RAM) avec **2 disques vides de 2 Go** (ceux du Lab 7.1). **Hôte** : VMware Workstation, au moins 6 Go d'espace disque libre, accès Internet pour installer un paquet. L'hyperviseur Windows doit être **désactivé**. / **VM**: `srv1-vmware` (2 vCPUs, 2 GB RAM) with **2 empty 2 GB disks** (those from Lab 7.1). **Host**: VMware Workstation, at least 6 GB of free disk space, Internet access to install a package. The Windows hypervisor must be **off**. |
| **Acquis visés / Learning outcomes** | AA3 |

=== "Français"

    !!! abstract "Objectifs"
        - Créer des tables de partitions GPT et MBR et des partitions avec `parted`
        - Créer des systèmes de fichiers ext4 et XFS et les comparer
        - Monter les volumes à la main puis de façon permanente avec `/etc/fstab` (UUID)
        - Observer les inodes et le journal, et vérifier un système de fichiers

=== "English"

    !!! abstract "Objectives"
        - Create GPT and MBR partition tables and partitions with `parted`
        - Create ext4 and XFS file systems and compare them
        - Mount volumes by hand then permanently with `/etc/fstab` (UUID)
        - Observe inodes and the journal, and check a file system

## Contexte et schéma / Context and diagram

=== "Français"

    Vous préparez deux disques vides de la VM : le premier reçoit une table **GPT** et deux partitions (ext4 et XFS), le second une table **MBR** pour comparer. Vous passez ainsi du disque brut des chapitres précédents à un volume **utilisable pour stocker des fichiers**.

    ```mermaid
    flowchart LR
      D1["Disque 1<br/>table GPT"] --> P1["Partition 1<br/>ext4"]
      D1 --> P2["Partition 2<br/>XFS"]
      D2["Disque 2<br/>table MBR"] --> P3["Partition 1<br/>ext4"]
      P1 --> M1["/mnt/ext4data"]
      P2 --> M2["/mnt/xfsdata"]
    ```

    !!! danger "Disques vides uniquement"
        Créer une table de partitions et un système de fichiers **efface** le contenu du disque. Utilisez uniquement les **deux disques vides de 2 Go**, repérés avec `lsblk` à l'étape 1. Ne touchez jamais au disque système de la VM.

=== "English"

    You prepare two empty disks of the VM: the first receives a **GPT** table and two partitions (ext4 and XFS), the second an **MBR** table for comparison. You thus go from the raw disk of previous chapters to a volume **usable for storing files**.

    ```mermaid
    flowchart LR
      D1["Disk 1<br/>GPT table"] --> P1["Partition 1<br/>ext4"]
      D1 --> P2["Partition 2<br/>XFS"]
      D2["Disk 2<br/>MBR table"] --> P3["Partition 1<br/>ext4"]
      P1 --> M1["/mnt/ext4data"]
      P2 --> M2["/mnt/xfsdata"]
    ```

    !!! danger "Empty disks only"
        Creating a partition table and a file system **erases** the disk's content. Use only the **two empty 2 GB disks**, found with `lsblk` in step 1. Never touch the VM's system disk.

## Étapes / Steps

### Étape 1 : Préparer la VM et repérer les disques / Step 1: Prepare the VM and find the disks

=== "Français"

    1. Éteignez la VM et restaurez le snapshot `etat-disques` (disques vierges). Démarrez-la.
    2. Repérez les disques de 2 Go et installez les outils nécessaires :

=== "English"

    1. Shut the VM down and restore the `etat-disques` snapshot (blank disks). Start it.
    2. Find the 2 GB disks and install the required tools:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT
sudo apt update && sudo apt install -y parted xfsprogs
```

=== "Français"

    Notez deux disques vides, sans système de fichiers ni point de montage : `<D1>` (qui recevra la table GPT) et `<D2>` (table MBR). Les partitions d'un disque `sdb` se nomment `sdb1`, `sdb2` ; celles d'un disque NVMe `nvme0n1` se nomment `nvme0n1p1`, `nvme0n1p2`.

    !!! success "Point de contrôle"
        - [ ] `<D1>` et `<D2>` sont deux disques de 2 Go vides ; `parted` et `xfsprogs` sont installés

=== "English"

    Note two empty disks, without file system or mount point: `<D1>` (which will receive the GPT table) and `<D2>` (MBR table). The partitions of a disk `sdb` are named `sdb1`, `sdb2`; those of an NVMe disk `nvme0n1` are named `nvme0n1p1`, `nvme0n1p2`.

    !!! success "Checkpoint"
        - [ ] `<D1>` and `<D2>` are two empty 2 GB disks; `parted` and `xfsprogs` are installed

### Étape 2 : Table GPT et deux partitions / Step 2: GPT table and two partitions

```bash
sudo parted /dev/<D1> --script mklabel gpt
sudo parted /dev/<D1> --script mkpart donnees-ext4 ext4 1MiB 1GiB
sudo parted /dev/<D1> --script mkpart donnees-xfs xfs 1GiB 100%
sudo parted /dev/<D1> print
lsblk /dev/<D1>
```

=== "Français"

    La première partition commence à 1 Mio pour rester **alignée** sur les blocs du disque. Relevez dans `print` le type de table (`Partition Table: gpt`), les tailles et les noms des partitions.

    !!! success "Point de contrôle"
        - [ ] `<D1>` porte une table `gpt` et deux partitions (environ 1 Gio chacune)

=== "English"

    The first partition starts at 1 MiB to stay **aligned** with the disk's blocks. In `print`, note the table type (`Partition Table: gpt`), the sizes and the partition names.

    !!! success "Checkpoint"
        - [ ] `<D1>` has a `gpt` table and two partitions (about 1 GiB each)

### Étape 3 : Table MBR / Step 3: MBR table

```bash
sudo parted /dev/<D2> --script mklabel msdos
sudo parted /dev/<D2> --script mkpart primary ext4 1MiB 1GiB
sudo parted /dev/<D2> print
sudo fdisk -l /dev/<D1> /dev/<D2> | grep -E "Disk /dev|Disklabel type"
```

=== "Français"

    Comparez : la ligne `Disklabel type` indique `gpt` pour `<D1>` et `dos` (MBR) pour `<D2>`. Notez les limites du MBR rappelées au cours (4 partitions primaires, 2 Tio).

    !!! success "Point de contrôle"
        - [ ] `fdisk -l` montre un disque `gpt` et un disque `dos`

=== "English"

    Compare: the `Disklabel type` line shows `gpt` for `<D1>` and `dos` (MBR) for `<D2>`. Note the MBR limits recalled in the lesson (4 primary partitions, 2 TiB).

    !!! success "Checkpoint"
        - [ ] `fdisk -l` shows one `gpt` disk and one `dos` disk

### Étape 4 : Créer des systèmes de fichiers / Step 4: Create file systems

```bash
sudo mkfs.ext4 -L ext4data /dev/<D1>1
sudo mkfs.xfs  -L xfsdata  /dev/<D1>2
sudo mkfs.ext4 -L mbrdata  /dev/<D2>1
sudo blkid /dev/<D1>1 /dev/<D1>2 /dev/<D2>1
lsblk -f /dev/<D1> /dev/<D2>
```

=== "Français"

    `mkfs.ext4` affiche le nombre d'**inodes**, la taille de bloc et la création du **journal**. Relevez les **UUID** et les types : vous en aurez besoin à l'étape 6. Remplacez `<D1>1` par `<D1>p1` si votre disque est un NVMe.

    !!! success "Point de contrôle"
        - [ ] `blkid` liste trois systèmes de fichiers (`ext4`, `xfs`, `ext4`) avec leur UUID et leur étiquette

=== "English"

    `mkfs.ext4` shows the number of **inodes**, the block size and the creation of the **journal**. Note the **UUIDs** and types: you will need them in step 6. Replace `<D1>1` with `<D1>p1` if your disk is NVMe.

    !!! success "Checkpoint"
        - [ ] `blkid` lists three file systems (`ext4`, `xfs`, `ext4`) with their UUID and label

### Étape 5 : Monter, écrire, observer / Step 5: Mount, write, observe

```bash
sudo mkdir -p /mnt/ext4data /mnt/xfsdata
sudo mount /dev/<D1>1 /mnt/ext4data
sudo mount /dev/<D1>2 /mnt/xfsdata
df -hT /mnt/ext4data /mnt/xfsdata
df -i /mnt/ext4data /mnt/xfsdata
sudo ls -la /mnt/ext4data /mnt/xfsdata
```

=== "Français"

    Relevez la **taille utilisable** de chaque système (un peu inférieure à celle de la partition, à cause des métadonnées et du journal), le **type**, et le **nombre d'inodes**. Notez que `/mnt/ext4data` contient un répertoire `lost+found`, propre à ext4. Écrivez maintenant des fichiers et observez leurs inodes :

=== "English"

    Note the **usable size** of each system (slightly below the partition's, due to metadata and the journal), the **type**, and the **number of inodes**. Note that `/mnt/ext4data` contains a `lost+found` directory, specific to ext4. Now write files and observe their inodes:

```bash
sudo dd if=/dev/urandom of=/mnt/ext4data/fichier.bin bs=1M count=50 status=none
sudo dd if=/dev/urandom of=/mnt/xfsdata/fichier.bin bs=1M count=50 status=none
sudo ls -li /mnt/ext4data /mnt/xfsdata
sudo stat /mnt/ext4data/fichier.bin
df -h /mnt/ext4data /mnt/xfsdata
```

=== "Français"

    `ls -li` affiche le **numéro d'inode** de chaque fichier ; `stat` détaille taille, blocs, inode et dates. L'espace utilisé a augmenté d'environ 50 Mo sur chaque volume.

    !!! success "Point de contrôle"
        - [ ] Les deux volumes sont montés avec les types attendus
        - [ ] Un fichier de 50 Mo existe sur chacun, et vous avez relevé leurs numéros d'inode

=== "English"

    `ls -li` shows each file's **inode number**; `stat` details size, blocks, inode and dates. Used space increased by about 50 MB on each volume.

    !!! success "Checkpoint"
        - [ ] Both volumes are mounted with the expected types
        - [ ] A 50 MB file exists on each, and you noted their inode numbers

### Étape 6 : Rendre les montages permanents / Step 6: Make the mounts permanent

=== "Français"

    Relevez les UUID avec `sudo blkid`, puis ajoutez deux lignes à `/etc/fstab` (**avec des UUID**, et l'option `nofail`, car ces volumes ne sont pas indispensables au démarrage). Remplacez `<UUID_EXT4>` et `<UUID_XFS>` :

=== "English"

    Get the UUIDs with `sudo blkid`, then add two lines to `/etc/fstab` (**with UUIDs**, and the `nofail` option, since these volumes are not essential at boot). Replace `<UUID_EXT4>` and `<UUID_XFS>`:

```bash
sudo cp /etc/fstab /etc/fstab.sauvegarde
sudo nano /etc/fstab
```

```text
UUID=<UUID_EXT4>  /mnt/ext4data  ext4  defaults,nofail  0  2
UUID=<UUID_XFS>   /mnt/xfsdata   xfs   defaults,nofail  0  2
```

=== "Français"

    **Avant de redémarrer**, testez la syntaxe et le montage, après avoir démonté les volumes :

=== "English"

    **Before restarting**, test the syntax and the mount, after unmounting the volumes:

```bash
sudo umount /mnt/ext4data /mnt/xfsdata
sudo findmnt --verify
sudo mount -a
df -hT | grep mnt
sudo reboot
```

=== "Français"

    Après le redémarrage, reconnectez-vous et vérifiez que les deux volumes sont montés **et que les fichiers sont toujours là**.

    !!! warning "Gardez la sauvegarde de `fstab`"
        Si la VM ne démarre plus correctement à cause d'une ligne erronée, vous pouvez restaurer `/etc/fstab.sauvegarde` depuis le mode de secours, ou restaurer le snapshot `etat-disques`.

    !!! success "Point de contrôle"
        - [ ] `findmnt --verify` ne signale aucune erreur ; `mount -a` réussit
        - [ ] Après le redémarrage, les deux volumes sont montés automatiquement, `fichier.bin` est présent

=== "English"

    After the restart, log in again and check that both volumes are mounted **and that the files are still there**.

    !!! warning "Keep the `fstab` backup"
        If the VM no longer boots correctly because of a faulty line, you can restore `/etc/fstab.sauvegarde` from rescue mode, or restore the `etat-disques` snapshot.

    !!! success "Checkpoint"
        - [ ] `findmnt --verify` reports no error; `mount -a` succeeds
        - [ ] After the restart, both volumes are mounted automatically, `fichier.bin` is present

### Étape 7 : Examiner le journal et vérifier les systèmes de fichiers / Step 7: Inspect the journal and check the file systems

```bash
sudo tune2fs -l /dev/<D1>1 | grep -iE "features|journal|inode count|block size"
sudo umount /mnt/ext4data /mnt/xfsdata
sudo fsck -f /dev/<D1>1
sudo xfs_repair -n /dev/<D1>2
```

=== "Français"

    `tune2fs -l` liste les fonctions d'ext4 (dont `has_journal`). `fsck -f` force la vérification d'un système ext4 **démonté** ; `xfs_repair -n` examine XFS **sans rien modifier** (option `-n`). Les deux doivent indiquer un système de fichiers cohérent. Remontez ensuite les volumes :

=== "English"

    `tune2fs -l` lists ext4's features (including `has_journal`). `fsck -f` forces a check of an **unmounted** ext4 system; `xfs_repair -n` examines XFS **without modifying anything** (`-n` option). Both should report a consistent file system. Then remount the volumes:

```bash
sudo mount -a
df -hT | grep mnt
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] `has_journal` figure dans les fonctions de l'ext4
        - [ ] Les vérifications ne signalent aucune erreur, et les volumes sont remontés

=== "English"

    !!! success "Checkpoint"
        - [ ] `has_journal` appears in ext4's features
        - [ ] The checks report no error, and the volumes are remounted

### Étape 8 : Fiche de synthèse / Step 8: Summary sheet

=== "Français"

    Remplissez la fiche ci-dessous : c'est le **livrable** du lab.

    | Rubrique | Valeur |
    |---|---|
    | Noms de `<D1>` et `<D2>` ; type de table de chacun | … |
    | Partitions créées (nom, début, fin, taille) sur `<D1>` | … |
    | UUID des trois systèmes de fichiers | … |
    | Taille utilisable ext4 / XFS ; nombre d'inodes de chacun | … / … |
    | Numéros d'inode des deux fichiers de test | … |
    | Lignes ajoutées à `/etc/fstab` | … |
    | Volumes montés après redémarrage ? fichiers présents ? | oui / non |
    | Résultat de `fsck` et de `xfs_repair -n` | … |

=== "English"

    Fill in the sheet below: this is the lab **deliverable**.

    | Item | Value |
    |---|---|
    | Names of `<D1>` and `<D2>`; table type of each | … |
    | Partitions created (name, start, end, size) on `<D1>` | … |
    | UUIDs of the three file systems | … |
    | Usable size ext4 / XFS; number of inodes of each | … / … |
    | Inode numbers of the two test files | … |
    | Lines added to `/etc/fstab` | … |
    | Volumes mounted after restart? files present? | yes / no |
    | Result of `fsck` and `xfs_repair -n` | … |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi la taille utilisable d'un système de fichiers est-elle inférieure à celle de la partition ?"
        Le système de fichiers réserve de l'espace pour ses propres structures : table d'inodes, journal, métadonnées, et parfois une réserve pour l'administrateur. Cet espace n'est pas disponible pour les fichiers de l'utilisateur.

    ??? question "Pourquoi utiliser l'option `nofail` pour ces volumes dans `/etc/fstab` ?"
        Sans `nofail`, l'absence ou l'erreur d'un volume au démarrage peut bloquer le démarrage et conduire au mode de secours. Pour un volume de données qui n'est pas indispensable au système, `nofail` permet au système de démarrer même si le volume est indisponible.

    ??? question "Que gagne-t-on avec un système de fichiers journalisé après une coupure de courant ?"
        Les modifications de métadonnées sont d'abord notées dans le journal : après une coupure, le système rejoue ou annule les opérations incomplètes en quelques instants, au lieu de parcourir tout le volume. Cela évite des vérifications longues et limite le risque d'incohérences.

=== "English"

    ??? question "Why is a file system's usable size lower than the partition's?"
        The file system reserves space for its own structures: inode table, journal, metadata, and sometimes a reserve for the administrator. This space is not available for user files.

    ??? question "Why use the `nofail` option for these volumes in `/etc/fstab`?"
        Without `nofail`, a volume's absence or error at boot can block startup and lead to rescue mode. For a data volume that is not essential to the system, `nofail` lets the system boot even if the volume is unavailable.

    ??? question "What is gained with a journaled file system after a power cut?"
        Metadata changes are first noted in the journal: after a cut, the system replays or cancels incomplete operations in moments, instead of scanning the whole volume. This avoids long checks and limits the risk of inconsistencies.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | `mkfs.xfs: command not found` | `sudo apt install -y xfsprogs` |
    | `parted: command not found` | `sudo apt install -y parted` |
    | `Error: ... contains a mounted partition` ou « device busy » | Un volume est monté ou une grappe RAID l'utilise encore : `lsblk`, `sudo umount`, `sudo mdadm --stop` |
    | La partition n'apparaît pas dans `lsblk` | `sudo partprobe /dev/<D1>` puis `lsblk` ; vérifier le nom (`p1` pour un NVMe) |
    | `mount: wrong fs type, bad option, bad superblock` | Mauvais type ou périphérique sans système de fichiers : vérifier avec `blkid` |
    | Le volume ne se monte pas après le redémarrage | UUID erroné dans `fstab` ; tester avec `sudo mount -a` ; comparer avec `sudo blkid` |
    | La VM démarre en mode de secours | Ligne `fstab` invalide sans `nofail` : restaurer `/etc/fstab.sauvegarde` ou le snapshot |
    | `fsck` refuse de vérifier | Le système est monté : le démonter d'abord |
    | Un `dd` échoue : « No space left on device » | Volume plein ; vérifier avec `df -h` |

=== "English"

    | Symptom | Hints |
    |---|---|
    | `mkfs.xfs: command not found` | `sudo apt install -y xfsprogs` |
    | `parted: command not found` | `sudo apt install -y parted` |
    | `Error: ... contains a mounted partition` or "device busy" | A volume is mounted or a RAID array still uses it: `lsblk`, `sudo umount`, `sudo mdadm --stop` |
    | The partition does not appear in `lsblk` | `sudo partprobe /dev/<D1>` then `lsblk`; check the name (`p1` for NVMe) |
    | `mount: wrong fs type, bad option, bad superblock` | Wrong type or device without a file system: check with `blkid` |
    | The volume does not mount after restart | Wrong UUID in `fstab`; test with `sudo mount -a`; compare with `sudo blkid` |
    | The VM boots into rescue mode | Invalid `fstab` line without `nofail`: restore `/etc/fstab.sauvegarde` or the snapshot |
    | `fsck` refuses to check | The system is mounted: unmount it first |
    | A `dd` fails: "No space left on device" | Volume full; check with `df -h` |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    Pour tout défaire proprement, restaurez simplement le snapshot **`etat-disques`** : il rétablit les disques vierges **et** le fichier `/etc/fstab` d'origine. Si vous préférez nettoyer à la main : démontez les volumes, supprimez les deux lignes ajoutées à `/etc/fstab`, puis effacez les signatures des disques (`sudo wipefs -a /dev/<D1>1 /dev/<D1>2 /dev/<D2>1`, puis `sudo wipefs -a /dev/<D1> /dev/<D2>`). Les [Labs 9.2](lab-9-2-lvm-extension-a-chaud-et-snapshot.md) et [9.3](lab-9-3-zfs-pools-et-snapshots.md) partent de disques vierges.

=== "English"

    To undo everything cleanly, simply restore the **`etat-disques`** snapshot: it brings back blank disks **and** the original `/etc/fstab` file. If you prefer cleaning by hand: unmount the volumes, remove the two lines added to `/etc/fstab`, then erase the disks' signatures (`sudo wipefs -a /dev/<D1>1 /dev/<D1>2 /dev/<D2>1`, then `sudo wipefs -a /dev/<D1> /dev/<D2>`). [Labs 9.2](lab-9-2-lvm-extension-a-chaud-et-snapshot.md) and [9.3](lab-9-3-zfs-pools-et-snapshots.md) start from blank disks.

[Retour au chapitre 9 / Back to chapter 9](cours.md)