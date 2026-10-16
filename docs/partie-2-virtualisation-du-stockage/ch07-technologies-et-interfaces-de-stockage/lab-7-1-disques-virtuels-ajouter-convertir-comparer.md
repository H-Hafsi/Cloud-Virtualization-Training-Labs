# Lab 7.1 : Disques virtuels, ajouter, convertir, comparer / Virtual Disks: Add, Convert, Compare

|                                              |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Durée / Duration**                         | 75 à 90 min / 75 to 90 min                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| **Niveau / Level**                           | Guidé / Guided                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| **Prérequis / Prerequisites**                | [Chapitre 7 : cours](cours.md) ; [Lab 3.2](../../partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch03-virtualbox-et-vmware-workstation/lab-3-2-vmware-workstation-meme-vm-import-et-export.md) réalisé (VM `srv1-vmware`, snapshot `etat-propre`)                                                                                                                                                                                                                                                                                               |
| **Ressources minimales / Minimum resources** | **VM** : `srv1-vmware` (2 vCPU, 2 Go de RAM) avec 3 disques supplémentaires de 2 Go chacun. **Hôte** : VMware Workstation, au moins 15 Go d'espace disque libre (un des disques est réservé en totalité), accès Internet pour installer un paquet. L'hyperviseur Windows doit être **désactivé**. / **VM**: `srv1-vmware` (2 vCPUs, 2 GB RAM) with 3 extra disks of 2 GB each. **Host**: VMware Workstation, at least 15 GB of free disk space (one disk is fully reserved), Internet access to install a package. The Windows hypervisor must be **off**. |
| **Acquis visés / Learning outcomes**         | AA3                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |

=== "Français"

    !!! abstract "Objectifs"
        - Ajouter des disques virtuels sur différents contrôleurs (SCSI, NVMe) avec un provisionnement dynamique ou fixe
        - Les observer depuis l'invité et depuis l'hôte
        - Convertir une image de disque entre les formats raw, qcow2, VMDK, VDI et VHDX avec `qemu-img`
        - Vérifier l'intégrité d'une conversion et créer un disque chaîné
        - Comparer taille réelle et taille déclarée

=== "English"

    !!! abstract "Objectives"
        - Add virtual disks on different controllers (SCSI, NVMe) with dynamic or fixed provisioning
        - Observe them from the guest and from the host
        - Convert a disk image between raw, qcow2, VMDK, VDI and VHDX formats with `qemu-img`
        - Verify the integrity of a conversion and create a chained disk
        - Compare actual and declared size

## Contexte et schéma / Context and diagram

=== "Français"

    Vous travaillez sur la VM de référence `srv1-vmware`. Dans la première partie, vous lui ajoutez **trois disques vides** de 2 Go sur des contrôleurs et des provisionnements différents ; ces disques serviront au [Lab 7.2](lab-7-2-mesurer-les-performances-avec-fio.md) (mesure de performances). Dans la seconde partie, vous manipulez des **images de disque** dans la VM avec `qemu-img`.

    ```mermaid
    flowchart LR
      VM["VM srv1-vmware"] --- A["Disque A<br/>SCSI, dynamique"]
      VM --- B["Disque B<br/>SCSI, réservé"]
      VM --- C["Disque C<br/>NVMe, dynamique"]
      VM --> Q["Images dans la VM<br/>raw, qcow2, VMDK, VDI, VHDX"]
    ```

    !!! warning "Disques vides, non formatés"
        Les trois disques ajoutés restent **vides et sans système de fichiers** : les partitions et systèmes de fichiers sont étudiés au chapitre 9. Au Lab 7.2, ils seront testés tels quels. Ne les utilisez pour rien d'autre, car ce test **écrit sur le disque entier**.

=== "English"

    You work on the `srv1-vmware` reference VM. In the first part, you add **three empty disks** of 2 GB on different controllers and provisioning types; these disks will be used in [Lab 7.2](lab-7-2-mesurer-les-performances-avec-fio.md) (performance measurement). In the second part, you handle **disk images** inside the VM with `qemu-img`.

    ```mermaid
    flowchart LR
      VM["VM srv1-vmware"] --- A["Disk A<br/>SCSI, dynamic"]
      VM --- B["Disk B<br/>SCSI, reserved"]
      VM --- C["Disk C<br/>NVMe, dynamic"]
      VM --> Q["Images inside the VM<br/>raw, qcow2, VMDK, VDI, VHDX"]
    ```

    !!! warning "Empty, unformatted disks"
        The three added disks stay **empty and without a file system**: partitions and file systems are studied in chapter 9. In Lab 7.2, they will be tested as they are. Do not use them for anything else, as this test **writes on the whole disk**.

