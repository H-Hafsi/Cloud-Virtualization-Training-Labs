# Chapitre 4 : Hyper-V / Hyper-V

=== "Français"

    !!! abstract "Objectifs du chapitre"
        - Décrire l'architecture d'Hyper-V et expliquer pourquoi il est un hyperviseur de type 1 bien qu'installé dans Windows
        - Citer les prérequis et les outils de gestion d'Hyper-V
        - Choisir la génération d'une VM et le type de disque virtuel adaptés
        - Distinguer les trois types de commutateurs virtuels et le commutateur par défaut
        - Expliquer les checkpoints standard et de production, l'export et l'import
        - Situer Hyper-V par rapport à VirtualBox et à VMware Workstation
        - Acquis d'apprentissage visé : **AA2**

        Matériel requis pour ce cours : aucun. Les labs associés indiquent leurs propres ressources.

=== "English"

    !!! abstract "Chapter objectives"
        - Describe the Hyper-V architecture and explain why it is a type 1 hypervisor although installed inside Windows
        - List the prerequisites and management tools of Hyper-V
        - Choose the suitable VM generation and virtual disk type
        - Tell the three virtual switch types and the default switch apart
        - Explain standard and production checkpoints, export and import
        - Place Hyper-V relative to VirtualBox and VMware Workstation
        - Target learning outcome: **AA2**

        Required equipment for this lesson: none. The associated labs state their own resources.

## 1. Architecture et prérequis / Architecture and prerequisites

=== "Français"

    **Hyper-V** est l'hyperviseur de Microsoft. Il est fourni avec Windows Server (comme **rôle**) et avec les éditions **Pro, Enterprise et Éducation** de Windows 10 et 11 (comme **fonctionnalité** à activer). Il n'existe pas en édition Famille.

    Au chapitre 2, nous avons vu que les hyperviseurs de type 2 s'exécutent comme des applications. Hyper-V est différent : quand on l'active, l'hyperviseur **démarre avant Windows** et prend le contrôle du matériel. Windows lui-même s'exécute alors dans une partition privilégiée, la **partition racine**, et les machines virtuelles dans des **partitions enfants**.

    ```mermaid
    flowchart TB
      subgraph P["Partitions"]
        direction LR
        R["Partition racine<br/>(Windows hôte + outils de gestion)"]
        C1["Partition enfant<br/>VM 1"]
        C2["Partition enfant<br/>VM 2"]
      end
      P --> H["Hyperviseur Hyper-V"]
      H --> M["Matériel"]
    ```

    C'est pourquoi on classe Hyper-V parmi les hyperviseurs de **type 1**, même sur un poste de travail, et pourquoi il peut entrer en conflit avec les hyperviseurs de type 2 (voir le [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md)).

    | Prérequis | Détail |
    |---|---|
    | **Édition de Windows** | Pro, Enterprise ou Éducation (client) ; Windows Server |
    | **Processeur** | 64 bits, avec traduction d'adresses de second niveau (SLAT) et virtualisation activée dans le micrologiciel |
    | **Mémoire et disque** | Selon votre profil de poste : voir l'annexe [Environnement](../../annexes/environnement.md) |

    ### Les outils de gestion

    | Outil | Usage |
    |---|---|
    | **Gestionnaire Hyper-V** | Interface graphique (`virtmgmt.msc`) : créer, configurer et exploiter les VM |
    | **PowerShell, module Hyper-V** | Automatisation, avec des commandes comme `New-VM`, `Get-VM`, `Start-VM`, `Checkpoint-VM`, `New-VMSwitch`, `Export-VM`, `Import-VM` |

