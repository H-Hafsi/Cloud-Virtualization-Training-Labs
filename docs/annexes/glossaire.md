# Glossaire / Glossary

=== "Français"

    Ce glossaire regroupe les termes clés du cours, classés par **thème** (dans l'ordre des Parties), puis par **ordre alphabétique**. La colonne « Ch. » indique le chapitre où le terme est étudié en détail. Les termes anglais sont donnés tels qu'on les rencontre dans la documentation des outils.

    !!! tip "Conseil"
        Pour comparer le vocabulaire propre à chaque hyperviseur (VMware, Proxmox VE, Hyper-V, KVM), consultez la [correspondance des termes entre hyperviseurs](correspondance-hyperviseurs.md). Les commandes sont dans l'[aide-mémoire](aide-memoire-commandes.md).

=== "English"

    This glossary gathers the key terms of the course, grouped by **theme** (in the order of the Parts), then in **alphabetical order**. The "Ch." column gives the chapter where the term is studied in detail. English terms are given as found in the tools' documentation.

    !!! tip "Tip"
        To compare the vocabulary of each hypervisor (VMware, Proxmox VE, Hyper-V, KVM), see the [terminology mapping between hypervisors](correspondance-hyperviseurs.md). Commands are in the [cheat sheet](aide-memoire-commandes.md).

## 1. Datacenter et virtualisation / Datacenter and virtualization

=== "Français"

    | Terme | Définition | Ch. |
    |---|---|---|
    | **Baie (rack)** | Armoire normalisée qui accueille serveurs, commutateurs et équipements de stockage. | 1 |
    | **cgroups** | Fonction du noyau Linux qui limite et mesure les ressources (CPU, mémoire, E/S) d'un groupe de processus. | 1 |
    | **Clone** | Copie d'une machine virtuelle. Un clone *complet* est indépendant ; un clone *lié* dépend du disque de la machine d'origine. | 2 |
    | **Conteneur** | Processus isolé par les namespaces et limité par les cgroups, qui partage le noyau de l'hôte (contrairement à une VM). | 1 |
    | **Datacenter (centre de données)** | Site regroupant serveurs, stockage, réseau, alimentation électrique et refroidissement pour héberger des services informatiques. | 1 |
    | **Hôte / invité (host / guest)** | L'hôte est la machine physique (ou la VM) qui exécute l'hyperviseur ; l'invité est le système qui tourne dans la VM. | 2 |
    | **Hyperviseur** | Logiciel qui crée et exécute des machines virtuelles en partageant les ressources matérielles. | 2 |
    | **Hyperviseur de type 1 (bare metal)** | Hyperviseur installé directement sur le matériel (ESXi, Proxmox VE avec KVM, Hyper-V Server). | 2 |
    | **Hyperviseur de type 2 (hébergé)** | Hyperviseur exécuté comme une application dans un système d'exploitation (VirtualBox, VMware Workstation). | 2 |
    | **Namespaces** | Fonction du noyau Linux qui donne à un groupe de processus sa propre vue du système (processus, réseau, points de montage…). | 1 |
    | **N+1 (redondance)** | Principe consistant à prévoir un équipement de plus que le besoin (alimentation, refroidissement, serveur) pour absorber une panne. | 1 |
    | **Onduleur (UPS)** | Alimentation de secours à batterie qui maintient les équipements lors d'une coupure électrique. | 1 |
    | **OVF / OVA** | Formats ouverts d'échange de machines virtuelles. OVF décrit la machine en plusieurs fichiers ; OVA regroupe le tout dans une seule archive. | 3 |
    | **Paravirtualisation (virtio)** | Technique où l'invité utilise des pilotes adaptés à la virtualisation, plus rapides qu'un matériel émulé. `virtio` est la famille de pilotes utilisée avec KVM. | 2, 12 |
    | **PUE (Power Usage Effectiveness)** | Rapport entre l'énergie totale consommée par le datacenter et celle consommée par les équipements informatiques. Plus il est proche de 1, plus le site est efficace. | 1 |
    | **Snapshot (instantané)** | État d'une VM (ou d'un volume) à un instant donné, auquel on peut revenir. Ce n'est pas une sauvegarde. | 2 |
    | **Surallocation (overcommit)** | Attribution aux VM de plus de vCPU ou de mémoire que la machine physique n'en possède, en comptant sur le fait qu'elles n'en utilisent pas toutes en même temps. | 2 |
    | **Thin / thick provisioning** | *Thin* : le disque virtuel grandit au fil de l'utilisation. *Thick* : tout l'espace est réservé dès la création. | 2, 7 |
    | **Tier (classification)** | Niveaux de disponibilité d'un datacenter (de I à IV), définis par l'Uptime Institute. Plus le niveau est élevé, plus la redondance est forte. | 1 |
    | **U (unité de rack)** | Unité de hauteur d'une baie : 1 U = 44,45 mm. | 1 |
    | **vCPU** | Processeur virtuel attribué à une VM. | 2 |
    | **VT-x / AMD-V** | Extensions matérielles des processeurs Intel et AMD qui permettent une virtualisation efficace. | 2 |
    | **Virtualisation imbriquée (nested)** | Exécution d'un hyperviseur à l'intérieur d'une VM. Elle doit être exposée à la VM par l'hyperviseur de niveau inférieur. | 2, 12 à 16 |
    | **Machine virtuelle (VM)** | Ordinateur simulé par logiciel, avec son propre système d'exploitation, ses disques et ses cartes réseau virtuels. | 2 |

