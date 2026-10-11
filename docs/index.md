---
title: Syllabus
template: home.html
hide:
  - navigation
  - toc
---

## Syllabus { #syllabus }

!!! info "Site en construction"
    Toutes les pages existent ; celles marquées « Under construction » sont en cours de rédaction et seront complétées au fil de l'avancement du cours.

### Fiche d'identification

| Rubrique | Description |
|---|---|
| Intitulé | Cloud et virtualisation : concevoir et exploiter une infrastructure virtualisée |
| Public | Master DevOps et Cloud Computing, semestre 1 |
| Volume horaire | 84 h (28 séances de 3 h : environ 22 h de cours, 62 h de TP et de projets) |
| Crédits | 6 ECTS |
| Prérequis | Administration Linux, réseau (TCP/IP, DNS, routage), notions de scripting shell. Aucune connaissance préalable de la virtualisation ni du cloud. |
| Outils de TP | VirtualBox, VMware Workstation, Hyper-V, Vagrant, KVM et libvirt, Proxmox VE, VMware ESXi et vCenter (quota gratuit et essai de 60 jours), DevStack (OpenStack), OpenTofu, Ansible, LocalStack ; invités Ubuntu Server 24.04 LTS ; trois profils de machine (A : 16 Go, B : 16 Go avec VM de 8 Go, C : 32 Go) ; voir [l'environnement de TP](annexes/environnement.md) |

### Objectif général

Concevoir, déployer, automatiser et exploiter une infrastructure virtualisée (réseau, stockage, hyperviseurs, cloud privé) et situer ses choix par rapport aux offres du cloud public, avec des outils open source ou à quota gratuit.

### Acquis d'apprentissage

À la fin du cours, l'étudiant est capable de :

- **AA1** : décrire l'architecture d'un datacenter et expliquer les principes de la virtualisation (comprendre)
- **AA2** : installer, configurer et comparer les hyperviseurs de poste : VirtualBox, VMware Workstation et Hyper-V (appliquer)
- **AA3** : concevoir et configurer la connectivité réseau entre machines virtuelles (appliquer)
- **AA4** : automatiser la création d'environnements virtuels avec Vagrant et cloud-init (créer)
- **AA5** : expliquer les technologies, les interfaces et les architectures de stockage (comprendre)
- **AA6** : configurer un stockage virtualisé (RAID, LVM, NFS, iSCSI) et diagnostiquer une panne de disque (analyser)
- **AA7** : déployer et administrer des hyperviseurs serveur et leurs fonctions de cluster : KVM, Proxmox VE, ESXi et vCenter (appliquer)
- **AA8** : expliquer les modèles du cloud et déployer un cloud privé OpenStack (analyser)
- **AA9** : automatiser une infrastructure par le code et évaluer des services de cloud public selon coût, sécurité et responsabilité partagée (créer, évaluer)

### Compétences visées

=== "Savoirs"

    Architecture et composants d'un datacenter, principes de la virtualisation (CPU, mémoire, E/S), hyperviseurs de types 1 et 2, réseaux virtuels, interfaces et technologies de stockage, RAID, LVM, stockage en réseau (NAS, SAN, NFS, iSCSI), stockage défini par logiciel, concepts de vSphere, Proxmox VE et OpenStack, modèles de service et de déploiement du cloud.

=== "Savoir-faire"

    Administrer VirtualBox, VMware Workstation et Hyper-V, connecter des machines virtuelles, décrire un environnement avec Vagrant, configurer RAID, LVM, NFS et iSCSI, administrer KVM, Proxmox VE et ESXi, mettre en place cluster, migration et haute disponibilité, déployer des instances OpenStack, automatiser avec OpenTofu et Ansible, superviser et sauvegarder.

=== "Savoir-être"

    Rigueur et reproductibilité (infrastructure as code), esprit critique dans le choix d'une plateforme, souci des coûts et de la sécurité, travail en binôme, veille technologique.

### Organisation du cours

#### Partie 1 : Datacenter et virtualisation sur poste de travail (21 h) { #partie-1 }

[Présentation de la Partie 1](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/index.md)

| Chapitre | Contenu | Ressources |
|---|---|---|
| 1. Architecture et composants d'un datacenter (AA1) | Salles et baies, serveurs rack et lames, alimentation, refroidissement, PUE, niveaux Tier, réseau spine-leaf, stockage, sécurité physique, edge | [Cours](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch01-architecture-et-composants-d-un-datacenter/cours.md) · [Lab 1.1 : Dimensionner une salle serveur](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch01-architecture-et-composants-d-un-datacenter/lab-1-1-dimensionner-une-salle-serveur.md) · [Lab 1.2 : Inventorier une machine](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch01-architecture-et-composants-d-un-datacenter/lab-1-2-inventorier-une-machine.md) · [Quiz](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch01-architecture-et-composants-d-un-datacenter/quiz.md) |
| 2. Principes de la virtualisation (AA1) | Machine virtuelle et conteneur, hyperviseurs de types 1 et 2, VT-x et AMD-V, virtualisation du CPU, de la mémoire (ballooning, surallocation) et des E/S (virtio, passthrough), virtualisation imbriquée | [Cours](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch02-principes-de-la-virtualisation/cours.md) · [Lab 2.1 : Préparer le poste pour la virtualisation](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) · [Quiz](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch02-principes-de-la-virtualisation/quiz.md) |
| 3. VirtualBox et VMware Workstation (AA2) | Création de VM, additions invitées, snapshots, clones, formats VDI et VMDK, import et export OVF, virtualisation imbriquée | [Cours](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch03-virtualbox-et-vmware-workstation/cours.md) · [Lab 3.1 : VirtualBox : une VM Ubuntu Server](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch03-virtualbox-et-vmware-workstation/lab-3-1-virtualbox-une-vm-ubuntu-server.md) · [Lab 3.2 : VMware Workstation : même VM, import et export](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch03-virtualbox-et-vmware-workstation/lab-3-2-vmware-workstation-meme-vm-import-et-export.md) · [Quiz](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch03-virtualbox-et-vmware-workstation/quiz.md) |
| 4. Hyper-V (AA2) | Éditions de Windows compatibles, génération 1 et 2, disques VHDX, checkpoints, coexistence avec les autres hyperviseurs | [Cours](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch04-hyper-v/cours.md) · [Lab 4.1 : Activer Hyper-V et créer une VM](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch04-hyper-v/lab-4-1-activer-hyper-v-et-creer-une-vm.md) · [Lab 4.2 : Checkpoints, disques et export](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch04-hyper-v/lab-4-2-checkpoints-disques-et-export.md) · [Quiz](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch04-hyper-v/quiz.md) |
| 5. Connexion entre VM et réseaux virtuels (AA3) | Modes NAT, pont, hôte seul et réseau interne, commutateur virtuel, VLAN, tableau de correspondance entre outils, routage entre VM | [Cours](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch05-connexion-entre-vm-et-reseaux-virtuels/cours.md) · [Lab 5.1 : Réseaux virtuels sous VirtualBox et VMware](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch05-connexion-entre-vm-et-reseaux-virtuels/lab-5-1-reseaux-virtuels-sous-virtualbox-et-vmware.md) · [Lab 5.2 : Commutateurs virtuels sous Hyper-V](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch05-connexion-entre-vm-et-reseaux-virtuels/lab-5-2-commutateurs-virtuels-sous-hyper-v.md) · [Lab 5.3 : Routage, NAT et VLAN sous Linux](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch05-connexion-entre-vm-et-reseaux-virtuels/lab-5-3-routage-nat-et-vlan-sous-linux.md) · [Quiz](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch05-connexion-entre-vm-et-reseaux-virtuels/quiz.md) |
| 6. Vagrant : décrire ses environnements (AA4) | Vagrantfile, boxes, providers (VirtualBox, VMware, Hyper-V, libvirt), réseau, dossiers synchronisés, provisioning shell et Ansible, multi-machines | [Cours](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch06-vagrant-decrire-ses-environnements/cours.md) · [Lab 6.1 : Premier Vagrantfile](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch06-vagrant-decrire-ses-environnements/lab-6-1-premier-vagrantfile.md) · [Lab 6.2 : Environnement multi-machines avec provisioning](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch06-vagrant-decrire-ses-environnements/lab-6-2-environnement-multi-machines-avec-provisioning.md) · [Lab 6.3 : Même environnement sur un autre provider](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch06-vagrant-decrire-ses-environnements/lab-6-3-meme-environnement-sur-un-autre-provider.md) · [Quiz](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/ch06-vagrant-decrire-ses-environnements/quiz.md) |
| **Mini-projet 1** | Laboratoire réseau reproductible | [Énoncé](partie-1-datacenter-et-virtualisation-sur-poste-de-travail/mini-projet-1.md) |

#### Partie 2 : Virtualisation du stockage (18 h) { #partie-2 }

[Présentation de la Partie 2](partie-2-virtualisation-du-stockage/index.md)

| Chapitre | Contenu | Ressources |
|---|---|---|
| 7. Technologies et interfaces de stockage (AA5) | HDD, SSD et NVMe ; interfaces SATA, SAS, SCSI, NVMe et Fibre Channel ; IOPS, débit, latence ; HBA ; contrôleurs virtuels ; formats raw, qcow2, VMDK, VDI et VHDX ; provisionnement léger et épais | [Cours](partie-2-virtualisation-du-stockage/ch07-technologies-et-interfaces-de-stockage/cours.md) · [Lab 7.1 : Disques virtuels : ajouter, convertir, comparer](partie-2-virtualisation-du-stockage/ch07-technologies-et-interfaces-de-stockage/lab-7-1-disques-virtuels-ajouter-convertir-comparer.md) · [Lab 7.2 : Mesurer les performances avec fio](partie-2-virtualisation-du-stockage/ch07-technologies-et-interfaces-de-stockage/lab-7-2-mesurer-les-performances-avec-fio.md) · [Quiz](partie-2-virtualisation-du-stockage/ch07-technologies-et-interfaces-de-stockage/quiz.md) |
| 8. RAID (AA5, AA6) | Niveaux 0, 1, 5, 6 et 10, parité, disque de secours, RAID matériel, logiciel et pseudo-matériel, reconstruction, calcul de capacité | [Cours](partie-2-virtualisation-du-stockage/ch08-raid/cours.md) · [Lab 8.1 : RAID 1 et RAID 5 avec mdadm](partie-2-virtualisation-du-stockage/ch08-raid/lab-8-1-raid-1-et-raid-5-avec-mdadm.md) · [Lab 8.2 : Panne, remplacement et reconstruction](partie-2-virtualisation-du-stockage/ch08-raid/lab-8-2-panne-remplacement-et-reconstruction.md) · [Lab 8.3 : Choisir un niveau RAID selon un cahier des charges](partie-2-virtualisation-du-stockage/ch08-raid/lab-8-3-choisir-un-niveau-raid-selon-un-cahier-des-charges.md) · [Quiz](partie-2-virtualisation-du-stockage/ch08-raid/quiz.md) |
| 9. Disques, LVM et systèmes de fichiers (AA6) | MBR et GPT, ext4 et XFS, LVM (PV, VG, LV), extension à chaud, snapshots, ZFS (aperçu) | [Cours](partie-2-virtualisation-du-stockage/ch09-disques-lvm-et-systemes-de-fichiers/cours.md) · [Lab 9.1 : Partitions et systèmes de fichiers](partie-2-virtualisation-du-stockage/ch09-disques-lvm-et-systemes-de-fichiers/lab-9-1-partitions-et-systemes-de-fichiers.md) · [Lab 9.2 : LVM : extension à chaud et snapshot](partie-2-virtualisation-du-stockage/ch09-disques-lvm-et-systemes-de-fichiers/lab-9-2-lvm-extension-a-chaud-et-snapshot.md) · [Lab 9.3 : ZFS : pools et snapshots](partie-2-virtualisation-du-stockage/ch09-disques-lvm-et-systemes-de-fichiers/lab-9-3-zfs-pools-et-snapshots.md) · [Quiz](partie-2-virtualisation-du-stockage/ch09-disques-lvm-et-systemes-de-fichiers/quiz.md) |
| 10. Stockage en réseau (AA5, AA6) | DAS, NAS et SAN ; NFS et SMB ; iSCSI (cible, initiateur, IQN, LUN, CHAP) ; Fibre Channel et FCoE (concepts) ; multipath | [Cours](partie-2-virtualisation-du-stockage/ch10-stockage-en-reseau/cours.md) · [Lab 10.1 : NFS et Samba](partie-2-virtualisation-du-stockage/ch10-stockage-en-reseau/lab-10-1-nfs-et-samba.md) · [Lab 10.2 : Cible et initiateur iSCSI](partie-2-virtualisation-du-stockage/ch10-stockage-en-reseau/lab-10-2-cible-et-initiateur-iscsi.md) · [Lab 10.3 : Multipath sur deux réseaux](partie-2-virtualisation-du-stockage/ch10-stockage-en-reseau/lab-10-3-multipath-sur-deux-reseaux.md) · [Quiz](partie-2-virtualisation-du-stockage/ch10-stockage-en-reseau/quiz.md) |
| 11. Stockage défini par logiciel (AA5, AA7) | Ceph (RADOS, OSD, RBD, CephFS), vSAN (concept), stockage objet S3, déduplication et compression | [Cours](partie-2-virtualisation-du-stockage/ch11-stockage-defini-par-logiciel/cours.md) · [Lab 11.1 : Stockage objet avec MinIO](partie-2-virtualisation-du-stockage/ch11-stockage-defini-par-logiciel/lab-11-1-stockage-objet-avec-minio.md) · [Lab 11.2 : Grappe Ceph à trois nœuds](partie-2-virtualisation-du-stockage/ch11-stockage-defini-par-logiciel/lab-11-2-grappe-ceph-a-trois-nuds.md) · [Quiz](partie-2-virtualisation-du-stockage/ch11-stockage-defini-par-logiciel/quiz.md) |
| **Mini-projet 2** | Serveur de stockage d'entreprise | [Énoncé](partie-2-virtualisation-du-stockage/mini-projet-2.md) |

#### Partie 3 : Hyperviseurs serveur et plateformes d'entreprise (18 h) { #partie-3 }

[Présentation de la Partie 3](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/index.md)

| Chapitre | Contenu | Ressources |
|---|---|---|
| 12. KVM, QEMU et libvirt (AA4, AA7) | Architecture KVM et QEMU, virsh, virt-install, réseaux et pools de stockage libvirt, images cloud et cloud-init | [Cours](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch12-kvm-qemu-et-libvirt/cours.md) · [Lab 12.1 : Premier invité KVM](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch12-kvm-qemu-et-libvirt/lab-12-1-premier-invite-kvm.md) · [Lab 12.2 : Réseaux et stockage libvirt](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch12-kvm-qemu-et-libvirt/lab-12-2-reseaux-et-stockage-libvirt.md) · [Lab 12.3 : Images cloud et cloud-init](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch12-kvm-qemu-et-libvirt/lab-12-3-images-cloud-et-cloud-init.md) · [Quiz](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch12-kvm-qemu-et-libvirt/quiz.md) |
| 13. Proxmox VE : installation et exploitation (AA7) | Architecture, installation, VM et conteneurs LXC, stockage, réseau (ponts vmbr), modèles, sauvegardes avec vzdump | [Cours](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch13-proxmox-ve-installation-et-exploitation/cours.md) · [Lab 13.1 : Installer Proxmox VE en VM imbriquée](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch13-proxmox-ve-installation-et-exploitation/lab-13-1-installer-proxmox-ve-en-vm-imbriquee.md) · [Lab 13.2 : VM, conteneurs LXC et modèles](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch13-proxmox-ve-installation-et-exploitation/lab-13-2-vm-conteneurs-lxc-et-modeles.md) · [Lab 13.3 : Stockage et sauvegardes](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch13-proxmox-ve-installation-et-exploitation/lab-13-3-stockage-et-sauvegardes.md) · [Quiz](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch13-proxmox-ve-installation-et-exploitation/quiz.md) |
| 14. Proxmox VE : cluster et haute disponibilité (AA7) | Cluster, quorum et QDevice, stockage partagé, migration à chaud, haute disponibilité, aperçu de Ceph intégré | [Cours](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch14-proxmox-ve-cluster-et-haute-disponibilite/cours.md) · [Lab 14.1 : Constituer un cluster Proxmox VE](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch14-proxmox-ve-cluster-et-haute-disponibilite/lab-14-1-constituer-un-cluster-proxmox-ve.md) · [Lab 14.2 : Migration à chaud sur stockage partagé](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch14-proxmox-ve-cluster-et-haute-disponibilite/lab-14-2-migration-a-chaud-sur-stockage-partage.md) · [Lab 14.3 : Panne de nœud et haute disponibilité](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch14-proxmox-ve-cluster-et-haute-disponibilite/lab-14-3-panne-de-nud-et-haute-disponibilite.md) · [Quiz](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch14-proxmox-ve-cluster-et-haute-disponibilite/quiz.md) |
| 15. VMware vSphere (1) : concepts et ESXi gratuit (AA7) | ESXi, vCenter, VMFS, datastores, commutateurs standard et distribués, vMotion, Storage vMotion, HA, DRS, FT, vSAN, NSX (survol), modèle de licences | [Cours](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch15-vmware-vsphere-1-concepts-et-esxi-gratuit/cours.md) · [Lab 15.1 : Installer ESXi gratuit en VM imbriquée](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch15-vmware-vsphere-1-concepts-et-esxi-gratuit/lab-15-1-installer-esxi-gratuit-en-vm-imbriquee.md) · [Lab 15.2 : VM, datastore, commutateur et groupes de ports](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch15-vmware-vsphere-1-concepts-et-esxi-gratuit/lab-15-2-vm-datastore-commutateur-et-groupes-de-ports.md) · [Lab 15.3 : Datastore NFS ou iSCSI](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch15-vmware-vsphere-1-concepts-et-esxi-gratuit/lab-15-3-datastore-nfs-ou-iscsi.md) · [Quiz](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch15-vmware-vsphere-1-concepts-et-esxi-gratuit/quiz.md) |
| 16. VMware vSphere (2) : vCenter et fonctions avancées (AA7) | VCSA, inventaire, clusters, vMotion, HA, DRS, essai de 60 jours | [Cours](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch16-vmware-vsphere-2-vcenter-et-fonctions-avancees/cours.md) · [Lab 16.1 : Déployer vCenter](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch16-vmware-vsphere-2-vcenter-et-fonctions-avancees/lab-16-1-deployer-vcenter.md) · [Lab 16.2 : Cluster et vMotion](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch16-vmware-vsphere-2-vcenter-et-fonctions-avancees/lab-16-2-cluster-et-vmotion.md) · [Lab 16.3 : HA et DRS](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch16-vmware-vsphere-2-vcenter-et-fonctions-avancees/lab-16-3-ha-et-drs.md) · [Quiz](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/ch16-vmware-vsphere-2-vcenter-et-fonctions-avancees/quiz.md) |
| **Mini-projet 3** | Plateformes de virtualisation comparées | [Énoncé](partie-3-hyperviseurs-serveur-et-plateformes-d-entreprise/mini-projet-3.md) |

#### Partie 4 : Cloud privé, Infrastructure as Code et cloud public (21 h) { #partie-4 }

[Présentation de la Partie 4](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/index.md)

| Chapitre | Contenu | Ressources |
|---|---|---|
| 17. Modèles et architectures du cloud (AA8) | Définition du NIST, IaaS, PaaS et SaaS, modèles de déploiement, responsabilité partagée, du virtualisé au cloud (API, libre-service, élasticité) | [Cours](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch17-modeles-et-architectures-du-cloud/cours.md) · [Lab 17.1 : Étude de cas : choisir un modèle de cloud](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch17-modeles-et-architectures-du-cloud/lab-17-1-etude-de-cas-choisir-un-modele-de-cloud.md) · [Lab 17.2 : Piloter Proxmox VE par son API REST](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch17-modeles-et-architectures-du-cloud/lab-17-2-piloter-proxmox-ve-par-son-api-rest.md) · [Quiz](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch17-modeles-et-architectures-du-cloud/quiz.md) |
| 18. OpenStack : architecture et services (AA8) | Keystone, Glance, Nova, Neutron, Cinder, Swift, Horizon, Heat | [Cours](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch18-openstack-architecture-et-services/cours.md) · [Lab 18.1 : Installer DevStack](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch18-openstack-architecture-et-services/lab-18-1-installer-devstack.md) · [Lab 18.2 : Explorer Horizon et la ligne de commande](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch18-openstack-architecture-et-services/lab-18-2-explorer-horizon-et-la-ligne-de-commande.md) · [Quiz](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch18-openstack-architecture-et-services/quiz.md) |
| 19. OpenStack en pratique (AA8) | Images, flavors, réseaux Neutron, adresses flottantes, groupes de sécurité, volumes Cinder | [Cours](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch19-openstack-en-pratique/cours.md) · [Lab 19.1 : Lancer une instance](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch19-openstack-en-pratique/lab-19-1-lancer-une-instance.md) · [Lab 19.2 : Réseaux et adresses flottantes](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch19-openstack-en-pratique/lab-19-2-reseaux-et-adresses-flottantes.md) · [Lab 19.3 : Volumes et snapshots](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch19-openstack-en-pratique/lab-19-3-volumes-et-snapshots.md) · [Quiz](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch19-openstack-en-pratique/quiz.md) |
| 20. Infrastructure as Code : OpenTofu et Ansible (AA4, AA9) | OpenTofu (HCL, état, providers), Ansible (inventaire, playbooks, rôles), articulation entre provisionnement et configuration | [Cours](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch20-infrastructure-as-code-opentofu-et-ansible/cours.md) · [Lab 20.1 : OpenTofu avec libvirt](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch20-infrastructure-as-code-opentofu-et-ansible/lab-20-1-opentofu-avec-libvirt.md) · [Lab 20.2 : OpenTofu avec Proxmox VE ou OpenStack](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch20-infrastructure-as-code-opentofu-et-ansible/lab-20-2-opentofu-avec-proxmox-ve-ou-openstack.md) · [Lab 20.3 : Configurer avec Ansible](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch20-infrastructure-as-code-opentofu-et-ansible/lab-20-3-configurer-avec-ansible.md) · [Quiz](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch20-infrastructure-as-code-opentofu-et-ansible/quiz.md) |
| 21. Cloud public : AWS, Azure et GCP (AA9) | Équivalence des services (calcul, stockage, réseau, identité), régions et zones, offres gratuites et crédits d'essai, FinOps, sécurité | [Cours](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch21-cloud-public-aws-azure-et-gcp/cours.md) · [Lab 21.1 : Stockage objet et IAM avec LocalStack](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch21-cloud-public-aws-azure-et-gcp/lab-21-1-stockage-objet-et-iam-avec-localstack.md) · [Lab 21.2 : Estimer et comparer les coûts](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch21-cloud-public-aws-azure-et-gcp/lab-21-2-estimer-et-comparer-les-couts.md) · [Quiz](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch21-cloud-public-aws-azure-et-gcp/quiz.md) |
| 22. Exploitation et continuité de service (AA6, AA9) | Supervision, sauvegarde, PRA et PCA, durcissement des hyperviseurs | [Cours](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch22-exploitation-et-continuite-de-service/cours.md) · [Lab 22.1 : Supervision avec Prometheus et Grafana](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch22-exploitation-et-continuite-de-service/lab-22-1-supervision-avec-prometheus-et-grafana.md) · [Lab 22.2 : Sauvegarde et restauration avec restic](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch22-exploitation-et-continuite-de-service/lab-22-2-sauvegarde-et-restauration-avec-restic.md) · [Quiz](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/ch22-exploitation-et-continuite-de-service/quiz.md) |
| **Mini-projet 4** | Cloud privé automatisé | [Énoncé](partie-4-cloud-prive-infrastructure-as-code-et-cloud-public/mini-projet-4.md) |

#### Projet final

[Mini-datacenter virtuel](projet-final.md) (6 h, Binôme, 6 h en séance et travail personnel, dossier et soutenance)

### Méthodes pédagogiques

Cours interactifs courts, TP guidés puis autonomes sur machines virtuelles, études de cas, mini-projets en binôme à la fin de chaque partie, projet final de synthèse. Autonomie croissante : guidé, semi-guidé, autonome.

### Évaluation

| Modalité | Pondération |
|---|---|
| Contrôle continu : TP notés et quiz | 30 % |
| Mini-projets et projet final : réalisation, dossier et soutenance | 40 % |
| Examen pratique chronométré | 30 % |

### Références

- NIST, The NIST Definition of Cloud Computing (SP 800-145)
- A. Tanenbaum et H. Bos, Modern Operating Systems (chapitre sur la virtualisation et le cloud)
- Documentation officielle : VirtualBox (virtualbox.org/manual), Hyper-V (learn.microsoft.com), Vagrant (developer.hashicorp.com/vagrant), libvirt (libvirt.org)
- Documentation officielle : Proxmox VE (pve.proxmox.com), OpenStack (docs.openstack.org), Ceph (docs.ceph.com)
- Documentation officielle : VMware vSphere (techdocs.broadcom.com), OpenTofu (opentofu.org), Ansible (docs.ansible.com), LocalStack (docs.localstack.cloud)
