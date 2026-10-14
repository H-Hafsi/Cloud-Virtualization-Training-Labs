# Lab 3.2 : VMware Workstation, même VM, import et export / VMware Workstation: Same VM, Import and Export

| | |
|---|---|
| **Durée / Duration** | 60 à 75 min / 60 to 75 min |
| **Niveau / Level** | Guidé / Guided |
| **Prérequis / Prerequisites** | [Chapitre 3 : cours](cours.md) ; [Lab 3.1](lab-3-1-virtualbox-une-vm-ubuntu-server.md) réalisé (VM `srv1` et snapshot `etat-propre`) |
| **Ressources minimales / Minimum resources** | **VM** : 2 vCPU, 2 Go de RAM, disque de 25 Go (dynamique), une seule VM allumée à la fois. **Hôte** : au moins 4 Go de RAM libres et 40 Go d'espace disque libre (VM source, archive OVA, VM importée et clone). / **VM**: 2 vCPUs, 2 GB RAM, 25 GB disk (dynamic), only one VM powered on at a time. **Host**: at least 4 GB of free RAM and 40 GB of free disk space (source VM, OVA archive, imported VM and clone). |
| **Acquis visés / Learning outcomes** | AA2 |

=== "Français"

    !!! abstract "Objectifs"
        - Exporter une VM de VirtualBox au format OVA et l'importer dans VMware Workstation
        - Corriger la configuration réseau d'une VM déplacée d'un hyperviseur à l'autre
        - Configurer les réseaux virtuels de VMware Workstation (NAT et hôte seul)
        - Prendre un snapshot, cloner une VM et préparer le clone pour éviter les doublons
        - Activer la virtualisation imbriquée et la tester
        - Comparer les fichiers et les formats des deux hyperviseurs

=== "English"

    !!! abstract "Objectives"
        - Export a VM from VirtualBox as an OVA and import it into VMware Workstation
        - Fix the network configuration of a VM moved from one hypervisor to the other
        - Configure VMware Workstation virtual networks (NAT and host-only)
        - Take a snapshot, clone a VM and prepare the clone to avoid duplicates
        - Enable nested virtualization and test it
        - Compare the files and formats of the two hypervisors

## Contexte et schéma / Context and diagram

=== "Français"

    Vous allez faire voyager la VM de référence `srv1` de VirtualBox vers VMware Workstation, l'hyperviseur principal du cours pour les labs imbriqués. Ce trajet illustre la **portabilité** d'une VM (chapitre 2) et ses limites : le matériel virtuel change, donc les noms d'interfaces réseau aussi.

    ```mermaid
    flowchart LR
      VB["VirtualBox<br/>VM srv1"] -- "Export OVA" --> F["Fichier .ova"]
      F -- "Import" --> VW["VMware Workstation<br/>VM importée"]
      VW -- "Clone complet" --> C["VM clonée"]
      VW -- "Export OVF" --> F2["Archive pour<br/>un autre poste"]
    ```

    !!! warning "Une seule VM allumée à la fois"
        Les VM `srv1` de VirtualBox, la VM importée et le clone portent la **même configuration** au départ. Les allumer en même temps provoque des conflits d'adresse. Éteignez chaque VM avant de passer à l'étape suivante. Les adresses IP du réseau hôte seul suivent le plan de l'annexe [Environnement](../../annexes/environnement.md) (notées `<IP_VM>` et `<IP_CLONE>`).

=== "English"

    You will move the `srv1` reference VM from VirtualBox to VMware Workstation, the course's main hypervisor for nested labs. This trip illustrates the **portability** of a VM (chapter 2) and its limits: the virtual hardware changes, so network interface names do too.

    ```mermaid
    flowchart LR
      VB["VirtualBox<br/>VM srv1"] -- "OVA export" --> F[".ova file"]
      F -- "Import" --> VW["VMware Workstation<br/>imported VM"]
      VW -- "Full clone" --> C["Cloned VM"]
      VW -- "OVF export" --> F2["Archive for<br/>another computer"]
    ```

    !!! warning "Only one VM powered on at a time"
        The VirtualBox `srv1` VM, the imported VM and the clone have the **same configuration** at the start. Powering them on at the same time causes address conflicts. Shut each VM down before moving to the next step. Host-only network IP addresses follow the plan of the [Environment](../../annexes/environnement.md) annex (written `<IP_VM>` and `<IP_CLONE>`).

