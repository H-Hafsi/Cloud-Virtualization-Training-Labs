# Lab 3.1 : VirtualBox, une VM Ubuntu Server / VirtualBox: an Ubuntu Server VM

| | |
|---|---|
| **Durée / Duration** | 60 à 75 min / 60 to 75 min |
| **Niveau / Level** | Guidé / Guided |
| **Prérequis / Prerequisites** | [Chapitre 3 : cours](cours.md) ; [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) réalisé (virtualisation activée, ISO Ubuntu Server téléchargé et vérifié, dossiers créés) |
| **Ressources minimales / Minimum resources** | **VM** : 2 vCPU, 2 Go de RAM, disque de 25 Go (dynamique). **Hôte** : au moins 4 Go de RAM libres et 30 Go d'espace disque libre. Accès Internet. / **VM**: 2 vCPUs, 2 GB RAM, 25 GB disk (dynamic). **Host**: at least 4 GB of free RAM and 30 GB of free disk space. Internet access. |
| **Acquis visés / Learning outcomes** | AA2 |

=== "Français"

    !!! abstract "Objectifs"
        - Installer VirtualBox et créer une VM Ubuntu Server de référence
        - Configurer deux cartes réseau : NAT (Internet) et hôte seul (administration)
        - Se connecter à la VM en SSH depuis votre poste Windows
        - Prendre un snapshot de référence `etat-propre` et le restaurer
        - Observer le provisionnement dynamique du disque virtuel

=== "English"

    !!! abstract "Objectives"
        - Install VirtualBox and create a reference Ubuntu Server VM
        - Configure two network cards: NAT (Internet) and host-only (administration)
        - Connect to the VM over SSH from your Windows computer
        - Take an `etat-propre` reference snapshot and restore it
        - Observe the dynamic provisioning of the virtual disk

## Contexte et schéma / Context and diagram

=== "Français"

    Vous construisez la **VM de référence** qui servira de base à de nombreux labs. Elle possède deux cartes : l'une en **NAT** pour les mises à jour, l'autre en **hôte seul** pour s'y connecter depuis votre poste. Cette VM sera réutilisée au [Lab 3.2](lab-3-2-vmware-workstation-meme-vm-import-et-export.md) : **ne la supprimez pas** à la fin.

    ```mermaid
    flowchart LR
      P["Poste Windows<br/>(hôte)"] --- HO["Réseau hôte seul<br/>(administration)"]
      HO --- V["VM Ubuntu Server<br/>(VirtualBox)"]
      V --- NAT["NAT<br/>(Internet)"]
    ```

    !!! info "Libellés de l'interface"
        Les noms de menus et d'options ci-dessous sont ceux de l'interface en anglais de VirtualBox ; si votre interface est en français, retrouvez les équivalents. Les adresses IP du réseau hôte seul suivent le **plan d'adressage de l'annexe [Environnement](../../annexes/environnement.md)** : elles sont notées `<IP_VM>` et `<IP_HOTE>` dans ce lab.

=== "English"

    You are building the **reference VM** that will be the base for many labs. It has two cards: one in **NAT** for updates, the other in **host-only** to connect from your computer. This VM will be reused in [Lab 3.2](lab-3-2-vmware-workstation-meme-vm-import-et-export.md): **do not delete it** at the end.

    ```mermaid
    flowchart LR
      P["Windows computer<br/>(host)"] --- HO["Host-only network<br/>(administration)"]
      HO --- V["Ubuntu Server VM<br/>(VirtualBox)"]
      V --- NAT["NAT<br/>(Internet)"]
    ```

    !!! info "Interface labels"
        The menu and option names below are those of VirtualBox's English interface; if your interface is in French, find the equivalents. Host-only network IP addresses follow the **addressing plan of the [Environment](../../annexes/environnement.md) annex**: they are written `<IP_VM>` and `<IP_HOTE>` in this lab.

## Étapes / Steps

### Étape 1 : Installer VirtualBox / Step 1: Install VirtualBox

