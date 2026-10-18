# Lab 8.1 : RAID 1 et RAID 5 avec mdadm / RAID 1 and RAID 5 with mdadm

| | |
|---|---|
| **Durée / Duration** | 75 à 90 min / 75 to 90 min |
| **Niveau / Level** | Guidé / Guided |
| **Prérequis / Prerequisites** | [Chapitre 8 : cours](cours.md) ; [Lab 7.1](../ch07-technologies-et-interfaces-de-stockage/lab-7-1-disques-virtuels-ajouter-convertir-comparer.md) réalisé (VM `srv1-vmware`, trois disques vides, snapshot `etat-disques`) |
| **Ressources minimales / Minimum resources** | **VM** : `srv1-vmware` (2 vCPU, 2 Go de RAM) avec **5 disques vides de 2 Go** (les 3 du Lab 7.1 et 2 de plus), tous dynamiques. **Hôte** : VMware Workstation, au moins 10 Go d'espace disque libre, accès Internet. L'hyperviseur Windows doit être **désactivé**. / **VM**: `srv1-vmware` (2 vCPUs, 2 GB RAM) with **5 empty 2 GB disks** (the 3 from Lab 7.1 and 2 more), all dynamic. **Host**: VMware Workstation, at least 10 GB of free disk space, Internet access. The Windows hypervisor must be **off**. |
| **Acquis visés / Learning outcomes** | AA3 |

=== "Français"

    !!! abstract "Objectifs"
        - Créer une grappe RAID 1 et une grappe RAID 5 avec `mdadm`
        - Lire l'état d'une grappe : `/proc/mdstat`, `mdadm --detail`
        - Vérifier la capacité utile d'après la formule du cours
        - Écrire des données de référence et vérifier leur intégrité par empreinte
        - Rendre la configuration permanente et la retrouver après un redémarrage

=== "English"

    !!! abstract "Objectives"
        - Create a RAID 1 array and a RAID 5 array with `mdadm`
        - Read an array's state: `/proc/mdstat`, `mdadm --detail`
        - Check the usable capacity against the lesson's formula
        - Write reference data and verify its integrity by checksum
        - Make the configuration permanent and find it again after a restart

## Contexte et schéma / Context and diagram

=== "Français"

    Vous construisez deux grappes dans la VM : un **RAID 1** (miroir de deux disques) et un **RAID 5** (trois disques avec parité répartie). Rappel du chapitre 8 : sur des disques virtuels stockés sur un même disque physique, le RAID n'offre **aucune vraie tolérance aux pannes** ; ce lab sert à comprendre les **mécanismes**.

    ```mermaid
    flowchart LR
      D1["Disque 1"] --- M0["md0<br/>RAID 1"]
      D2["Disque 2"] --- M0
      D3["Disque 3"] --- M1["md1<br/>RAID 5"]
      D4["Disque 4"] --- M1
      D5["Disque 5"] --- M1
    ```

    !!! danger "Disques vides uniquement"
        La création d'une grappe **efface** les disques utilisés. Vérifiez à l'étape 1 que les cinq disques sont **vides** et ne contiennent pas le système de la VM. Les fichiers de la grappe ne reçoivent pas de système de fichiers dans ce lab (étudié au chapitre 9) : on écrit et on vérifie des données brutes.

=== "English"

    You build two arrays in the VM: a **RAID 1** (mirror of two disks) and a **RAID 5** (three disks with distributed parity). Reminder from chapter 8: on virtual disks stored on the same physical disk, RAID provides **no real fault tolerance**; this lab is for understanding the **mechanisms**.

    ```mermaid
    flowchart LR
      D1["Disk 1"] --- M0["md0<br/>RAID 1"]
      D2["Disk 2"] --- M0
      D3["Disk 3"] --- M1["md1<br/>RAID 5"]
      D4["Disk 4"] --- M1
      D5["Disk 5"] --- M1
    ```

    !!! danger "Empty disks only"
        Creating an array **erases** the disks used. Check in step 1 that the five disks are **empty** and do not hold the VM's system. The arrays receive no file system in this lab (studied in chapter 9): raw data is written and verified.

## Étapes / Steps

### Étape 1 : Préparer les cinq disques / Step 1: Prepare the five disks