=== "English"

    **Hyper-V** is Microsoft's hypervisor. It ships with Windows Server (as a **role**) and with the **Pro, Enterprise and Education** editions of Windows 10 and 11 (as a **feature** to enable). It is not available in Home edition.

    In chapter 2 we saw that type 2 hypervisors run as applications. Hyper-V is different: when enabled, the hypervisor **starts before Windows** and takes control of the hardware. Windows itself then runs in a privileged partition, the **root partition**, and virtual machines in **child partitions**.

    ```mermaid
    flowchart TB
      subgraph P["Partitions"]
        direction LR
        R["Root partition<br/>(host Windows + management tools)"]
        C1["Child partition<br/>VM 1"]
        C2["Child partition<br/>VM 2"]
      end
      P --> H["Hyper-V hypervisor"]
      H --> M["Hardware"]
    ```

    This is why Hyper-V is classed as a **type 1** hypervisor, even on a workstation, and why it can conflict with type 2 hypervisors (see [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md)).

    | Prerequisite | Detail |
    |---|---|
    | **Windows edition** | Pro, Enterprise or Education (client); Windows Server |
    | **Processor** | 64-bit, with second-level address translation (SLAT) and virtualization enabled in the firmware |
    | **Memory and disk** | According to your computer profile: see the [Environment](../../annexes/environnement.md) annex |

    ### Management tools

    | Tool | Use |
    |---|---|
    | **Hyper-V Manager** | Graphical interface (`virtmgmt.msc`): create, configure and operate VMs |
    | **PowerShell, Hyper-V module** | Automation, with commands such as `New-VM`, `Get-VM`, `Start-VM`, `Checkpoint-VM`, `New-VMSwitch`, `Export-VM`, `Import-VM` |

## 2. La machine virtuelle : génération, mémoire, disques / The virtual machine: generation, memory, disks

=== "Français"

    ### Génération 1 ou génération 2

    | | Génération 1 | Génération 2 |
    |---|---|---|
    | **Micrologiciel** | BIOS | UEFI |
    | **Matériel** | Périphériques hérités émulés (par exemple contrôleur IDE) | Périphériques synthétiques, plus rapides (contrôleur SCSI) |
    | **Démarrage sécurisé (Secure Boot)** | Non | Oui |
    | **Systèmes invités** | Anciens systèmes 32 bits et 64 bits | Systèmes 64 bits récents |

    Pour Ubuntu Server 64 bits, la **génération 2** est le choix usuel. Attention : par défaut, le démarrage sécurisé d'une VM de génération 2 utilise un modèle de certificats prévu pour Windows, qui **empêche le démarrage de Linux**. Il faut soit choisir le modèle de certificats « Microsoft UEFI Certificate Authority », soit désactiver le démarrage sécurisé pour cette VM (réglage vu au Lab 4.1).

    ### Mémoire

    - **Mémoire statique** : la VM reçoit une quantité fixe de RAM.
    - **Mémoire dynamique** : on fixe une mémoire de démarrage, un minimum et un maximum ; Hyper-V ajuste selon les besoins. Elle permet de surallouer (chapitre 2), avec les risques qui y sont associés.

    ### Disques virtuels

    | Format | Caractéristiques |
    |---|---|
    | **VHD** | Format ancien, taille limitée (2 To) |
    | **VHDX** | Format recommandé : plus grande taille (jusqu'à 64 To), plus résistant aux coupures |

    | Type de disque | Principe |
    |---|---|
    | **Taille fixe** | Tout l'espace est réservé à la création (*thick*) |
    | **Taille dynamique** | Le fichier grandit avec les données écrites (*thin*) |
    | **Différentiel** | Disque « enfant » qui ne stocke que les différences par rapport à un disque parent ; il dépend de ce parent (principe du clone lié) |

    ### Services d'intégration

    Les **services d'intégration** améliorent la coopération entre l'hôte et l'invité : synchronisation de l'heure, arrêt propre, signal de vie (*heartbeat*), échange de données, sauvegarde cohérente. Les pilotes sont intégrés au noyau Linux ; des paquets d'outils complémentaires existent pour les démons correspondants.

=== "English"

    ### Generation 1 or generation 2

    | | Generation 1 | Generation 2 |
    |---|---|---|
    | **Firmware** | BIOS | UEFI |
    | **Hardware** | Emulated legacy devices (for example IDE controller) | Synthetic, faster devices (SCSI controller) |
    | **Secure Boot** | No | Yes |
    | **Guest systems** | Older 32-bit and 64-bit systems | Recent 64-bit systems |

    For 64-bit Ubuntu Server, **generation 2** is the usual choice. Beware: by default, Secure Boot on a generation 2 VM uses a certificate template designed for Windows, which **prevents Linux from booting**. Either choose the "Microsoft UEFI Certificate Authority" template, or disable Secure Boot for that VM (setting covered in Lab 4.1).

    ### Memory

    - **Static memory**: the VM gets a fixed amount of RAM.
    - **Dynamic memory**: you set a startup memory, a minimum and a maximum; Hyper-V adjusts as needed. It allows overcommit (chapter 2), with the associated risks.

    ### Virtual disks

    | Format | Characteristics |
    |---|---|
    | **VHD** | Legacy format, limited size (2 TB) |
    | **VHDX** | Recommended format: larger size (up to 64 TB), more resilient to power loss |

    | Disk type | Principle |
    |---|---|
    | **Fixed size** | All the space is reserved at creation (*thick*) |
    | **Dynamically expanding** | The file grows with the data written (*thin*) |
    | **Differencing** | "Child" disk that stores only the differences from a parent disk; it depends on that parent (linked clone principle) |

    ### Integration services

    **Integration services** improve cooperation between host and guest: time synchronization, clean shutdown, heartbeat, data exchange, consistent backup. The drivers are built into the Linux kernel; additional tool packages exist for the corresponding daemons.

## 3. Les commutateurs virtuels / Virtual switches

=== "Français"

    Une VM Hyper-V se connecte au réseau par un **commutateur virtuel**. Il en existe trois types :

    | Type | Les VM peuvent communiquer avec… | Usage typique |
    |---|---|---|
    | **Externe** | Les autres VM, l'hôte et le **réseau physique** (le commutateur est lié à une carte réseau de l'hôte) | Accès au réseau local et à Internet |
    | **Interne** | Les autres VM et l'**hôte**, mais pas le réseau physique | Administration depuis l'hôte, labs isolés |
    | **Privé** | **Uniquement** les autres VM du même commutateur | Réseaux totalement isolés entre VM |

    Lorsque Hyper-V est activé sous Windows 10 ou 11, un commutateur nommé **Default Switch** est créé automatiquement. Il fournit un accès à Internet par **traduction d'adresses (NAT)** et une adresse par DHCP, mais ses réglages ne sont pas modifiables et le sous-réseau peut changer d'un redémarrage à l'autre : il convient aux essais, pas à un plan d'adressage fixe.

    Les commutateurs Hyper-V correspondent aux modes vus au chapitre 3 : l'externe joue le rôle d'un **pont**, l'interne celui d'un **hôte seul**, le privé celui d'un **réseau interne**. Le chapitre 5 approfondit les réseaux virtuels.

