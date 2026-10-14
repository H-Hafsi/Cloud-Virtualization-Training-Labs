# Chapitre 2 : Principes de la virtualisation / Principles of Virtualization

=== "Français"

    !!! abstract "Objectifs du chapitre"
        - Expliquer pourquoi et comment on virtualise des ressources informatiques
        - Distinguer émulation, virtualisation complète, paravirtualisation et assistance matérielle
        - Comparer les hyperviseurs de type 1 et de type 2
        - Décrire les composants et les fichiers d'une machine virtuelle
        - Expliquer la surallocation, le *thin provisioning*, les snapshots et les clones
        - Comparer une machine virtuelle et un conteneur
        - Acquis d'apprentissage visé : **AA1**

        Matériel requis pour ce cours : aucun. Le [Lab 2.1](lab-2-1-preparer-le-poste-pour-la-virtualisation.md) prépare votre poste pour les chapitres suivants.

=== "English"

    !!! abstract "Chapter objectives"
        - Explain why and how IT resources are virtualized
        - Tell emulation, full virtualization, paravirtualization and hardware assistance apart
        - Compare type 1 and type 2 hypervisors
        - Describe the components and files of a virtual machine
        - Explain overcommit, thin provisioning, snapshots and clones
        - Compare a virtual machine with a container
        - Target learning outcome: **AA1**

        Required equipment for this lesson: none. [Lab 2.1](lab-2-1-preparer-le-poste-pour-la-virtualisation.md) prepares your computer for the following chapters.

## 1. Pourquoi virtualiser ? / Why virtualize?

=== "Français"

    Au chapitre 1, nous avons vu qu'un serveur physique consomme de l'énergie, occupe de la place et exige du refroidissement en permanence, même lorsqu'il est peu utilisé. La **virtualisation** consiste à créer, par logiciel, des ressources informatiques qui se comportent comme des ressources physiques : plusieurs **machines virtuelles** (VM) isolées partagent ainsi le même matériel.

    | Avantage | Explication |
    |---|---|
    | **Consolidation** | Plusieurs charges de travail peu actives cohabitent sur une seule machine physique : moins de serveurs, moins d'énergie, moins de baies |
    | **Isolation** | Chaque VM possède son propre système ; une défaillance logicielle dans l'une n'affecte pas les autres |
    | **Flexibilité** | Création, copie et suppression d'une VM en quelques minutes plutôt qu'une commande de matériel |
    | **Reproductibilité** | Un environnement peut être décrit, copié, restauré (snapshots, modèles) |
    | **Portabilité** | Une VM est un ensemble de fichiers que l'on peut déplacer d'un hôte à l'autre |
    | **Apprentissage** | On peut construire un réseau de plusieurs machines sur un seul poste, sans matériel dédié |

    **Exemple (fictif).** Dix serveurs physiques utilisent chacun environ 10 % de leur capacité de calcul. La charge cumulée représente environ un serveur à 100 %. Prévue avec une marge confortable, elle pourrait tenir sur deux ou trois hôtes virtualisés au lieu de dix machines.

    !!! warning "Contrepartie"
        Regrouper de nombreuses VM sur un hôte concentre aussi le risque : la panne de cet hôte arrête toutes ses VM. L'hôte devient un **point de défaillance unique** (chapitre 1) tant qu'on n'a pas prévu de redondance (chapitres 14 et 16).

=== "English"

    In chapter 1 we saw that a physical server permanently consumes power, takes up space and requires cooling, even when it is lightly used. **Virtualization** means creating, in software, IT resources that behave like physical ones: several isolated **virtual machines** (VMs) thus share the same hardware.

    | Benefit | Explanation |
    |---|---|
    | **Consolidation** | Several lightly loaded workloads coexist on a single physical machine: fewer servers, less power, fewer racks |
    | **Isolation** | Each VM has its own system; a software failure in one does not affect the others |
    | **Flexibility** | Creating, copying and deleting a VM takes minutes rather than a hardware order |
    | **Reproducibility** | An environment can be described, copied, restored (snapshots, templates) |
    | **Portability** | A VM is a set of files that can be moved from one host to another |
    | **Learning** | A network of several machines can be built on a single computer, with no dedicated hardware |

    **Example (fictional).** Ten physical servers each use about 10% of their computing capacity. The combined load is about one server at 100%. Planned with a comfortable margin, it could fit on two or three virtualized hosts instead of ten machines.

    !!! warning "Trade-off"
        Grouping many VMs on one host also concentrates risk: the failure of that host stops all its VMs. The host becomes a **single point of failure** (chapter 1) until redundancy is provided (chapters 14 and 16).

