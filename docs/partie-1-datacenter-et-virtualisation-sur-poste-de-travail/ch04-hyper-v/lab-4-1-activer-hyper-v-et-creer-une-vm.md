# Lab 4.1 : Activer Hyper-V et créer une VM / Enabling Hyper-V and Creating a VM

| | |
|---|---|
| **Durée / Duration** | 60 à 75 min / 60 to 75 min |
| **Niveau / Level** | Guidé / Guided |
| **Prérequis / Prerequisites** | [Chapitre 4 : cours](cours.md) ; [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) réalisé (ISO Ubuntu Server vérifié, dossiers créés, fiche de préparation) |
| **Ressources minimales / Minimum resources** | **VM** : 2 vCPU, 2 Go de RAM, disque VHDX de 25 Go (dynamique). **Hôte** : Windows 10 ou 11 **Pro, Enterprise ou Éducation**, compte administrateur, au moins 4 Go de RAM libres, 30 Go d'espace disque libre, un redémarrage. L'hyperviseur Windows doit être **activé** pour ce lab. / **VM**: 2 vCPUs, 2 GB RAM, 25 GB VHDX disk (dynamic). **Host**: Windows 10 or 11 **Pro, Enterprise or Education**, administrator account, at least 4 GB of free RAM, 30 GB of free disk space, one restart. The Windows hypervisor must be **on** for this lab. |
| **Acquis visés / Learning outcomes** | AA2 |

=== "Français"

    !!! abstract "Objectifs"
        - Vérifier que l'édition de Windows et le processeur permettent Hyper-V, puis l'activer
        - Créer un commutateur virtuel interne pour l'administration et utiliser le *Default Switch* pour l'accès à Internet
        - Créer une VM Ubuntu Server de génération 2 avec PowerShell
        - Résoudre le problème du démarrage sécurisé avec Linux
        - Se connecter à la VM en SSH et vérifier les services d'intégration

=== "English"

    !!! abstract "Objectives"
        - Check that the Windows edition and processor allow Hyper-V, then enable it
        - Create an internal virtual switch for administration and use the *Default Switch* for Internet access
        - Create a generation 2 Ubuntu Server VM with PowerShell
        - Solve the Secure Boot problem with Linux
        - Connect to the VM over SSH and check the integration services

## Contexte et schéma / Context and diagram

=== "Français"

    Vous reproduisez avec Hyper-V la **VM de référence** déjà construite avec VirtualBox puis VMware : deux cartes, l'une pour Internet, l'autre pour l'administration depuis votre poste. Ici, la carte Internet est connectée au **Default Switch** (NAT) et la carte d'administration à un commutateur **interne** que vous créez.

    ```mermaid
    flowchart LR
      P["Poste Windows<br/>(partition racine)"] --- G["Commutateur interne<br/>gestion-interne"]
      G --- V["VM srv1-hv<br/>(génération 2)"]
      V --- D["Default Switch<br/>(NAT, Internet)"]
    ```

    !!! warning "À lire avant de commencer"
        Activer Hyper-V change la façon dont Windows démarre et **demande un redémarrage**. Tant qu'il est actif, VirtualBox et VMware Workstation fonctionnent avec moins de performances. Pour revenir à l'état précédent à la fin du chapitre, utilisez la procédure du [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md). Les adresses du réseau d'administration suivent le plan de l'annexe [Environnement](../../annexes/environnement.md) (notées `<IP_HOTE>` et `<IP_VM>`).

=== "English"

    You reproduce with Hyper-V the **reference VM** already built with VirtualBox then VMware: two cards, one for the Internet, the other for administration from your computer. Here, the Internet card is connected to the **Default Switch** (NAT) and the administration card to an **internal** switch that you create.

    ```mermaid
    flowchart LR
      P["Windows computer<br/>(root partition)"] --- G["Internal switch<br/>gestion-interne"]
      G --- V["VM srv1-hv<br/>(generation 2)"]
      V --- D["Default Switch<br/>(NAT, Internet)"]
    ```

    !!! warning "Read before starting"
        Enabling Hyper-V changes the way Windows boots and **requires a restart**. While it is active, VirtualBox and VMware Workstation run with lower performance. To return to the previous state at the end of the chapter, use the procedure from [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md). Administration network addresses follow the plan of the [Environment](../../annexes/environnement.md) annex (written `<IP_HOTE>` and `<IP_VM>`).