=== "English"

    | Term | Definition | Ch. |
    |---|---|---|
    | **Rack** | Standardized cabinet housing servers, switches and storage equipment. | 1 |
    | **cgroups** | Linux kernel feature that limits and measures the resources (CPU, memory, I/O) of a group of processes. | 1 |
    | **Clone** | Copy of a virtual machine. A *full* clone is independent; a *linked* clone depends on the source machine's disk. | 2 |
    | **Container** | Process isolated by namespaces and limited by cgroups, sharing the host kernel (unlike a VM). | 1 |
    | **Datacenter** | Facility grouping servers, storage, network, power and cooling to host IT services. | 1 |
    | **Host / guest** | The host is the physical machine (or VM) running the hypervisor; the guest is the system running inside the VM. | 2 |
    | **Hypervisor** | Software that creates and runs virtual machines by sharing hardware resources. | 2 |
    | **Type 1 hypervisor (bare metal)** | Hypervisor installed directly on the hardware (ESXi, Proxmox VE with KVM, Hyper-V Server). | 2 |
    | **Type 2 hypervisor (hosted)** | Hypervisor running as an application inside an operating system (VirtualBox, VMware Workstation). | 2 |
    | **Namespaces** | Linux kernel feature that gives a group of processes its own view of the system (processes, network, mount points...). | 1 |
    | **N+1 (redundancy)** | Principle of providing one more unit than needed (power, cooling, server) to absorb a failure. | 1 |
    | **UPS (uninterruptible power supply)** | Battery-backed power source that keeps equipment running during a power cut. | 1 |
    | **OVF / OVA** | Open formats for exchanging virtual machines. OVF describes the machine in several files; OVA packs everything into a single archive. | 3 |
    | **Paravirtualization (virtio)** | Technique where the guest uses drivers designed for virtualization, faster than emulated hardware. `virtio` is the driver family used with KVM. | 2, 12 |
    | **PUE (Power Usage Effectiveness)** | Ratio of the total energy used by the datacenter to the energy used by the IT equipment. The closer to 1, the more efficient the site. | 1 |
    | **Snapshot** | State of a VM (or volume) at a given moment, to which you can return. It is not a backup. | 2 |
    | **Overcommit** | Assigning VMs more vCPUs or memory than the physical machine has, relying on the fact that they do not all use it at once. | 2 |
    | **Thin / thick provisioning** | *Thin*: the virtual disk grows as it is used. *Thick*: all the space is reserved at creation. | 2, 7 |
    | **Tier (classification)** | Datacenter availability levels (I to IV) defined by the Uptime Institute. The higher the level, the stronger the redundancy. | 1 |
    | **U (rack unit)** | Height unit of a rack: 1 U = 44.45 mm. | 1 |
    | **vCPU** | Virtual processor assigned to a VM. | 2 |
    | **VT-x / AMD-V** | Hardware extensions of Intel and AMD processors that enable efficient virtualization. | 2 |
    | **Nested virtualization** | Running a hypervisor inside a VM. The lower-level hypervisor must expose it to the VM. | 2, 12 to 16 |
    | **Virtual machine (VM)** | Computer simulated by software, with its own operating system, virtual disks and virtual network cards. | 2 |

## 2. Outils de poste et réseaux virtuels / Desktop tools and virtual networks