=== "Français"

    1. Téléchargez VirtualBox depuis le **site officiel** (virtualbox.org) et lancez l'installateur avec les options par défaut.
    2. L'installateur prévient que les connexions réseau vont être **brièvement interrompues** : acceptez.
    3. Si vous utilisez Hyper-V ou l'hyperviseur de Windows, vérifiez son état comme au [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) : pour ce lab, **désactivé** est recommandé.
    4. Lancez VirtualBox. Notez sa version (menu **Help**, **About VirtualBox**).

    !!! success "Point de contrôle"
        - [ ] VirtualBox démarre et affiche sa fenêtre principale
        - [ ] Vous avez noté sa version

=== "English"

    1. Download VirtualBox from the **official website** (virtualbox.org) and run the installer with default options.
    2. The installer warns that network connections will be **briefly interrupted**: accept.
    3. If you use Hyper-V or the Windows hypervisor, check its state as in [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md): for this lab, **off** is recommended.
    4. Start VirtualBox. Note its version (menu **Help**, **About VirtualBox**).

    !!! success "Checkpoint"
        - [ ] VirtualBox starts and shows its main window
        - [ ] You noted its version

### Étape 2 : Créer la machine virtuelle / Step 2: Create the virtual machine

=== "Français"

    Cliquez sur **New** et renseignez :

    | Paramètre | Valeur |
    |---|---|
    | Name | Un nom explicite, par exemple `srv1` |
    | Folder | Le dossier des machines virtuelles créé au Lab 2.1 |
    | ISO Image | L'ISO Ubuntu Server téléchargé |
    | Type / Version | Linux / Ubuntu (64-bit) |
    | Skip Unattended Installation | **Cochée** (nous installons à la main) |
    | Base Memory | 2048 MB |
    | Processors | 2 |
    | Hard Disk | Create a Virtual Hard Disk Now, **25 GB**, **sans** cocher « Pre-Allocate Full Size » (disque dynamique) |

    Terminez l'assistant sans démarrer la VM.

    !!! success "Point de contrôle"
        - [ ] La VM apparaît dans la liste, à l'état « Powered Off »
        - [ ] Le disque est configuré en allocation dynamique

=== "English"

    Click **New** and fill in:

    | Setting | Value |
    |---|---|
    | Name | An explicit name, for example `srv1` |
    | Folder | The virtual machines folder created in Lab 2.1 |
    | ISO Image | The downloaded Ubuntu Server ISO |
    | Type / Version | Linux / Ubuntu (64-bit) |
    | Skip Unattended Installation | **Checked** (we install by hand) |
    | Base Memory | 2048 MB |
    | Processors | 2 |
    | Hard Disk | Create a Virtual Hard Disk Now, **25 GB**, **without** checking "Pre-Allocate Full Size" (dynamic disk) |

    Finish the wizard without starting the VM.

    !!! success "Checkpoint"
        - [ ] The VM appears in the list, in the "Powered Off" state
        - [ ] The disk is configured with dynamic allocation

### Étape 3 : Configurer le réseau / Step 3: Configure the network

=== "Français"

    1. Ouvrez **Tools**, puis **Network** (gestionnaire de réseau) et vérifiez qu'un **réseau hôte seul** existe ; créez-le sinon. Réglez sa plage d'adresses selon l'annexe Environnement et relevez l'adresse de l'hôte sur ce réseau (`<IP_HOTE>`).
    2. Sélectionnez la VM, ouvrez **Settings**, **Network** :
        - **Adapter 1** : *Enabled*, *Attached to* **NAT** ;
        - **Adapter 2** : *Enabled*, *Attached to* **Host-only Adapter**, puis choisissez le réseau hôte seul vérifié ci-dessus.
    3. Validez avec **OK**.

    !!! success "Point de contrôle"
        - [ ] Adapter 1 en NAT, Adapter 2 en hôte seul
        - [ ] Vous avez noté l'adresse de l'hôte sur le réseau hôte seul

=== "English"

    1. Open **Tools**, then **Network** (network manager) and check that a **host-only network** exists; create it otherwise. Set its address range according to the Environment annex and note the host's address on this network (`<IP_HOTE>`).
    2. Select the VM, open **Settings**, **Network**:
        - **Adapter 1**: *Enabled*, *Attached to* **NAT**;
        - **Adapter 2**: *Enabled*, *Attached to* **Host-only Adapter**, then choose the host-only network checked above.
    3. Confirm with **OK**.

    !!! success "Checkpoint"
        - [ ] Adapter 1 on NAT, Adapter 2 on host-only
        - [ ] You noted the host's address on the host-only network

