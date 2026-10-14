# Lab 4.2 : Checkpoints, disques et export / Checkpoints, Disks and Export

|                                              |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Durée / Duration**                         | 75 à 90 min / 75 to 90 min                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| **Niveau / Level**                           | Guidé / Guided                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| **Prérequis / Prerequisites**                | [Chapitre 4 : cours](cours.md) ; [Lab 4.1](lab-4-1-activer-hyper-v-et-creer-une-vm.md) réalisé (VM `srv1-hv` éteinte, accessible en SSH)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| **Ressources minimales / Minimum resources** | **VM** : `srv1-hv` (2 vCPU, 2 Go de RAM, disque de 25 Go dynamique), une seule VM allumée à la fois. **Hôte** : Windows Pro, Enterprise ou Éducation avec Hyper-V actif, au moins 4 Go de RAM libres, **40 Go** d'espace disque libre (checkpoints, disques de test, export et copie). L'hyperviseur Windows doit être **activé**. / **VM**: `srv1-hv` (2 vCPUs, 2 GB RAM, 25 GB dynamic disk), only one VM powered on at a time. **Host**: Windows Pro, Enterprise or Education with Hyper-V on, at least 4 GB of free RAM, **40 GB** of free disk space (checkpoints, test disks, export and copy). The Windows hypervisor must be **on**. |
| **Acquis visés / Learning outcomes**         | AA2                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |

=== "Français"

    !!! abstract "Objectifs"
        - Créer, restaurer et supprimer des checkpoints, et observer leur effet sur les fichiers de disque
        - Distinguer la restauration d'un checkpoint éteint (disque seul) et d'un checkpoint standard (avec mémoire)
        - Ajouter, agrandir, convertir un disque virtuel et créer un disque différentiel
        - Exporter la VM et la réimporter en copie avec un nouvel identifiant

=== "English"

    !!! abstract "Objectives"
        - Create, restore and delete checkpoints, and observe their effect on disk files
        - Tell apart restoring a powered-off checkpoint (disk only) and a standard checkpoint (with memory)
        - Add, extend and convert a virtual disk and create a differencing disk
        - Export the VM and re-import it as a copy with a new identifier

## Contexte et schéma / Context and diagram