=== "Français"

    | Terme | Définition | Ch. |
    |---|---|---|
    | **Box (Vagrant)** | Image de base empaquetée qu'utilise Vagrant pour créer une machine. | 6 |
    | **Checkpoint (Hyper-V)** | Équivalent Hyper-V du snapshot : point de restauration d'une VM. | 4 |
    | **Commutateur virtuel (virtual switch)** | Commutateur logiciel qui relie les cartes réseau des VM entre elles et, selon son type, au réseau physique. Hyper-V distingue les types externe, interne et privé. | 5 |
    | **Hôte seul (host-only)** | Réseau virtuel qui relie les VM entre elles et à l'hôte, sans accès au réseau extérieur. | 5 |
    | **NAT** | Traduction d'adresses : les VM sortent vers l'extérieur derrière l'adresse de l'hôte, sans être joignables directement de l'extérieur. | 5 |
    | **Pont (bridge)** | Mode réseau où la VM apparaît comme une machine à part entière sur le réseau physique. | 5 |
    | **Provider (Vagrant)** | Moteur de virtualisation utilisé par Vagrant (VirtualBox, VMware, libvirt, Hyper-V…). À ne pas confondre avec le *provider* d'OpenTofu. | 6 |
    | **Provisioning** | Configuration automatique d'une machine après sa création (scripts, Ansible…). | 6 |
    | **Routage** | Acheminement des paquets entre réseaux IP différents, par une machine configurée en routeur. | 5 |
    | **Vagrant** | Outil qui décrit des environnements de machines virtuelles dans un fichier et les crée de façon reproductible. | 6 |
    | **Vagrantfile** | Fichier (en Ruby) qui décrit les machines, leur réseau et leur configuration. | 6 |
    | **VirtualBox** | Hyperviseur de type 2 libre. | 3 |
    | **VLAN** | Réseau local virtuel : découpage logique d'un même réseau physique en domaines de diffusion séparés (norme 802.1Q). | 5 |
    | **VMware Workstation** | Hyperviseur de type 2 de VMware, utilisé dans ce cours pour héberger les machines imbriquées. | 3 |

=== "English"

    | Term | Definition | Ch. |
    |---|---|---|
    | **Box (Vagrant)** | Packaged base image that Vagrant uses to create a machine. | 6 |
    | **Checkpoint (Hyper-V)** | Hyper-V equivalent of a snapshot: a VM restore point. | 4 |
    | **Virtual switch** | Software switch that connects VM network cards to each other and, depending on its type, to the physical network. Hyper-V distinguishes external, internal and private types. | 5 |
    | **Host-only** | Virtual network that links VMs to each other and to the host, with no access to the outside network. | 5 |
    | **NAT** | Network address translation: VMs reach the outside behind the host's address, without being directly reachable from outside. | 5 |
    | **Bridge** | Network mode where the VM appears as a full machine on the physical network. | 5 |
    | **Provider (Vagrant)** | Virtualization engine used by Vagrant (VirtualBox, VMware, libvirt, Hyper-V...). Not to be confused with the OpenTofu *provider*. | 6 |
    | **Provisioning** | Automatic configuration of a machine after its creation (scripts, Ansible...). | 6 |
    | **Routing** | Forwarding of packets between different IP networks, by a machine configured as a router. | 5 |
    | **Vagrant** | Tool that describes virtual machine environments in a file and creates them reproducibly. | 6 |
    | **Vagrantfile** | File (written in Ruby) describing the machines, their network and their configuration. | 6 |
    | **VirtualBox** | Open source type 2 hypervisor. | 3 |
    | **VLAN** | Virtual local area network: logical split of one physical network into separate broadcast domains (802.1Q standard). | 5 |
    | **VMware Workstation** | VMware's type 2 hypervisor, used in this course to host nested machines. | 3 |

## 3. Stockage / Storage

