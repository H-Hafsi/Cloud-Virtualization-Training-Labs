# Chapitre 7 : Technologies et interfaces de stockage / Storage Technologies and Interfaces

=== "Français"

    !!! abstract "Objectifs du chapitre"
        - Comparer les technologies de stockage : disque dur, SSD SATA, SSD NVMe
        - Distinguer les interfaces SATA, SAS et NVMe et leurs débits théoriques
        - Définir et relier capacité, débit, IOPS, latence et profondeur de file
        - Décrire le chemin d'une écriture dans une machine virtuelle et les formats de disques virtuels
        - Expliquer le provisionnement dynamique, le coût des snapshots et les contrôleurs virtuels
        - Planifier une mesure de performances fiable avec `fio`
        - Acquis d'apprentissage visé : **AA3**

        Matériel requis pour ce cours : aucun. Les labs associés indiquent leurs propres ressources.

=== "English"

    !!! abstract "Chapter objectives"
        - Compare storage technologies: hard disk, SATA SSD, NVMe SSD
        - Tell SATA, SAS and NVMe interfaces apart and their theoretical bandwidth
        - Define and relate capacity, throughput, IOPS, latency and queue depth
        - Describe the path of a write in a virtual machine and virtual disk formats
        - Explain dynamic provisioning, the cost of snapshots and virtual controllers
        - Plan a reliable performance measurement with `fio`
        - Target learning outcome: **AA3**

        Required equipment for this lesson: none. The associated labs state their own resources.

## 1. Les technologies de stockage / Storage technologies

=== "Français"

    Jusqu'ici, un disque virtuel était pour vous un simple fichier de quelques giga-octets. Derrière ce fichier, il y a toujours un **support physique** dont les caractéristiques déterminent les performances de toutes vos VM.

    | Technologie | Principe | Points forts | Limites |
    |---|---|---|---|
    | **Disque dur (HDD)** | Plateaux magnétiques en rotation (5 400 à 15 000 tr/min) lus par une tête mobile | Faible coût par Go, grande capacité | Latence de l'ordre de la milliseconde (déplacement de la tête, rotation) ; performances faibles sur les accès aléatoires ; sensible aux chocs |
    | **SSD SATA** | Mémoire flash NAND pilotée par un contrôleur, sans pièce mobile | Latence très faible, nombreux accès aléatoires par seconde | Débit plafonné par l'interface SATA ; usure des cellules à l'écriture |
    | **SSD NVMe** | Même mémoire flash, mais reliée directement au bus **PCIe** | Débits et IOPS les plus élevés, files de commandes parallèles | Plus coûteux ; chauffe sous charge soutenue |

    Quelques notions complémentaires :

    - **Mémoire flash :** chaque cellule supporte un nombre limité de cycles d'écriture (endurance) ; le contrôleur répartit l'usure (*wear levelling*). Plus on stocke de bits par cellule (TLC, QLC), plus la capacité est élevée et l'endurance basse.
    - **TRIM :** commande par laquelle le système indique au SSD quels blocs ne servent plus, ce qui préserve ses performances.
    - **Disques en production :** les serveurs utilisent des modèles conçus pour un fonctionnement continu et surveillent leur état (SMART), car un disque finit toujours par tomber en panne : c'est l'objet de la redondance (chapitre 8).

    !!! info "Ordres de grandeur"
        Les valeurs varient beaucoup selon les modèles. À titre indicatif : latence d'accès d'un HDD de l'ordre de plusieurs millisecondes, d'un SSD SATA de l'ordre de la dizaine à la centaine de microsecondes, d'un SSD NVMe plus faible encore. Vous **mesurerez** vos propres valeurs au Lab 7.2.

=== "English"

    So far, a virtual disk was for you a simple file of a few gigabytes. Behind that file there is always a **physical medium** whose characteristics determine the performance of all your VMs.

    | Technology | Principle | Strengths | Limits |
    |---|---|---|---|
    | **Hard disk (HDD)** | Spinning magnetic platters (5,400 to 15,000 rpm) read by a moving head | Low cost per GB, large capacity | Latency in the millisecond range (head movement, rotation); poor random access performance; shock-sensitive |
    | **SATA SSD** | NAND flash memory driven by a controller, with no moving parts | Very low latency, many random accesses per second | Throughput capped by the SATA interface; cell wear on writes |
    | **NVMe SSD** | Same flash memory, but connected directly to the **PCIe** bus | Highest throughput and IOPS, parallel command queues | More expensive; heats up under sustained load |

    A few additional notions:

    - **Flash memory:** each cell withstands a limited number of write cycles (endurance); the controller spreads wear (*wear levelling*). The more bits stored per cell (TLC, QLC), the higher the capacity and the lower the endurance.
    - **TRIM:** command by which the system tells the SSD which blocks are no longer used, preserving its performance.
    - **Production disks:** servers use models designed for continuous operation and monitor their health (SMART), because a disk always ends up failing: this is what redundancy is for (chapter 8).

    !!! info "Orders of magnitude"
        Values vary a lot between models. As a guide: access latency of an HDD in the range of several milliseconds, of a SATA SSD in the range of tens to hundreds of microseconds, of an NVMe SSD even lower. You will **measure** your own values in Lab 7.2.

