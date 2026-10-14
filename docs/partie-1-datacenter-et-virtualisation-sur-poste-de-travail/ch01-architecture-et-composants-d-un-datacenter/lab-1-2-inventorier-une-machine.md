# Lab 1.2 : Inventorier une machine / Inventorying a Machine

| | |
|---|---|
| **Durée / Duration** | 45 à 60 min / 45 to 60 min |
| **Niveau / Level** | Guidé / Guided |
| **Prérequis / Prerequisites** | [Chapitre 1 : cours](cours.md) ; [Lab 1.1](lab-1-1-dimensionner-une-salle-serveur.md) |
| **Ressources minimales / Minimum resources** | Aucune machine virtuelle. Votre poste Windows 10 ou 11 (PowerShell, déjà installé). Aucun droit administrateur requis en principe. / No virtual machine. Your Windows 10 or 11 computer (PowerShell, already installed). No administrator rights normally required. |
| **Acquis visés / Learning outcomes** | AA1 |

=== "Français"

    !!! abstract "Objectifs"
        - Identifier les composants matériels d'une machine : processeur, mémoire, stockage, réseau, micrologiciel
        - Relever ces informations avec des commandes PowerShell et avec les outils graphiques de Windows
        - Rédiger une fiche d'inventaire exploitable
        - Comparer votre poste à un serveur de datacenter

=== "English"

    !!! abstract "Objectives"
        - Identify the hardware components of a machine: processor, memory, storage, network, firmware
        - Collect this information with PowerShell commands and with Windows graphical tools
        - Write a usable inventory sheet
        - Compare your computer with a datacenter server

## Contexte et schéma / Context and diagram

=== "Français"

    Avant de virtualiser quoi que ce soit, il faut savoir **de quoi l'on dispose**. Dans ce lab, vous inventoriez **votre propre poste**, qui servira d'hôte pour les machines virtuelles des chapitres suivants. Les composants que vous allez relever sont ceux d'un serveur du cours (section 5 du chapitre 1), à une autre échelle.

    ```mermaid
    flowchart TB
      M["Machine"] --> F["Micrologiciel (BIOS / UEFI)"]
      M --> C["Processeur"]
      M --> R["Mémoire"]
      M --> S["Stockage"]
      M --> N["Réseau"]
      M --> O["Système d'exploitation"]
    ```

=== "English"

    Before virtualizing anything, you need to know **what you have**. In this lab you inventory **your own computer**, which will serve as the host for the virtual machines in the following chapters. The components you will collect are those of a course server (section 5 of chapter 1), at a different scale.

    ```mermaid
    flowchart TB
      M["Machine"] --> F["Firmware (BIOS / UEFI)"]
      M --> C["Processor"]
      M --> R["Memory"]
      M --> S["Storage"]
      M --> N["Network"]
      M --> O["Operating system"]
    ```

## Étapes / Steps

### Étape 1 : Ouvrir PowerShell et enregistrer la session / Step 1: Open PowerShell and record the session

=== "Français"

    Ouvrez **PowerShell** (menu Démarrer, tapez `PowerShell`). Lancez l'enregistrement de la session : tout ce qui s'affiche sera conservé dans un fichier, que vous utiliserez pour rédiger votre fiche.

=== "English"

    Open **PowerShell** (Start menu, type `PowerShell`). Start recording the session: everything displayed will be saved to a file that you will use to write your sheet.