=== "Français"

    | Terme | Définition | Ch. |
    |---|---|---|
    | **Bucket** | Conteneur logique d'objets dans un stockage objet. | 11 |
    | **Ceph** | Système de stockage distribué libre qui offre des interfaces bloc, fichier et objet sur une grappe de serveurs. | 11 |
    | **Copy-on-write** | Technique où les données modifiées sont écrites ailleurs au lieu d'écraser les anciennes. Les snapshots LVM et ZFS s'appuient sur ce principe. | 9 |
    | **CRUSH** | Algorithme de Ceph qui calcule où placer les données sans annuaire central. | 11 |
    | **DAS / NAS / SAN** | *DAS* : stockage branché directement à un serveur. *NAS* : stockage partagé au niveau fichier sur le réseau (NFS, SMB). *SAN* : stockage au niveau bloc sur un réseau dédié (iSCSI, par exemple). | 10 |
    | **fio** | Outil de mesure des performances de stockage. | 7 |
    | **Hot spare** | Disque de réserve qui remplace automatiquement un disque défaillant dans une grappe RAID. | 8 |
    | **IOPS** | Opérations d'entrée-sortie par seconde, mesure de la capacité d'un stockage à traiter de petites requêtes. | 7 |
    | **IQN** | Nom unique (iSCSI Qualified Name) qui identifie une cible ou un initiateur iSCSI. | 10 |
    | **iSCSI** | Protocole qui transporte des commandes de stockage bloc sur un réseau IP. | 10 |
    | **Initiateur / cible (iSCSI)** | La cible expose des volumes ; l'initiateur est le client qui s'y connecte et les voit comme des disques locaux. | 10 |
    | **LUN** | Volume de stockage bloc présenté par une cible iSCSI. | 10 |
    | **LVM** | Gestionnaire de volumes logiques de Linux. Il organise les disques en volumes physiques (PV), groupes de volumes (VG) et volumes logiques (LV), que l'on peut redimensionner à chaud. | 9 |
    | **mdadm** | Outil Linux de gestion du RAID logiciel. | 8 |
    | **Multipath** | Utilisation de plusieurs chemins réseau vers un même stockage pour la tolérance de panne et la performance. | 10 |
    | **NFS** | Protocole de partage de fichiers en réseau, courant entre systèmes Unix et Linux. | 10 |
    | **NVMe / SATA / SAS** | Interfaces de connexion des disques. NVMe, conçue pour les SSD, offre les meilleures performances. | 7 |
    | **OSD (Ceph)** | Démon de Ceph qui gère un disque et stocke les données. | 11 |
    | **MON (Ceph)** | Démon de Ceph qui maintient la carte de la grappe ; un nombre impair de moniteurs permet de garder le quorum. | 11 |
    | **Parité** | Information de contrôle calculée à partir des données, qui permet de reconstruire un disque perdu (RAID 5 et 6). | 8 |
    | **Pool (Ceph / ZFS)** | Ensemble logique de stockage. Dans ZFS, le *pool* regroupe des disques ; dans Ceph, il regroupe des objets selon des règles de réplication. | 9, 11 |
    | **RAID** | Regroupement de plusieurs disques pour améliorer la tolérance de panne, la performance ou les deux. Niveaux étudiés : 0, 1, 5, 6 et 10. | 8 |
    | **RAID 0 / 1 / 5 / 6 / 10** | *0* : répartition sans redondance. *1* : miroir. *5* : répartition avec une parité (tolère une panne). *6* : deux parités (tolère deux pannes). *10* : miroirs répartis. | 8 |
    | **Reconstruction (rebuild)** | Recalcul des données d'un disque remplacé à partir des disques restants. Pendant cette phase, la grappe est plus vulnérable. | 8 |
    | **Samba / SMB** | SMB est le protocole de partage de fichiers de Windows ; Samba l'implémente sous Linux. | 10 |
    | **S3** | Interface de stockage objet d'Amazon, devenue un standard de fait. MinIO et LocalStack proposent une API compatible. | 11, 21 |
    | **Scrub (ZFS)** | Parcours des données qui vérifie leur intégrité grâce aux sommes de contrôle et répare les erreurs quand une copie saine existe. | 9 |
    | **Stockage défini par logiciel (SDS)** | Stockage dont l'intelligence est assurée par un logiciel sur du matériel standard, plutôt que par une baie propriétaire. | 11 |
    | **Stockage objet** | Stockage de données sous forme d'objets identifiés par une clé, accessibles par une API HTTP. | 11 |
    | **ZFS (OpenZFS)** | Système de fichiers et gestionnaire de volumes avec sommes de contrôle, snapshots et pools. | 9 |