## 2. Les interfaces : SATA, SAS, NVMe / Interfaces: SATA, SAS, NVMe

=== "Français"

    | Interface | Usage typique | Débit théorique par lien | Particularités |
    |---|---|---|---|
    | **SATA III** | Postes de travail, serveurs d'entrée de gamme | 6 Gbit/s, soit environ 600 Mo/s utiles | Une seule file de 32 commandes ; un seul sens de transfert à la fois |
    | **SAS-3** | Serveurs et baies de stockage d'entreprise | 12 Gbit/s | Double port (chemins redondants), transferts dans les deux sens, grands câblages possibles |
    | **NVMe sur PCIe 3.0 ×4** | SSD haute performance | Environ 4 Go/s | Jusqu'à 64 000 files de 64 000 commandes ; latence réduite |
    | **NVMe sur PCIe 4.0 ×4** | SSD plus récents | Environ 8 Go/s | Même protocole, bus plus rapide |

    Un **contrôleur HBA** relie les disques SATA ou SAS à la carte mère du serveur. Un contrôleur **RAID matériel** ajoute en outre la gestion de grappes de disques (chapitre 8).

    **Pourquoi ces différences comptent-elles ?** Un SSD SATA ne peut pas dépasser le débit de son interface, quelles que soient ses capacités internes. Un SSD NVMe lève cette limite et traite beaucoup de requêtes en parallèle, ce qui profite aux serveurs qui hébergent de nombreuses VM.

=== "English"

    | Interface | Typical use | Theoretical throughput per link | Peculiarities |
    |---|---|---|---|
    | **SATA III** | Workstations, entry-level servers | 6 Gbit/s, about 600 MB/s usable | A single queue of 32 commands; one transfer direction at a time |
    | **SAS-3** | Enterprise servers and storage arrays | 12 Gbit/s | Dual port (redundant paths), transfers in both directions, large cabling possible |
    | **NVMe over PCIe 3.0 ×4** | High-performance SSDs | About 4 GB/s | Up to 64,000 queues of 64,000 commands; reduced latency |
    | **NVMe over PCIe 4.0 ×4** | Newer SSDs | About 8 GB/s | Same protocol, faster bus |

    An **HBA** (host bus adapter) connects SATA or SAS disks to the server's motherboard. A **hardware RAID** controller also adds disk array management (chapter 8).

    **Why do these differences matter?** A SATA SSD cannot exceed its interface throughput, whatever its internal capabilities. An NVMe SSD removes this limit and handles many requests in parallel, which benefits servers hosting many VMs.

## 3. Mesurer les performances : débit, IOPS, latence / Measuring performance: throughput, IOPS, latency

=== "Français"

    | Grandeur | Définition | Unité |
    |---|---|---|
    | **Capacité** | Quantité de données stockables | Go, To |
    | **Débit** (*throughput*) | Quantité de données transférées par seconde | Mo/s |
    | **IOPS** | Nombre d'opérations d'entrée-sortie par seconde | opérations/s |
    | **Latence** | Durée d'une opération, de la demande à la réponse | ms, µs |
    | **Profondeur de file** (*queue depth*) | Nombre de requêtes envoyées en parallèle | nombre |

    **Relation fondamentale :** débit = IOPS × taille du bloc.

    - 20 000 IOPS en blocs de 4 Kio donnent environ 80 000 Kio/s, soit **environ 80 Mo/s**.
    - 500 IOPS en blocs de 1 Mio donnent **500 Mio/s**.

    Un même disque peut donc afficher un très grand nombre d'IOPS avec un faible débit (petits blocs, accès aléatoires) ou un grand débit avec peu d'IOPS (gros blocs, accès séquentiels). **Un chiffre isolé ne caractérise pas un disque** : il faut préciser le profil.

    | Paramètre du profil | Valeurs courantes |
    |---|---|
    | **Motif d'accès** | Séquentiel (fichiers volumineux, sauvegardes) ou aléatoire (bases de données, VM) |
    | **Type d'opération** | Lecture, écriture ou mixte (par exemple 70 % de lectures) |
    | **Taille de bloc** | 4 Kio (petits accès) à 1 Mio (grands transferts) |
    | **Profondeur de file** | 1 (une requête à la fois) à 32 ou plus |

    Une **latence** faible et stable compte autant que le débit moyen : une VM perçoit surtout le temps d'attente de chaque opération.