### Étape 4 : Installer Ubuntu Server / Step 4: Install Ubuntu Server

=== "Français"

    Démarrez la VM (**Start**) : elle boote sur l'ISO. Dans l'installateur, utilisez les flèches, ++tab++ et ++enter++ :

    | Écran | Choix |
    |---|---|
    | Langue, clavier | Selon votre clavier (testez-le dans le champ proposé) |
    | Type d'installation | Ubuntu Server (version standard) |
    | Réseau | Laissez les valeurs par défaut : deux interfaces sont listées ; seule celle en NAT obtient une adresse par défaut |
    | Proxy, miroir | Valeurs par défaut |
    | Stockage | Utiliser le disque entier (configuration par défaut) |
    | Profil | Votre nom, le **nom du serveur** (par exemple `srv1`), un nom d'utilisateur et un mot de passe |
    | SSH | **Cochez** « Install OpenSSH server » |
    | Snaps proposés | Ne rien sélectionner |

    Lancez l'installation, attendez la fin, puis choisissez **Reboot Now**. Si l'installateur demande de retirer le support d'installation, appuyez sur ++enter++ : VirtualBox éjecte l'ISO.

    !!! success "Point de contrôle"
        - [ ] Après le redémarrage, une invite de connexion s'affiche
        - [ ] Vous pouvez vous connecter avec le compte créé

=== "English"

    Start the VM (**Start**): it boots from the ISO. In the installer, use the arrow keys, ++tab++ and ++enter++:

    | Screen | Choice |
    |---|---|
    | Language, keyboard | According to your keyboard (test it in the proposed field) |
    | Installation type | Ubuntu Server (standard version) |
    | Network | Leave the defaults: two interfaces are listed; only the NAT one gets an address by default |
    | Proxy, mirror | Defaults |
    | Storage | Use the entire disk (default configuration) |
    | Profile | Your name, the **server name** (for example `srv1`), a username and a password |
    | SSH | **Check** "Install OpenSSH server" |
    | Suggested snaps | Select nothing |

    Start the installation, wait for it to finish, then choose **Reboot Now**. If the installer asks you to remove the installation medium, press ++enter++: VirtualBox ejects the ISO.

    !!! success "Checkpoint"
        - [ ] After the restart, a login prompt appears
        - [ ] You can log in with the account you created

### Étape 5 : Mettre à jour et relever les informations / Step 5: Update and collect information

=== "Français"

    Connectez-vous dans la console de la VM, puis exécutez :

=== "English"

    Log in on the VM console, then run:

```bash
sudo apt update && sudo apt full-upgrade -y
lsb_release -a
hostnamectl
free -h
df -h /
ip -br link
ip -br addr
```

=== "Français"

    Résultat attendu : la distribution et sa version, le nom d'hôte, environ 2 Go de RAM, la taille du système de fichiers racine, et la liste des deux interfaces réseau. Notez le **nom de l'interface en hôte seul** (la seconde, souvent `enp0s8`) : elle n'a pas encore d'adresse IPv4.

    !!! success "Point de contrôle"
        - [ ] La mise à jour s'est terminée sans erreur (accès Internet par NAT confirmé)
        - [ ] Vous connaissez le nom de l'interface hôte seul

=== "English"

    Expected result: the distribution and its version, the hostname, about 2 GB of RAM, the root file system size, and the list of the two network interfaces. Note the **name of the host-only interface** (the second one, often `enp0s8`): it does not have an IPv4 address yet.

    !!! success "Checkpoint"
        - [ ] The update finished without error (Internet access through NAT confirmed)
        - [ ] You know the name of the host-only interface

### Étape 6 : Attribuer une adresse statique à l'interface d'administration / Step 6: Give a static address to the management interface