## Étapes / Steps

### Étape 1 : Préparer la VM / Step 1: Prepare the VM

=== "Français"

    1. Éteignez `srv1-vmware`. Si elle a été modifiée depuis, restaurez le snapshot `etat-propre` (**VM**, **Snapshot**, **Snapshot Manager**).
    2. Vérifiez que l'hyperviseur Windows est désactivé (`(Get-CimInstance Win32_ComputerSystem).HypervisorPresent` vaut `False`, voir le Lab 2.1).
    3. Notez l'**espace libre** de l'hôte sur le disque des VM.

    !!! success "Point de contrôle"
        - [ ] VM éteinte, état propre, hyperviseur Windows désactivé

=== "English"

    1. Shut `srv1-vmware` down. If it was changed since, restore the `etat-propre` snapshot (**VM**, **Snapshot**, **Snapshot Manager**).
    2. Check that the Windows hypervisor is off (`(Get-CimInstance Win32_ComputerSystem).HypervisorPresent` is `False`, see Lab 2.1).
    3. Note the host's **free space** on the VM disk.

    !!! success "Checkpoint"
        - [ ] VM powered off, clean state, Windows hypervisor off

### Étape 2 : Ajouter trois disques / Step 2: Add three disks

=== "Français"

    Dans **Edit virtual machine settings**, cliquez sur **Add**, **Hard Disk**, **Next**, puis pour chaque disque :

    | Disque | Type de contrôleur | Capacité | « Allocate all disk space now » | Fichier |
    |---|---|---|---|---|
    | **A** | SCSI | 2 Go | **décoché** (dynamique) | `disque-a-scsi-dyn.vmdk` |
    | **B** | SCSI | 2 Go | **coché** (réservé) | `disque-b-scsi-fixe.vmdk` |
    | **C** | NVMe | 2 Go | **décoché** (dynamique) | `disque-c-nvme-dyn.vmdk` |

    Choisissez **Create a new virtual disk**, puis **Store virtual disk as a single file** pour chacun. La création du disque B prend un peu plus de temps, car l'espace est réservé d'avance.

    !!! success "Point de contrôle"
        - [ ] Les trois disques apparaissent dans la liste du matériel de la VM

=== "English"

    In **Edit virtual machine settings**, click **Add**, **Hard Disk**, **Next**, then for each disk:

    | Disk | Controller type | Capacity | "Allocate all disk space now" | File |
    |---|---|---|---|---|
    | **A** | SCSI | 2 GB | **unchecked** (dynamic) | `disque-a-scsi-dyn.vmdk` |
    | **B** | SCSI | 2 GB | **checked** (reserved) | `disque-b-scsi-fixe.vmdk` |
    | **C** | NVMe | 2 GB | **unchecked** (dynamic) | `disque-c-nvme-dyn.vmdk` |

    Choose **Create a new virtual disk**, then **Store virtual disk as a single file** for each. Creating disk B takes a bit longer, as the space is reserved in advance.

    !!! success "Checkpoint"
        - [ ] The three disks appear in the VM's hardware list

### Étape 3 : Observer les fichiers sur l'hôte / Step 3: Observe the files on the host

=== "Français"

    Avant de démarrer la VM, comparez la taille réelle des fichiers des disques (remplacez le chemin par celui du dossier de la VM) :