## 2. Comment ça marche ? Les techniques / How does it work? The techniques

=== "Français"

    | Technique | Principe | Performance |
    |---|---|---|
    | **Émulation** | Un logiciel simule entièrement le matériel, y compris le processeur, éventuellement d'une autre architecture | Faible |
    | **Virtualisation complète** | Le système invité, non modifié, croit tourner sur du matériel réel ; l'hyperviseur intercepte les opérations sensibles | Bonne avec assistance matérielle |
    | **Paravirtualisation** | Le système invité sait qu'il est virtualisé et utilise des pilotes adaptés (par exemple `virtio` avec KVM) | Très bonne, surtout pour les disques et le réseau |
    | **Assistance matérielle** | Les processeurs fournissent des modes dédiés à la virtualisation : **Intel VT-x** et **AMD-V** | Quasi native pour le calcul |

    Les hyperviseurs modernes **combinent** l'assistance matérielle (pour le processeur) et des pilotes paravirtualisés (pour les disques et le réseau). Cette assistance doit être **activée dans le micrologiciel** (BIOS/UEFI) de la machine : vous l'avez relevée au [Lab 1.2](../ch01-architecture-et-composants-d-un-datacenter/lab-1-2-inventorier-une-machine.md) (`VirtualizationFirmwareEnabled`).

    ### Comment les ressources sont partagées

    | Ressource | Principe de partage |
    |---|---|
    | **Processeur** | Les **vCPU** des VM se partagent le temps des cœurs physiques ; l'hyperviseur les ordonnance |
    | **Mémoire** | Une quantité de RAM est attribuée à chaque VM |
    | **Disque** | Chaque VM voit un disque virtuel, qui est en réalité un fichier ou un volume sur l'hôte |
    | **Réseau** | Chaque VM possède une carte virtuelle, reliée à un commutateur virtuel (chapitre 5) |

=== "English"

    | Technique | Principle | Performance |
    |---|---|---|
    | **Emulation** | Software fully simulates the hardware, including the processor, possibly of another architecture | Low |
    | **Full virtualization** | The unmodified guest system believes it runs on real hardware; the hypervisor intercepts sensitive operations | Good with hardware assistance |
    | **Paravirtualization** | The guest system knows it is virtualized and uses adapted drivers (for example `virtio` with KVM) | Very good, especially for disks and network |
    | **Hardware assistance** | Processors provide modes dedicated to virtualization: **Intel VT-x** and **AMD-V** | Near native for computing |

    Modern hypervisors **combine** hardware assistance (for the processor) and paravirtualized drivers (for disks and network). This assistance must be **enabled in the machine's firmware** (BIOS/UEFI): you collected it in [Lab 1.2](../ch01-architecture-et-composants-d-un-datacenter/lab-1-2-inventorier-une-machine.md) (`VirtualizationFirmwareEnabled`).

    ### How resources are shared

    | Resource | Sharing principle |
    |---|---|
    | **Processor** | The VMs' **vCPUs** share the time of the physical cores; the hypervisor schedules them |
    | **Memory** | An amount of RAM is assigned to each VM |
    | **Disk** | Each VM sees a virtual disk, which is actually a file or a volume on the host |
    | **Network** | Each VM has a virtual card, connected to a virtual switch (chapter 5) |

## 3. Les hyperviseurs : type 1 et type 2 / Hypervisors: type 1 and type 2

=== "Français"

    L'**hyperviseur** est le logiciel qui crée les VM et arbitre l'accès au matériel.

    ```mermaid
    flowchart TB
      subgraph T1["Type 1 (bare metal)"]
        direction TB
        V1["VM 1"] --- H1["Hyperviseur"]
        V2["VM 2"] --- H1
        H1 --- M1["Matériel"]
      end
      subgraph T2["Type 2 (hébergé)"]
        direction TB
        V3["VM 3"] --- H2["Hyperviseur<br/>(application)"]
        V4["VM 4"] --- H2
        H2 --- OS["Système d'exploitation hôte"]
        OS --- M2["Matériel"]
      end
    ```

    | | Type 1 (bare metal) | Type 2 (hébergé) |
    |---|---|---|
    | **Installation** | Directement sur le matériel | Comme une application dans un système d'exploitation |
    | **Performance** | Meilleure : peu de couches | Légèrement inférieure : le système hôte s'intercale |
    | **Usage typique** | Serveurs, datacenters | Postes de travail, apprentissage, tests |
    | **Exemples** | VMware ESXi, Proxmox VE (avec KVM), Hyper-V | VirtualBox, VMware Workstation |

    !!! info "Cas particuliers"
        **KVM** est un module intégré au noyau Linux : on le classe souvent comme hyperviseur de type 1, car c'est le noyau lui-même qui joue ce rôle. Sous Windows, **Hyper-V** s'installe comme une fonctionnalité du système, mais prend la main sur le matériel : il se comporte lui aussi comme un hyperviseur de type 1. Cette particularité peut entrer en conflit avec les hyperviseurs de type 2 (voir le [Lab 2.1](lab-2-1-preparer-le-poste-pour-la-virtualisation.md)).