=== "Français"

    1. Éteignez la VM et restaurez le snapshot `etat-disques` (trois disques vierges). Ajoutez **deux disques** de 2 Go, SCSI, dynamiques, comme au Lab 7.1 (étape 2) : `disque-d-scsi-dyn.vmdk` et `disque-e-scsi-dyn.vmdk`. Démarrez la VM.
    2. Listez les disques et installez l'outil :

=== "English"

    1. Shut the VM down and restore the `etat-disques` snapshot (three blank disks). Add **two disks** of 2 GB, SCSI, dynamic, as in Lab 7.1 (step 2): `disque-d-scsi-dyn.vmdk` and `disque-e-scsi-dyn.vmdk`. Start the VM.
    2. List the disks and install the tool:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT
sudo apt update && sudo apt install -y mdadm
mdadm --version
```

=== "Français"

    Identifiez les **cinq disques de 2 Go** (colonnes `FSTYPE` et `MOUNTPOINT` vides) et notez leurs noms : `<D1>`, `<D2>` pour le RAID 1 ; `<D3>`, `<D4>`, `<D5>` pour le RAID 5 (par exemple `sdb` à `sdf`, ou `nvme0n1` pour le disque NVMe). Le disque système, plus grand et monté, ne doit **jamais** être utilisé.

    !!! success "Point de contrôle"
        - [ ] Cinq disques de 2 Go sans système de fichiers ni point de montage
        - [ ] Les cinq noms sont notés ; `mdadm` est installé

=== "English"

    Identify the **five 2 GB disks** (`FSTYPE` and `MOUNTPOINT` columns empty) and note their names: `<D1>`, `<D2>` for the RAID 1; `<D3>`, `<D4>`, `<D5>` for the RAID 5 (for example `sdb` to `sdf`, or `nvme0n1` for the NVMe disk). The system disk, larger and mounted, must **never** be used.

    !!! success "Checkpoint"
        - [ ] Five 2 GB disks with no file system or mount point
        - [ ] The five names are noted; `mdadm` is installed

### Étape 2 : Créer le RAID 1 / Step 2: Create the RAID 1

```bash
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/<D1> /dev/<D2>
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

=== "Français"

    Si `mdadm` demande de confirmer la création (à propos des métadonnées et du démarrage), répondez `y`. Lisez la sortie :

    | Élément | Signification |
    |---|---|
    | `md0 : active raid1` | La grappe est active, au niveau RAID 1 |
    | `[2/2] [UU]` | 2 disques attendus, 2 disques actifs (`U` = *up*) ; `_` signalerait un disque manquant |
    | `resync = ...%` | Synchronisation initiale des deux disques, en cours ou terminée |
    | `State` (dans `--detail`) | `clean` quand tout est cohérent, `degraded` si un disque manque |
    | `Active Devices`, `Working Devices`, `Failed Devices`, `Spare Devices` | Nombres de disques actifs, en état de marche, en panne et de secours |

    Suivez la synchronisation jusqu'à son terme : `watch cat /proc/mdstat` (++ctrl+c++ pour quitter).

    !!! success "Point de contrôle"
        - [ ] `/dev/md0` existe, `[2/2] [UU]`, état `clean` après la synchronisation

=== "English"

    If `mdadm` asks to confirm the creation (about metadata and boot), answer `y`. Read the output:

    | Element | Meaning |
    |---|---|
    | `md0 : active raid1` | The array is active, at RAID 1 level |
    | `[2/2] [UU]` | 2 disks expected, 2 disks active (`U` = *up*); `_` would signal a missing disk |
    | `resync = ...%` | Initial synchronization of the two disks, in progress or finished |
    | `State` (in `--detail`) | `clean` when everything is consistent, `degraded` if a disk is missing |
    | `Active Devices`, `Working Devices`, `Failed Devices`, `Spare Devices` | Numbers of active, working, failed and spare disks |

    Follow the synchronization to the end: `watch cat /proc/mdstat` (++ctrl+c++ to quit).

    !!! success "Checkpoint"
        - [ ] `/dev/md0` exists, `[2/2] [UU]`, state `clean` after synchronization