## Étapes / Steps

### Étape 1 : Vérifier les prérequis / Step 1: Check the prerequisites

=== "Français"

    Ouvrez **PowerShell en administrateur** et lancez :

=== "English"

    Open **PowerShell as administrator** and run:

```powershell
(Get-CimInstance Win32_OperatingSystem).Caption
(Get-CimInstance Win32_Processor).SecondLevelAddressTranslationExtensions
```

=== "Français"

    Le premier résultat doit indiquer une édition **Pro, Enterprise ou Éducation** (pas « Famille ») ; le second doit valoir `True` (SLAT disponible). La virtualisation matérielle activée dans le micrologiciel a été vérifiée au Lab 2.1.

    !!! success "Point de contrôle"
        - [ ] Édition compatible
        - [ ] SLAT disponible

=== "English"

    The first result must show a **Pro, Enterprise or Education** edition (not "Home"); the second must be `True` (SLAT available). Hardware virtualization enabled in the firmware was checked in Lab 2.1.

    !!! success "Checkpoint"
        - [ ] Compatible edition
        - [ ] SLAT available

### Étape 2 : Activer Hyper-V / Step 2: Enable Hyper-V

=== "Français"

    Dans PowerShell administrateur, assurez-vous que l'hyperviseur est autorisé à démarrer, puis activez la fonctionnalité Hyper-V :

=== "English"

    In administrator PowerShell, make sure the hypervisor is allowed to start, then enable the Hyper-V feature:

