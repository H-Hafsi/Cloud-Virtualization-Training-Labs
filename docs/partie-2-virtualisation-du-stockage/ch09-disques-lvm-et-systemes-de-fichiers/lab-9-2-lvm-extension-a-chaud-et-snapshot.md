# Lab 9.2 : LVM, extension à chaud et snapshot / LVM: Online Extension and Snapshot

|                                              |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Durée / Duration**                         | 75 à 90 min / 75 to 90 min                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| **Niveau / Level**                           | Guidé / Guided                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| **Prérequis / Prerequisites**                | [Chapitre 9 : cours](cours.md) ; [Lab 9.1](lab-9-1-partitions-et-systemes-de-fichiers.md) réalisé (mêmes disques vides, snapshot `etat-disques`)                                                                                                                                                                                                                                                                                                                                                                                                         |
| **Ressources minimales / Minimum resources** | **VM** : `srv1-vmware` (2 vCPU, 2 Go de RAM) avec **3 disques vides de 2 Go** (ceux du Lab 7.1). **Hôte** : VMware Workstation, au moins 8 Go d'espace disque libre (les disques dynamiques grossissent pendant les écritures), accès Internet. L'hyperviseur Windows doit être **désactivé**. / **VM**: `srv1-vmware` (2 vCPUs, 2 GB RAM) with **3 empty 2 GB disks** (those from Lab 7.1). **Host**: VMware Workstation, at least 8 GB of free disk space (dynamic disks grow during writes), Internet access. The Windows hypervisor must be **off**. |
| **Acquis visés / Learning outcomes**         | AA3                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |

=== "Français"

    !!! abstract "Objectifs"
        - Construire une pile LVM : volume physique (PV), groupe de volumes (VG), volume logique (LV)
        - Agrandir un volume monté **sans interruption**, puis constater la limite de l'espace libre du VG
        - Ajouter un disque au VG et étendre un LV sur deux disques
        - Créer un snapshot LVM, y retrouver une donnée supprimée, et constater ce qui se passe quand il est trop petit
        - Déplacer des données d'un disque à un autre avec `pvmove` (facultatif)

=== "English"

    !!! abstract "Objectives"
        - Build an LVM stack: physical volume (PV), volume group (VG), logical volume (LV)
        - Grow a mounted volume **without interruption**, then observe the limit of the VG's free space
        - Add a disk to the VG and extend an LV over two disks
        - Create an LVM snapshot, recover a deleted file from it, and see what happens when it is too small
        - Move data from one disk to another with `pvmove` (optional)

## Contexte et schéma / Context and diagram

=== "Français"

    Vous construisez une pile LVM dans la VM et vous la faites évoluer **pendant qu'elle sert des données** : agrandissement, ajout d'un disque, snapshot. Un petit programme écrit un fichier journal chaque seconde pendant les opérations, pour prouver qu'aucune interruption n'a lieu.

    ```mermaid
    flowchart TB
      D1["Disque 1"] --> PV1["PV 1"]
      PV1 --> VG["VG vg_lab"]
      VG --> LV["LV lv_data<br/>(monté sur /mnt/lvdata)"]
      D2["Disque 2<br/>(ajouté à chaud)"] -.-> VG
      D3["Disque 3<br/>(facultatif, pvmove)"] -.-> VG
      LV -.-> S["Snapshot snap_data"]
    ```

    !!! danger "Disques vides uniquement"
        `pvcreate` **efface** le début du disque et en fait un volume physique LVM. N'utilisez que les **trois disques vides de 2 Go** (`<D1>`, `<D2>`, `<D3>`), repérés avec `lsblk` à l'étape 1. Ne touchez jamais au disque système de la VM.