## Étapes / Steps

### Étape 1 : Installer VMware Workstation / Step 1: Install VMware Workstation

=== "Français"

    1. Téléchargez VMware Workstation depuis le **site officiel** de l'éditeur, selon les conditions de licence en vigueur à la date de la séance (voir l'annexe Environnement), et installez-le avec les options par défaut.
    2. Vérifiez l'état de l'hyperviseur Windows comme au [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) : pour ce lab, **désactivé** est recommandé.
    3. Lancez VMware Workstation et notez sa version (menu **Help**, **About**).

    !!! success "Point de contrôle"
        - [ ] VMware Workstation démarre
        - [ ] Vous avez noté sa version

=== "English"

    1. Download VMware Workstation from the vendor's **official website**, according to the license terms in force on the session date (see the Environment annex), and install it with default options.
    2. Check the Windows hypervisor state as in [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md): for this lab, **off** is recommended.
    3. Start VMware Workstation and note its version (menu **Help**, **About**).

    !!! success "Checkpoint"
        - [ ] VMware Workstation starts
        - [ ] You noted its version

### Étape 2 : Exporter la VM depuis VirtualBox / Step 2: Export the VM from VirtualBox

=== "Français"

    1. Dans la VM `srv1`, éteignez proprement : `sudo poweroff`.
    2. Dans VirtualBox : **File**, **Export Appliance**, sélectionnez `srv1`, choisissez un dossier de destination et le nom `srv1.ova`, puis validez.

    Équivalent en ligne de commande (PowerShell, remplacez le chemin de destination) :