```powershell
bcdedit /set hypervisorlaunchtype auto
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

=== "Français"

    PowerShell propose de redémarrer : acceptez (`Y`). Une autre méthode graphique : touche Windows, tapez `optionalfeatures`, cochez **Hyper-V** (plateforme et outils de gestion), validez et redémarrez.

=== "English"

    PowerShell offers to restart: accept (`Y`). Another graphical method: Windows key, type `optionalfeatures`, check **Hyper-V** (platform and management tools), confirm and restart.

### Étape 3 : Vérifier l'installation / Step 3: Check the installation

=== "Français"

    Après le redémarrage, ouvrez **PowerShell en administrateur** :

=== "English"

    After the restart, open **PowerShell as administrator**:

```powershell
(Get-CimInstance Win32_ComputerSystem).HypervisorPresent
Get-Service vmms
Get-VMSwitch
Get-VMHost | Select-Object VirtualMachinePath, VirtualHardDiskPath
```

=== "Français"

    Résultat attendu : `HypervisorPresent` vaut `True` ; le service `vmms` (gestion des machines virtuelles Hyper-V) est `Running` ; `Get-VMSwitch` liste le **Default Switch** ; le dernier résultat affiche les dossiers par défaut des VM et des disques, qui se trouvent sur le disque système.

    Faites pointer ces dossiers vers le dossier de VM préparé au Lab 2.1 (remplacez le chemin) :

=== "English"

    Expected result: `HypervisorPresent` is `True`; the `vmms` service (Hyper-V virtual machine management) is `Running`; `Get-VMSwitch` lists the **Default Switch**; the last result shows the default VM and disk folders, which are on the system disk.

    Point these folders to the VM folder prepared in Lab 2.1 (replace the path):

```powershell
Set-VMHost -VirtualMachinePath "<DOSSIER_VM>" -VirtualHardDiskPath "<DOSSIER_VM>"
```

=== "Français"

    Ouvrez aussi le **Gestionnaire Hyper-V** (touche Windows, tapez `virtmgmt.msc`) et repérez votre ordinateur dans le volet de gauche.

    !!! success "Point de contrôle"
        - [ ] `HypervisorPresent` vaut `True` et `vmms` est `Running`
        - [ ] Le *Default Switch* est listé
        - [ ] Les dossiers des VM pointent vers votre dossier de travail

=== "English"

    Also open **Hyper-V Manager** (Windows key, type `virtmgmt.msc`) and find your computer in the left pane.

    !!! success "Checkpoint"
        - [ ] `HypervisorPresent` is `True` and `vmms` is `Running`
        - [ ] The *Default Switch* is listed
        - [ ] The VM folders point to your working folder

### Étape 4 : Créer le commutateur interne d'administration / Step 4: Create the internal administration switch

=== "Français"

    Créez un commutateur **interne**, puis attribuez à l'hôte une adresse sur ce réseau (remplacez `<IP_HOTE>` par l'adresse prévue dans l'annexe Environnement, avec le préfixe correspondant, ici `24`) :

=== "English"

    Create an **internal** switch, then give the host an address on this network (replace `<IP_HOTE>` with the address planned in the Environment annex, with the matching prefix, here `24`):

```powershell
New-VMSwitch -Name "gestion-interne" -SwitchType Internal
Get-NetAdapter | Where-Object Name -like "vEthernet*"
New-NetIPAddress -InterfaceAlias "vEthernet (gestion-interne)" -IPAddress <IP_HOTE> -PrefixLength 24
```

=== "Français"

    La deuxième commande liste les cartes virtuelles de l'hôte : la carte `vEthernet (gestion-interne)` est celle qui relie votre poste à ce commutateur.

    !!! success "Point de contrôle"
        - [ ] `Get-VMSwitch` liste `gestion-interne` (type `Internal`)
        - [ ] La carte `vEthernet (gestion-interne)` porte l'adresse `<IP_HOTE>` (`ipconfig` pour vérifier)

=== "English"

    The second command lists the host's virtual adapters: the `vEthernet (gestion-interne)` adapter is the one connecting your computer to this switch.

    !!! success "Checkpoint"
        - [ ] `Get-VMSwitch` lists `gestion-interne` (type `Internal`)
        - [ ] The `vEthernet (gestion-interne)` adapter carries the `<IP_HOTE>` address (`ipconfig` to check)

### Étape 5 : Créer la VM de génération 2 / Step 5: Create the generation 2 VM

=== "Français"

    Remplacez `<DOSSIER_VM>` par votre dossier des VM et `<CHEMIN_ISO>` par le chemin complet de l'ISO Ubuntu Server :

=== "English"

    Replace `<DOSSIER_VM>` with your VM folder and `<CHEMIN_ISO>` with the full path of the Ubuntu Server ISO:

```powershell
New-VM -Name "srv1-hv" -Generation 2 -MemoryStartupBytes 2GB -NewVHDPath "<DOSSIER_VM>\srv1-hv\srv1-hv.vhdx" -NewVHDSizeBytes 25GB -SwitchName "Default Switch"
Set-VMProcessor -VMName "srv1-hv" -Count 2
Set-VMMemory -VMName "srv1-hv" -DynamicMemoryEnabled $false
Add-VMNetworkAdapter -VMName "srv1-hv" -SwitchName "gestion-interne"
Add-VMDvdDrive -VMName "srv1-hv" -Path "<CHEMIN_ISO>"
Set-VMFirmware -VMName "srv1-hv" -FirstBootDevice (Get-VMDvdDrive -VMName "srv1-hv")
```

=== "Français"

    Puis **résolvez le problème du démarrage sécurisé** : sans cette commande, la VM refuserait de démarrer sur l'ISO d'Ubuntu (voir le cours, section 2).

=== "English"

    Then **solve the Secure Boot problem**: without this command, the VM would refuse to boot from the Ubuntu ISO (see the lesson, section 2).

```powershell
Set-VMFirmware -VMName "srv1-hv" -SecureBootTemplate MicrosoftUEFICertificateAuthority
```

=== "Français"

    Résultat attendu : `Get-VM` liste `srv1-hv` à l'état `Off`. Vous pouvez aussi créer la VM avec le **Gestionnaire Hyper-V** (**Action**, **New**, **Virtual Machine**, génération 2, 2048 Mo, connexion au *Default Switch*, disque de 25 Go, installation depuis l'ISO), puis régler le **démarrage sécurisé** dans **Settings**, **Security**, et ajouter la seconde carte réseau dans **Settings**, **Add Hardware**.

    !!! success "Point de contrôle"
        - [ ] La VM `srv1-hv` existe, éteinte
        - [ ] Elle a deux cartes réseau (*Default Switch* et `gestion-interne`) : `Get-VMNetworkAdapter -VMName srv1-hv`

=== "English"

    Expected result: `Get-VM` lists `srv1-hv` in the `Off` state. You can also create the VM with **Hyper-V Manager** (**Action**, **New**, **Virtual Machine**, generation 2, 2048 MB, connection to the *Default Switch*, 25 GB disk, install from the ISO), then set **Secure Boot** in **Settings**, **Security**, and add the second network card in **Settings**, **Add Hardware**.

    !!! success "Checkpoint"
        - [ ] The `srv1-hv` VM exists, powered off
        - [ ] It has two network cards (*Default Switch* and `gestion-interne`): `Get-VMNetworkAdapter -VMName srv1-hv`

### Étape 6 : Installer Ubuntu Server / Step 6: Install Ubuntu Server

=== "Français"

    Démarrez la VM et ouvrez sa console :

=== "English"

    Start the VM and open its console:

```powershell
Start-VM -Name "srv1-hv"
vmconnect.exe localhost "srv1-hv"
```

=== "Français"

    Suivez l'installateur comme au [Lab 3.1](../ch03-virtualbox-et-vmware-workstation/lab-3-1-virtualbox-une-vm-ubuntu-server.md) (étape 4) : disque entier, nom de serveur explicite (par exemple `srv1-hv`), utilisateur et mot de passe, **cochez « Install OpenSSH server »**. À la fin, choisissez **Reboot Now** ; si l'installateur demande de retirer le support, appuyez sur ++enter++.

    !!! success "Point de contrôle"
        - [ ] Après le redémarrage, une invite de connexion s'affiche
        - [ ] Vous pouvez vous connecter avec le compte créé

=== "English"

    Follow the installer as in [Lab 3.1](../ch03-virtualbox-et-vmware-workstation/lab-3-1-virtualbox-une-vm-ubuntu-server.md) (step 4): entire disk, explicit server name (for example `srv1-hv`), user and password, **check "Install OpenSSH server"**. At the end, choose **Reboot Now**; if the installer asks to remove the medium, press ++enter++.

    !!! success "Checkpoint"
        - [ ] After the restart, a login prompt appears
        - [ ] You can log in with the account you created

### Étape 7 : Mettre à jour, configurer le réseau d'administration / Step 7: Update, configure the administration network

=== "Français"

    Sur la console de la VM :

=== "English"

    On the VM console:

```bash
sudo apt update && sudo apt full-upgrade -y
ip -br link
ip -br addr
```

=== "Français"

    Relevez le nom de la **seconde interface** (celle du commutateur `gestion-interne`, sans adresse). Créez le fichier Netplan, comme au Lab 3.1 (étape 6), en remplaçant `<NOM_INTERFACE>` et `<IP_VM>` :

=== "English"

    Note the name of the **second interface** (the one on the `gestion-interne` switch, without an address). Create the Netplan file, as in Lab 3.1 (step 6), replacing `<NOM_INTERFACE>` and `<IP_VM>`:

```bash
sudo nano /etc/netplan/99-gestion.yaml
```

```yaml
network:
  version: 2
  ethernets:
    <NOM_INTERFACE>:
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

    !!! success "Point de contrôle"
        - [ ] La mise à jour s'est terminée (accès Internet par le *Default Switch* confirmé)
        - [ ] L'interface d'administration porte `<IP_VM>`