=== "English"

    | Quantity | Definition | Unit |
    |---|---|---|
    | **Capacity** | Amount of data that can be stored | GB, TB |
    | **Throughput** | Amount of data transferred per second | MB/s |
    | **IOPS** | Number of input/output operations per second | operations/s |
    | **Latency** | Duration of an operation, from request to response | ms, µs |
    | **Queue depth** | Number of requests sent in parallel | number |

    **Fundamental relation:** throughput = IOPS × block size.

    - 20,000 IOPS with 4 KiB blocks give about 80,000 KiB/s, i.e. **about 80 MB/s**.
    - 500 IOPS with 1 MiB blocks give **500 MiB/s**.

    The same disk can therefore show a very large number of IOPS with low throughput (small blocks, random access) or high throughput with few IOPS (large blocks, sequential access). **A single figure does not characterize a disk**: the profile must be specified.

    | Profile parameter | Common values |
    |---|---|
    | **Access pattern** | Sequential (large files, backups) or random (databases, VMs) |
    | **Operation type** | Read, write or mixed (for example 70% reads) |
    | **Block size** | 4 KiB (small accesses) to 1 MiB (large transfers) |
    | **Queue depth** | 1 (one request at a time) to 32 or more |

    A low and stable **latency** matters as much as average throughput: a VM mostly feels the waiting time of each operation.

## 4. Le stockage d'une machine virtuelle / A virtual machine's storage