```powershell
Start-Transcript -Path "$HOME\inventaire.txt"
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] PowerShell affiche un message indiquant que l'enregistrement a démarré

=== "English"

    !!! success "Checkpoint"
        - [ ] PowerShell displays a message saying the transcript has started

### Étape 2 : Identité de la machine et système / Step 2: Machine identity and system

=== "Français"

    Relevez le constructeur, le modèle, le système d'exploitation et le micrologiciel (BIOS ou UEFI).

=== "English"

    Collect the manufacturer, model, operating system and firmware (BIOS or UEFI).

```powershell
Get-CimInstance Win32_ComputerSystem | Select-Object Manufacturer, Model
Get-CimInstance Win32_OperatingSystem | Select-Object Caption, Version, OSArchitecture
Get-CimInstance Win32_BIOS | Select-Object Manufacturer, SMBIOSBIOSVersion, ReleaseDate
```

=== "Français"

    Résultat attendu : trois petits tableaux (constructeur et modèle ; nom, version et architecture du système ; fabricant, version et date du BIOS).

    !!! success "Point de contrôle"
        - [ ] Vous avez noté le modèle de la machine et la version du système
        - [ ] Vous avez noté la version et la date du micrologiciel

=== "English"

    Expected result: three small tables (manufacturer and model; OS name, version and architecture; BIOS vendor, version and date).

    !!! success "Checkpoint"
        - [ ] You noted the machine model and the OS version
        - [ ] You noted the firmware version and date

### Étape 3 : Processeur / Step 3: Processor

=== "Français"

    Relevez le nom du processeur, son nombre de **cœurs physiques**, son nombre de **processeurs logiques** et sa fréquence maximale. Le champ `VirtualizationFirmwareEnabled` indique si la virtualisation matérielle est activée dans le micrologiciel : notez-le, il sera expliqué au chapitre 2.

=== "English"

    Collect the processor name, its number of **physical cores**, its number of **logical processors** and its maximum frequency. The `VirtualizationFirmwareEnabled` field tells whether hardware virtualization is enabled in the firmware: note it, it will be explained in chapter 2.

```powershell
Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores, NumberOfLogicalProcessors, MaxClockSpeed, VirtualizationFirmwareEnabled
```

=== "Français"

    Résultat attendu : une ligne par processeur physique installé, avec les cinq champs demandés (`MaxClockSpeed` est en MHz).

    !!! success "Point de contrôle"
        - [ ] Vous avez le nombre de cœurs et le nombre de processeurs logiques
        - [ ] Vous avez noté la valeur de `VirtualizationFirmwareEnabled` (`True` ou `False`)

=== "English"

    Expected result: one row per installed physical processor, with the five requested fields (`MaxClockSpeed` is in MHz).

    !!! success "Checkpoint"
        - [ ] You have the number of cores and the number of logical processors
        - [ ] You noted the value of `VirtualizationFirmwareEnabled` (`True` or `False`)

### Étape 4 : Mémoire / Step 4: Memory

=== "Français"

    Relevez la quantité totale de mémoire (en Go), puis le détail des barrettes installées.

=== "English"

    Collect the total amount of memory (in GB), then the detail of the installed modules.

```powershell
[math]::Round((Get-CimInstance Win32_ComputerSystem).TotalPhysicalMemory / 1GB, 1)
Get-CimInstance Win32_PhysicalMemory | Select-Object BankLabel, Capacity, Speed, Manufacturer
```

=== "Français"

    Résultat attendu : un nombre (la RAM en Go), puis une ligne par barrette. `Capacity` est en octets : divisez par 1 073 741 824 pour obtenir des Go, et `Speed` est en MHz.

    !!! success "Point de contrôle"
        - [ ] Vous connaissez la RAM totale en Go
        - [ ] Vous avez le nombre de barrettes et la capacité de chacune

=== "English"

    Expected result: one number (RAM in GB), then one row per module. `Capacity` is in bytes: divide by 1,073,741,824 to get GB, and `Speed` is in MHz.

    !!! success "Checkpoint"
        - [ ] You know the total RAM in GB
        - [ ] You have the number of modules and the capacity of each

### Étape 5 : Stockage / Step 5: Storage

=== "Français"

    Relevez les disques physiques (type, interface, taille, état de santé), puis les volumes (lettre, système de fichiers, taille et espace libre).

=== "English"

    Collect the physical disks (type, interface, size, health status), then the volumes (letter, file system, size and free space).

```powershell
Get-PhysicalDisk | Select-Object FriendlyName, MediaType, BusType, Size, HealthStatus
Get-Volume | Where-Object DriveLetter | Select-Object DriveLetter, FileSystem, Size, SizeRemaining
```

=== "Français"

    Résultat attendu : un tableau des disques (`MediaType` indique SSD ou HDD, `BusType` l'interface, par exemple NVMe ou SATA) et un tableau des volumes. Les tailles sont en octets.

    !!! success "Point de contrôle"
        - [ ] Vous savez si votre stockage est de type SSD ou HDD
        - [ ] Vous connaissez l'espace libre en Go (utile pour dimensionner vos futures VM)

=== "English"

    Expected result: a disk table (`MediaType` indicates SSD or HDD, `BusType` the interface, for example NVMe or SATA) and a volume table. Sizes are in bytes.

    !!! success "Checkpoint"
        - [ ] You know whether your storage is SSD or HDD
        - [ ] You know the free space in GB (useful for sizing your future VMs)

### Étape 6 : Réseau / Step 6: Network

=== "Français"

    Relevez les cartes réseau actives, leur vitesse de liaison et leur adresse IPv4.

=== "English"

    Collect the active network cards, their link speed and their IPv4 address.

```powershell
Get-NetAdapter | Where-Object Status -eq 'Up' | Select-Object Name, InterfaceDescription, LinkSpeed, MacAddress
Get-NetIPAddress -AddressFamily IPv4 | Select-Object InterfaceAlias, IPAddress, PrefixLength
```

=== "Français"

    Résultat attendu : la liste des cartes à l'état `Up` (connectées), puis les adresses IPv4 associées à chaque interface.

    !!! success "Point de contrôle"
        - [ ] Vous avez identifié la carte qui vous donne accès à Internet et son adresse IP

=== "English"

    Expected result: the list of cards in the `Up` state (connected), then the IPv4 addresses attached to each interface.

    !!! success "Checkpoint"
        - [ ] You identified the card that gives you Internet access and its IP address

### Étape 7 : Retrouver les mêmes informations avec les outils graphiques / Step 7: Find the same information with graphical tools

=== "Français"

    1. Ouvrez le **Gestionnaire des tâches** (++ctrl+shift+esc++) puis l'onglet **Performance** : retrouvez le processeur, la mémoire, le disque et le réseau.
    2. Ouvrez **Informations système** (touche Windows, tapez `msinfo32`) : retrouvez le constructeur, le modèle et la version du BIOS.

    Comparez avec ce que PowerShell vous a donné. Les valeurs doivent concorder.

=== "English"

    1. Open **Task Manager** (++ctrl+shift+esc++) then the **Performance** tab: find the processor, memory, disk and network.
    2. Open **System Information** (Windows key, type `msinfo32`): find the manufacturer, model and BIOS version.

    Compare with what PowerShell gave you. The values should match.

### Étape 8 : Terminer l'enregistrement / Step 8: Stop the recording

```powershell
Stop-Transcript
```

=== "Français"

    Le fichier `inventaire.txt` se trouve dans votre dossier personnel. Ouvrez-le : c'est la matière première de votre fiche.

=== "English"

    The `inventaire.txt` file is in your home folder. Open it: it is the raw material for your sheet.

### Étape 9 : Rédiger la fiche d'inventaire / Step 9: Write the inventory sheet

=== "Français"

    Remplissez la fiche ci-dessous : c'est le **livrable** du lab.

    | Rubrique | Valeur relevée |
    |---|---|
    | Constructeur et modèle | … |
    | Système d'exploitation (nom, version) | … |
    | Micrologiciel (version, date) | … |
    | Processeur (nom) | … |
    | Cœurs physiques / processeurs logiques | … / … |
    | Virtualisation activée dans le micrologiciel | … |
    | Mémoire totale (Go) et nombre de barrettes | … |
    | Stockage : type (SSD/HDD), interface, taille (Go) | … |
    | Espace libre sur le disque système (Go) | … |
    | Carte réseau active, vitesse de liaison, adresse IPv4 | … |

    **Comparaison avec le Lab 1.1.** Recherchez sur la fiche du constructeur la puissance thermique nominale (**TDP**) de votre processeur. Estimez ensuite la consommation maximale de votre machine en ajoutant une marge pour le reste des composants, et comparez-la aux 450 W d'un serveur de calcul du Lab 1.1. Notez votre estimation et les hypothèses que vous avez faites.

=== "English"

    Fill in the sheet below: this is the lab **deliverable**.

    | Item | Collected value |
    |---|---|
    | Manufacturer and model | … |
    | Operating system (name, version) | … |
    | Firmware (version, date) | … |
    | Processor (name) | … |
    | Physical cores / logical processors | … / … |
    | Virtualization enabled in firmware | … |
    | Total memory (GB) and number of modules | … |
    | Storage: type (SSD/HDD), interface, size (GB) | … |
    | Free space on the system disk (GB) | … |
    | Active network card, link speed, IPv4 address | … |

    **Comparison with Lab 1.1.** Look up the nominal thermal power (**TDP**) of your processor on the manufacturer's datasheet. Then estimate your machine's maximum consumption by adding a margin for the other components, and compare it with the 450 W of a compute server in Lab 1.1. Write down your estimate and the assumptions you made.

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi le nombre de processeurs logiques peut-il être supérieur au nombre de cœurs physiques ?"
        Certains processeurs exécutent deux fils d'exécution par cœur (multithreading simultané). Le système voit alors deux processeurs logiques pour un cœur physique, sans que la puissance de calcul soit pour autant doublée.

    ??? question "Citez trois différences entre votre poste et un serveur de datacenter."
        Par exemple : alimentations redondantes ; contrôleur de gestion à distance (BMC) ; mémoire avec correction d'erreurs ; format adapté à la baie ; refroidissement et fonctionnement continu prévus pour 24 h/24 ; extensibilité (nombre de barrettes, de disques et de cartes réseau).

    ??? question "Pourquoi conserver un inventaire à jour des machines d'une infrastructure ?"
        Pour planifier la capacité (ressources disponibles), préparer les maintenances et les remplacements, suivre les versions de micrologiciels et diagnostiquer plus vite en cas de panne.

=== "English"

    ??? question "Why can the number of logical processors be higher than the number of physical cores?"
        Some processors run two threads per core (simultaneous multithreading). The system then sees two logical processors for one physical core, without the computing power being doubled.

    ??? question "Name three differences between your computer and a datacenter server."
        For example: redundant power supplies; remote management controller (BMC); error-correcting memory; rack-friendly form factor; cooling and continuous operation designed for 24/7; expandability (number of memory modules, disks and network cards).

    ??? question "Why keep an up-to-date inventory of an infrastructure's machines?"
        To plan capacity (available resources), prepare maintenance and replacements, track firmware versions and diagnose faster when a failure occurs.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | `Get-PhysicalDisk` ou une autre commande renvoie « Accès refusé » | Fermer PowerShell et le relancer **en administrateur** (clic droit, « Exécuter en tant qu'administrateur ») |
    | Une commande n'est pas reconnue | Vérifier l'orthographe ; utiliser PowerShell (et non l'invite de commandes `cmd`) |
    | `VirtualizationFirmwareEnabled` renvoie `False` ou rien | Noter la valeur telle quelle ; l'activation dans le BIOS est traitée au [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) |
    | Tailles illisibles (nombres très grands) | Les valeurs sont en octets : diviser par `1GB` dans PowerShell, par exemple `$x / 1GB` |
    | Le fichier `inventaire.txt` est introuvable | Il se trouve dans votre dossier personnel (`$HOME`) ; relancer `Start-Transcript` si l'enregistrement n'a pas démarré |

=== "English"

    | Symptom | Hints |
    |---|---|
    | `Get-PhysicalDisk` or another command returns "Access denied" | Close PowerShell and reopen it **as administrator** (right-click, "Run as administrator") |
    | A command is not recognized | Check the spelling; use PowerShell (not the `cmd` command prompt) |
    | `VirtualizationFirmwareEnabled` returns `False` or nothing | Note the value as is; enabling it in the BIOS is covered in [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) |
    | Unreadable sizes (very large numbers) | Values are in bytes: divide by `1GB` in PowerShell, for example `$x / 1GB` |
    | The `inventaire.txt` file cannot be found | It is in your home folder (`$HOME`); rerun `Start-Transcript` if recording did not start |

## Clôture / Wrap-up

=== "Français"

    Ce lab ne modifie rien sur votre machine (il ne fait que lire des informations) : il n'y a rien à nettoyer. **Conservez votre fiche d'inventaire** : au chapitre 2, vous la réutiliserez pour vérifier que votre poste peut héberger des machines virtuelles.

=== "English"

    This lab changes nothing on your machine (it only reads information): there is nothing to clean up. **Keep your inventory sheet**: in chapter 2 you will reuse it to check that your computer can host virtual machines.

[Retour au chapitre 1 / Back to chapter 1](cours.md)