=== "Français"

    Créez un fichier de configuration Netplan pour la seconde interface (remplacez `enp0s8` par le nom relevé et `<IP_VM>` par une adresse libre du réseau hôte seul, avec son préfixe, par exemple `/24`, selon l'annexe Environnement) :

=== "English"

    Create a Netplan configuration file for the second interface (replace `enp0s8` with the name you noted and `<IP_VM>` with a free address of the host-only network, with its prefix, for example `/24`, according to the Environment annex):

```bash
sudo nano /etc/netplan/99-gestion.yaml
```

```yaml
network:
  version: 2
  ethernets:
    enp0s8:
      dhcp4: false
      addresses:
        - <IP_VM>/24
```

```bash
sudo chmod 600 /etc/netplan/99-gestion.yaml
sudo netplan apply
ip -br addr
```

=== "Français"

    Résultat attendu : l'interface en hôte seul affiche maintenant `<IP_VM>`. Respectez l'indentation du fichier YAML (des espaces, pas de tabulations).

    !!! success "Point de contrôle"
        - [ ] `ip -br addr` montre l'adresse statique sur l'interface hôte seul
        - [ ] L'interface NAT conserve son adresse et l'accès à Internet

=== "English"

    Expected result: the host-only interface now shows `<IP_VM>`. Respect the YAML file indentation (spaces, no tabs).

    !!! success "Checkpoint"
        - [ ] `ip -br addr` shows the static address on the host-only interface
        - [ ] The NAT interface keeps its address and Internet access

### Étape 7 : Se connecter en SSH depuis Windows / Step 7: Connect over SSH from Windows

=== "Français"

    Dans **PowerShell** sur votre poste (remplacez par votre nom d'utilisateur et l'adresse de la VM) :

=== "English"

    In **PowerShell** on your computer (replace with your username and the VM's address):

```powershell
ping <IP_VM>
ssh utilisateur@<IP_VM>
```

=== "Français"

    À la première connexion, SSH demande de confirmer l'empreinte de la machine : répondez `yes`. Saisissez ensuite le mot de passe. Tapez `exit` pour quitter.

    !!! success "Point de contrôle"
        - [ ] Le `ping` aboutit
        - [ ] Vous obtenez une invite de commande de la VM par SSH

=== "English"

    On first connection, SSH asks you to confirm the machine's fingerprint: answer `yes`. Then enter the password. Type `exit` to leave.

    !!! success "Checkpoint"
        - [ ] `ping` succeeds
        - [ ] You get a VM command prompt over SSH

### Étape 8 : Outils invité (facultatif) / Step 8: Guest tools (optional)

=== "Français"

    Sur un serveur sans interface graphique, les outils invité sont facultatifs. Ils améliorent l'intégration (par exemple l'arrêt propre depuis VirtualBox). Dans la VM :

=== "English"

    On a server without a graphical interface, guest tools are optional. They improve integration (for example a clean shutdown from VirtualBox). In the VM:

```bash
sudo apt install -y virtualbox-guest-utils
```

### Étape 9 : Snapshot de référence et restauration / Step 9: Reference snapshot and restore

=== "Français"

    1. Éteignez proprement la VM : `sudo poweroff`.
    2. Dans VirtualBox, sélectionnez la VM, ouvrez l'onglet **Snapshots** et cliquez sur **Take**. Nommez le snapshot **`etat-propre`**.
    3. Redémarrez la VM, connectez-vous et créez un fichier de test :

=== "English"

    1. Shut the VM down cleanly: `sudo poweroff`.
    2. In VirtualBox, select the VM, open the **Snapshots** tab and click **Take**. Name the snapshot **`etat-propre`**.
    3. Restart the VM, log in and create a test file:

```bash
touch ~/fichier-de-test && ls ~
```

=== "Français"

    4. Éteignez la VM (`sudo poweroff`), puis dans l'onglet **Snapshots** sélectionnez `etat-propre` et cliquez sur **Restore** (ne conservez pas l'état courant).
    5. Redémarrez la VM et vérifiez que le fichier a disparu :

=== "English"

    4. Shut the VM down (`sudo poweroff`), then in the **Snapshots** tab select `etat-propre` and click **Restore** (do not keep the current state).
    5. Restart the VM and check that the file is gone:

```bash
ls ~
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Le snapshot `etat-propre` existe
        - [ ] Après restauration, `fichier-de-test` n'existe plus

=== "English"

    !!! success "Checkpoint"
        - [ ] The `etat-propre` snapshot exists
        - [ ] After restoring, `fichier-de-test` no longer exists

### Étape 10 : Observer le disque dynamique / Step 10: Observe the dynamic disk

=== "Français"

    Sur votre poste, comparez la taille déclarée du disque (25 Go) à la taille réelle du fichier `.vdi` dans le dossier de la VM. Remplacez le chemin par celui de votre dossier des VM :

=== "English"

    On your computer, compare the declared disk size (25 GB) with the actual size of the `.vdi` file in the VM folder. Replace the path with your VM folder:

```powershell
Get-ChildItem -Recurse "C:\chemin\vers\vos-vm\srv1" -Include *.vdi | Select-Object Name, @{n='Go';e={[math]::Round($_.Length/1GB,2)}}
```

=== "Français"

    Résultat attendu : une taille bien inférieure à 25 Go, qui correspond aux données réellement écrites (provisionnement dynamique, chapitre 2). Elle augmentera au fil de l'utilisation. Les fichiers de snapshot se trouvent dans un sous-dossier de la VM.

    !!! success "Point de contrôle"
        - [ ] Vous avez comparé taille déclarée et taille réelle du disque

=== "English"

    Expected result: a size well below 25 GB, matching the data actually written (dynamic provisioning, chapter 2). It will grow with use. Snapshot files are in a subfolder of the VM.

    !!! success "Checkpoint"
        - [ ] You compared declared and actual disk size

### Étape 11 : Piloter la VM en ligne de commande (facultatif) / Step 11: Drive the VM from the command line (optional)

=== "Français"

    Dans PowerShell, l'outil `VBoxManage` se trouve dans le dossier d'installation de VirtualBox (par défaut `C:\Program Files\Oracle\VirtualBox`) :

=== "English"

    In PowerShell, the `VBoxManage` tool is in VirtualBox's installation folder (by default `C:\Program Files\Oracle\VirtualBox`):

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" list vms
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" startvm "srv1" --type headless
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" controlvm "srv1" acpipowerbutton
```

=== "Français"

    La deuxième commande démarre la VM **sans fenêtre** (vous vous y connectez par SSH) ; la troisième envoie une demande d'arrêt propre.

=== "English"

    The second command starts the VM **without a window** (you connect through SSH); the third sends a clean shutdown request.

### Étape 12 : Fiche de la VM de référence / Step 12: Reference VM sheet

=== "Français"

    Remplissez la fiche ci-dessous : c'est le **livrable** du lab.

    | Rubrique | Valeur |
    |---|---|
    | Version de VirtualBox | … |
    | Nom de la VM et nom d'hôte | … |
    | vCPU / RAM / disque déclaré | … / … / … |
    | Taille réelle du fichier `.vdi` (Go) | … |
    | Version d'Ubuntu Server | … |
    | Adresse de l'interface NAT | … |
    | Interface et adresse en hôte seul | … |
    | Connexion SSH depuis Windows réussie | oui / non |
    | Snapshot `etat-propre` créé et restauré | oui / non |

=== "English"

    Fill in the sheet below: this is the lab **deliverable**.

    | Item | Value |
    |---|---|
    | VirtualBox version | … |
    | VM name and hostname | … |
    | vCPU / RAM / declared disk | … / … / … |
    | Actual size of the `.vdi` file (GB) | … |
    | Ubuntu Server version | … |
    | NAT interface address | … |
    | Host-only interface and address | … |
    | SSH connection from Windows successful | yes / no |
    | `etat-propre` snapshot created and restored | yes / no |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi la VM a-t-elle deux cartes réseau et à quoi sert chacune ?"
        La carte en **NAT** donne accès à Internet (mises à jour, téléchargements) sans exposer la VM sur le réseau local. La carte en **hôte seul** crée un réseau privé avec votre poste, utilisé pour l'administration en SSH avec une adresse stable.

    ??? question "Pourquoi éteindre la VM avant de prendre le snapshot de référence ?"
        Un snapshot pris VM éteinte ne contient que l'état du disque : il est plus petit et sa restauration produit un démarrage propre. Un snapshot pris VM allumée conserve aussi la mémoire, ce qui est plus lourd et restaure l'état « en fonctionnement ».

    ??? question "Pourquoi le fichier `.vdi` est-il bien plus petit que les 25 Go déclarés ?"
        Le disque est en provisionnement dynamique : le fichier ne grossit qu'avec les données réellement écrites. Il faut donc surveiller l'espace libre de l'hôte, car les disques peuvent croître jusqu'à leur taille maximale.