=== "English"

    | Term | Definition | Ch. |
    |---|---|---|
    | **Bucket** | Logical container of objects in object storage. | 11 |
    | **Ceph** | Open source distributed storage system offering block, file and object interfaces on a cluster of servers. | 11 |
    | **Copy-on-write** | Technique where modified data is written elsewhere instead of overwriting the old data. LVM and ZFS snapshots rely on this principle. | 9 |
    | **CRUSH** | Ceph algorithm that computes where to place data without a central directory. | 11 |
    | **DAS / NAS / SAN** | *DAS*: storage attached directly to a server. *NAS*: file-level storage shared over the network (NFS, SMB). *SAN*: block-level storage on a dedicated network (iSCSI, for example). | 10 |
    | **fio** | Storage performance measurement tool. | 7 |
    | **Hot spare** | Spare disk that automatically replaces a failed disk in a RAID array. | 8 |
    | **IOPS** | Input/output operations per second, a measure of a storage system's ability to handle small requests. | 7 |
    | **IQN** | Unique name (iSCSI Qualified Name) identifying an iSCSI target or initiator. | 10 |
    | **iSCSI** | Protocol that carries block storage commands over an IP network. | 10 |
    | **Initiator / target (iSCSI)** | The target exposes volumes; the initiator is the client that connects to it and sees them as local disks. | 10 |
    | **LUN** | Block storage volume presented by an iSCSI target. | 10 |
    | **LVM** | Linux Logical Volume Manager. It organizes disks into physical volumes (PV), volume groups (VG) and logical volumes (LV), which can be resized online. | 9 |
    | **mdadm** | Linux tool for managing software RAID. | 8 |
    | **Multipath** | Use of several network paths to the same storage for fault tolerance and performance. | 10 |
    | **NFS** | Network file sharing protocol, common between Unix and Linux systems. | 10 |
    | **NVMe / SATA / SAS** | Disk connection interfaces. NVMe, designed for SSDs, offers the best performance. | 7 |
    | **OSD (Ceph)** | Ceph daemon that manages one disk and stores data. | 11 |
    | **MON (Ceph)** | Ceph daemon that maintains the cluster map; an odd number of monitors makes it possible to keep quorum. | 11 |
    | **Parity** | Check information computed from the data, which allows a lost disk to be rebuilt (RAID 5 and 6). | 8 |
    | **Pool (Ceph / ZFS)** | Logical storage group. In ZFS, the *pool* groups disks; in Ceph, it groups objects according to replication rules. | 9, 11 |
    | **RAID** | Grouping of several disks to improve fault tolerance, performance or both. Levels studied: 0, 1, 5, 6 and 10. | 8 |
    | **RAID 0 / 1 / 5 / 6 / 10** | *0*: striping without redundancy. *1*: mirroring. *5*: striping with one parity (tolerates one failure). *6*: two parities (tolerates two failures). *10*: striped mirrors. | 8 |
    | **Rebuild** | Recomputing the data of a replaced disk from the remaining disks. During this phase, the array is more vulnerable. | 8 |
    | **Samba / SMB** | SMB is the Windows file sharing protocol; Samba implements it on Linux. | 10 |
    | **S3** | Amazon's object storage interface, now a de facto standard. MinIO and LocalStack provide a compatible API. | 11, 21 |
    | **Scrub (ZFS)** | Pass over the data that checks its integrity using checksums and repairs errors when a healthy copy exists. | 9 |
    | **Software-defined storage (SDS)** | Storage whose intelligence is provided by software on commodity hardware, rather than by a proprietary array. | 11 |
    | **Object storage** | Storing data as objects identified by a key, accessible through an HTTP API. | 11 |
    | **ZFS (OpenZFS)** | File system and volume manager with checksums, snapshots and pools. | 9 |

## 4. Hyperviseurs serveur et plateformes / Server hypervisors and platforms

=== "Français"

    | Terme | Définition | Ch. |
    |---|---|---|
    | **Cloud-init** | Outil qui configure une instance au premier démarrage (utilisateur, clés SSH, réseau, paquets) à partir de données fournies par la plateforme. | 12 |
    | **Cluster** | Ensemble de machines (nœuds) qui coopèrent et sont gérées comme un tout. | 14, 16 |
    | **Corosync** | Moteur de communication entre les nœuds d'un cluster Proxmox VE (appartenance au cluster et quorum). | 14 |
    | **Datastore (vSphere)** | Espace de stockage où ESXi place les fichiers des VM : volume VMFS, partage NFS ou LUN iSCSI. | 15 |
    | **DRS** | Fonction vSphere qui équilibre la charge entre les hôtes d'un cluster en déplaçant des VM. | 16 |
    | **ESXi** | Hyperviseur de type 1 de VMware. | 15 |
    | **Fencing** | Isolement d'un nœud défaillant pour garantir qu'il ne continue pas à écrire sur le stockage avant le redémarrage de ses VM ailleurs. | 14 |
    | **Groupe de ports (port group)** | Regroupement de paramètres réseau (VLAN, politiques) auquel on rattache les cartes des VM sur un commutateur ESXi. | 15 |
    | **Haute disponibilité (HA)** | Capacité d'un service à continuer ou à redémarrer rapidement après une panne, par exemple en relançant les VM d'un nœud tombé sur un autre nœud. | 14, 16 |
    | **KVM** | Module du noyau Linux qui transforme Linux en hyperviseur de type 1. | 12 |
    | **libvirt / virsh** | libvirt est une bibliothèque et un démon de gestion de la virtualisation (KVM, QEMU…) ; `virsh` est son interface en ligne de commande. | 12 |
    | **LXC** | Conteneurs système Linux, gérés dans Proxmox VE à côté des machines virtuelles. | 13 |
    | **Image cloud** | Image de disque préinstallée, prévue pour être configurée au démarrage par cloud-init. | 12 |
    | **Migration à chaud (live migration)** | Déplacement d'une VM en fonctionnement d'un hôte à un autre, avec une interruption quasi nulle. Elle suppose en général un stockage partagé. | 14, 16 |
    | **Modèle (template)** | VM figée qui sert de base pour créer rapidement d'autres machines. | 13 |
    | **Proxmox VE** | Plateforme libre de virtualisation fondée sur KVM et LXC, avec interface web et gestion de cluster. | 13, 14 |
    | **QDevice** | Petit service externe qui apporte un vote supplémentaire à un cluster Proxmox VE pour maintenir le quorum, notamment avec deux nœuds. | 14 |
    | **QEMU** | Émulateur et moteur de virtualisation ; associé à KVM, il fournit les périphériques virtuels des VM. | 12 |
    | **Quorum** | Majorité de votes nécessaire pour qu'un cluster continue à fonctionner. Elle évite que deux moitiés séparées du cluster agissent chacune de leur côté. | 14 |
    | **Stockage partagé** | Stockage accessible par tous les nœuds d'un cluster (NFS, iSCSI, Ceph…). | 14 |
    | **vCenter** | Serveur de gestion centralisée de plusieurs hôtes ESXi et de leurs fonctions avancées. | 16 |
    | **vMotion** | Migration à chaud d'une VM entre deux hôtes ESXi. | 16 |
    | **vSphere** | Ensemble de la plateforme de virtualisation VMware (ESXi, vCenter et fonctions associées). | 15, 16 |
    | **vSphere HA** | Fonction qui redémarre automatiquement sur un autre hôte les VM d'un hôte ESXi en panne. | 16 |
    | **VMFS** | Système de fichiers de VMware conçu pour que plusieurs hôtes ESXi partagent un même volume de stockage. | 15 |