### Étape 3 : Écrire et vérifier des données / Step 3: Write and verify data

=== "Français"

    Créez un fichier de référence de 100 Mio, écrivez-le tel quel sur la grappe (sans système de fichiers) et vérifiez qu'on le relit à l'identique :

=== "English"

    Create a 100 MiB reference file, write it as is onto the array (without a file system) and check that it reads back identically:

```bash
dd if=/dev/urandom of=~/ref.bin bs=1M count=100 status=none
sha256sum ~/ref.bin
sudo dd if=~/ref.bin of=/dev/md0 bs=1M oflag=direct status=none
sudo head -c 104857600 /dev/md0 | sha256sum
```

=== "Français"

    Les deux empreintes doivent être **identiques**. Relevez ensuite la capacité de la grappe :

=== "English"

    Both checksums must be **identical**. Then note the array's capacity:

```bash
lsblk -o NAME,SIZE,TYPE /dev/<D1> /dev/<D2> /dev/md0
```

=== "Français"

    Résultat attendu : `md0` fait à peu près **la taille d'un seul disque** (2 Go, légèrement moins à cause des métadonnées), conformément à la capacité utile du RAID 1 (C).

    !!! success "Point de contrôle"
        - [ ] Les empreintes de `ref.bin` et de `md0` sont identiques
        - [ ] La capacité de `md0` correspond à celle d'un disque

=== "English"

    Expected result: `md0` is about **the size of a single disk** (2 GB, slightly less due to metadata), in line with RAID 1's usable capacity (C).

    !!! success "Checkpoint"
        - [ ] The checksums of `ref.bin` and `md0` are identical
        - [ ] `md0`'s capacity matches that of one disk

### Étape 4 : Créer le RAID 5 / Step 4: Create the RAID 5

```bash
sudo mdadm --create /dev/md1 --level=5 --raid-devices=3 /dev/<D3> /dev/<D4> /dev/<D5>
cat /proc/mdstat
sudo mdadm --detail /dev/md1
```

=== "Français"

    Pendant la création, la grappe passe par une phase de **construction de la parité** (`recovery` ou `resync` dans `/proc/mdstat`) : les disques sont utilisables, mais la redondance n'est complète qu'à la fin. Dans `--detail`, relevez `Layout` (disposition de la parité, par exemple `left-symmetric`) et `Chunk Size` (taille du bloc de répartition). Attendez la fin de la construction (`watch cat /proc/mdstat`), puis calculez la capacité attendue avec la formule du cours **(n − 1) × C** et comparez :

=== "English"

    During creation, the array goes through a **parity building** phase (`recovery` or `resync` in `/proc/mdstat`): the disks are usable, but redundancy is only complete at the end. In `--detail`, note `Layout` (parity layout, for example `left-symmetric`) and `Chunk Size` (stripe chunk size). Wait for the build to finish (`watch cat /proc/mdstat`), then compute the expected capacity with the lesson's formula **(n − 1) × C** and compare:

```bash
lsblk -o NAME,SIZE,TYPE /dev/md1
```

=== "Français"

    Écrivez le fichier de référence sur cette grappe et vérifiez-le :

=== "English"

    Write the reference file onto this array and verify it:

```bash
sudo dd if=~/ref.bin of=/dev/md1 bs=1M oflag=direct status=none
sudo head -c 104857600 /dev/md1 | sha256sum
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] `md1` est `active raid5`, `[3/3] [UUU]`, état `clean`
        - [ ] Capacité observée ≈ (3 − 1) × 2 Go ; empreinte identique à celle de `ref.bin`

=== "English"

    !!! success "Checkpoint"
        - [ ] `md1` is `active raid5`, `[3/3] [UUU]`, state `clean`
        - [ ] Observed capacity ≈ (3 − 1) × 2 GB; checksum identical to `ref.bin`'s

### Étape 5 : Examiner les métadonnées / Step 5: Examine the metadata

```bash
sudo mdadm --examine /dev/<D3>
```

=== "Français"

    Chaque disque membre porte un **superbloc** (métadonnées) qui l'identifie : UUID de la grappe, niveau, rôle du disque, état. C'est ce qui permet de **réassembler** une grappe sur un autre système. Repérez l'UUID et le champ `Device Role`.

=== "English"

    Each member disk carries a **superblock** (metadata) that identifies it: array UUID, level, disk role, state. This is what allows an array to be **reassembled** on another system. Find the UUID and the `Device Role` field.

### Étape 6 : Rendre la configuration permanente / Step 6: Make the configuration permanent

```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u
sudo reboot
```

=== "Français"

    Après le redémarrage, reconnectez-vous et vérifiez que les deux grappes sont revenues, **avec leurs noms** et leurs données :

=== "English"

    After the restart, log in again and check that both arrays are back, **with their names** and their data:

```bash
cat /proc/mdstat
sudo head -c 104857600 /dev/md0 | sha256sum
sudo head -c 104857600 /dev/md1 | sha256sum
sha256sum ~/ref.bin
```

=== "Français"

    Sans l'entrée dans `mdadm.conf` et la mise à jour de l'image de démarrage, une grappe peut réapparaître sous un autre nom (`md127`, par exemple). Si les noms ont changé, relevez-les et reprenez la configuration.

    !!! success "Point de contrôle"
        - [ ] `md0` et `md1` sont actives après le redémarrage
        - [ ] Les empreintes des deux grappes sont celles de `ref.bin`

=== "English"

    Without the `mdadm.conf` entry and the boot image update, an array may reappear under another name (`md127`, for example). If the names changed, note them and redo the configuration.

    !!! success "Checkpoint"
        - [ ] `md0` and `md1` are active after the restart
        - [ ] Both arrays' checksums are those of `ref.bin`

### Étape 7 : Fiche de synthèse / Step 7: Summary sheet

=== "Français"

    Remplissez la fiche ci-dessous : c'est le **livrable** du lab.

    | Rubrique | Valeur |
    |---|---|
    | Noms des cinq disques et affectation (RAID 1 / RAID 5) | … |
    | RAID 1 : état, `[x/y]`, capacité observée | … |
    | RAID 5 : état, `[x/y]`, `Layout`, `Chunk Size`, capacité observée | … |
    | Capacité attendue (formule du cours) pour le RAID 5 | … |
    | Empreintes : `ref.bin`, `md0`, `md1` | … |
    | UUID d'une grappe (`--examine` ou `--detail`) | … |
    | Grappes retrouvées après redémarrage, mêmes noms ? | oui / non |

=== "English"

    Fill in the sheet below: this is the lab **deliverable**.

    | Item | Value |
    |---|---|
    | Names of the five disks and assignment (RAID 1 / RAID 5) | … |
    | RAID 1: state, `[x/y]`, observed capacity | … |
    | RAID 5: state, `[x/y]`, `Layout`, `Chunk Size`, observed capacity | … |
    | Expected capacity (lesson formula) for the RAID 5 | … |
    | Checksums: `ref.bin`, `md0`, `md1` | … |
    | UUID of an array (`--examine` or `--detail`) | … |
    | Arrays found after restart, same names? | yes / no |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi la capacité du RAID 5 de trois disques de 2 Go est-elle d'environ 4 Go ?"
        Un RAID 5 consacre l'équivalent d'un disque à la parité : la capacité utile est (n − 1) × C = 2 × 2 Go = 4 Go. La parité est répartie sur les trois disques, mais le total « perdu » est celui d'un disque.

    ??? question "Que signifie `[UUU]` dans `/proc/mdstat`, et que montrerait `[U_U]` ?"
        Chaque `U` correspond à un disque actif. `[UUU]` : les trois disques fonctionnent. `[U_U]` : un disque (le deuxième) est absent ou en panne : la grappe est **dégradée** (Lab 8.2).

    ??? question "Cette grappe protège-t-elle vraiment les données contre la panne du disque de votre poste ?"
        Non : les cinq disques virtuels sont des fichiers stockés sur le même disque physique. Si celui-ci tombe, tous disparaissent ensemble. La grappe ne protège que contre la panne d'un disque virtuel, ce qui sert ici à comprendre le fonctionnement.

=== "English"

    ??? question "Why is the capacity of a three-disk 2 GB RAID 5 about 4 GB?"
        A RAID 5 dedicates the equivalent of one disk to parity: usable capacity is (n − 1) × C = 2 × 2 GB = 4 GB. Parity is spread over the three disks, but the total "lost" is that of one disk.

    ??? question "What does `[UUU]` mean in `/proc/mdstat`, and what would `[U_U]` show?"
        Each `U` stands for an active disk. `[UUU]`: all three disks work. `[U_U]`: one disk (the second) is missing or failed: the array is **degraded** (Lab 8.2).

    ??? question "Does this array really protect the data against the failure of your computer's disk?"
        No: the five virtual disks are files stored on the same physical disk. If it fails, they all disappear together. The array only protects against the failure of a virtual disk, which serves here to understand how it works.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | `mdadm: command not found` | `sudo apt install -y mdadm` (accès Internet par la carte NAT) |
    | `mdadm: device or resource busy` ou « Cannot get exclusive access » | Le disque est déjà utilisé (monté, ou membre d'une autre grappe) : vérifier avec `lsblk` et `cat /proc/mdstat` ; en cas de reste d'une ancienne grappe, `sudo mdadm --stop /dev/mdX` puis `sudo mdadm --zero-superblock /dev/<disque>` |
    | `mdadm: ... No such file or directory` | Mauvais nom de disque : revérifier avec `lsblk` |
    | La grappe réapparaît sous le nom `md127` | Entrée manquante dans `/etc/mdadm/mdadm.conf` ou image de démarrage non mise à jour (étape 6) |
    | Synchronisation très lente | Les disques virtuels partagent le même disque physique ; attendre, ne pas lancer d'autre charge lourde |
    | Les empreintes diffèrent | La synchronisation n'était pas finie ou `dd` a été interrompu ; recréer le fichier et recommencer l'écriture |
    | Le mot de passe `sudo` est redemandé après le redémarrage | Normal : nouvelle session |

=== "English"

    | Symptom | Hints |
    |---|---|
    | `mdadm: command not found` | `sudo apt install -y mdadm` (Internet access through the NAT card) |
    | `mdadm: device or resource busy` or "Cannot get exclusive access" | The disk is already in use (mounted, or member of another array): check with `lsblk` and `cat /proc/mdstat`; for a leftover of an old array, `sudo mdadm --stop /dev/mdX` then `sudo mdadm --zero-superblock /dev/<disk>` |
    | `mdadm: ... No such file or directory` | Wrong disk name: recheck with `lsblk` |
    | The array reappears under the name `md127` | Missing entry in `/etc/mdadm/mdadm.conf` or boot image not updated (step 6) |
    | Very slow synchronization | The virtual disks share the same physical disk; wait, do not start another heavy load |
    | The checksums differ | Synchronization was not finished or `dd` was interrupted; recreate the file and redo the write |
    | The `sudo` password is asked again after the restart | Normal: new session |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    **Conservez les grappes** si vous enchaînez avec le [Lab 8.2](lab-8-2-panne-remplacement-et-reconstruction.md) (panne et reconstruction), qui part de cet état. Pour tout défaire : arrêtez les grappes, effacez les métadonnées des disques, puis retirez les lignes ajoutées dans `/etc/mdadm/mdadm.conf` et mettez à jour l'image de démarrage :

=== "English"

    **Keep the arrays** if you continue with [Lab 8.2](lab-8-2-panne-remplacement-et-reconstruction.md) (failure and rebuild), which starts from this state. To undo everything: stop the arrays, erase the disks' metadata, then remove the lines added to `/etc/mdadm/mdadm.conf` and update the boot image:

```bash
sudo mdadm --stop /dev/md0 /dev/md1
sudo mdadm --zero-superblock /dev/<D1> /dev/<D2> /dev/<D3> /dev/<D4> /dev/<D5>
sudo nano /etc/mdadm/mdadm.conf
sudo update-initramfs -u
```

=== "Français"

    Pour retrouver des disques vierges, restaurez simplement le snapshot `etat-disques` (les deux disques ajoutés dans ce lab seront alors à supprimer des paramètres de la VM).

=== "English"

    To get blank disks back, simply restore the `etat-disques` snapshot (the two disks added in this lab will then have to be removed from the VM settings).

[Retour au chapitre 8 / Back to chapter 8](cours.md)