# Lab 8.2 : Panne, remplacement et reconstruction / Failure, Replacement and Rebuild

| | |
|---|---|
| **Durée / Duration** | 75 à 90 min / 75 to 90 min |
| **Niveau / Level** | Guidé / Guided |
| **Prérequis / Prerequisites** | [Chapitre 8 : cours](cours.md) ; [Lab 8.1](lab-8-1-raid-1-et-raid-5-avec-mdadm.md) réalisé (grappes `md0` en RAID 1 et `md1` en RAID 5, fichier `~/ref.bin`) |
| **Ressources minimales / Minimum resources** | **VM** : `srv1-vmware` (2 vCPU, 2 Go de RAM) avec les 5 disques du Lab 8.1 et **2 disques vides supplémentaires de 2 Go**, dynamiques. **Hôte** : VMware Workstation, au moins 12 Go d'espace disque libre. L'hyperviseur Windows doit être **désactivé**. / **VM**: `srv1-vmware` (2 vCPUs, 2 GB RAM) with the 5 disks from Lab 8.1 and **2 extra empty 2 GB disks**, dynamic. **Host**: VMware Workstation, at least 12 GB of free disk space. The Windows hypervisor must be **off**. |
| **Acquis visés / Learning outcomes** | AA3 |

=== "Français"

    !!! abstract "Objectifs"
        - Simuler la panne d'un disque dans un RAID 1 et dans un RAID 5, et constater le mode dégradé
        - Vérifier que les données restent accessibles malgré la panne
        - Remplacer un disque et suivre la reconstruction
        - Utiliser un disque de secours (*hot spare*) pour une reconstruction automatique
        - Constater les limites : une seconde panne dans un RAID 5, et le contrôle de cohérence (*scrub*)

=== "English"

    !!! abstract "Objectives"
        - Simulate a disk failure in a RAID 1 and in a RAID 5, and observe degraded mode
        - Verify that data remains accessible despite the failure
        - Replace a disk and follow the rebuild
        - Use a hot spare for an automatic rebuild
        - Observe the limits: a second failure in a RAID 5, and the consistency check (*scrub*)

## Contexte et schéma / Context and diagram

=== "Français"

    Vous reprenez les grappes du Lab 8.1 et vous les mettez à l'épreuve : une **panne simulée**, puis un **remplacement** manuel sur le RAID 1, et une reconstruction **automatique** sur un disque de secours pour le RAID 5. Une panne est simulée par `mdadm --fail` : aucun disque n'est vraiment endommagé, ce qui rend l'exercice sans danger pour votre poste.

    ```mermaid
    flowchart LR
      N["Grappe saine<br/>[UU]"] -- "--fail" --> D["Mode dégradé<br/>[U_]"]
      D -- "--remove puis --add" --> R["Reconstruction<br/>(recovery)"]
      R --> N
    ```

    !!! warning "Des disques vides pour remplacer"
        Les disques de remplacement et de secours (`<D6>` et `<D7>`) doivent être **vides** et d'une taille **au moins égale** à celle des disques membres : un disque plus petit est refusé. Ne sélectionnez jamais le disque système de la VM.

=== "English"

    You take the Lab 8.1 arrays and put them to the test: a **simulated failure**, then a manual **replacement** on the RAID 1, and an **automatic** rebuild onto a hot spare for the RAID 5. A failure is simulated with `mdadm --fail`: no disk is really damaged, which makes the exercise safe for your computer.

    ```mermaid
    flowchart LR
      N["Healthy array<br/>[UU]"] -- "--fail" --> D["Degraded mode<br/>[U_]"]
      D -- "--remove then --add" --> R["Rebuild<br/>(recovery)"]
      R --> N
    ```

    !!! warning "Empty disks as replacements"
        Replacement and spare disks (`<D6>` and `<D7>`) must be **empty** and at least **as large** as the member disks: a smaller disk is refused. Never select the VM's system disk.

## Étapes / Steps

### Étape 1 : Vérifier l'état de départ et ajouter deux disques / Step 1: Check the starting state and add two disks

=== "Français"

    1. Vérifiez que les deux grappes sont saines et que les données sont intactes :

=== "English"

    1. Check that both arrays are healthy and that the data is intact:

```bash
cat /proc/mdstat
sudo head -c 104857600 /dev/md0 | sha256sum
sudo head -c 104857600 /dev/md1 | sha256sum
sha256sum ~/ref.bin
```

=== "Français"

    2. Éteignez la VM, ajoutez **deux disques** de 2 Go, SCSI, dynamiques (comme au Lab 7.1, étape 2), puis redémarrez-la. Identifiez-les avec `lsblk` : ils sont vides, sans système de fichiers. Notez leurs noms : `<D6>` (remplacement du RAID 1) et `<D7>` (disque de secours du RAID 5). Rappel des disques du Lab 8.1 : `<D1>` et `<D2>` pour `md0` ; `<D3>`, `<D4>` et `<D5>` pour `md1`.

    !!! success "Point de contrôle"
        - [ ] `md0` en `[UU]` et `md1` en `[UUU]`, empreintes identiques à celle de `ref.bin`
        - [ ] `<D6>` et `<D7>` sont identifiés, vides, et de même taille que les disques membres

=== "English"

    2. Shut the VM down, add **two disks** of 2 GB, SCSI, dynamic (as in Lab 7.1, step 2), then restart it. Identify them with `lsblk`: they are empty, without file system. Note their names: `<D6>` (RAID 1 replacement) and `<D7>` (RAID 5 hot spare). Reminder of the Lab 8.1 disks: `<D1>` and `<D2>` for `md0`; `<D3>`, `<D4>` and `<D5>` for `md1`.

    !!! success "Checkpoint"
        - [ ] `md0` in `[UU]` and `md1` in `[UUU]`, checksums identical to `ref.bin`'s
        - [ ] `<D6>` and `<D7>` are identified, empty, and the same size as the member disks

### Étape 2 : Panne dans le RAID 1 / Step 2: Failure in the RAID 1

=== "Français"

    Simulez la panne du disque `<D2>` :

=== "English"

    Simulate the failure of disk `<D2>`:

```bash
sudo mdadm /dev/md0 --fail /dev/<D2>
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

=== "Français"

    Constatez : `[2/1] [U_]`, le disque marqué `(F)`, un état `degraded` et un disque en `faulty` dans `--detail`. Vérifiez que **les données sont toujours accessibles** et qu'on peut **continuer à écrire** (dans une autre zone de la grappe) :

=== "English"

    Observe: `[2/1] [U_]`, the disk marked `(F)`, a `degraded` state and a disk `faulty` in `--detail`. Check that **the data is still accessible** and that you can **keep writing** (in another area of the array):

```bash
sudo head -c 104857600 /dev/md0 | sha256sum
dd if=/dev/urandom of=~/ref2.bin bs=1M count=50 status=none
sha256sum ~/ref2.bin
sudo dd if=~/ref2.bin of=/dev/md0 bs=1M seek=200 oflag=direct status=none
sudo dd if=/dev/md0 bs=1M skip=200 count=50 status=none | sha256sum
sudo dmesg | grep -i "md0" | tail -5
```

=== "Français"

    Les empreintes doivent correspondre à `ref.bin` (premier bloc) et à `ref2.bin` (zone écrite en mode dégradé). Les messages du noyau (`dmesg`) gardent la trace de la panne.

    !!! success "Point de contrôle"
        - [ ] `md0` est en `[U_]`, `degraded`, et les deux empreintes sont correctes
        - [ ] Le journal du noyau mentionne le disque défaillant

=== "English"

    The checksums must match `ref.bin` (first block) and `ref2.bin` (area written in degraded mode). Kernel messages (`dmesg`) trace the failure.

    !!! success "Checkpoint"
        - [ ] `md0` is in `[U_]`, `degraded`, and both checksums are correct
        - [ ] The kernel log mentions the failed disk

### Étape 3 : Remplacer le disque et suivre la reconstruction / Step 3: Replace the disk and follow the rebuild

```bash
sudo mdadm /dev/md0 --remove /dev/<D2>
sudo mdadm /dev/md0 --add /dev/<D6>
watch cat /proc/mdstat
```

=== "Français"

    On **retire** d'abord le disque défaillant, puis on **ajoute** le disque de remplacement : la reconstruction (`recovery`) démarre aussitôt et recopie les données de `<D1>` sur `<D6>`. Notez le pourcentage et la durée (++ctrl+c++ pour quitter `watch`). À la fin, relevez l'état puis vérifiez **les deux zones de données** :

=== "English"

    The failed disk is first **removed**, then the replacement disk is **added**: the rebuild (`recovery`) starts at once and copies data from `<D1>` onto `<D6>`. Note the percentage and the duration (++ctrl+c++ to quit `watch`). At the end, note the state then verify **both data areas**:

```bash
sudo mdadm --detail /dev/md0
sudo head -c 104857600 /dev/md0 | sha256sum
sudo dd if=/dev/md0 bs=1M skip=200 count=50 status=none | sha256sum
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] `md0` est de nouveau en `[UU]`, état `clean`
        - [ ] Les données écrites avant **et pendant** la panne sont intactes