=== "English"

    | Term | Definition | Ch. |
    |---|---|---|
    | **Cloud-init** | Tool that configures an instance at first boot (user, SSH keys, network, packages) from data supplied by the platform. | 12 |
    | **Cluster** | Set of machines (nodes) that cooperate and are managed as a whole. | 14, 16 |
    | **Corosync** | Communication engine between the nodes of a Proxmox VE cluster (membership and quorum). | 14 |
    | **Datastore (vSphere)** | Storage space where ESXi places VM files: VMFS volume, NFS share or iSCSI LUN. | 15 |
    | **DRS** | vSphere feature that balances load between the hosts of a cluster by moving VMs. | 16 |
    | **ESXi** | VMware's type 1 hypervisor. | 15 |
    | **Fencing** | Isolating a failed node to guarantee it does not keep writing to storage before its VMs are restarted elsewhere. | 14 |
    | **Port group** | Group of network settings (VLAN, policies) to which VM network cards are attached on an ESXi switch. | 15 |
    | **High availability (HA)** | Ability of a service to keep running or to restart quickly after a failure, for example by restarting the VMs of a failed node on another node. | 14, 16 |
    | **KVM** | Linux kernel module that turns Linux into a type 1 hypervisor. | 12 |
    | **libvirt / virsh** | libvirt is a virtualization management library and daemon (KVM, QEMU...); `virsh` is its command-line interface. | 12 |
    | **LXC** | Linux system containers, managed in Proxmox VE alongside virtual machines. | 13 |
    | **Cloud image** | Preinstalled disk image meant to be configured at boot by cloud-init. | 12 |
    | **Live migration** | Moving a running VM from one host to another with almost no interruption. It generally requires shared storage. | 14, 16 |
    | **Template** | Frozen VM used as a base to quickly create other machines. | 13 |
    | **Proxmox VE** | Open source virtualization platform based on KVM and LXC, with a web interface and cluster management. | 13, 14 |
    | **QDevice** | Small external service that adds an extra vote to a Proxmox VE cluster to maintain quorum, notably with two nodes. | 14 |
    | **QEMU** | Emulator and virtualization engine; paired with KVM, it provides the VMs' virtual devices. | 12 |
    | **Quorum** | Majority of votes needed for a cluster to keep operating. It prevents two separated halves of the cluster from each acting on their own. | 14 |
    | **Shared storage** | Storage accessible by all nodes of a cluster (NFS, iSCSI, Ceph...). | 14 |
    | **vCenter** | Central management server for several ESXi hosts and their advanced features. | 16 |
    | **vMotion** | Live migration of a VM between two ESXi hosts. | 16 |
    | **vSphere** | VMware's whole virtualization platform (ESXi, vCenter and related features). | 15, 16 |
    | **vSphere HA** | Feature that automatically restarts the VMs of a failed ESXi host on another host. | 16 |
    | **VMFS** | VMware file system designed so that several ESXi hosts can share one storage volume. | 15 |

## 5. Cloud, Infrastructure as Code et exploitation / Cloud, Infrastructure as Code and operations