=== "English"

    You build an LVM stack in the VM and make it evolve **while it serves data**: growing, adding a disk, snapshot. A small program writes a log file every second during the operations, to prove that no interruption occurs.

    ```mermaid
    flowchart TB
      D1["Disk 1"] --> PV1["PV 1"]
      PV1 --> VG["VG vg_lab"]
      VG --> LV["LV lv_data<br/>(mounted on /mnt/lvdata)"]
      D2["Disk 2<br/>(hot-added)"] -.-> VG
      D3["Disk 3<br/>(optional, pvmove)"] -.-> VG
      LV -.-> S["Snapshot snap_data"]
    ```

    !!! danger "Empty disks only"
        `pvcreate` **erases** the start of the disk and turns it into an LVM physical volume. Use only the **three empty 2 GB disks** (`<D1>`, `<D2>`, `<D3>`), found with `lsblk` in step 1. Never touch the VM's system disk.

## Étapes / Steps

### Étape 1 : Préparer la VM / Step 1: Prepare the VM

=== "Français"

    1. Éteignez la VM et restaurez le snapshot `etat-disques` (disques vierges, `fstab` d'origine). Démarrez-la.
    2. Repérez les trois disques vides de 2 Go et installez LVM si nécessaire :

=== "English"

    1. Shut the VM down and restore the `etat-disques` snapshot (blank disks, original `fstab`). Start it.
    2. Find the three empty 2 GB disks and install LVM if needed:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT
sudo apt update && sudo apt install -y lvm2
```

=== "Français"

    Notez `<D1>`, `<D2>` et `<D3>`.

    !!! success "Point de contrôle"
        - [ ] Trois disques de 2 Go vides, notés ; `lvm2` installé

=== "English"

    Note `<D1>`, `<D2>` and `<D3>`.

    !!! success "Checkpoint"
        - [ ] Three empty 2 GB disks, noted; `lvm2` installed

### Étape 2 : Construire la pile LVM / Step 2: Build the LVM stack

```bash
sudo pvcreate /dev/<D1>
sudo vgcreate vg_lab /dev/<D1>
sudo lvcreate -L 1G -n lv_data vg_lab
sudo pvs
sudo vgs
sudo lvs
sudo vgdisplay vg_lab | grep -E "PE Size|Total PE|Free  PE"
```

=== "Français"

    Relevez : la taille du VG (un peu moins de 2 Go), la taille des **extents** (`PE Size`, 4 Mio par défaut), le nombre total d'extents et l'espace **libre** restant (`Free PE`). Le volume logique se trouve dans `/dev/vg_lab/lv_data`.

    !!! success "Point de contrôle"
        - [ ] `pvs`, `vgs`, `lvs` montrent un PV, un VG `vg_lab` et un LV `lv_data` de 1 Go
        - [ ] Vous avez relevé la taille d'extent et l'espace libre du VG

=== "English"

    Note: the VG's size (a little under 2 GB), the **extent** size (`PE Size`, 4 MiB by default), the total number of extents and the remaining **free** space (`Free PE`). The logical volume is at `/dev/vg_lab/lv_data`.

    !!! success "Checkpoint"
        - [ ] `pvs`, `vgs`, `lvs` show one PV, one `vg_lab` VG and one 1 GB `lv_data` LV
        - [ ] You noted the extent size and the VG's free space

### Étape 3 : Système de fichiers, montage et données de référence / Step 3: File system, mount and reference data

```bash
sudo mkfs.ext4 -L lvdata /dev/vg_lab/lv_data
sudo mkdir -p /mnt/lvdata
sudo mount /dev/vg_lab/lv_data /mnt/lvdata
sudo dd if=/dev/urandom of=/mnt/lvdata/ref.bin bs=1M count=100 status=none
sudo sha256sum /mnt/lvdata/ref.bin
df -h /mnt/lvdata
```

=== "Français"

    Notez l'empreinte de `ref.bin` et la taille du volume. Lancez maintenant le **programme de contrôle** : il ajoute une ligne horodatée au fichier `journal.log` chaque seconde, en arrière-plan.

=== "English"

    Note `ref.bin`'s checksum and the volume's size. Now start the **monitoring program**: it adds a timestamped line to `journal.log` every second, in the background.

```bash
sudo bash -c 'while true; do date >> /mnt/lvdata/journal.log; sleep 1; done' &
sleep 5
sudo tail -3 /mnt/lvdata/journal.log
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Le volume est monté, `ref.bin` est écrit et son empreinte est notée
        - [ ] `journal.log` grandit d'une ligne par seconde

=== "English"

    !!! success "Checkpoint"
        - [ ] The volume is mounted, `ref.bin` is written and its checksum is noted
        - [ ] `journal.log` grows by one line per second

### Étape 4 : Agrandir à chaud, puis atteindre la limite / Step 4: Grow online, then hit the limit

=== "Français"

    **Le volume reste monté et utilisé.** Ajoutez 512 Mo au volume logique **et** agrandissez le système de fichiers d'un seul geste (`-r`) :

=== "English"

    **The volume stays mounted and in use.** Add 512 MB to the logical volume **and** grow the file system in a single step (`-r`):

```bash
sudo lvextend -r -L +512M /dev/vg_lab/lv_data
df -h /mnt/lvdata
sudo lvs
```

=== "Français"

    La taille passe d'environ 1 Go à environ 1,5 Go, sans démontage. Tentez maintenant d'ajouter 2 Go, ce que le VG ne peut pas fournir :

=== "English"

    The size goes from about 1 GB to about 1.5 GB, with no unmounting. Now try to add 2 GB, which the VG cannot provide:

```bash
sudo lvextend -r -L +2G /dev/vg_lab/lv_data
sudo vgs
```

=== "Français"

    LVM refuse avec un message d'**espace libre insuffisant** : un LV ne peut pas dépasser l'espace libre de son VG. Il faut d'abord **agrandir le VG**.

    !!! success "Point de contrôle"
        - [ ] Le volume passe à environ 1,5 Go à chaud
        - [ ] La seconde extension est refusée ; vous avez lu le message et l'espace libre du VG

=== "English"

    LVM refuses with an **insufficient free space** message: an LV cannot exceed its VG's free space. The **VG must be enlarged** first.

    !!! success "Checkpoint"
        - [ ] The volume goes to about 1.5 GB online
        - [ ] The second extension is refused; you read the message and the VG's free space

### Étape 5 : Ajouter un disque et étendre sur deux disques / Step 5: Add a disk and extend over two disks

```bash
sudo pvcreate /dev/<D2>
sudo vgextend vg_lab /dev/<D2>
sudo vgs
sudo lvextend -r -L +2G /dev/vg_lab/lv_data
df -h /mnt/lvdata
sudo lvs -o +devices
```

=== "Français"

    L'extension réussit, cette fois : le LV fait environ 3,5 Go, **plus grand qu'un seul disque**. La colonne `Devices` montre qu'il s'étend sur `<D1>` **et** `<D2>`. Vérifiez que rien n'a été perdu ni interrompu, puis arrêtez le programme de contrôle :

=== "English"

    The extension succeeds this time: the LV is about 3.5 GB, **larger than a single disk**. The `Devices` column shows it spans `<D1>` **and** `<D2>`. Check that nothing was lost or interrupted, then stop the monitoring program:

```bash
sudo sha256sum /mnt/lvdata/ref.bin
sudo wc -l /mnt/lvdata/journal.log
sudo tail -3 /mnt/lvdata/journal.log
sudo pkill -f journal.log
```

=== "Français"

    L'empreinte de `ref.bin` est **inchangée**, et le nombre de lignes de `journal.log` correspond à peu près au nombre de secondes écoulées depuis l'étape 3 : le programme n'a subi **aucune interruption** pendant les extensions.

    !!! success "Point de contrôle"
        - [ ] Le LV fait environ 3,5 Go et s'étend sur deux disques (`lvs -o +devices`)
        - [ ] L'empreinte de `ref.bin` est identique ; `journal.log` n'a pas de trou

=== "English"

    `ref.bin`'s checksum is **unchanged**, and the number of lines in `journal.log` roughly matches the seconds elapsed since step 3: the program suffered **no interruption** during the extensions.

    !!! success "Checkpoint"
        - [ ] The LV is about 3.5 GB and spans two disks (`lvs -o +devices`)
        - [ ] `ref.bin`'s checksum is identical; `journal.log` has no gap

### Étape 6 : Snapshot LVM et récupération d'une donnée / Step 6: LVM snapshot and data recovery

=== "Français"

    Créez un snapshot de **300 Mo** du volume monté, puis **supprimez** `ref.bin` (comme une erreur d'utilisateur) :

=== "English"

    Create a **300 MB** snapshot of the mounted volume, then **delete** `ref.bin` (like a user error):

```bash
sudo lvcreate -s -n snap_data -L 300M /dev/vg_lab/lv_data
sudo lvs
sudo rm /mnt/lvdata/ref.bin
ls /mnt/lvdata
```

=== "Français"

    Montez le snapshot en **lecture seule** et récupérez le fichier supprimé :

=== "English"

    Mount the snapshot **read-only** and recover the deleted file:

```bash
sudo mkdir -p /mnt/snap
sudo mount -o ro /dev/vg_lab/snap_data /mnt/snap
ls /mnt/snap
sudo cp /mnt/snap/ref.bin /mnt/lvdata/ref.bin
sudo sha256sum /mnt/lvdata/ref.bin
sudo lvs
```

=== "Français"

    Le snapshot voit le volume **tel qu'il était à sa création** : `ref.bin` y est encore. Après la copie, l'empreinte retrouve la valeur notée. La colonne `Data%` de `lvs` montre la part de l'espace du snapshot déjà utilisée (blocs modifiés depuis sa création).

    !!! success "Point de contrôle"
        - [ ] `ref.bin` a disparu du volume, existe dans le snapshot, et a été restauré avec la bonne empreinte
        - [ ] Vous avez relevé `Data%` du snapshot

=== "English"

    The snapshot sees the volume **as it was at its creation**: `ref.bin` is still there. After the copy, the checksum returns to the noted value. The `Data%` column of `lvs` shows the share of the snapshot's space already used (blocks modified since its creation).

    !!! success "Checkpoint"
        - [ ] `ref.bin` disappeared from the volume, exists in the snapshot, and was restored with the right checksum
        - [ ] You noted the snapshot's `Data%`

### Étape 7 : Un snapshot trop petit / Step 7: A snapshot that is too small

=== "Français"

    Écrivez beaucoup plus de données (700 Mo) que l'espace du snapshot (300 Mo) sur le volume d'origine, puis observez le snapshot :

=== "English"

    Write much more data (700 MB) than the snapshot's space (300 MB) onto the origin volume, then observe the snapshot:

```bash
sudo dd if=/dev/urandom of=/mnt/lvdata/gros.bin bs=1M count=700 status=none
sudo lvs -a
sudo ls /mnt/snap
```

=== "Français"

    Le snapshot est maintenant **plein** : `Data%` atteint 100 % et les attributs de `lvs` signalent un snapshot **invalide**. Son contenu n'est plus utilisable. C'est le risque décrit au cours : le snapshot doit être dimensionné selon le volume de modifications attendu. Nettoyez :

=== "English"

    The snapshot is now **full**: `Data%` reaches 100% and `lvs` attributes flag an **invalid** snapshot. Its content is no longer usable. This is the risk described in the lesson: the snapshot must be sized according to the expected amount of change. Clean up:

```bash
sudo umount /mnt/snap
sudo lvremove -y vg_lab/snap_data
sudo rm /mnt/lvdata/gros.bin
sudo lvs
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Le snapshot a été invalidé par le dépassement de son espace
        - [ ] Le snapshot est supprimé ; le volume d'origine fonctionne toujours

=== "English"

    !!! success "Checkpoint"
        - [ ] The snapshot was invalidated by exceeding its space
        - [ ] The snapshot is removed; the origin volume still works

### Étape 8 : Déplacer des données d'un disque à un autre (facultatif) / Step 8: Move data from one disk to another (optional)

=== "Français"

    Vous allez « remplacer » le disque `<D1>` **à chaud** par `<D3>` : ajoutez `<D3>` au VG, vidangez `<D1>` vers les autres disques, puis retirez-le. Vérifiez d'abord que l'espace libre des autres PV suffit (colonne `PFree` de `pvs`) :

=== "English"

    You will "replace" disk `<D1>` **online** with `<D3>`: add `<D3>` to the VG, empty `<D1>` onto the other disks, then remove it. First check that the free space on the other PVs is enough (`PFree` column of `pvs`):

```bash
sudo pvcreate /dev/<D3>
sudo vgextend vg_lab /dev/<D3>
sudo pvs
sudo pvmove /dev/<D1>
sudo vgreduce vg_lab /dev/<D1>
sudo pvremove /dev/<D1>
sudo lvs -o +devices
sudo sha256sum /mnt/lvdata/ref.bin
```

=== "Français"

    `pvmove` déplace les extents de `<D1>` vers `<D2>` et `<D3>` **pendant que le volume reste monté**. `<D1>` est ensuite retiré du VG et libéré. L'empreinte de `ref.bin` reste la même. Si `pvmove` signale un manque de place, vérifiez `PFree` et supprimez des données ou ajoutez un disque.

    !!! success "Point de contrôle"
        - [ ] `<D1>` n'appartient plus au VG ; `lvs -o +devices` ne le mentionne plus
        - [ ] Le volume est resté monté et les données sont intactes

=== "English"

    `pvmove` moves `<D1>`'s extents to `<D2>` and `<D3>` **while the volume stays mounted**. `<D1>` is then removed from the VG and freed. `ref.bin`'s checksum stays the same. If `pvmove` reports a lack of space, check `PFree` and delete data or add a disk.

    !!! success "Checkpoint"
        - [ ] `<D1>` no longer belongs to the VG; `lvs -o +devices` no longer mentions it
        - [ ] The volume stayed mounted and the data is intact

### Étape 9 : Fiche de synthèse / Step 9: Summary sheet

=== "Français"

    Remplissez la fiche ci-dessous : c'est le **livrable** du lab.

    | Rubrique | Valeur |
    |---|---|
    | Taille d'extent et nombre total d'extents du VG | … |
    | Taille du LV : initiale / après `+512M` / après `+2G` | … / … / … |
    | Message d'erreur lors de la première extension de 2 Go | … |
    | Disques sur lesquels s'étend le LV après l'ajout de `<D2>` | … |
    | Empreinte de `ref.bin` avant / après les extensions | … / … |
    | `journal.log` : lignes attendues / observées | … / … |
    | `Data%` du snapshot après la suppression de `ref.bin` | … |
    | État du snapshot après l'écriture de 700 Mo | … |
    | Résultat de `pvmove` (facultatif) | … |

=== "English"

    Fill in the sheet below: this is the lab **deliverable**.

    | Item | Value |
    |---|---|
    | Extent size and total number of VG extents | … |
    | LV size: initial / after `+512M` / after `+2G` | … / … / … |
    | Error message on the first 2 GB extension | … |
    | Disks the LV spans after adding `<D2>` | … |
    | `ref.bin` checksum before / after the extensions | … / … |
    | `journal.log`: expected / observed lines | … / … |
    | Snapshot `Data%` after deleting `ref.bin` | … |
    | Snapshot state after writing 700 MB | … |
    | `pvmove` result (optional) | … |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Quel est l'avantage d'une extension « à chaud » en production ?"
        Le service continue de fonctionner pendant l'opération : pas d'arrêt planifié, pas de démontage, pas de perte de disponibilité. Les applications qui écrivent dans le volume ne sont pas interrompues.

    ??? question "Pourquoi l'extension de 2 Go a-t-elle échoué avant l'ajout de `<D2>` ?"
        Un volume logique tire son espace du **groupe de volumes** : il ne peut pas dépasser l'espace libre de ce VG. Le VG ne comptait que `<D1>`, presque entièrement utilisé. Ajouter un PV agrandit la réserve disponible.

    ??? question "Un snapshot LVM est-il une sauvegarde ? Que se passe-t-il s'il est trop petit ?"
        Non : il dépend du volume d'origine et stocke seulement les blocs modifiés ; si le volume ou le disque est perdu, le snapshot l'est aussi. Il est utile pour une copie cohérente à un instant donné ou pour un retour en arrière rapide, mais pas contre un sinistre. S'il est trop petit, il se remplit et devient invalide : son contenu est perdu.

=== "English"

    ??? question "What is the benefit of an “online” extension in production?"
        The service keeps running during the operation: no planned downtime, no unmounting, no loss of availability. Applications writing into the volume are not interrupted.

    ??? question "Why did the 2 GB extension fail before adding `<D2>`?"
        A logical volume draws its space from the **volume group**: it cannot exceed that VG's free space. The VG only contained `<D1>`, almost fully used. Adding a PV enlarges the available pool.

    ??? question "Is an LVM snapshot a backup? What happens if it is too small?"
        No: it depends on the origin volume and only stores modified blocks; if the volume or the disk is lost, so is the snapshot. It is useful for a consistent point-in-time copy or a quick rollback, but not against a disaster. If it is too small, it fills up and becomes invalid: its content is lost.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | `pvcreate` : « Device ... excluded by a filter » ou « has a signature » | Le disque contient d'anciennes données ou une table de partitions : le restaurer vierge (snapshot `etat-disques`) ou `sudo wipefs -a /dev/<disque>` s'il est vide et inutilisé |
    | `lvextend -r` : « Insufficient free space » | Espace libre du VG insuffisant : ajouter un PV (`vgextend`) ou réduire la demande (`vgs` pour le voir) |
    | `lvextend` réussit mais `df` n'a pas changé | Option `-r` oubliée : agrandir ensuite avec `sudo resize2fs /dev/vg_lab/lv_data` |
    | `mount` du snapshot : « can't read superblock » | Le snapshot est invalide (plein) : le supprimer et en recréer un plus grand |
    | `pvmove` : « No extents available » ou échec | Espace libre insuffisant sur les autres PV : ajouter un disque ou libérer de l'espace |
    | `umount` : « target is busy » | Le programme de contrôle écrit encore dans le volume : `sudo pkill -f journal.log` |
    | Les noms `sdb`, `sdc` ne correspondent pas | L'ordre varie : se fier à `lsblk` (taille, transport) |

=== "English"

    | Symptom | Hints |
    |---|---|
    | `pvcreate`: "Device ... excluded by a filter" or "has a signature" | The disk holds old data or a partition table: restore it blank (snapshot `etat-disques`) or `sudo wipefs -a /dev/<disk>` if it is empty and unused |
    | `lvextend -r`: "Insufficient free space" | VG free space insufficient: add a PV (`vgextend`) or reduce the request (`vgs` to see it) |
    | `lvextend` succeeds but `df` did not change | `-r` option forgotten: then grow with `sudo resize2fs /dev/vg_lab/lv_data` |
    | Snapshot `mount`: "can't read superblock" | The snapshot is invalid (full): remove it and create a larger one |
    | `pvmove`: "No extents available" or failure | Not enough free space on the other PVs: add a disk or free up space |
    | `umount`: "target is busy" | The monitoring program is still writing in the volume: `sudo pkill -f journal.log` |
    | The names `sdb`, `sdc` do not match | Order varies: rely on `lsblk` (size, transport) |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    Arrêtez le programme de contrôle et démontez le volume, puis supprimez la pile LVM dans l'ordre inverse de sa construction :

=== "English"

    Stop the monitoring program and unmount the volume, then delete the LVM stack in the reverse order of its construction:

```bash
sudo pkill -f journal.log
sudo umount /mnt/lvdata
sudo lvremove -y vg_lab/lv_data
sudo vgremove -y vg_lab
sudo pvremove /dev/<D2> /dev/<D3>
```

=== "Français"

    Le plus simple pour retrouver des disques vierges est de restaurer le snapshot `etat-disques`. Le [Lab 9.3](lab-9-3-zfs-pools-et-snapshots.md) part de disques vierges.

=== "English"

    The simplest way to get blank disks back is to restore the `etat-disques` snapshot. [Lab 9.3](lab-9-3-zfs-pools-et-snapshots.md) starts from blank disks.

[Retour au chapitre 9 / Back to chapter 9](cours.md)