=== "English"

    Before starting the VM, compare the actual size of the disk files (replace the path with the VM's folder):

```powershell
Get-ChildItem "C:\chemin\vers\vos-vm\srv1-vmware" -Filter "disque-*.vmdk" | Select-Object Name, @{n='Mo';e={[math]::Round($_.Length/1MB,1)}}
```

=== "Français"

    Résultat attendu : le disque B (réservé) occupe environ **2 Go** dès sa création ; les disques A et C (dynamiques) ne pèsent que quelques centaines de Ko à quelques Mo. Notez ces tailles dans la fiche.

    !!! success "Point de contrôle"
        - [ ] Vous avez relevé trois tailles et expliqué pourquoi B est différent

=== "English"

    Expected result: disk B (reserved) takes about **2 GB** from creation; disks A and C (dynamic) are only a few hundred KB to a few MB. Note these sizes in the sheet.

    !!! success "Checkpoint"
        - [ ] You noted three sizes and explained why B differs

### Étape 4 : Observer les disques dans l'invité / Step 4: Observe the disks in the guest

=== "Français"

    Démarrez la VM, connectez-vous et listez les disques :

=== "English"

    Start the VM, log in and list the disks:

```bash
lsblk -o NAME,SIZE,TYPE,TRAN,ROTA,MODEL
for d in /sys/block/sd* /sys/block/nvme*; do echo "$d rotational=$(cat $d/queue/rotational)"; done
```

=== "Français"

    Résultat attendu : le disque système et les disques SCSI apparaissent en `sd?`, le disque NVMe en `nvme0n1`, chacun de 2 Go. La colonne `TRAN` indique le transport, `ROTA` indique si le système croit à un disque rotatif. Notez les **noms des périphériques** (`<DISQUE_A>`, `<DISQUE_B>`, `<DISQUE_C>`) : ils servent au Lab 7.2. Pour les identifier sans ambiguïté, les trois disques de 2 Go peuvent être départagés par leur contrôleur (SCSI ou NVMe) et par l'ordre d'ajout.

    Prenez enfin un snapshot de la VM éteinte nommé **`etat-disques`** : il fournira un point de départ propre au Lab 7.2 (`sudo poweroff`, puis **VM**, **Snapshot**, **Take Snapshot**).

    !!! success "Point de contrôle"
        - [ ] Trois nouveaux disques de 2 Go visibles dans `lsblk`, dont un NVMe
        - [ ] Les noms de périphériques sont notés ; le snapshot `etat-disques` existe

=== "English"

    Expected result: the system disk and the SCSI disks appear as `sd?`, the NVMe disk as `nvme0n1`, each of 2 GB. The `TRAN` column shows the transport, `ROTA` shows whether the system believes it is a rotating disk. Note the **device names** (`<DISQUE_A>`, `<DISQUE_B>`, `<DISQUE_C>`): they are used in Lab 7.2. To identify them unambiguously, the three 2 GB disks can be told apart by their controller (SCSI or NVMe) and by their order of addition.

    Finally take a snapshot of the powered-off VM named **`etat-disques`**: it will provide a clean starting point for Lab 7.2 (`sudo poweroff`, then **VM**, **Snapshot**, **Take Snapshot**).

    !!! success "Checkpoint"
        - [ ] Three new 2 GB disks visible in `lsblk`, one of them NVMe
        - [ ] Device names are noted; the `etat-disques` snapshot exists

### Étape 5 : Provisionnement au niveau d'un fichier / Step 5: Provisioning at file level

=== "Français"

    Redémarrez la VM. Installez `qemu-utils` puis comparez un fichier **creux** (*sparse*, alloué à la demande) et un fichier **entièrement alloué**, sur le système de fichiers de la VM :

=== "English"

    Restart the VM. Install `qemu-utils` then compare a **sparse** file (allocated on demand) and a **fully allocated** file, on the VM's file system:

```bash
sudo apt update && sudo apt install -y qemu-utils
mkdir -p ~/images && cd ~/images
truncate -s 1G creux.img
fallocate -l 1G alloue.img
ls -ls creux.img alloue.img
du -h creux.img alloue.img
```

=== "Français"

    Résultat attendu : les deux fichiers ont une taille apparente de 1 Go, mais `du` montre que `creux.img` n'occupe presque rien sur le disque, alors que `alloue.img` occupe 1 Go. C'est exactement la différence entre provisionnement **dynamique** et **fixe** que vous venez de voir sur l'hôte.

    !!! success "Point de contrôle"
        - [ ] Vous avez observé deux tailles différentes pour deux fichiers de même taille apparente

=== "English"

    Expected result: both files have an apparent size of 1 GB, but `du` shows that `creux.img` uses almost nothing on disk, while `alloue.img` takes 1 GB. This is exactly the difference between **dynamic** and **fixed** provisioning you just saw on the host.

    !!! success "Checkpoint"
        - [ ] You observed two different sizes for two files of the same apparent size

### Étape 6 : Créer une image et la convertir / Step 6: Create an image and convert it

=== "Français"

    Créez une image **raw** de 1 Go, écrivez 200 Mo de données aléatoires au début, puis convertissez-la dans quatre autres formats :

=== "English"

    Create a 1 GB **raw** image, write 200 MB of random data at the beginning, then convert it into four other formats:

```bash
cd ~/images
qemu-img create -f raw disque.raw 1G
dd if=/dev/urandom of=disque.raw bs=1M count=200 conv=notrunc status=none
for f in qcow2 vmdk vdi vhdx; do qemu-img convert -f raw -O $f disque.raw disque.$f; done
ls -ls disque.*
qemu-img info disque.qcow2
```

=== "Français"

    Complétez la fiche avec, pour chaque format, la **taille virtuelle** (déclarée) et la **taille sur disque** (`ls -ls` ou `qemu-img info`). Les formats dynamiques ne conservent que les données écrites, avec un petit surcoût de métadonnées.

    **Vérifier l'intégrité.** Reconvertissez le VMDK en raw, puis comparez les empreintes :

=== "English"

    Complete the sheet with, for each format, the **virtual size** (declared) and the **size on disk** (`ls -ls` or `qemu-img info`). Dynamic formats only keep the written data, with a small metadata overhead.

    **Check integrity.** Convert the VMDK back to raw, then compare checksums:

```bash
qemu-img convert -f vmdk -O raw disque.vmdk retour.raw
sha256sum disque.raw retour.raw
qemu-img check disque.qcow2
```

=== "Français"

    Résultat attendu : les deux empreintes SHA-256 sont **identiques** (la conversion n'a rien altéré) ; `qemu-img check` ne signale pas d'erreur.

    !!! success "Point de contrôle"
        - [ ] Cinq formats comparés (taille virtuelle et taille sur disque)
        - [ ] Les empreintes de `disque.raw` et `retour.raw` sont identiques

=== "English"

    Expected result: both SHA-256 checksums are **identical** (the conversion altered nothing); `qemu-img check` reports no error.

    !!! success "Checkpoint"
        - [ ] Five formats compared (virtual size and size on disk)
        - [ ] The checksums of `disque.raw` and `retour.raw` are identical

### Étape 7 : Un disque chaîné (backing file) / Step 7: A chained disk (backing file)

=== "Français"

    Créez un disque qcow2 « enfant » qui s'appuie sur `disque.qcow2` comme disque parent, et observez la chaîne :

=== "English"

    Create a "child" qcow2 disk that relies on `disque.qcow2` as its parent disk, and observe the chain:

```bash
qemu-img create -f qcow2 -b disque.qcow2 -F qcow2 enfant.qcow2
qemu-img info --backing-chain enfant.qcow2
ls -ls enfant.qcow2 disque.qcow2
```

=== "Français"

    L'enfant est minuscule : il ne stocke que les **différences** par rapport à son parent, comme un disque différentiel Hyper-V (`.avhdx`, chapitre 4) ou un snapshot. Il **dépend** du parent : si celui-ci est modifié ou supprimé, l'enfant devient inutilisable.

    !!! success "Point de contrôle"
        - [ ] `qemu-img info --backing-chain` montre le parent et l'enfant
        - [ ] Vous savez expliquer pourquoi l'enfant ne peut pas vivre sans le parent

=== "English"

    The child is tiny: it only stores the **differences** from its parent, like a Hyper-V differencing disk (`.avhdx`, chapter 4) or a snapshot. It **depends** on the parent: if the parent is modified or deleted, the child becomes unusable.

    !!! success "Checkpoint"
        - [ ] `qemu-img info --backing-chain` shows the parent and the child
        - [ ] You can explain why the child cannot live without the parent

### Étape 8 : Fiche de synthèse / Step 8: Summary sheet

=== "Français"

    Remplissez la fiche ci-dessous : c'est le **livrable** du lab.

    | Rubrique | Valeur |
    |---|---|
    | Tailles sur l'hôte : disque A / B / C (Mo) | … / … / … |
    | Noms de périphériques dans l'invité : A / B / C | … / … / … |
    | Transport et `ROTA` observés pour chacun | … |
    | Fichier creux / alloué : taille apparente et taille occupée | … / … |
    | Taille virtuelle et taille sur disque : raw, qcow2, VMDK, VDI, VHDX | … |
    | Intégrité : empreintes identiques ? | oui / non |
    | Taille de l'enfant qcow2 et de son parent | … / … |

=== "English"

    Fill in the sheet below: this is the lab **deliverable**.

    | Item | Value |
    |---|---|
    | Sizes on the host: disk A / B / C (MB) | … / … / … |
    | Device names in the guest: A / B / C | … / … / … |
    | Transport and `ROTA` observed for each | … |
    | Sparse / allocated file: apparent size and occupied size | … / … |
    | Virtual size and size on disk: raw, qcow2, VMDK, VDI, VHDX | … |
    | Integrity: identical checksums? | yes / no |
    | Size of the qcow2 child and of its parent | … / … |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi le disque réservé occupe-t-il 2 Go dès sa création, contrairement aux disques dynamiques ?"
        Le provisionnement fixe réserve tout l'espace d'avance : l'hyperviseur écrit le fichier entier à la création. Un disque dynamique ne crée que ses métadonnées et grossit avec les données écrites. Contrepartie : le fixe est plus prévisible, le dynamique plus économique.

    ??? question "Pourquoi vérifie-t-on l'empreinte après une conversion de format ?"
        Pour s'assurer que le contenu n'a pas été altéré : un format qui ne se convertit pas fidèlement rendrait le disque inutilisable ou corromprait des données. Comparer les empreintes du disque d'origine et du disque reconverti en est la preuve.

    ??? question "Quels risques présente une chaîne de disques parent-enfant ?"
        L'enfant dépend du parent : supprimer ou modifier le parent détruit l'enfant. Une chaîne longue ralentit les accès (chaque niveau est consulté) et complique la sauvegarde et la restauration. C'est le même risque que celui des snapshots.

=== "English"

    ??? question "Why does the reserved disk take 2 GB from creation, unlike the dynamic disks?"
        Fixed provisioning reserves all the space in advance: the hypervisor writes the whole file at creation. A dynamic disk only creates its metadata and grows with the data written. Trade-off: fixed is more predictable, dynamic more economical.

    ??? question "Why verify the checksum after a format conversion?"
        To make sure the content has not been altered: a format that does not convert faithfully would make the disk unusable or corrupt data. Comparing the checksums of the original disk and of the reconverted disk is the proof.

    ??? question "What risks does a parent-child disk chain present?"
        The child depends on the parent: deleting or modifying the parent destroys the child. A long chain slows access (each level is consulted) and complicates backup and restore. It is the same risk as with snapshots.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | L'ajout du disque échoue : espace insuffisant | Libérer de l'espace sur le disque de l'hôte (le disque B réserve 2 Go) |
    | Le disque NVMe n'apparaît pas dans la VM | Vérifier le type de contrôleur choisi (NVMe) à l'ajout ; redémarrer la VM ; utiliser une version récente de VMware Workstation si l'option est absente |
    | Les noms `sdb`, `sdc` ne correspondent pas à l'ordre d'ajout | Normal : l'ordre dépend du contrôleur et de la détection ; se fier à `lsblk` (taille, transport) plutôt qu'à l'ordre |
    | `apt install qemu-utils` échoue | Accès Internet par la carte NAT ; `sudo apt update` |
    | `qemu-img: command not found` | Le paquet n'est pas installé : `sudo apt install -y qemu-utils` |
    | `qemu-img create -b` demande le format du parent | Ajouter `-F qcow2` (format du fichier parent) |
    | `dd` ou une conversion échoue pour manque de place | Espace libre insuffisant dans la VM (`df -h ~`) : supprimer des fichiers de `~/images` |

=== "English"

    | Symptom | Hints |
    |---|---|
    | Adding the disk fails: not enough space | Free up space on the host's disk (disk B reserves 2 GB) |
    | The NVMe disk does not appear in the VM | Check the controller type chosen (NVMe) when adding; restart the VM; use a recent VMware Workstation version if the option is missing |
    | The names `sdb`, `sdc` do not match the order of addition | Normal: order depends on the controller and detection; rely on `lsblk` (size, transport) rather than order |
    | `apt install qemu-utils` fails | Internet access through the NAT card; `sudo apt update` |
    | `qemu-img: command not found` | The package is not installed: `sudo apt install -y qemu-utils` |
    | `qemu-img create -b` asks for the parent's format | Add `-F qcow2` (format of the parent file) |
    | `dd` or a conversion fails for lack of space | Not enough free space in the VM (`df -h ~`): delete files from `~/images` |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    Supprimez les images de test dans la VM : `rm -rf ~/images`. **Conservez les trois disques ajoutés** (A, B, C) et le snapshot `etat-disques` : ils servent au [Lab 7.2](lab-7-2-mesurer-les-performances-avec-fio.md). Pour repartir d'un état propre sans les disques, restaurez le snapshot `etat-propre`, puis supprimez les fichiers `disque-*.vmdk` restants dans le dossier de la VM.

=== "English"

    Delete the test images in the VM: `rm -rf ~/images`. **Keep the three added disks** (A, B, C) and the `etat-disques` snapshot: they are used in [Lab 7.2](lab-7-2-mesurer-les-performances-avec-fio.md). To start again from a clean state without the disks, restore the `etat-propre` snapshot, then delete the remaining `disque-*.vmdk` files in the VM's folder.

[Retour au chapitre 7 / Back to chapter 7](cours.md)