=== "English"

    1. In the `srv1` VM, shut down cleanly: `sudo poweroff`.
    2. In VirtualBox: **File**, **Export Appliance**, select `srv1`, choose a destination folder and the name `srv1.ova`, then confirm.

    Command-line equivalent (PowerShell, replace the destination path):

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" export "srv1" -o "C:\chemin\vers\srv1.ova"
Get-Item "C:\chemin\vers\srv1.ova" | Select-Object Name, @{n='Go';e={[math]::Round($_.Length/1GB,2)}}
```

=== "Français"

    Résultat attendu : un fichier `srv1.ova` dont la taille correspond aux données réellement écrites sur le disque, et non aux 25 Go déclarés.

    !!! success "Point de contrôle"
        - [ ] Le fichier `srv1.ova` existe
        - [ ] Vous avez noté sa taille

=== "English"

    Expected result: an `srv1.ova` file whose size matches the data actually written on the disk, not the declared 25 GB.

    !!! success "Checkpoint"
        - [ ] The `srv1.ova` file exists
        - [ ] You noted its size

### Étape 3 : Préparer les réseaux de VMware Workstation / Step 3: Prepare VMware Workstation networks

=== "Français"

    1. Dans VMware Workstation, ouvrez **Edit**, **Virtual Network Editor**, puis **Change Settings** (droits administrateur).
    2. Vérifiez les réseaux par défaut : **VMnet8** (type NAT) et **VMnet1** (type *Host-only*). Créez-les s'ils sont absents.
    3. Sélectionnez **VMnet1** et réglez son adresse de sous-réseau selon l'annexe Environnement (réseau de gestion). Validez.
    4. Dans PowerShell, relevez l'adresse de votre poste sur ce réseau (`<IP_HOTE>`) :

=== "English"

    1. In VMware Workstation, open **Edit**, **Virtual Network Editor**, then **Change Settings** (administrator rights).
    2. Check the default networks: **VMnet8** (NAT type) and **VMnet1** (*Host-only* type). Create them if missing.
    3. Select **VMnet1** and set its subnet address according to the Environment annex (management network). Confirm.
    4. In PowerShell, note your computer's address on this network (`<IP_HOTE>`):

```powershell
ipconfig
```

=== "Français"

    Cherchez la carte nommée **VMware Network Adapter VMnet1** dans le résultat. Les adresses statiques que vous attribuerez aux VM doivent appartenir à ce réseau et ne pas entrer dans la plage du serveur DHCP de VMware (désactivez le DHCP de VMnet1 si besoin).

    !!! success "Point de contrôle"
        - [ ] VMnet8 en NAT et VMnet1 en hôte seul sont présents
        - [ ] Vous avez noté l'adresse de l'hôte sur VMnet1

=== "English"

    Look for the adapter named **VMware Network Adapter VMnet1** in the output. The static addresses you assign to VMs must belong to this network and stay out of VMware's DHCP server range (turn off VMnet1's DHCP if needed).

    !!! success "Checkpoint"
        - [ ] VMnet8 in NAT and VMnet1 in host-only are present
        - [ ] You noted the host's address on VMnet1

### Étape 4 : Importer l'OVA dans VMware Workstation / Step 4: Import the OVA into VMware Workstation

=== "Français"

    1. Dans VMware Workstation : **File**, **Open**, sélectionnez `srv1.ova`.
    2. Donnez un nom à la VM (par exemple `srv1-vmware`) et choisissez son dossier de stockage (le dossier des VM du Lab 2.1). Cliquez sur **Import**.
    3. Si l'import échoue avec un message de non-conformité, cliquez sur **Retry** : VMware relâche alors certaines vérifications.
    4. N'allumez pas encore la VM.

    !!! success "Point de contrôle"
        - [ ] La VM `srv1-vmware` apparaît dans la bibliothèque, éteinte

=== "English"

    1. In VMware Workstation: **File**, **Open**, select `srv1.ova`.
    2. Give the VM a name (for example `srv1-vmware`) and choose its storage folder (the VM folder from Lab 2.1). Click **Import**.
    3. If the import fails with a compliance message, click **Retry**: VMware then relaxes some checks.
    4. Do not power the VM on yet.

    !!! success "Checkpoint"
        - [ ] The `srv1-vmware` VM appears in the library, powered off

### Étape 5 : Ajuster le matériel virtuel de la VM importée / Step 5: Adjust the imported VM's virtual hardware

=== "Français"

    Sélectionnez la VM, ouvrez **Edit virtual machine settings** et vérifiez :

    | Élément | Valeur |
    |---|---|
    | Memory | 2048 MB |
    | Processors | 2 |
    | Hard Disk | Disque `.vmdk` issu de l'import |
    | Network Adapter | **NAT** (VMnet8) |
    | Network Adapter 2 | **Host-only** (VMnet1) ; ajoutez-le (**Add**, **Network Adapter**) s'il manque |

    !!! success "Point de contrôle"
        - [ ] Deux cartes réseau : NAT et hôte seul
        - [ ] Mémoire et processeurs conformes

=== "English"

    Select the VM, open **Edit virtual machine settings** and check:

    | Element | Value |
    |---|---|
    | Memory | 2048 MB |
    | Processors | 2 |
    | Hard Disk | `.vmdk` disk coming from the import |
    | Network Adapter | **NAT** (VMnet8) |
    | Network Adapter 2 | **Host-only** (VMnet1); add it (**Add**, **Network Adapter**) if missing |

    !!! success "Checkpoint"
        - [ ] Two network cards: NAT and host-only
        - [ ] Memory and processors as expected

### Étape 6 : Démarrer et réparer le réseau / Step 6: Start and repair the network

=== "Français"

    Démarrez la VM et connectez-vous sur sa console. Si VMware demande si la VM a été **déplacée ou copiée**, répondez **I Copied It**. Vérifiez le réseau :

=== "English"

    Start the VM and log in on its console. If VMware asks whether the VM was **moved or copied**, answer **I Copied It**. Check the network:

```bash
ip -br link
ip -br addr
ls /etc/netplan
sudo cat /etc/netplan/*.yaml
```

=== "Français"

    Constatez que les **noms des interfaces ont changé** (par exemple `ens33` au lieu de `enp0s3`) : le matériel virtuel n'est plus le même, alors que les fichiers Netplan font encore référence aux anciens noms. Modifiez chaque fichier concerné pour remplacer les anciens noms par les nouveaux (`sudo nano` sur le fichier), en veillant à garder l'interface NAT en DHCP et à reporter dans `99-gestion.yaml` le nouveau nom de l'interface hôte seul et l'adresse `<IP_VM>`. Appliquez :

=== "English"

    Note that the **interface names have changed** (for example `ens33` instead of `enp0s3`): the virtual hardware is no longer the same, while the Netplan files still refer to the old names. Edit each affected file to replace the old names with the new ones (`sudo nano` on the file), keeping the NAT interface on DHCP and updating `99-gestion.yaml` with the new host-only interface name and the `<IP_VM>` address. Apply:

```bash
sudo netplan apply
ip -br addr
ping -c 2 archive.ubuntu.com
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] L'interface NAT obtient une adresse et le `ping` vers Internet aboutit
        - [ ] L'interface hôte seul porte l'adresse `<IP_VM>`
        - [ ] Depuis PowerShell : `ssh utilisateur@<IP_VM>` fonctionne

=== "English"

    !!! success "Checkpoint"
        - [ ] The NAT interface gets an address and the `ping` to the Internet succeeds
        - [ ] The host-only interface carries the `<IP_VM>` address
        - [ ] From PowerShell: `ssh user@<IP_VM>` works

### Étape 7 : Outils invité de VMware / Step 7: VMware guest tools

=== "Français"

    Vérifiez la présence des outils invité de VMware (sous Linux, le paquet `open-vm-tools`) et installez-les si besoin :

=== "English"

    Check that VMware guest tools are present (on Linux, the `open-vm-tools` package) and install them if needed:

```bash
systemctl status open-vm-tools --no-pager
sudo apt install -y open-vm-tools
```

### Étape 8 : Snapshot de référence dans VMware / Step 8: Reference snapshot in VMware

=== "Français"

    1. Éteignez proprement la VM : `sudo poweroff`.
    2. Dans VMware Workstation : **VM**, **Snapshot**, **Take Snapshot**. Nommez-le **`etat-propre`**.

    !!! success "Point de contrôle"
        - [ ] Le snapshot `etat-propre` apparaît dans le **Snapshot Manager**

=== "English"

    1. Shut the VM down cleanly: `sudo poweroff`.
    2. In VMware Workstation: **VM**, **Snapshot**, **Take Snapshot**. Name it **`etat-propre`**.

    !!! success "Checkpoint"
        - [ ] The `etat-propre` snapshot appears in the **Snapshot Manager**

### Étape 9 : Cloner la VM et préparer le clone / Step 9: Clone the VM and prepare the clone

=== "Français"

    1. VM éteinte : **VM**, **Manage**, **Clone**. Choisissez l'état actuel, puis **Create a full clone**. Nommez-le `srv2`.
    2. Démarrez **uniquement le clone** (la VM d'origine reste éteinte). Un clone est une copie parfaite : il possède le **même nom d'hôte, les mêmes clés SSH, le même identifiant de machine et la même adresse** que l'original. Préparez-le :

=== "English"

    1. With the VM powered off: **VM**, **Manage**, **Clone**. Choose the current state, then **Create a full clone**. Name it `srv2`.
    2. Start **only the clone** (the original VM stays off). A clone is a perfect copy: it has the **same hostname, SSH keys, machine identifier and address** as the original. Prepare it:

```bash
sudo hostnamectl set-hostname srv2
sudo truncate -s 0 /etc/machine-id
sudo rm -f /var/lib/dbus/machine-id
sudo ln -s /etc/machine-id /var/lib/dbus/machine-id
sudo rm -f /etc/ssh/ssh_host_*
sudo dpkg-reconfigure openssh-server
sudo nano /etc/netplan/99-gestion.yaml    # remplacer <IP_VM> par <IP_CLONE>
sudo netplan apply
sudo reboot
```

=== "Français"

    Après le redémarrage, vérifiez :

=== "English"

    After the restart, check:

```bash
hostnamectl
cat /etc/machine-id
ip -br addr
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Le nom d'hôte est `srv2` et l'adresse hôte seul est `<IP_CLONE>`
        - [ ] L'identifiant de machine (`machine-id`) est différent de celui de `srv1-vmware` (comparez après avoir allumé l'original plus tard, ou notez-le)
        - [ ] `ssh utilisateur@<IP_CLONE>` fonctionne (acceptez la nouvelle empreinte SSH)

=== "English"

    !!! success "Checkpoint"
        - [ ] The hostname is `srv2` and the host-only address is `<IP_CLONE>`
        - [ ] The machine identifier (`machine-id`) differs from that of `srv1-vmware` (compare after powering the original on later, or note it)
        - [ ] `ssh user@<IP_CLONE>` works (accept the new SSH fingerprint)

### Étape 10 : Virtualisation imbriquée / Step 10: Nested virtualization

=== "Français"

    1. Éteignez la VM `srv1-vmware`.
    2. **Edit virtual machine settings**, **Processors**, cochez **Virtualize Intel VT-x/EPT or AMD-V/RVI**.
    3. Démarrez la VM et exécutez :

=== "English"

    1. Shut down the `srv1-vmware` VM.
    2. **Edit virtual machine settings**, **Processors**, check **Virtualize Intel VT-x/EPT or AMD-V/RVI**.
    3. Start the VM and run:

```bash
grep -Ec '(vmx|svm)' /proc/cpuinfo
```

=== "Français"

    Un résultat **supérieur à 0** signifie que la VM voit les extensions de virtualisation : elle pourra exécuter un hyperviseur (chapitres 12 à 16 et 18). Un résultat de 0 indique que la virtualisation imbriquée n'est pas disponible (voir le dépannage).

    !!! success "Point de contrôle"
        - [ ] Le compteur est supérieur à 0

=== "English"

    A result **greater than 0** means the VM sees the virtualization extensions: it will be able to run a hypervisor (chapters 12 to 16 and 18). A result of 0 means nested virtualization is not available (see troubleshooting).

    !!! success "Checkpoint"
        - [ ] The counter is greater than 0

### Étape 11 : Comparer les fichiers des deux hyperviseurs / Step 11: Compare the files of both hypervisors

=== "Français"

    Éteignez les VM, puis listez le dossier de la VM VMware, et comparez avec celui de la VM VirtualBox (Lab 3.1). Remplacez le chemin par le vôtre :

=== "English"

    Shut the VMs down, then list the VMware VM's folder, and compare it with the VirtualBox VM's (Lab 3.1). Replace the path with yours:

```powershell
Get-ChildItem "C:\chemin\vers\vos-vm\srv1-vmware" | Select-Object Name, @{n='Mo';e={[math]::Round($_.Length/1MB,1)}}
```

=== "Français"

    Repérez le fichier de configuration (`.vmx`), le disque virtuel (`.vmdk`) et les fichiers liés aux snapshots. Comparez avec le `.vbox` et le `.vdi` de VirtualBox.

=== "English"

    Identify the configuration file (`.vmx`), the virtual disk (`.vmdk`) and the snapshot-related files. Compare them with VirtualBox's `.vbox` and `.vdi`.

### Étape 12 : Exporter depuis VMware (facultatif) / Step 12: Export from VMware (optional)

=== "Français"

    VM éteinte, selon l'édition de VMware Workstation : **File**, **Export to OVF**. Choisissez un dossier de destination et le nom `srv2.ovf` (ou `.ova` si l'option est proposée). L'archive obtenue pourra être importée sur un autre poste ou dans VirtualBox, comme dans la direction inverse.

=== "English"

    With the VM powered off, depending on the VMware Workstation edition: **File**, **Export to OVF**. Choose a destination folder and the name `srv2.ovf` (or `.ova` if the option is offered). The resulting archive can be imported on another computer or into VirtualBox, just as in the opposite direction.

### Étape 13 : Fiche de comparaison / Step 13: Comparison sheet

=== "Français"

    Remplissez la fiche ci-dessous : c'est le **livrable** du lab.

    | Rubrique | VirtualBox | VMware Workstation |
    |---|---|---|
    | Version de l'outil | … | … |
    | Fichier de configuration (extension) | … | … |
    | Fichier de disque virtuel (extension) | … | … |
    | Noms des interfaces réseau dans la VM | … | … |
    | Taille de l'archive OVA (Go) | … | — |
    | Réseaux utilisés (NAT / hôte seul) | … | … |
    | Virtualisation imbriquée : compteur `vmx` / `svm` | — | … |
    | Clone : nom d'hôte, adresse, `machine-id` différents de l'original | — | oui / non |
    | Difficultés rencontrées lors de l'import | — | … |

=== "English"

    Fill in the sheet below: this is the lab **deliverable**.

    | Item | VirtualBox | VMware Workstation |
    |---|---|---|
    | Tool version | … | … |
    | Configuration file (extension) | … | … |
    | Virtual disk file (extension) | … | … |
    | Network interface names inside the VM | … | … |
    | OVA archive size (GB) | … | — |
    | Networks used (NAT / host-only) | … | … |
    | Nested virtualization: `vmx` / `svm` counter | — | … |
    | Clone: hostname, address, `machine-id` different from the original | — | yes / no |
    | Difficulties met during import | — | … |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi les noms des interfaces réseau ont-ils changé après l'import ?"
        Le matériel virtuel présenté à la VM n'est pas le même sous VirtualBox et sous VMware (carte réseau et emplacement différents). Linux nomme les interfaces d'après leur emplacement matériel : les noms changent, et les fichiers de configuration qui citent les anciens noms ne s'appliquent plus.

    ??? question "Pourquoi régénérer le `machine-id` et les clés SSH d'un clone ?"
        Ces éléments identifient une machine. S'ils sont identiques sur plusieurs machines, des services peuvent les confondre (par exemple un serveur DHCP qui attribue la même adresse), et les clés SSH partagées affaiblissent la sécurité : n'importe quelle copie peut se faire passer pour une autre.

    ??? question "Quand préférer exporter une VM en OVA plutôt que la cloner ?"
        Pour transférer la VM vers un autre poste ou un autre hyperviseur, ou pour l'archiver : l'OVA est un fichier portable. Le clone sert à obtenir rapidement plusieurs machines sur le même hyperviseur.

=== "English"

    ??? question "Why did the network interface names change after the import?"
        The virtual hardware presented to the VM is not the same under VirtualBox and VMware (different network card and slot). Linux names interfaces after their hardware location: the names change, and configuration files citing the old names no longer apply.

    ??? question "Why regenerate a clone's `machine-id` and SSH keys?"
        These elements identify a machine. If they are identical on several machines, services may confuse them (for example a DHCP server handing out the same address), and shared SSH keys weaken security: any copy can impersonate another.

    ??? question "When should you export a VM as an OVA rather than clone it?"
        To move the VM to another computer or another hypervisor, or to archive it: the OVA is a portable file. A clone is for quickly getting several machines on the same hypervisor.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | L'import de l'OVA échoue | Cliquer sur **Retry** pour relâcher les vérifications ; sinon réexporter depuis VirtualBox avec un autre format OVF, ou convertir le disque (voir l'encadré ci-dessous) |
    | Pas de réseau après l'import | Noms d'interfaces différents : corriger les fichiers `/etc/netplan/*.yaml` (étape 6) |
    | VMware refuse de démarrer la VM, message sur VT-x/AMD-V | Virtualisation désactivée ou conflit avec l'hyperviseur Windows : voir le [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) |
    | `ssh` ou `ping` vers la VM échoue | Adresse hors du sous-réseau de VMnet1 ; carte **VMware Network Adapter VMnet1** désactivée dans Windows (Connexions réseau) ; pare-feu Windows |
    | Deux machines répondent à la même adresse | Une autre VM (VirtualBox, VMware, clone) est allumée avec la même adresse : éteindre les autres |
    | Compteur `vmx` / `svm` égal à 0 | VM non éteinte lors du réglage ; option non cochée ; hyperviseur Windows actif (voir Lab 2.1) ; processeur ne le permettant pas |
    | Le clone se connecte avec la même adresse que l'original | Étape 9 oubliée : modifier `99-gestion.yaml` puis `sudo netplan apply` |

    !!! tip "Variante : convertir le disque au lieu d'importer l'OVA"
        On peut convertir le disque de VirtualBox au format VMware puis créer une VM VMware qui l'utilise (**Use an existing virtual disk**) :
        `VBoxManage clonemedium disk "srv1.vdi" "srv1.vmdk" --format VMDK`.
        Travaillez sur une copie du disque, et attendez-vous aux mêmes changements de noms d'interfaces.

=== "English"

    | Symptom | Hints |
    |---|---|
    | The OVA import fails | Click **Retry** to relax checks; otherwise re-export from VirtualBox with another OVF format, or convert the disk (see the box below) |
    | No network after the import | Different interface names: fix the `/etc/netplan/*.yaml` files (step 6) |
    | VMware refuses to start the VM, message about VT-x/AMD-V | Virtualization disabled or conflict with the Windows hypervisor: see [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) |
    | `ssh` or `ping` to the VM fails | Address outside the VMnet1 subnet; **VMware Network Adapter VMnet1** disabled in Windows (Network Connections); Windows firewall |
    | Two machines answer to the same address | Another VM (VirtualBox, VMware, clone) is powered on with the same address: shut the others down |
    | `vmx` / `svm` counter equals 0 | VM was not off when setting the option; option not checked; Windows hypervisor active (see Lab 2.1); processor not supporting it |
    | The clone connects with the same address as the original | Step 9 forgotten: edit `99-gestion.yaml` then `sudo netplan apply` |

    !!! tip "Variant: convert the disk instead of importing the OVA"
        You can convert the VirtualBox disk to the VMware format then create a VMware VM that uses it (**Use an existing virtual disk**):
        `VBoxManage clonemedium disk "srv1.vdi" "srv1.vmdk" --format VMDK`.
        Work on a copy of the disk, and expect the same interface name changes.

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    Conservez `srv1-vmware` et son snapshot `etat-propre` : cette VM est la base des labs suivants. Le clone `srv2` peut être supprimé (**VM**, **Manage**, **Delete from Disk**) pour libérer de l'espace, ainsi que l'archive `srv1.ova` si vous n'en avez plus besoin. Pour repartir d'un état propre, restaurez `etat-propre` (**VM**, **Snapshot**, **Snapshot Manager**).

=== "English"

    Keep `srv1-vmware` and its `etat-propre` snapshot: this VM is the base for the following labs. The `srv2` clone can be deleted (**VM**, **Manage**, **Delete from Disk**) to free space, as can the `srv1.ova` archive if you no longer need it. To return to a clean state, restore `etat-propre` (**VM**, **Snapshot**, **Snapshot Manager**).

[Retour au chapitre 3 / Back to chapter 3](cours.md)