=== "Français"

    | Terme | Définition | Ch. |
    |---|---|---|
    | **Adresse flottante (floating IP)** | Adresse IP publique que l'on associe à une instance OpenStack pour la rendre joignable de l'extérieur. | 19 |
    | **Ansible** | Outil de configuration qui s'exécute sans agent (généralement par SSH) à partir de fichiers YAML. | 20 |
    | **API REST** | Interface programmable fondée sur HTTP, par laquelle un outil pilote une plateforme (Proxmox VE, OpenStack, services cloud). | 17 |
    | **Cinder** | Service OpenStack de volumes de stockage bloc. | 18, 19 |
    | **Cloud computing** | Accès à la demande, par le réseau, à des ressources informatiques mutualisées, facturées ou mesurées selon l'usage. | 17 |
    | **Cloud privé / public / hybride** | *Privé* : infrastructure dédiée à une organisation. *Public* : infrastructure d'un fournisseur partagée entre clients. *Hybride* : combinaison des deux. | 17 |
    | **DevStack** | Ensemble de scripts qui installe un OpenStack complet sur une seule machine, destiné à l'apprentissage et aux tests. Il ne convient pas à la production. | 18 |
    | **Exporter** | Programme qui expose des métriques à Prometheus (par exemple `node_exporter` pour les métriques d'une machine Linux). | 22 |
    | **Flavor (gabarit)** | Définition OpenStack des ressources d'une instance : vCPU, mémoire, disque. | 19 |
    | **Glance** | Service OpenStack de gestion des images. | 18 |
    | **Grafana** | Outil de visualisation qui construit des tableaux de bord à partir de sources de données comme Prometheus. | 22 |
    | **Horizon** | Interface web d'OpenStack. | 18 |
    | **IaaS / PaaS / SaaS** | Modèles de service. *IaaS* : infrastructure (machines, réseau, stockage). *PaaS* : plateforme d'exécution d'applications. *SaaS* : application complète fournie en service. | 17 |
    | **IAM** | Gestion des identités et des accès : qui peut faire quoi sur quelles ressources. | 21 |
    | **Idempotence** | Propriété d'une opération qui donne le même résultat qu'on l'exécute une ou plusieurs fois. Elle est centrale dans Ansible. | 20 |
    | **Infrastructure as Code (IaC)** | Pratique qui consiste à décrire l'infrastructure dans des fichiers versionnés plutôt qu'à la configurer à la main. | 20 |
    | **Instance** | Machine virtuelle lancée dans un cloud. | 19 |
    | **Inventaire (Ansible)** | Liste des machines (et de leurs groupes) que cible Ansible. | 20 |
    | **Keystone** | Service OpenStack d'identité et d'authentification. | 18 |
    | **LocalStack** | Outil qui émule localement certains services d'un cloud public, ici ceux d'AWS. | 21 |
    | **Métrique** | Mesure numérique d'un système (utilisation du processeur, mémoire, latence…), relevée à intervalles réguliers. | 22 |
    | **Neutron** | Service OpenStack de réseau. | 18, 19 |
    | **Nova** | Service OpenStack de calcul, qui crée et gère les instances. | 18, 19 |
    | **OpenStack** | Ensemble de services libres qui permettent de construire un cloud privé de type IaaS. | 18, 19 |
    | **OpenTofu** | Outil libre d'infrastructure as code déclaratif, dérivé de Terraform. | 20 |
    | **Playbook (Ansible)** | Fichier YAML qui décrit les tâches à appliquer à un groupe de machines. | 20 |
    | **Plan / apply (OpenTofu)** | `plan` montre les changements prévus ; `apply` les exécute. | 20 |
    | **PRA / PCA** | Plan de reprise d'activité (rétablir le service après un sinistre) et plan de continuité d'activité (maintenir le service pendant l'incident). | 22 |
    | **Prometheus** | Système de supervision qui collecte et stocke des métriques sous forme de séries temporelles et les interroge avec le langage PromQL. | 22 |
    | **Provider (OpenTofu)** | Extension qui permet à OpenTofu de piloter une plateforme donnée (libvirt, Proxmox VE, OpenStack…). | 20 |
    | **Région / zone de disponibilité** | Dans un cloud public, une région est une zone géographique ; une zone de disponibilité est un site isolé au sein d'une région. | 21 |
    | **Règle 3-2-1** | Bonne pratique de sauvegarde : 3 copies des données, sur 2 supports différents, dont 1 hors site. | 22 |
    | **restic** | Outil de sauvegarde libre avec chiffrement et déduplication. | 22 |
    | **RPO / RTO** | *RPO* : durée maximale de données que l'on accepte de perdre. *RTO* : durée maximale pendant laquelle on accepte que le service soit indisponible. | 22 |
    | **Groupe de sécurité (security group)** | Ensemble de règles de filtrage réseau appliquées aux instances d'un cloud. | 19 |
    | **SLA** | Accord de niveau de service : engagement de disponibilité pris par un fournisseur. | 17, 21 |
    | **Supervision (monitoring)** | Collecte et analyse continues de métriques et d'événements pour détecter les problèmes. | 22 |
    | **État (state, OpenTofu)** | Fichier où OpenTofu enregistre les ressources qu'il gère. Il peut contenir des informations sensibles et ne doit pas être publié. | 20 |

=== "English"

    | Term | Definition | Ch. |
    |---|---|---|
    | **Floating IP** | Public IP address associated with an OpenStack instance to make it reachable from outside. | 19 |
    | **Ansible** | Configuration tool that runs agentless (usually over SSH) from YAML files. | 20 |
    | **REST API** | Programmable interface based on HTTP, through which a tool drives a platform (Proxmox VE, OpenStack, cloud services). | 17 |
    | **Cinder** | OpenStack block storage volume service. | 18, 19 |
    | **Cloud computing** | On-demand network access to shared computing resources, billed or measured by usage. | 17 |
    | **Private / public / hybrid cloud** | *Private*: infrastructure dedicated to one organization. *Public*: provider infrastructure shared between customers. *Hybrid*: a combination of both. | 17 |
    | **DevStack** | Set of scripts that installs a complete OpenStack on a single machine, meant for learning and testing. It is not suitable for production. | 18 |
    | **Exporter** | Program that exposes metrics to Prometheus (for example `node_exporter` for a Linux machine's metrics). | 22 |
    | **Flavor** | OpenStack definition of an instance's resources: vCPUs, memory, disk. | 19 |
    | **Glance** | OpenStack image management service. | 18 |
    | **Grafana** | Visualization tool that builds dashboards from data sources such as Prometheus. | 22 |
    | **Horizon** | OpenStack web interface. | 18 |
    | **IaaS / PaaS / SaaS** | Service models. *IaaS*: infrastructure (machines, network, storage). *PaaS*: application runtime platform. *SaaS*: complete application delivered as a service. | 17 |
    | **IAM** | Identity and access management: who can do what on which resources. | 21 |
    | **Idempotence** | Property of an operation that gives the same result whether it is run once or several times. It is central to Ansible. | 20 |
    | **Infrastructure as Code (IaC)** | Practice of describing infrastructure in versioned files rather than configuring it by hand. | 20 |
    | **Instance** | Virtual machine launched in a cloud. | 19 |
    | **Inventory (Ansible)** | List of machines (and their groups) targeted by Ansible. | 20 |
    | **Keystone** | OpenStack identity and authentication service. | 18 |
    | **LocalStack** | Tool that emulates some public cloud services locally, here those of AWS. | 21 |
    | **Metric** | Numerical measurement of a system (CPU usage, memory, latency...), collected at regular intervals. | 22 |
    | **Neutron** | OpenStack networking service. | 18, 19 |
    | **Nova** | OpenStack compute service, which creates and manages instances. | 18, 19 |
    | **OpenStack** | Set of open source services for building an IaaS private cloud. | 18, 19 |
    | **OpenTofu** | Open source declarative infrastructure as code tool, derived from Terraform. | 20 |
    | **Playbook (Ansible)** | YAML file describing the tasks to apply to a group of machines. | 20 |
    | **Plan / apply (OpenTofu)** | `plan` shows the planned changes; `apply` carries them out. | 20 |
    | **DRP / BCP** | Disaster recovery plan (restore service after a disaster) and business continuity plan (keep service running during the incident). In French: PRA / PCA. | 22 |
    | **Prometheus** | Monitoring system that collects and stores metrics as time series and queries them with the PromQL language. | 22 |
    | **Provider (OpenTofu)** | Plugin that lets OpenTofu drive a given platform (libvirt, Proxmox VE, OpenStack...). | 20 |
    | **Region / availability zone** | In a public cloud, a region is a geographic area; an availability zone is an isolated site within a region. | 21 |
    | **3-2-1 rule** | Backup best practice: 3 copies of the data, on 2 different media, with 1 off-site. | 22 |
    | **restic** | Open source backup tool with encryption and deduplication. | 22 |
    | **RPO / RTO** | *RPO*: maximum amount of data (as a time window) you accept to lose. *RTO*: maximum time you accept the service to be unavailable. | 22 |
    | **Security group** | Set of network filtering rules applied to the instances of a cloud. | 19 |
    | **SLA** | Service level agreement: availability commitment made by a provider. | 17, 21 |
    | **Monitoring** | Continuous collection and analysis of metrics and events to detect problems. | 22 |
    | **State (OpenTofu)** | File where OpenTofu records the resources it manages. It may contain sensitive information and must not be published. | 20 |

## Voir aussi / See also

- [Environnement de TP / Lab environment](environnement.md)
- [Correspondance des termes entre hyperviseurs / Hypervisor terminology mapping](correspondance-hyperviseurs.md)
- [Aide-mémoire des commandes / Command cheat sheet](aide-memoire-commandes.md)