=== "Français"

    Quand un programme écrit dans une VM, la donnée traverse plusieurs couches avant d'atteindre le support physique :

    ```mermaid
    flowchart LR
      A["Application<br/>dans la VM"] --> F["Système de fichiers<br/>de l'invité"]
      F --> C["Contrôleur<br/>de disque virtuel"]
      C --> H["Hyperviseur"]
      H --> V["Fichier de disque virtuel<br/>sur l'hôte"]
      V --> HF["Système de fichiers<br/>de l'hôte (cache)"]
      HF --> D["Disque physique"]
    ```

    Chaque couche peut ajouter de la latence ou un **cache** : le cache de l'invité et celui de l'hôte peuvent rendre une mesure plus optimiste que la réalité du disque.

    ### Contrôleurs virtuels

    | Contrôleur | Principe |
    |---|---|
    | **IDE / SATA émulés** | Matériel ancien ou courant simulé par l'hyperviseur : compatible partout, plus lent |
    | **SCSI virtuel et `virtio`** | Contrôleurs conçus pour la virtualisation (paravirtualisation, chapitre 2) : plus rapides |
    | **NVMe virtuel** | Proposé par certains hyperviseurs ; utile pour tester ce protocole |

    ### Formats de disques virtuels

    | Format | Hyperviseur | Particularités |
    |---|---|---|
    | **raw** | KVM/QEMU | Aucune structure : l'image est une copie brute du disque ; performances maximales, mais ni snapshots ni allocation dynamique propres au format |
    | **qcow2** | KVM/QEMU | Allocation dynamique, snapshots, chaînage (*backing file*), compression |
    | **VMDK** | VMware (et lu par d'autres) | Allocation dynamique ou fixe, snapshots |
    | **VDI** | VirtualBox | Allocation dynamique ou fixe |
    | **VHDX** | Hyper-V | Jusqu'à 64 To, résistant aux coupures, disque différentiel |

    On **convertit** un disque d'un format à l'autre avec l'outil de l'hyperviseur (`VBoxManage clonemedium`, `Convert-VHD`) ou avec `qemu-img convert` (paquet `qemu-utils` sous Ubuntu). Le Lab 7.1 vous fait ajouter, convertir et comparer des disques.

    ### Provisionnement et snapshots : le coût de la souplesse

    - **Dynamique (*thin*)** : économise la place, mais chaque première écriture doit **allouer** de l'espace, ce qui ralentit légèrement ; l'hôte peut manquer d'espace si les disques grandissent.
    - **Fixe (*thick*)** : l'espace est réservé d'avance ; accès plus prévisibles.
    - **Snapshots et disques différentiels** : chaque niveau ajouté oblige à parcourir une chaîne de fichiers ; plus la chaîne est longue, plus la VM ralentit (chapitre 2).

=== "English"

    When a program writes in a VM, the data crosses several layers before reaching the physical medium:

    ```mermaid
    flowchart LR
      A["Application<br/>inside the VM"] --> F["Guest<br/>file system"]
      F --> C["Virtual disk<br/>controller"]
      C --> H["Hypervisor"]
      H --> V["Virtual disk file<br/>on the host"]
      V --> HF["Host file system<br/>(cache)"]
      HF --> D["Physical disk"]
    ```

    Each layer can add latency or a **cache**: the guest's and the host's caches can make a measurement more optimistic than the disk's reality.

    ### Virtual controllers

    | Controller | Principle |
    |---|---|
    | **Emulated IDE / SATA** | Old or common hardware simulated by the hypervisor: compatible everywhere, slower |
    | **Virtual SCSI and `virtio`** | Controllers designed for virtualization (paravirtualization, chapter 2): faster |
    | **Virtual NVMe** | Offered by some hypervisors; useful to test this protocol |

    ### Virtual disk formats

    | Format | Hypervisor | Peculiarities |
    |---|---|---|
    | **raw** | KVM/QEMU | No structure: the image is a raw copy of the disk; maximum performance, but no format-level snapshots or dynamic allocation |
    | **qcow2** | KVM/QEMU | Dynamic allocation, snapshots, chaining (*backing file*), compression |
    | **VMDK** | VMware (and read by others) | Dynamic or fixed allocation, snapshots |
    | **VDI** | VirtualBox | Dynamic or fixed allocation |
    | **VHDX** | Hyper-V | Up to 64 TB, power-loss resilient, differencing disk |

    A disk is **converted** from one format to another with the hypervisor's tool (`VBoxManage clonemedium`, `Convert-VHD`) or with `qemu-img convert` (`qemu-utils` package on Ubuntu). Lab 7.1 has you add, convert and compare disks.

    ### Provisioning and snapshots: the cost of flexibility

    - **Dynamic (*thin*)**: saves space, but each first write must **allocate** space, which slows it slightly; the host can run out of space if disks grow.
    - **Fixed (*thick*)**: space is reserved in advance; more predictable access.
    - **Snapshots and differencing disks**: each added level forces a walk along a chain of files; the longer the chain, the slower the VM (chapter 2).

## 5. Mesurer et surveiller / Measuring and monitoring

=== "Français"

    | Outil | Usage |
    |---|---|
    | **`fio`** | Générateur de charge d'entrées-sorties : on décrit un profil (motif, taille de bloc, profondeur de file) et il mesure débit, IOPS et latences (Lab 7.2) |
    | **`iostat`** | Statistiques d'utilisation des disques en temps réel (paquet `sysstat`) |
    | **`lsblk`** | Liste des disques, partitions et points de montage |
    | **`smartctl`** | État de santé d'un disque physique (paquet `smartmontools`) |

    Paramètres usuels de `fio` : `--rw` (`read`, `write`, `randread`, `randwrite`, `randrw`), `--bs` (taille de bloc), `--iodepth` (profondeur de file), `--direct=1` (contourne le cache du système), `--runtime` (durée), `--size` (taille du fichier de test).

    **Règles pour une mesure fiable :**

    - tester un **fichier** sur un système de fichiers dédié, **jamais un disque contenant des données** : un test d'écriture sur un périphérique brut **détruit son contenu** ;
    - utiliser `--direct=1` et une taille de test **supérieure à la mémoire** pour limiter l'effet des caches ;
    - **répéter** la mesure et noter les conditions (autres VM, charge de l'hôte) ;
    - comparer des profils **identiques** : même bloc, même file, même durée.

    !!! tip "À retenir"
        - HDD : grande capacité, latence en millisecondes ; SSD SATA : faible latence, débit limité par l'interface ; NVMe : débits et parallélisme les plus élevés.
        - SATA III plafonne autour de 600 Mo/s ; NVMe sur PCIe dépasse plusieurs Go/s.
        - Débit = IOPS × taille de bloc ; un chiffre isolé ne caractérise pas un disque.
        - Une écriture de VM traverse le système de fichiers de l'invité, le contrôleur virtuel, l'hyperviseur et le système de fichiers de l'hôte ; les caches faussent les mesures.
        - Formats : raw, qcow2, VMDK, VDI, VHDX ; on les convertit avec `qemu-img`, `VBoxManage`, `Convert-VHD`.
        - Dynamique : économique mais moins prévisible ; fixe : réservé d'avance ; snapshots : coût croissant avec la chaîne.
        - Avec `fio`, toujours tester un fichier, jamais un disque contenant des données.

=== "English"

    | Tool | Use |
    |---|---|
    | **`fio`** | I/O load generator: you describe a profile (pattern, block size, queue depth) and it measures throughput, IOPS and latencies (Lab 7.2) |
    | **`iostat`** | Real-time disk usage statistics (`sysstat` package) |
    | **`lsblk`** | List of disks, partitions and mount points |
    | **`smartctl`** | Health status of a physical disk (`smartmontools` package) |

    Usual `fio` parameters: `--rw` (`read`, `write`, `randread`, `randwrite`, `randrw`), `--bs` (block size), `--iodepth` (queue depth), `--direct=1` (bypasses the system cache), `--runtime` (duration), `--size` (test file size).

    **Rules for a reliable measurement:**

    - test a **file** on a dedicated file system, **never a disk holding data**: a write test on a raw device **destroys its content**;
    - use `--direct=1` and a test size **larger than memory** to limit the effect of caches;
    - **repeat** the measurement and note the conditions (other VMs, host load);
    - compare **identical** profiles: same block, same queue, same duration.

    !!! tip "Key takeaways"
        - HDD: large capacity, latency in milliseconds; SATA SSD: low latency, throughput limited by the interface; NVMe: highest throughput and parallelism.
        - SATA III caps at around 600 MB/s; NVMe over PCIe exceeds several GB/s.
        - Throughput = IOPS × block size; a single figure does not characterize a disk.
        - A VM write crosses the guest file system, the virtual controller, the hypervisor and the host file system; caches skew measurements.
        - Formats: raw, qcow2, VMDK, VDI, VHDX; convert them with `qemu-img`, `VBoxManage`, `Convert-VHD`.
        - Dynamic: economical but less predictable; fixed: reserved in advance; snapshots: cost grows with the chain.
        - With `fio`, always test a file, never a disk holding data.

## Pour vérifier votre compréhension / Check your understanding

=== "Français"

    ??? question "Un SSD réalise 20 000 IOPS en blocs de 4 Kio. Quel débit cela représente-t-il ? Ce SSD peut-il saturer une interface SATA III ?"
        Débit = 20 000 × 4 Kio = 80 000 Kio/s, soit environ **80 Mo/s**. Ce débit est très inférieur aux quelque 600 Mo/s de SATA III : en accès aléatoire de petits blocs, ce SSD ne saturerait pas l'interface. Il la saturerait en lectures séquentielles de gros blocs, si le SSD en est capable.

    ??? question "Pourquoi un test de performance dans une VM peut-il surestimer les performances du disque physique ?"
        Les données peuvent être servies par les **caches** de l'invité ou de l'hôte plutôt que par le disque. On limite cet effet avec `--direct=1` et une taille de test supérieure à la mémoire, et en sachant que le résultat dépend aussi de l'hôte et des autres VM.

    ??? question "Pourquoi le format raw est-il le plus rapide, mais rarement choisi seul ?"
        Il n'a aucune structure intermédiaire à gérer, donc peu de surcoût. Mais il n'offre pas d'allocation dynamique, de snapshots ni de chaînage propres au format : ces fonctions sont celles de formats comme qcow2.

=== "English"

    ??? question "An SSD does 20,000 IOPS with 4 KiB blocks. What throughput is that? Can this SSD saturate a SATA III interface?"
        Throughput = 20,000 × 4 KiB = 80,000 KiB/s, about **80 MB/s**. This is far below SATA III's roughly 600 MB/s: with random small-block access, this SSD would not saturate the interface. It would with sequential reads of large blocks, if the SSD is capable of it.

    ??? question "Why can a performance test inside a VM overestimate the physical disk's performance?"
        Data may be served by the guest's or the host's **caches** rather than by the disk. This effect is limited with `--direct=1` and a test size larger than memory, bearing in mind that the result also depends on the host and other VMs.

    ??? question "Why is the raw format the fastest, but rarely chosen alone?"
        It has no intermediate structure to manage, hence little overhead. But it offers no format-level dynamic allocation, snapshots or chaining: these features belong to formats like qcow2.

## Ressources du chapitre / Chapter resources

- [Lab 7.1 : Disques virtuels, ajouter, convertir, comparer / Virtual disks: add, convert, compare](lab-7-1-disques-virtuels-ajouter-convertir-comparer.md)
- [Lab 7.2 : Mesurer les performances avec fio / Measuring performance with fio](lab-7-2-mesurer-les-performances-avec-fio.md)
- [Quiz du chapitre / Chapter quiz](quiz.md)
- [Glossaire / Glossary](../../annexes/glossaire.md)
- [Environnement de TP / Lab environment](../../annexes/environnement.md)