=== "Français"

    Ce lab met en pratique les notions du cours : checkpoints, types de disques, export et import. Vous travaillez sur la VM `srv1-hv` du Lab 4.1. Toutes les commandes se lancent depuis **PowerShell en administrateur**, avec des chemins à adapter (`<DOSSIER_VM>` : dossier de la VM, `<DOSSIER_EXPORT>` : dossier d'export).

    ```mermaid
    flowchart LR
      V["VM srv1-hv"] --> C["Checkpoints"]
      V --> D["Disques<br/>(ajout, taille, type)"]
      V --> E["Export"]
      E --> I["Import en copie"]
    ```

    !!! warning "Une seule VM allumée à la fois"
        La copie importée à l'étape 9 est identique à l'original (nom d'hôte, adresse, clés SSH). Ne l'allumez pas en même temps que `srv1-hv`.

=== "English"

    This lab puts the lesson's notions into practice: checkpoints, disk types, export and import. You work on the `srv1-hv` VM from Lab 4.1. All commands are run from **PowerShell as administrator**, with paths to adapt (`<DOSSIER_VM>`: the VM's folder, `<DOSSIER_EXPORT>`: the export folder).

    ```mermaid
    flowchart LR
      V["VM srv1-hv"] --> C["Checkpoints"]
      V --> D["Disks<br/>(add, size, type)"]
      V --> E["Export"]
      E --> I["Import as copy"]
    ```

    !!! warning "Only one VM powered on at a time"
        The copy imported in step 9 is identical to the original (hostname, address, SSH keys). Do not power it on at the same time as `srv1-hv`.

## Étapes / Steps

### Étape 1 : État de départ / Step 1: Starting state

```powershell
Get-VM -Name "srv1-hv" | Select-Object Name, State, CheckpointType
Get-VMCheckpoint -VMName "srv1-hv"
```

=== "Français"

    Résultat attendu : la VM est à l'état `Off`, le type de checkpoint est `Production` (valeur par défaut) et aucun checkpoint n'existe encore.

    !!! success "Point de contrôle"
        - [ ] VM éteinte, aucun checkpoint listé

=== "English"

    Expected result: the VM is in the `Off` state, the checkpoint type is `Production` (default value) and no checkpoint exists yet.

    !!! success "Checkpoint"
        - [ ] VM powered off, no checkpoint listed

### Étape 2 : Checkpoint de référence, VM éteinte / Step 2: Reference checkpoint, VM powered off

```powershell
Checkpoint-VM -Name "srv1-hv" -SnapshotName "etat-propre"
Get-VMCheckpoint -VMName "srv1-hv"
(Get-VMHardDiskDrive -VMName "srv1-hv").Path
Get-VHD -Path (Get-VMHardDiskDrive -VMName "srv1-hv").Path | Select-Object VhdType, ParentPath, FileSize
```

=== "Français"

    Un checkpoint pris VM éteinte ne contient que l'état du disque. Résultat attendu : le chemin du disque de la VM se termine désormais par **`.avhdx`**, et `VhdType` vaut `Differencing` avec un `ParentPath` qui désigne le `.vhdx` d'origine : la VM écrit maintenant dans un disque différentiel, le disque d'origine étant figé.

    !!! success "Point de contrôle"
        - [ ] Le checkpoint `etat-propre` existe
        - [ ] Le disque actif de la VM est un `.avhdx` de type `Differencing`

=== "English"

    A checkpoint taken with the VM off contains only the disk state. Expected result: the VM's disk path now ends in **`.avhdx`**, and `VhdType` is `Differencing` with a `ParentPath` pointing to the original `.vhdx`: the VM now writes to a differencing disk, the original disk being frozen.

    !!! success "Checkpoint"
        - [ ] The `etat-propre` checkpoint exists
        - [ ] The VM's active disk is a `.avhdx` of type `Differencing`

### Étape 3 : Checkpoint standard, avec la mémoire / Step 3: Standard checkpoint, with memory

=== "Français"

    Passez au type **Standard**, démarrez la VM, puis créez un fichier de test et un processus qui tourne en arrière-plan (via SSH, ou la console de la VM) :

=== "English"

    Switch to the **Standard** type, start the VM, then create a test file and a background process (over SSH, or on the VM console):

```powershell
Set-VM -Name "srv1-hv" -CheckpointType Standard
Start-VM -Name "srv1-hv"
```

```bash
touch ~/avant-checkpoint
nohup sleep 7200 > /dev/null 2>&1 &
pgrep -a sleep
```

=== "Français"

    Prenez le checkpoint **pendant que la VM fonctionne** :

=== "English"

    Take the checkpoint **while the VM is running**:

```powershell
Checkpoint-VM -Name "srv1-hv" -SnapshotName "avec-memoire"
Get-VMCheckpoint -VMName "srv1-hv"
```

=== "Français"

    Retournez dans la VM : créez un second fichier et arrêtez le processus.

=== "English"

    Go back into the VM: create a second file and stop the process.

```bash
touch ~/apres-checkpoint
pkill sleep
ls ~
pgrep -a sleep
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Le checkpoint `avec-memoire` existe
        - [ ] Dans la VM : `avant-checkpoint` et `apres-checkpoint` existent, et le processus `sleep` est arrêté

=== "English"

    !!! success "Checkpoint"
        - [ ] The `avec-memoire` checkpoint exists
        - [ ] In the VM: `avant-checkpoint` and `apres-checkpoint` exist, and the `sleep` process is stopped

### Étape 4 : Restaurer le checkpoint avec mémoire / Step 4: Restore the checkpoint with memory

=== "Français"

    Coupez la VM (l'état courant va être abandonné de toute façon), restaurez, puis redémarrez :

=== "English"

    Turn the VM off (the current state is going to be discarded anyway), restore, then restart:

```powershell
Stop-VM -Name "srv1-hv" -TurnOff -Force
Restore-VMCheckpoint -VMName "srv1-hv" -Name "avec-memoire" -Confirm:$false
Get-VM -Name "srv1-hv" | Select-Object Name, State
Start-VM -Name "srv1-hv"
```

=== "Français"

    Après la restauration d'un checkpoint standard pris VM allumée, la VM peut apparaître à l'état `Saved` : la démarrer la **reprend** exactement là où elle en était. Reconnectez-vous en SSH (la session précédente est perdue) et vérifiez :

=== "English"

    After restoring a standard checkpoint taken while the VM was running, the VM may appear in the `Saved` state: starting it **resumes** it exactly where it was. Reconnect over SSH (the previous session is lost) and check:

```bash
ls ~
pgrep -a sleep
```

=== "Français"

    Résultat attendu : `avant-checkpoint` existe, `apres-checkpoint` n'existe plus, et le processus `sleep` **tourne encore** : la mémoire a été restaurée avec le disque.

    !!! success "Point de contrôle"
        - [ ] `apres-checkpoint` a disparu
        - [ ] Le processus `sleep` est de nouveau présent

=== "English"

    Expected result: `avant-checkpoint` exists, `apres-checkpoint` no longer exists, and the `sleep` process is **still running**: memory was restored along with the disk.

    !!! success "Checkpoint"
        - [ ] `apres-checkpoint` has disappeared
        - [ ] The `sleep` process is present again

### Étape 5 : Restaurer le checkpoint de référence (disque seul) / Step 5: Restore the reference checkpoint (disk only)

```powershell
Stop-VM -Name "srv1-hv" -TurnOff -Force
Restore-VMCheckpoint -VMName "srv1-hv" -Name "etat-propre" -Confirm:$false
Start-VM -Name "srv1-hv"
```

=== "Français"

    Reconnectez-vous et vérifiez : aucun des deux fichiers de test n'existe, et le processus `sleep` a disparu ; la VM a **démarré comme à froid**.

=== "English"

    Reconnect and check: neither test file exists, and the `sleep` process is gone; the VM **booted from scratch**.

```bash
ls ~
pgrep -a sleep
uptime
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Les deux fichiers de test et le processus ont disparu
        - [ ] `uptime` indique une VM qui vient de démarrer
        - [ ] Vous savez expliquer la différence avec l'étape 4

=== "English"

    !!! success "Checkpoint"
        - [ ] Both test files and the process have disappeared
        - [ ] `uptime` shows a VM that has just booted
        - [ ] You can explain the difference with step 4

### Étape 6 : Supprimer les checkpoints / Step 6: Delete the checkpoints

=== "Français"

    Éteignez proprement la VM, supprimez les checkpoints et observez la fusion des disques différentiels :

=== "English"

    Shut the VM down cleanly, delete the checkpoints and observe the merge of the differencing disks:

```powershell
Stop-VM -Name "srv1-hv"
Get-VMCheckpoint -VMName "srv1-hv" | Remove-VMCheckpoint
Start-Sleep -Seconds 30
(Get-VMHardDiskDrive -VMName "srv1-hv").Path
Get-ChildItem -Recurse "<DOSSIER_VM>\srv1-hv" -Include *.vhdx,*.avhdx | Select-Object Name, @{n='Go';e={[math]::Round($_.Length/1GB,2)}}
```

=== "Français"

    La fusion peut prendre un moment. Résultat attendu : plus aucun `.avhdx` ; le disque actif de la VM redevient le `.vhdx` d'origine, qui contient désormais l'état courant.

    !!! success "Point de contrôle"
        - [ ] `Get-VMCheckpoint` ne renvoie plus rien
        - [ ] Le disque actif de la VM est de nouveau un `.vhdx`

=== "English"

    The merge may take a moment. Expected result: no `.avhdx` anymore; the VM's active disk is again the original `.vhdx`, which now holds the current state.

    !!! success "Checkpoint"
        - [ ] `Get-VMCheckpoint` returns nothing anymore
        - [ ] The VM's active disk is a `.vhdx` again

### Étape 7 : Ajouter et agrandir un disque de données / Step 7: Add and extend a data disk

=== "Français"

    Créez un disque dynamique de 5 Go, attachez-le à la VM, démarrez-la :

=== "English"

    Create a 5 GB dynamic disk, attach it to the VM, start it:

```powershell
New-VHD -Path "<DOSSIER_VM>\srv1-hv\data.vhdx" -SizeBytes 5GB -Dynamic
Add-VMHardDiskDrive -VMName "srv1-hv" -Path "<DOSSIER_VM>\srv1-hv\data.vhdx"
Start-VM -Name "srv1-hv"
```

```bash
lsblk
```

=== "Français"

    Résultat attendu : un nouveau disque d'environ 5 Go apparaît dans la VM (sans partition ni système de fichiers : leur création est étudiée au chapitre 9). Éteignez la VM, agrandissez le disque à 8 Go, redémarrez :

=== "English"

    Expected result: a new disk of about 5 GB appears in the VM (without partition or file system: creating them is studied in chapter 9). Shut the VM down, extend the disk to 8 GB, restart:

```powershell
Stop-VM -Name "srv1-hv"
Resize-VHD -Path "<DOSSIER_VM>\srv1-hv\data.vhdx" -SizeBytes 8GB
Start-VM -Name "srv1-hv"
```

```bash
lsblk
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Le disque apparaît à environ 5 Go, puis à environ 8 Go après l'agrandissement

=== "English"

    !!! success "Checkpoint"
        - [ ] The disk appears at about 5 GB, then at about 8 GB after the extension

### Étape 8 : Types de disques : fixe et différentiel / Step 8: Disk types: fixed and differencing

=== "Français"

    VM éteinte, convertissez le disque de données en disque de **taille fixe**, créez un disque **différentiel** dont il est le parent, puis comparez :

=== "English"

    With the VM powered off, convert the data disk to a **fixed-size** disk, create a **differencing** disk with it as parent, then compare:

```powershell
Stop-VM -Name "srv1-hv"
Convert-VHD -Path "<DOSSIER_VM>\srv1-hv\data.vhdx" -DestinationPath "<DOSSIER_VM>\srv1-hv\data-fixe.vhdx" -VHDType Fixed
New-VHD -Path "<DOSSIER_VM>\srv1-hv\data-diff.vhdx" -ParentPath "<DOSSIER_VM>\srv1-hv\data.vhdx" -Differencing
Get-VHD -Path "<DOSSIER_VM>\srv1-hv\data.vhdx","<DOSSIER_VM>\srv1-hv\data-fixe.vhdx","<DOSSIER_VM>\srv1-hv\data-diff.vhdx" | Select-Object Path, VhdType, Size, FileSize, ParentPath
```

=== "Français"

    Résultat attendu : le disque dynamique est très petit, le disque fixe occupe environ **8 Go** (tout l'espace réservé), et le disque différentiel est minuscule et référence le disque d'origine par `ParentPath`.

    Nettoyez ensuite : détachez le disque de données de la VM et supprimez les trois fichiers de test :

=== "English"

    Expected result: the dynamic disk is very small, the fixed disk takes about **8 GB** (all the space reserved), and the differencing disk is tiny and references the original disk through `ParentPath`.

    Then clean up: detach the data disk from the VM and delete the three test files:

```powershell
Get-VMHardDiskDrive -VMName "srv1-hv" | Where-Object Path -like "*data.vhdx" | Remove-VMHardDiskDrive
Remove-Item "<DOSSIER_VM>\srv1-hv\data-diff.vhdx","<DOSSIER_VM>\srv1-hv\data-fixe.vhdx","<DOSSIER_VM>\srv1-hv\data.vhdx"
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Vous avez relevé trois tailles très différentes (dynamique, fixe, différentiel)
        - [ ] Le disque de données n'est plus attaché à la VM

=== "English"

    !!! success "Checkpoint"
        - [ ] You noted three very different sizes (dynamic, fixed, differencing)
        - [ ] The data disk is no longer attached to the VM

### Étape 9 : Exporter puis réimporter en copie / Step 9: Export then re-import as a copy

=== "Français"

    VM éteinte, exportez-la, puis notez l'identifiant de la VM d'origine :

=== "English"

    With the VM powered off, export it, then note the original VM's identifier:

```powershell
Export-VM -Name "srv1-hv" -Path "<DOSSIER_EXPORT>"
$orig = (Get-VM -Name "srv1-hv").Id
Get-ChildItem "<DOSSIER_EXPORT>\srv1-hv" -Recurse | Select-Object FullName
```

=== "Français"

    Repérez le fichier de configuration `.vmcx` dans le sous-dossier `Virtual Machines`. Importez la VM **en copie**, avec un nouvel identifiant (remplacez `<NOM>.vmcx` par le nom du fichier trouvé et le dossier de la copie) :

=== "English"

    Find the `.vmcx` configuration file in the `Virtual Machines` subfolder. Import the VM **as a copy**, with a new identifier (replace `<NOM>.vmcx` with the name of the file found and the copy folder):

```powershell
Import-VM -Path "<DOSSIER_EXPORT>\srv1-hv\Virtual Machines\<NOM>.vmcx" -Copy -GenerateNewId -VirtualMachinePath "<DOSSIER_VM>\copie" -VhdDestinationPath "<DOSSIER_VM>\copie\disques"
Get-VM | Select-Object Name, Id, State
Get-VM -Name "srv1-hv" | Where-Object Id -ne $orig | Rename-VM -NewName "srv1-hv-copie"
Get-VM | Select-Object Name, Id, State
```

=== "Français"

    Résultat attendu : deux VM portent d'abord le même nom avec des **identifiants différents** ; après le renommage, `srv1-hv` et `srv1-hv-copie` coexistent. La copie a ses propres fichiers dans `<DOSSIER_VM>\copie`.

    Pour utiliser la copie, démarrez-la **seule** (l'original reste éteint) et changez son nom d'hôte, son adresse et ses identifiants comme à l'étape 9 du [Lab 3.2](../ch03-virtualbox-et-vmware-workstation/lab-3-2-vmware-workstation-meme-vm-import-et-export.md).

    !!! success "Point de contrôle"
        - [ ] L'export existe dans `<DOSSIER_EXPORT>`
        - [ ] Deux VM aux identifiants différents coexistent, nommées `srv1-hv` et `srv1-hv-copie`

=== "English"

    Expected result: two VMs first share the same name with **different identifiers**; after renaming, `srv1-hv` and `srv1-hv-copie` coexist. The copy has its own files in `<DOSSIER_VM>\copie`.

    To use the copy, start it **alone** (the original stays off) and change its hostname, address and identifiers as in step 9 of [Lab 3.2](../ch03-virtualbox-et-vmware-workstation/lab-3-2-vmware-workstation-meme-vm-import-et-export.md).

    !!! success "Checkpoint"
        - [ ] The export exists in `<DOSSIER_EXPORT>`
        - [ ] Two VMs with different identifiers coexist, named `srv1-hv` and `srv1-hv-copie`

### Étape 10 : Fiche de synthèse / Step 10: Summary sheet

=== "Français"

    Remplissez la fiche ci-dessous : c'est le **livrable** du lab.

    | Rubrique | Valeur |
    |---|---|
    | Type de checkpoint par défaut de la VM | … |
    | Extension du disque actif après un checkpoint, et `VhdType` | … |
    | Après restauration du checkpoint avec mémoire : fichier de test, processus `sleep` | … |
    | Après restauration du checkpoint éteint : fichier de test, processus `sleep` | … |
    | Taille du disque de données : dynamique / fixe / différentiel (Go) | … / … / … |
    | Taille de l'export (Go) | … |
    | Identifiants de la VM d'origine et de la copie | … / … |

=== "English"

    Fill in the sheet below: this is the lab **deliverable**.

    | Item | Value |
    |---|---|
    | VM's default checkpoint type | … |
    | Active disk extension after a checkpoint, and `VhdType` | … |
    | After restoring the checkpoint with memory: test file, `sleep` process | … |
    | After restoring the powered-off checkpoint: test file, `sleep` process | … |
    | Data disk size: dynamic / fixed / differencing (GB) | … / … / … |
    | Export size (GB) | … |
    | Identifiers of the original VM and of the copy | … / … |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi le disque actif de la VM devient-il un `.avhdx` après un checkpoint ?"
        Le disque d'origine est figé pour conserver l'état du checkpoint ; les écritures suivantes vont dans un disque différentiel (`.avhdx`) qui référence le disque d'origine comme parent. Restaurer le checkpoint revient à abandonner ce disque différentiel.

    ??? question "Pourquoi une longue chaîne de checkpoints est-elle déconseillée ?"
        Chaque niveau ajoute un disque différentiel : la VM doit parcourir la chaîne pour lire ses données, ce qui la ralentit, et chaque disque consomme de l'espace. Supprimer un checkpoint déclenche en outre une fusion de disques qui peut être longue.

    ??? question "Pourquoi l'import d'une VM exportée se fait-il en mode « copier » avec un nouvel identifiant lorsque l'original existe encore ?"
        Chaque VM est identifiée de façon unique par Hyper-V : deux VM de même identifiant ne peuvent pas coexister sur un hôte. Le mode copie duplique les fichiers et génère un nouvel identifiant.

=== "English"

    ??? question "Why does the VM's active disk become an `.avhdx` after a checkpoint?"
        The original disk is frozen to preserve the checkpoint's state; subsequent writes go to a differencing disk (`.avhdx`) that references the original disk as its parent. Restoring the checkpoint amounts to discarding this differencing disk.

    ??? question "Why is a long checkpoint chain discouraged?"
        Each level adds a differencing disk: the VM has to walk the chain to read its data, which slows it down, and each disk uses space. Deleting a checkpoint also triggers a disk merge that can be long.

    ??? question "Why is a VM exported then imported in “copy” mode with a new identifier when the original still exists?"
        Each VM is uniquely identified by Hyper-V: two VMs with the same identifier cannot coexist on a host. Copy mode duplicates the files and generates a new identifier.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | `Checkpoint-VM` échoue sur une VM allumée en type `Production` | Les services d'intégration de sauvegarde ne sont pas actifs dans l'invité Linux : prendre le checkpoint VM éteinte (étape 2) ou utiliser le type `Standard` (étape 3) |
    | La VM est à l'état `Saved` après restauration | Normal pour un checkpoint avec mémoire : `Start-VM` la reprend |
    | `Remove-VMCheckpoint` semble ne rien faire | La fusion des disques s'effectue en arrière-plan, parfois VM éteinte : attendre et relancer `Get-ChildItem` |
    | `Resize-VHD` ou `Convert-VHD` signale que le fichier est utilisé | Éteindre la VM ; le disque ne doit pas être utilisé par une VM en fonctionnement |
    | Espace disque insuffisant pendant la conversion ou l'export | Libérer de l'espace ; le disque fixe et l'export demandent de la place (voir les ressources minimales) |
    | `Import-VM` signale que la VM existe déjà | Utiliser `-Copy -GenerateNewId` pour créer une copie avec un nouvel identifiant |
    | Fichier `.vmcx` introuvable | Lister l'export avec `Get-ChildItem -Recurse` : le fichier est dans `Virtual Machines` |
    | `Get-VM -Name srv1-hv` renvoie deux VM | C'est attendu après l'import ; différencier par l'identifiant (`Id`) comme à l'étape 9 |

=== "English"

    | Symptom | Hints |
    |---|---|
    | `Checkpoint-VM` fails on a running VM with `Production` type | The backup integration services are not active in the Linux guest: take the checkpoint with the VM off (step 2) or use the `Standard` type (step 3) |
    | The VM is in the `Saved` state after restoring | Normal for a checkpoint with memory: `Start-VM` resumes it |
    | `Remove-VMCheckpoint` seems to do nothing | The disk merge runs in the background, sometimes with the VM off: wait and rerun `Get-ChildItem` |
    | `Resize-VHD` or `Convert-VHD` reports that the file is in use | Shut the VM down; the disk must not be used by a running VM |
    | Not enough disk space during conversion or export | Free up space; the fixed disk and the export need room (see minimum resources) |
    | `Import-VM` reports that the VM already exists | Use `-Copy -GenerateNewId` to create a copy with a new identifier |
    | `.vmcx` file not found | List the export with `Get-ChildItem -Recurse`: the file is in `Virtual Machines` |
    | `Get-VM -Name srv1-hv` returns two VMs | Expected after the import; tell them apart by identifier (`Id`) as in step 9 |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    Supprimez la copie et l'export si vous n'en avez plus besoin : `Remove-VM -Name "srv1-hv-copie" -Force`, puis supprimez les dossiers `<DOSSIER_VM>\copie` et `<DOSSIER_EXPORT>`. Conservez `srv1-hv`, éteinte, sans checkpoint. Pour repartir d'une VM propre à tout moment, créez-lui un checkpoint de référence VM éteinte (étape 2). À la fin du chapitre, si vous devez revenir aux labs de VirtualBox ou de VMware Workstation, désactivez l'hyperviseur Windows (Lab 2.1).

=== "English"

    Delete the copy and the export if you no longer need them: `Remove-VM -Name "srv1-hv-copie" -Force`, then delete the `<DOSSIER_VM>\copie` and `<DOSSIER_EXPORT>` folders. Keep `srv1-hv`, powered off, with no checkpoint. To start again from a clean VM at any time, create a reference checkpoint with the VM off (step 2). At the end of the chapter, if you have to return to the VirtualBox or VMware Workstation labs, turn off the Windows hypervisor (Lab 2.1).

[Retour au chapitre 4 / Back to chapter 4](cours.md)