=== "English"

    A Hyper-V VM connects to the network through a **virtual switch**. There are three types:

    | Type | VMs can communicate with... | Typical use |
    |---|---|---|
    | **External** | Other VMs, the host and the **physical network** (the switch is bound to a host network card) | Access to the local network and the Internet |
    | **Internal** | Other VMs and the **host**, but not the physical network | Administration from the host, isolated labs |
    | **Private** | **Only** the other VMs on the same switch | Completely isolated networks between VMs |

    When Hyper-V is enabled on Windows 10 or 11, a switch named **Default Switch** is created automatically. It provides Internet access through **network address translation (NAT)** and an address by DHCP, but its settings cannot be changed and the subnet may change from one restart to the next: it suits quick tests, not a fixed addressing plan.

    Hyper-V switches match the modes seen in chapter 3: external plays the role of a **bridge**, internal that of **host-only**, private that of an **internal network**. Chapter 5 goes deeper into virtual networks.

## 4. Checkpoints, export et import / Checkpoints, export and import

=== "Français"

    ### Checkpoints

    Un **checkpoint** est le nom Hyper-V du snapshot (chapitre 2) : un point de restauration d'une VM.

    | Type | Contenu | Remarque |
    |---|---|---|
    | **De production** (par défaut) | État du disque capturé avec les mécanismes de sauvegarde cohérente du système invité ; **sans la mémoire** | Restauration équivalente à un démarrage propre ; recommandé |
    | **Standard** | État du disque **et de la mémoire** | Restaure la VM « en fonctionnement » ; plus lourd, déconseillé pour des applications transactionnelles |

    Techniquement, un checkpoint crée un disque différentiel (fichier `.avhdx`) qui reçoit les écritures suivantes. Comme tout snapshot, un checkpoint **n'est pas une sauvegarde**, et une longue chaîne de checkpoints ralentit la VM et consomme de l'espace.

    ### Export et import

    | Opération | Résultat |
    |---|---|
    | **Export** (`Export-VM`) | Copie complète de la VM (configuration, disques, checkpoints) dans un dossier |
    | **Import** (`Import-VM`) | Recrée la VM à partir d'un export. Trois modes : **enregistrer sur place** (utilise les fichiers là où ils sont), **restaurer** (copie en conservant l'identifiant unique), **copier** (copie avec un **nouvel** identifiant unique) |

    Pour dupliquer une VM sur le même hôte, il faut choisir le mode **copier** : deux VM de même identifiant ne peuvent coexister. Après une copie, **changez le nom d'hôte et l'adresse réseau** de l'invité, comme pour un clone (chapitre 3).