=== "English"

    The **hypervisor** is the software that creates VMs and arbitrates access to the hardware.

    ```mermaid
    flowchart TB
      subgraph T1["Type 1 (bare metal)"]
        direction TB
        V1["VM 1"] --- H1["Hypervisor"]
        V2["VM 2"] --- H1
        H1 --- M1["Hardware"]
      end
      subgraph T2["Type 2 (hosted)"]
        direction TB
        V3["VM 3"] --- H2["Hypervisor<br/>(application)"]
        V4["VM 4"] --- H2
        H2 --- OS["Host operating system"]
        OS --- M2["Hardware"]
      end
    ```

    | | Type 1 (bare metal) | Type 2 (hosted) |
    |---|---|---|
    | **Installation** | Directly on the hardware | As an application inside an operating system |
    | **Performance** | Better: few layers | Slightly lower: the host system sits in between |
    | **Typical use** | Servers, datacenters | Workstations, learning, testing |
    | **Examples** | VMware ESXi, Proxmox VE (with KVM), Hyper-V | VirtualBox, VMware Workstation |

    !!! info "Special cases"
        **KVM** is a module built into the Linux kernel: it is often classed as a type 1 hypervisor because the kernel itself plays that role. On Windows, **Hyper-V** is installed as a system feature but takes control of the hardware: it also behaves as a type 1 hypervisor. This can conflict with type 2 hypervisors (see [Lab 2.1](lab-2-1-preparer-le-poste-pour-la-virtualisation.md)).

## 4. La machine virtuelle / The virtual machine

=== "Français"

    ### Composants et fichiers

    Une VM est définie par un matériel **virtuel** et par un ensemble de **fichiers** sur l'hôte.

    | Élément | Contenu | Formats courants |
    |---|---|---|
    | **Configuration** | Nombre de vCPU, RAM, périphériques, réseau | `.vmx` (VMware), `.vbox` (VirtualBox), XML (libvirt) |
    | **Disque virtuel** | Contenu du disque du système invité | `.vmdk` (VMware), `.vdi` (VirtualBox), `.vhdx` (Hyper-V), `.qcow2` (QEMU/KVM) |
    | **État** | Mémoire sauvegardée, snapshots | Fichiers propres à chaque hyperviseur |

    ### Provisionnement du disque

    - **Thick provisioning** : tout l'espace du disque virtuel est réservé à la création.
    - **Thin provisioning** : le fichier grandit au fur et à mesure de l'utilisation, jusqu'à la taille maximale déclarée. On économise de la place, mais l'hôte peut manquer d'espace si les disques grandissent plus vite que prévu.

    ### Snapshots, clones et modèles

    | Notion | Définition | Précaution |
    |---|---|---|
    | **Snapshot** | État d'une VM à un instant donné, auquel on peut revenir | Ce n'est **pas une sauvegarde** : il dépend du disque d'origine ; une longue chaîne de snapshots ralentit la VM et consomme de l'espace |
    | **Clone complet** | Copie indépendante d'une VM | Occupe autant d'espace que l'original |
    | **Clone lié** | Copie qui réutilise le disque de l'original | Économise de l'espace mais dépend de l'original |
    | **Modèle (template)** | VM de référence figée servant à en créer d'autres | À maintenir à jour (correctifs) |

    ### Surallocation (overcommit)

    On peut attribuer aux VM davantage de vCPU ou de mémoire que l'hôte n'en possède, en comptant sur le fait qu'elles n'en utilisent pas toutes simultanément.

    **Exemple.** Un hôte de 8 cœurs exécute 4 VM de 4 vCPU : 16 vCPU pour 8 cœurs, soit un rapport de 2 vCPU par cœur. C'est acceptable si les VM sont rarement toutes occupées ; sinon elles se disputent les cœurs et ralentissent. La surallocation de la **mémoire** est plus risquée : si l'hôte en manque, il doit utiliser le disque, ce qui dégrade fortement les performances.