=== "English"

    !!! success "Checkpoint"
        - [ ] The update finished (Internet access through the *Default Switch* confirmed)
        - [ ] The administration interface carries `<IP_VM>`

### Étape 8 : Se connecter en SSH et vérifier l'intégration / Step 8: Connect over SSH and check integration

=== "Français"

    Depuis **PowerShell** sur votre poste :

=== "English"

    From **PowerShell** on your computer:

```powershell
ping <IP_VM>
ssh utilisateur@<IP_VM>
```

=== "Français"

    Dans la VM, vérifiez que les pilotes d'intégration Hyper-V sont chargés :

=== "English"

    Inside the VM, check that the Hyper-V integration drivers are loaded:

```bash
lsmod | grep hv_
```

=== "Français"

    Résultat attendu : plusieurs modules dont le nom commence par `hv_` (par exemple pour le bus, le réseau, le stockage et les services utilitaires). Ils proviennent du noyau Linux : aucune installation n'est nécessaire pour l'essentiel. Des paquets d'outils complémentaires (`linux-tools-virtual` et `linux-cloud-tools-virtual` sous Ubuntu) fournissent des démons facultatifs ; leur nom exact dépend du noyau installé.

    Depuis PowerShell, vérifiez que Hyper-V voit les services d'intégration de la VM :

=== "English"

    Expected result: several modules whose name starts with `hv_` (for example for the bus, network, storage and utility services). They come from the Linux kernel: nothing needs installing for the essentials. Additional tool packages (`linux-tools-virtual` and `linux-cloud-tools-virtual` on Ubuntu) provide optional daemons; their exact name depends on the installed kernel.

    From PowerShell, check that Hyper-V sees the VM's integration services:

```powershell
Get-VMIntegrationService -VMName "srv1-hv"
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Le `ping` aboutit et la connexion SSH fonctionne
        - [ ] `lsmod` affiche des modules `hv_`
        - [ ] Les services d'intégration apparaissent avec leur état (`Enabled`)

=== "English"

    !!! success "Checkpoint"
        - [ ] `ping` succeeds and the SSH connection works
        - [ ] `lsmod` shows `hv_` modules
        - [ ] Integration services appear with their state (`Enabled`)

### Étape 9 : Relever les caractéristiques de la VM / Step 9: Collect the VM's characteristics

```powershell
Get-VM -Name "srv1-hv" | Select-Object Name, State, Generation, ProcessorCount, MemoryStartup
Get-VHD -Path "<DOSSIER_VM>\srv1-hv\srv1-hv.vhdx" | Select-Object VhdFormat, VhdType, Size, FileSize
```

=== "Français"

    Comparez `Size` (taille déclarée) et `FileSize` (taille réelle) : `VhdType` doit valoir `Dynamic`, et le fichier est nettement plus petit que 25 Go (provisionnement dynamique, chapitre 2).

    Éteignez ensuite proprement la VM :

=== "English"

    Compare `Size` (declared size) and `FileSize` (actual size): `VhdType` should be `Dynamic`, and the file is well below 25 GB (dynamic provisioning, chapter 2).

    Then shut the VM down cleanly:

```powershell
Stop-VM -Name "srv1-hv"
```

### Étape 10 : Fiche de la VM Hyper-V / Step 10: Hyper-V VM sheet

=== "Français"

    Remplissez la fiche ci-dessous : c'est le **livrable** du lab.

    | Rubrique | Valeur |
    |---|---|
    | Édition de Windows | … |
    | `HypervisorPresent` après activation | … |
    | Commutateurs présents (noms et types) | … |
    | Nom, génération, vCPU, mémoire de la VM | … |
    | Modèle de démarrage sécurisé utilisé | … |
    | Format et type du disque, taille déclarée / réelle | … / … |
    | Interfaces de la VM et adresses | … |
    | Modules `hv_` chargés dans la VM | … |
    | Connexion SSH depuis Windows réussie | oui / non |

=== "English"

    Fill in the sheet below: this is the lab **deliverable**.

    | Item | Value |
    |---|---|
    | Windows edition | … |
    | `HypervisorPresent` after enabling | … |
    | Switches present (names and types) | … |
    | VM name, generation, vCPU, memory | … |
    | Secure Boot template used | … |
    | Disk format and type, declared / actual size | … / … |
    | VM interfaces and addresses | … |
    | `hv_` modules loaded in the VM | … |
    | SSH connection from Windows successful | yes / no |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi la VM n'a-t-elle pas pu démarrer sur l'ISO d'Ubuntu avec les réglages par défaut ?"
        Une VM de génération 2 utilise le démarrage sécurisé UEFI avec, par défaut, un modèle de certificats prévu pour Windows. Le chargeur de démarrage d'Ubuntu est signé par une autre autorité : le modèle « Microsoft UEFI Certificate Authority » l'accepte.

    ??? question "Pourquoi utiliser un commutateur interne pour l'administration plutôt que le Default Switch ?"
        Le *Default Switch* est géré par Windows : son sous-réseau peut changer d'un redémarrage à l'autre et ses réglages ne sont pas modifiables. Un commutateur interne avec adresses statiques donne un réseau d'administration stable, conforme au plan d'adressage.

    ??? question "Quelle différence y a-t-il entre `Stop-VM` et l'extinction depuis la console de la VM ?"
        `Stop-VM` demande un arrêt propre à l'invité grâce au service d'intégration d'arrêt, comme `sudo poweroff` depuis la VM. L'option `-TurnOff` couperait la VM brutalement, comme un arrachage de la prise, avec un risque de corruption.

=== "English"

    ??? question "Why could the VM not boot from the Ubuntu ISO with the default settings?"
        A generation 2 VM uses UEFI Secure Boot with, by default, a certificate template designed for Windows. Ubuntu's boot loader is signed by another authority: the "Microsoft UEFI Certificate Authority" template accepts it.

    ??? question "Why use an internal switch for administration rather than the Default Switch?"
        The *Default Switch* is managed by Windows: its subnet may change from one restart to the next and its settings cannot be changed. An internal switch with static addresses provides a stable administration network, in line with the addressing plan.

    ??? question "What is the difference between `Stop-VM` and shutting down from the VM console?"
        `Stop-VM` asks the guest for a clean shutdown through the shutdown integration service, just like `sudo poweroff` from inside the VM. The `-TurnOff` option would cut the VM abruptly, like pulling the plug, with a risk of corruption.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | `Enable-WindowsOptionalFeature` ne reconnaît pas la fonctionnalité | Édition Famille de Windows : Hyper-V n'y est pas disponible |
    | Message « l'hyperviseur n'est pas en cours d'exécution » au démarrage d'une VM | Exécuter `bcdedit /set hypervisorlaunchtype auto` puis redémarrer ; vérifier `HypervisorPresent` |
    | Les commandes Hyper-V renvoient « accès refusé » ou sont introuvables | Ouvrir PowerShell **en administrateur** (ou appartenir au groupe « Administrateurs Hyper-V ») ; le module Hyper-V doit être installé avec les outils de gestion |
    | La VM n'affiche rien ou reste sur un écran de démarrage sécurisé | Vérifier le modèle de démarrage sécurisé (étape 5) et l'ordre de démarrage (lecteur DVD en premier) |
    | `apt update` échoue | Vérifier que la première interface obtient une adresse (*Default Switch*) : `ip -br addr` |
    | `ping` vers la VM échoue depuis Windows | Vérifier l'adresse de la carte `vEthernet (gestion-interne)`, que l'adresse de la VM est dans le même réseau, et le pare-feu Windows |
    | La carte `vEthernet (gestion-interne)` n'apparaît pas | Vérifier que le commutateur a bien été créé avec le type `Internal` (`Get-VMSwitch`) |
    | VirtualBox ou VMware ne démarre plus de VM | Normal avec Hyper-V actif : performances réduites ou refus de démarrage ; voir la bascule du Lab 2.1 |

=== "English"

    | Symptom | Hints |
    |---|---|
    | `Enable-WindowsOptionalFeature` does not recognize the feature | Windows Home edition: Hyper-V is not available there |
    | Message "the hypervisor is not running" when starting a VM | Run `bcdedit /set hypervisorlaunchtype auto` then restart; check `HypervisorPresent` |
    | Hyper-V commands return "access denied" or are not found | Open PowerShell **as administrator** (or be a member of the "Hyper-V Administrators" group); the Hyper-V module must be installed with the management tools |
    | The VM shows nothing or stays on a Secure Boot screen | Check the Secure Boot template (step 5) and the boot order (DVD drive first) |
    | `apt update` fails | Check that the first interface gets an address (*Default Switch*): `ip -br addr` |
    | `ping` to the VM fails from Windows | Check the `vEthernet (gestion-interne)` adapter address, that the VM's address is in the same network, and the Windows firewall |
    | The `vEthernet (gestion-interne)` adapter does not appear | Check that the switch was created with type `Internal` (`Get-VMSwitch`) |
    | VirtualBox or VMware no longer starts VMs | Normal with Hyper-V active: reduced performance or refusal to start; see the switching procedure in Lab 2.1 |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    **Conservez la VM `srv1-hv`**, éteinte : elle sert au [Lab 4.2](lab-4-2-checkpoints-disques-et-export.md) (checkpoints, disques, export). Pour la supprimer et recommencer : `Remove-VM -Name "srv1-hv" -Force`, puis supprimez son dossier et son disque VHDX. Pour supprimer le commutateur : `Remove-VMSwitch -Name "gestion-interne"`. À la fin du chapitre, si vous devez revenir aux labs de VirtualBox ou de VMware Workstation, désactivez l'hyperviseur Windows (Lab 2.1).

=== "English"

    **Keep the `srv1-hv` VM**, powered off: it is used in [Lab 4.2](lab-4-2-checkpoints-disques-et-export.md) (checkpoints, disks, export). To delete it and start over: `Remove-VM -Name "srv1-hv" -Force`, then delete its folder and VHDX disk. To delete the switch: `Remove-VMSwitch -Name "gestion-interne"`. At the end of the chapter, if you have to return to the VirtualBox or VMware Workstation labs, turn off the Windows hypervisor (Lab 2.1).

[Retour au chapitre 4 / Back to chapter 4](cours.md)