=== "English"

    ### Checkpoints

    A **checkpoint** is Hyper-V's name for a snapshot (chapter 2): a VM restore point.

    | Type | Content | Note |
    |---|---|---|
    | **Production** (default) | Disk state captured using the guest system's consistent-backup mechanisms; **without memory** | Restoring is equivalent to a clean boot; recommended |
    | **Standard** | Disk **and memory** state | Restores the VM "as running"; heavier, not advised for transactional applications |

    Technically, a checkpoint creates a differencing disk (`.avhdx` file) that receives subsequent writes. Like any snapshot, a checkpoint **is not a backup**, and a long checkpoint chain slows the VM and uses space.

    ### Export and import

    | Operation | Result |
    |---|---|
    | **Export** (`Export-VM`) | Full copy of the VM (configuration, disks, checkpoints) into a folder |
    | **Import** (`Import-VM`) | Recreates the VM from an export. Three modes: **register in place** (uses the files where they are), **restore** (copies while keeping the unique identifier), **copy** (copies with a **new** unique identifier) |

    To duplicate a VM on the same host, the **copy** mode must be chosen: two VMs with the same identifier cannot coexist. After a copy, **change the guest's hostname and network address**, as for a clone (chapter 3).

## 5. Hyper-V dans ce cours / Hyper-V in this course

=== "Français"

    | Critère | Hyper-V | VirtualBox / VMware Workstation |
    |---|---|---|
    | **Type** | Type 1 (hyperviseur sous Windows) | Type 2 |
    | **Coût / licence** | Inclus dans les éditions Pro, Enterprise, Éducation | Voir le chapitre 3 |
    | **Interface** | Gestionnaire Hyper-V et PowerShell | Interface propre à chaque outil |
    | **Formats de disque** | `.vhdx` (`.vhd`) | `.vdi` / `.vmdk` |
    | **Concurrence avec d'autres hyperviseurs** | Hyper-V actif : les autres passent par lui avec moins de performances | Meilleures performances quand l'hyperviseur Windows est désactivé |

    **Règle de travail du cours :** l'hyperviseur Windows est **activé** pour les labs Hyper-V (chapitres 4 et 5, labs sur les commutateurs Hyper-V) et **désactivé** pour les labs de VirtualBox et de VMware Workstation et pour ceux des Parties 3 et 4. La bascule se fait comme au [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) et demande un **redémarrage**.

    !!! tip "À retenir"
        - Hyper-V est un hyperviseur de type 1 : il démarre avant Windows, qui devient la partition racine.
        - Disponible en éditions Pro, Enterprise et Éducation ; absent en édition Famille.
        - Génération 2 (UEFI, SCSI) pour les systèmes 64 bits récents ; pour Linux, adapter le démarrage sécurisé.
        - Disques : VHDX recommandé ; fixe, dynamique ou différentiel.
        - Commutateurs : externe (réseau physique), interne (hôte), privé (VM seules) ; Default Switch pour des essais.
        - Checkpoint de production (sans mémoire, par défaut) ou standard (avec mémoire) ; ce n'est pas une sauvegarde.
        - Export / import : choisir le mode « copier » pour dupliquer sur le même hôte.
        - Bascule de l'hyperviseur Windows selon le chapitre : activé pour Hyper-V, désactivé pour les autres labs lourds.