=== "English"

    ### Components and files

    A VM is defined by **virtual** hardware and a set of **files** on the host.

    | Element | Content | Common formats |
    |---|---|---|
    | **Configuration** | Number of vCPUs, RAM, devices, network | `.vmx` (VMware), `.vbox` (VirtualBox), XML (libvirt) |
    | **Virtual disk** | Content of the guest system's disk | `.vmdk` (VMware), `.vdi` (VirtualBox), `.vhdx` (Hyper-V), `.qcow2` (QEMU/KVM) |
    | **State** | Saved memory, snapshots | Files specific to each hypervisor |

    ### Disk provisioning

    - **Thick provisioning**: all the virtual disk space is reserved at creation.
    - **Thin provisioning**: the file grows as it is used, up to the declared maximum size. Space is saved, but the host can run out of space if disks grow faster than planned.

    ### Snapshots, clones and templates

    | Notion | Definition | Caution |
    |---|---|---|
    | **Snapshot** | State of a VM at a given moment, to which you can return | It is **not a backup**: it depends on the original disk; a long snapshot chain slows the VM and uses space |
    | **Full clone** | Independent copy of a VM | Takes as much space as the original |
    | **Linked clone** | Copy that reuses the original's disk | Saves space but depends on the original |
    | **Template** | Frozen reference VM used to create others | Must be kept up to date (patches) |

    ### Overcommit

    VMs can be assigned more vCPUs or memory than the host has, relying on the fact that they do not all use it at the same time.

    **Example.** An 8-core host runs 4 VMs of 4 vCPUs: 16 vCPUs for 8 cores, a ratio of 2 vCPUs per core. This is acceptable if the VMs are rarely all busy; otherwise they compete for cores and slow down. **Memory** overcommit is riskier: if the host runs short, it has to use the disk, which severely degrades performance.

## 5. Machine virtuelle ou conteneur ? / Virtual machine or container?

=== "Français"

    Un **conteneur** est un processus isolé par le noyau Linux, qui **partage le noyau de l'hôte**. Cette isolation repose sur deux fonctions du noyau :

    - les **namespaces**, qui donnent au conteneur sa propre vue du système (processus, réseau, points de montage) ;
    - les **cgroups**, qui limitent et mesurent les ressources qu'il peut consommer (CPU, mémoire, entrées-sorties).

    ```mermaid
    flowchart TB
      subgraph VMS["Machines virtuelles"]
        direction TB
        A1["Application"] --> G1["Système invité complet"] --> HV["Hyperviseur"]
      end
      subgraph CTN["Conteneurs"]
        direction TB
        A2["Application"] --> RT["Moteur de conteneurs"] --> K["Noyau de l'hôte"]
      end
    ```

    | Critère | Machine virtuelle | Conteneur |
    |---|---|---|
    | **Isolation** | Forte : un noyau par VM | Plus faible : noyau partagé |
    | **Démarrage** | Plus lent : un système à démarrer | Rapide : un simple processus |
    | **Poids** | Plus lourd : un système complet par VM | Léger |
    | **Système invité** | Libre (Linux, Windows…) | Même noyau que l'hôte |
    | **Usage typique** | Infrastructure, systèmes variés, forte isolation | Applications, déploiement rapide |

    Les deux technologies sont **complémentaires** : on exécute fréquemment des conteneurs à l'intérieur de VM.

    !!! tip "À retenir"
        - Virtualiser = faire partager un même matériel par plusieurs systèmes isolés (les VM).
        - Bénéfices : consolidation, isolation, flexibilité, portabilité ; contrepartie : concentration du risque.
        - Techniques : émulation, virtualisation complète, paravirtualisation, assistance matérielle (VT-x, AMD-V).
        - Type 1 : sur le matériel (serveurs). Type 2 : dans un système hôte (postes de travail).
        - Une VM = un matériel virtuel + des fichiers (configuration, disque, état).
        - Un snapshot n'est pas une sauvegarde.
        - Surallocation : plus de vCPU ou de RAM attribués que disponibles, à manier avec prudence (surtout la mémoire).
        - Un conteneur partage le noyau de l'hôte : plus léger, moins isolé qu'une VM.