=== "English"

    ??? question "Why does the VM have two network cards and what is each for?"
        The **NAT** card gives Internet access (updates, downloads) without exposing the VM on the local network. The **host-only** card creates a private network with your computer, used for SSH administration with a stable address.

    ??? question "Why shut the VM down before taking the reference snapshot?"
        A snapshot taken with the VM off contains only the disk state: it is smaller and restoring it gives a clean boot. A snapshot taken with the VM on also keeps memory, which is heavier and restores the "running" state.

    ??? question "Why is the `.vdi` file much smaller than the declared 25 GB?"
        The disk uses dynamic provisioning: the file only grows with the data actually written. The host's free space must therefore be watched, as disks can grow up to their maximum size.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | La VM refuse de démarrer, message sur VT-x ou AMD-V | Virtualisation désactivée dans l'UEFI ou conflit avec l'hyperviseur de Windows : voir le [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) |
    | Aucune version 64 bits proposée dans la liste des systèmes | Même cause : virtualisation matérielle non disponible pour VirtualBox |
    | La VM ne démarre pas sur l'ISO | Vérifier dans **Settings**, **Storage** que l'ISO est bien dans le lecteur optique et que l'ordre de démarrage place le lecteur optique avant le disque |
    | `apt update` échoue | Vérifier l'interface NAT (`ip -br addr`), la résolution DNS, l'accès Internet de l'hôte |
    | L'interface hôte seul n'apparaît pas dans la VM | Vérifier que l'Adapter 2 est activé et rattaché à un réseau hôte seul existant ; redémarrer la VM |
    | `netplan apply` signale une erreur | Vérifier l'indentation (espaces), le nom de l'interface et le format de l'adresse (`adresse/préfixe`) |
    | `ping` vers la VM échoue depuis Windows | Vérifier que l'adresse de la VM est dans le même réseau que l'adresse de l'hôte sur le réseau hôte seul ; vérifier le pare-feu Windows |
    | `ssh` renvoie « connection refused » | Vérifier que le serveur SSH est installé et actif dans la VM : `systemctl status ssh` ; l'installer si besoin : `sudo apt install openssh-server` |

=== "English"

    | Symptom | Hints |
    |---|---|
    | The VM refuses to start, message about VT-x or AMD-V | Virtualization disabled in UEFI or conflict with the Windows hypervisor: see [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) |
    | No 64-bit version offered in the system list | Same cause: hardware virtualization not available to VirtualBox |
    | The VM does not boot from the ISO | Check in **Settings**, **Storage** that the ISO is in the optical drive and that the boot order puts the optical drive before the disk |
    | `apt update` fails | Check the NAT interface (`ip -br addr`), DNS resolution, the host's Internet access |
    | The host-only interface does not appear in the VM | Check that Adapter 2 is enabled and attached to an existing host-only network; restart the VM |
    | `netplan apply` reports an error | Check indentation (spaces), the interface name and the address format (`address/prefix`) |
    | `ping` to the VM fails from Windows | Check that the VM's address is in the same network as the host's address on the host-only network; check the Windows firewall |
    | `ssh` returns "connection refused" | Check that the SSH server is installed and active in the VM: `systemctl status ssh`; install it if needed: `sudo apt install openssh-server` |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    **Ne supprimez pas la VM** : elle sert au Lab 3.2 (import et export). Éteignez-la proprement avant de fermer VirtualBox. Pour repartir d'un état propre à tout moment, restaurez le snapshot `etat-propre`. Si vous devez tout recommencer, supprimez la VM (**Remove**, puis **Delete all files**) et reprenez à l'étape 2.

=== "English"

    **Do not delete the VM**: it is used in Lab 3.2 (import and export). Shut it down cleanly before closing VirtualBox. To start again from a clean state at any time, restore the `etat-propre` snapshot. If you have to start over completely, delete the VM (**Remove**, then **Delete all files**) and resume at step 2.

[Retour au chapitre 3 / Back to chapter 3](cours.md)