=== "English"

    !!! success "Checkpoint"
        - [ ] `md0` is back in `[UU]`, state `clean`
        - [ ] Data written before **and during** the failure is intact

### Étape 4 : Ajouter un disque de secours au RAID 5 / Step 4: Add a hot spare to the RAID 5

```bash
sudo mdadm /dev/md1 --add /dev/<D7>
cat /proc/mdstat
sudo mdadm --detail /dev/md1
```

=== "Français"

    Comme la grappe n'est pas dégradée, le disque ajouté devient **disque de secours** : il est repéré par `(S)` dans `/proc/mdstat` et `Spare Devices : 1` dans `--detail`. Il n'est pas utilisé tant qu'aucun disque ne tombe en panne.

    !!! success "Point de contrôle"
        - [ ] `md1` en `[3/3] [UUU]` avec un disque de secours `(S)`

=== "English"

    As the array is not degraded, the added disk becomes a **hot spare**: it is marked `(S)` in `/proc/mdstat` and `Spare Devices : 1` in `--detail`. It is not used as long as no disk fails.

    !!! success "Checkpoint"
        - [ ] `md1` in `[3/3] [UUU]` with a hot spare `(S)`

### Étape 5 : Panne dans le RAID 5, reconstruction automatique / Step 5: Failure in the RAID 5, automatic rebuild

```bash
sudo mdadm /dev/md1 --fail /dev/<D4>
cat /proc/mdstat
```

=== "Français"

    Relancez immédiatement `cat /proc/mdstat` : `mdadm` a **automatiquement** affecté le disque de secours et lancé la reconstruction (`recovery`), sans intervention. Pendant cette phase, la grappe est en `[3/2]` : elle n'a plus de redondance. Vérifiez que les données restent accessibles malgré la parité en cours de reconstruction :

=== "English"

    Immediately rerun `cat /proc/mdstat`: `mdadm` **automatically** assigned the hot spare and started the rebuild (`recovery`), with no intervention. During this phase, the array is in `[3/2]`: it has no redundancy left. Check that data stays accessible while parity is being rebuilt:

```bash
sudo head -c 104857600 /dev/md1 | sha256sum
watch cat /proc/mdstat
```

=== "Français"

    Une fois la reconstruction terminée, la grappe est de nouveau en `[3/3] [UUU]` **sans qu'aucun disque n'ait été ajouté à la main**. Retirez le disque défaillant :

=== "English"

    Once the rebuild is finished, the array is back in `[3/3] [UUU]` **without any disk added by hand**. Remove the failed disk:

```bash
sudo mdadm /dev/md1 --remove /dev/<D4>
sudo mdadm --detail /dev/md1
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] La reconstruction a démarré sans commande `--add`
        - [ ] `md1` est de nouveau en `[UUU]` et son empreinte est celle de `ref.bin`
        - [ ] Vous avez noté la durée de la reconstruction et expliqué pourquoi la grappe est vulnérable pendant ce temps

=== "English"

    !!! success "Checkpoint"
        - [ ] The rebuild started without an `--add` command
        - [ ] `md1` is back in `[UUU]` and its checksum is `ref.bin`'s
        - [ ] You noted the rebuild duration and explained why the array is vulnerable during that time

### Étape 6 : Contrôler la cohérence (scrub) / Step 6: Check consistency (scrub)

```bash
echo check | sudo tee /sys/block/md1/md/sync_action
cat /proc/mdstat
cat /sys/block/md1/md/mismatch_cnt
```

=== "Français"

    L'opération `check` relit tous les blocs et recalcule les parités pour détecter les incohérences. Attendez la fin (`watch cat /proc/mdstat`), puis relevez `mismatch_cnt` : une valeur de **0** indique une grappe cohérente.

    !!! success "Point de contrôle"
        - [ ] Le contrôle a été lancé et terminé, `mismatch_cnt` est noté

=== "English"

    The `check` operation rereads all blocks and recomputes parities to detect inconsistencies. Wait for it to finish (`watch cat /proc/mdstat`), then note `mismatch_cnt`: a value of **0** indicates a consistent array.

    !!! success "Checkpoint"
        - [ ] The check was started and finished, `mismatch_cnt` is noted

### Étape 7 : Les limites du RAID 5, une seconde panne (facultatif) / Step 7: RAID 5 limits, a second failure (optional)

=== "Français"

    **Cette étape rend `md1` inutilisable** : elle n'a de sens que si vous n'avez plus besoin de la grappe. Simulez la panne de deux disques membres (`<D3>` puis `<D5>`) et tentez de lire :

=== "English"

    **This step makes `md1` unusable**: it only makes sense if you no longer need the array. Simulate the failure of two member disks (`<D3>` then `<D5>`) and try to read:

```bash
sudo mdadm /dev/md1 --fail /dev/<D3>
sudo mdadm /dev/md1 --fail /dev/<D5>
cat /proc/mdstat
sudo head -c 104857600 /dev/md1 | sha256sum
```

=== "Français"

    Résultat attendu : la grappe passe en échec, la lecture échoue ou renvoie une erreur. Un RAID 5 ne tolère **qu'une seule** panne ; il aurait fallu un RAID 6. Pour réutiliser les disques, supprimez la grappe et recommencez avec le Lab 8.1 (étape 4).

=== "English"

    Expected result: the array fails, the read fails or returns an error. A RAID 5 only tolerates **one** failure; a RAID 6 would have been needed. To reuse the disks, delete the array and start again with Lab 8.1 (step 4).

### Étape 8 : Fiche de synthèse / Step 8: Summary sheet

=== "Français"

    Remplissez la fiche ci-dessous : c'est le **livrable** du lab.

    | Étape | État de la grappe (`[x/y] [UU…]`) | Données intactes ? | Remarque (durée, messages) |
    |---|---|---|---|
    | RAID 1 sain (départ) | … | … | … |
    | RAID 1 après `--fail` | … | … | … |
    | RAID 1 pendant la reconstruction | … | … | … |
    | RAID 1 après remplacement | … | … | … |
    | RAID 5 avec disque de secours | … | … | … |
    | RAID 5 juste après `--fail` | … | … | … |
    | RAID 5 après reconstruction automatique | … | … | … |
    | Contrôle de cohérence : `mismatch_cnt` | … | — | … |

=== "English"

    Fill in the sheet below: this is the lab **deliverable**.

    | Step | Array state (`[x/y] [UU…]`) | Data intact? | Remark (duration, messages) |
    |---|---|---|---|
    | RAID 1 healthy (start) | … | … | … |
    | RAID 1 after `--fail` | … | … | … |
    | RAID 1 during rebuild | … | … | … |
    | RAID 1 after replacement | … | … | … |
    | RAID 5 with hot spare | … | … | … |
    | RAID 5 right after `--fail` | … | … | … |
    | RAID 5 after automatic rebuild | … | … | … |
    | Consistency check: `mismatch_cnt` | … | — | … |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Comment les données restent-elles accessibles quand un disque est en panne, en RAID 1 et en RAID 5 ?"
        En RAID 1, le disque restant contient une **copie complète** des données. En RAID 5, les blocs manquants sont **recalculés** à la volée à partir des blocs restants et de la parité (XOR, chapitre 8). Dans les deux cas, la grappe est plus lente et n'a plus de redondance.

    ??? question "Quel avantage apporte un disque de secours ?"
        La reconstruction démarre **immédiatement et automatiquement**, sans attendre qu'un administrateur remplace le disque. La période pendant laquelle la grappe est vulnérable (sans redondance) est donc la plus courte possible.

    ??? question "Pourquoi contrôler régulièrement la cohérence d'une grappe (scrub) ?"
        Pour détecter des incohérences ou des secteurs illisibles **avant** qu'une panne ne rende la reconstruction impossible : lors d'une reconstruction, tous les disques restants doivent être lus sans erreur.

=== "English"

    ??? question "How does data stay accessible when a disk fails, in RAID 1 and in RAID 5?"
        In RAID 1, the remaining disk holds a **complete copy** of the data. In RAID 5, missing blocks are **recomputed** on the fly from the remaining blocks and the parity (XOR, chapter 8). In both cases, the array is slower and has no redundancy left.

    ??? question "What advantage does a hot spare bring?"
        The rebuild starts **immediately and automatically**, without waiting for an administrator to replace the disk. The period during which the array is vulnerable (without redundancy) is therefore as short as possible.

    ??? question "Why regularly check an array's consistency (scrub)?"
        To detect inconsistencies or unreadable sectors **before** a failure makes rebuilding impossible: during a rebuild, all remaining disks must be read without error.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | `mdadm: set device faulty failed ... No such device` | Mauvais nom de disque ou disque déjà retiré : vérifier avec `cat /proc/mdstat` |
    | `mdadm: ... not large enough to join array` | Le disque de remplacement est plus petit que les membres : utiliser un disque d'au moins la même taille |
    | `mdadm: ... device or resource busy` à l'ajout | Le disque contient d'anciennes métadonnées ou est utilisé ailleurs : `sudo mdadm --zero-superblock /dev/<disque>` s'il est vide et inutilisé |
    | La reconstruction ne démarre pas avec le disque de secours | Le disque de secours n'a pas été ajouté **avant** la panne ; l'ajouter maintenant lance la reconstruction |
    | La reconstruction est lente | Les disques virtuels partagent le même disque physique ; attendre, éviter les charges concurrentes |
    | Après la seconde panne, `md1` est `inactive` | Résultat attendu d'un RAID 5 avec deux pannes (étape 7) |
    | Les grappes ont changé de nom | Voir le dépannage du Lab 8.1 (`mdadm.conf`, image de démarrage) |

=== "English"

    | Symptom | Hints |
    |---|---|
    | `mdadm: set device faulty failed ... No such device` | Wrong disk name or disk already removed: check with `cat /proc/mdstat` |
    | `mdadm: ... not large enough to join array` | The replacement disk is smaller than the members: use a disk at least as large |
    | `mdadm: ... device or resource busy` when adding | The disk holds old metadata or is used elsewhere: `sudo mdadm --zero-superblock /dev/<disk>` if it is empty and unused |
    | The rebuild does not start with the hot spare | The spare was not added **before** the failure; adding it now starts the rebuild |
    | The rebuild is slow | The virtual disks share the same physical disk; wait, avoid concurrent loads |
    | After the second failure, `md1` is `inactive` | Expected result of a RAID 5 with two failures (step 7) |
    | The arrays changed name | See Lab 8.1 troubleshooting (`mdadm.conf`, boot image) |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    Pour tout défaire : arrêtez les grappes, effacez les métadonnées de **tous** les disques utilisés (`<D1>` à `<D7>`), retirez les lignes ajoutées à `/etc/mdadm/mdadm.conf` et mettez à jour l'image de démarrage :

=== "English"

    To undo everything: stop the arrays, erase the metadata of **all** the disks used (`<D1>` to `<D7>`), remove the lines added to `/etc/mdadm/mdadm.conf` and update the boot image:

```bash
sudo mdadm --stop /dev/md0 /dev/md1
sudo mdadm --zero-superblock /dev/<D1> /dev/<D2> /dev/<D3> /dev/<D4> /dev/<D5> /dev/<D6> /dev/<D7>
sudo nano /etc/mdadm/mdadm.conf
sudo update-initramfs -u
```

=== "Français"

    Pour retrouver une VM sans disques supplémentaires, restaurez le snapshot `etat-propre` et supprimez les fichiers `disque-*.vmdk` des disques ajoutés. Le [Lab 8.3](lab-8-3-choisir-un-niveau-raid-selon-un-cahier-des-charges.md) est un exercice d'étude de cas : il ne demande pas de grappe.

=== "English"

    To get a VM without extra disks back, restore the `etat-propre` snapshot and delete the `disque-*.vmdk` files of the added disks. [Lab 8.3](lab-8-3-choisir-un-niveau-raid-selon-un-cahier-des-charges.md) is a case study exercise: it does not require an array.

[Retour au chapitre 8 / Back to chapter 8](cours.md)