=== "English"

    A **container** is a process isolated by the Linux kernel that **shares the host kernel**. This isolation relies on two kernel features:

    - **namespaces**, which give the container its own view of the system (processes, network, mount points);
    - **cgroups**, which limit and measure the resources it can consume (CPU, memory, I/O).

    ```mermaid
    flowchart TB
      subgraph VMS["Virtual machines"]
        direction TB
        A1["Application"] --> G1["Full guest system"] --> HV["Hypervisor"]
      end
      subgraph CTN["Containers"]
        direction TB
        A2["Application"] --> RT["Container engine"] --> K["Host kernel"]
      end
    ```

    | Criterion | Virtual machine | Container |
    |---|---|---|
    | **Isolation** | Strong: one kernel per VM | Weaker: shared kernel |
    | **Startup** | Slower: a system has to boot | Fast: a simple process |
    | **Weight** | Heavier: a full system per VM | Light |
    | **Guest system** | Free (Linux, Windows...) | Same kernel as the host |
    | **Typical use** | Infrastructure, varied systems, strong isolation | Applications, fast deployment |

    The two technologies are **complementary**: containers are frequently run inside VMs.

    !!! tip "Key takeaways"
        - Virtualizing = making several isolated systems (VMs) share the same hardware.
        - Benefits: consolidation, isolation, flexibility, portability; trade-off: concentrated risk.
        - Techniques: emulation, full virtualization, paravirtualization, hardware assistance (VT-x, AMD-V).
        - Type 1: on the hardware (servers). Type 2: inside a host system (workstations).
        - A VM = virtual hardware + files (configuration, disk, state).
        - A snapshot is not a backup.
        - Overcommit: more vCPUs or RAM assigned than available, to be handled with care (especially memory).
        - A container shares the host kernel: lighter, less isolated than a VM.

## Pour vérifier votre compréhension / Check your understanding

=== "Français"

    ??? question "Un hôte de 8 cœurs exécute 5 VM de 4 vCPU chacune. Quel est le rapport vCPU par cœur ? Est-ce nécessairement un problème ?"
        5 × 4 = 20 vCPU pour 8 cœurs, soit **2,5 vCPU par cœur**. Ce n'est pas nécessairement un problème si les VM sont rarement toutes actives en même temps. Si elles le sont, elles se disputent les cœurs et leurs performances baissent : il faut mesurer l'usage réel.

    ??? question "Quelle différence y a-t-il entre un snapshot et une sauvegarde ?"
        Un snapshot est un point de retour stocké **avec la VM**, qui dépend de son disque d'origine : si le disque ou l'hôte est perdu, le snapshot l'est aussi. Une sauvegarde est une copie **indépendante**, idéalement hors de l'hôte, qui permet de restaurer après une perte.

    ??? question "Pourquoi un conteneur démarre-t-il plus vite qu'une VM, et pourquoi est-il moins isolé ?"
        Un conteneur est un simple processus qui utilise le noyau de l'hôte : il n'y a pas de système à démarrer. Pour la même raison, tous les conteneurs partagent ce noyau : une faille du noyau les concerne tous, alors qu'une VM possède son propre noyau.

=== "English"

    ??? question "An 8-core host runs 5 VMs of 4 vCPUs each. What is the vCPU-per-core ratio? Is it necessarily a problem?"
        5 × 4 = 20 vCPUs for 8 cores, i.e. **2.5 vCPUs per core**. It is not necessarily a problem if the VMs are rarely all active at the same time. If they are, they compete for cores and their performance drops: actual usage has to be measured.

    ??? question "What is the difference between a snapshot and a backup?"
        A snapshot is a restore point stored **with the VM**, depending on its original disk: if the disk or the host is lost, so is the snapshot. A backup is an **independent** copy, ideally outside the host, that allows restoring after a loss.

    ??? question "Why does a container start faster than a VM, and why is it less isolated?"
        A container is a simple process using the host kernel: there is no system to boot. For the same reason, all containers share that kernel: a kernel flaw affects them all, whereas a VM has its own kernel.

## Ressources du chapitre / Chapter resources

- [Lab 2.1 : Préparer le poste pour la virtualisation / Preparing the computer for virtualization](lab-2-1-preparer-le-poste-pour-la-virtualisation.md)
- [Quiz du chapitre / Chapter quiz](quiz.md)
- [Glossaire / Glossary](../../annexes/glossaire.md)
- [Environnement de TP / Lab environment](../../annexes/environnement.md)