=== "English"

    | Criterion | Hyper-V | VirtualBox / VMware Workstation |
    |---|---|---|
    | **Type** | Type 1 (hypervisor beneath Windows) | Type 2 |
    | **Cost / license** | Included in Pro, Enterprise, Education editions | See chapter 3 |
    | **Interface** | Hyper-V Manager and PowerShell | Each tool's own interface |
    | **Disk formats** | `.vhdx` (`.vhd`) | `.vdi` / `.vmdk` |
    | **Competition with other hypervisors** | Hyper-V on: the others go through it with lower performance | Better performance when the Windows hypervisor is off |

    **Course working rule:** the Windows hypervisor is **on** for the Hyper-V labs (chapters 4 and 5, Hyper-V switch labs) and **off** for the VirtualBox and VMware Workstation labs and for those of Parts 3 and 4. Switching is done as in [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) and requires a **restart**.

    !!! tip "Key takeaways"
        - Hyper-V is a type 1 hypervisor: it starts before Windows, which becomes the root partition.
        - Available in Pro, Enterprise and Education editions; absent in Home edition.
        - Generation 2 (UEFI, SCSI) for recent 64-bit systems; for Linux, adapt Secure Boot.
        - Disks: VHDX recommended; fixed, dynamic or differencing.
        - Switches: external (physical network), internal (host), private (VMs only); Default Switch for quick tests.
        - Production checkpoint (no memory, default) or standard (with memory); not a backup.
        - Export / import: choose the "copy" mode to duplicate on the same host.
        - Windows hypervisor switching depends on the chapter: on for Hyper-V, off for the other heavy labs.

## Pour vérifier votre compréhension / Check your understanding

=== "Français"

    ??? question "Pourquoi Hyper-V est-il considéré comme un hyperviseur de type 1 alors qu'on l'active depuis Windows ?"
        Une fois activé, l'hyperviseur démarre **avant** Windows et contrôle directement le matériel. Windows devient la partition racine, une partition privilégiée parmi d'autres : il n'est plus au contact direct du matériel, contrairement à un hyperviseur de type 2 qui s'exécute comme une simple application.

    ??? question "Une VM Ubuntu Server de génération 2 ne démarre pas sur l'ISO. Quelle est la cause probable et comment la corriger ?"
        Le démarrage sécurisé (Secure Boot) utilise par défaut un modèle de certificats prévu pour Windows, qui refuse le chargeur de démarrage d'Ubuntu. On choisit le modèle « Microsoft UEFI Certificate Authority » ou on désactive le démarrage sécurisé pour cette VM.

    ??? question "Vous voulez que plusieurs VM communiquent entre elles et avec votre poste, sans accès au réseau de l'établissement. Quel type de commutateur choisissez-vous ?"
        Un commutateur **interne** : il relie les VM et l'hôte sans passer par le réseau physique. Un commutateur privé isolerait aussi l'hôte, et un commutateur externe donnerait accès au réseau physique.

=== "English"

    ??? question "Why is Hyper-V considered a type 1 hypervisor although you enable it from Windows?"
        Once enabled, the hypervisor starts **before** Windows and directly controls the hardware. Windows becomes the root partition, a privileged partition among others: it is no longer in direct contact with the hardware, unlike a type 2 hypervisor that runs as a simple application.

    ??? question "A generation 2 Ubuntu Server VM does not boot from the ISO. What is the probable cause and how do you fix it?"
        Secure Boot uses by default a certificate template designed for Windows, which rejects Ubuntu's boot loader. Choose the "Microsoft UEFI Certificate Authority" template or disable Secure Boot for that VM.

    ??? question "You want several VMs to communicate with each other and with your computer, without access to the institution's network. Which switch type do you choose?"
        An **internal** switch: it connects the VMs and the host without going through the physical network. A private switch would also isolate the host, and an external switch would give access to the physical network.

## Ressources du chapitre / Chapter resources

- [Lab 4.1 : Activer Hyper-V et créer une VM / Enabling Hyper-V and creating a VM](lab-4-1-activer-hyper-v-et-creer-une-vm.md)
- [Lab 4.2 : Checkpoints, disques et export / Checkpoints, disks and export](lab-4-2-checkpoints-disques-et-export.md)
- [Quiz du chapitre / Chapter quiz](quiz.md)
- [Glossaire / Glossary](../../annexes/glossaire.md)
- [Environnement de TP / Lab environment](../../annexes/environnement.md)