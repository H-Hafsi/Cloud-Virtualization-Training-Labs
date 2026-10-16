# Partie 1 : Datacenter et virtualisation sur poste de travail / Part 1: Datacenter and Virtualization on a Workstation

**Chapitres 1 à 6 / Chapters 1 to 6** · **Acquis visés / Learning outcomes : AA1, AA2** · **Mini-projet 1**

=== "Français"

    Cette partie pose les fondations de la matière. Vous partez du **datacenter physique** (énergie, refroidissement, redondance), vous découvrez les **principes de la virtualisation**, puis vous construisez vos premières machines virtuelles avec trois hyperviseurs de poste de travail : **VirtualBox**, **VMware Workstation** et **Hyper-V**. Vous apprenez ensuite à les **relier par des réseaux virtuels** (NAT, hôte seul, routage, VLAN) et à **décrire vos environnements dans un fichier** avec Vagrant. Le mini-projet 1 réunit l'ensemble : un mini-datacenter virtuel de trois machines, décrit comme du code.

    **Progression des labs :** guidé (chapitres 1 à 4), puis semi-guidé (routage et fournisseurs multiples, chapitres 5 et 6), jusqu'au mini-projet.

    **Avant de commencer.** Lisez l'annexe [Environnement de TP](../annexes/environnement.md) : elle précise les ressources minimales de votre poste, le plan d'adressage des réseaux virtuels et les règles de travail communes. Le [Lab 2.1](ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) prépare votre poste.

=== "English"

    This part lays the foundations of the course. You start from the **physical datacenter** (power, cooling, redundancy), discover the **principles of virtualization**, then build your first virtual machines with three desktop hypervisors: **VirtualBox**, **VMware Workstation** and **Hyper-V**. You then learn to **connect them with virtual networks** (NAT, host-only, routing, VLAN) and to **describe your environments in a file** with Vagrant. Mini-project 1 brings it all together: a virtual mini-datacenter of three machines, described as code.

    **Lab progression:** guided (chapters 1 to 4), then semi-guided (routing and multiple providers, chapters 5 and 6), up to the mini-project.

    **Before you start.** Read the [Lab environment](../annexes/environnement.md) annex: it states your computer's minimum resources, the virtual networks' addressing plan and the common working rules. [Lab 2.1](ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) prepares your computer.

## Organisation / Organization

=== "Français"

    | Chapitre | Contenu | Ressources |
    |---|---|---|
    | 1. Architecture et composants d'un datacenter (AA1) | Familles de datacenters, baies, énergie et refroidissement, PUE, redondance, classes Tier, serveurs et réseau | [Cours](ch01-architecture-et-composants-d-un-datacenter/cours.md) · [Lab 1.1 : Dimensionner une salle serveur](ch01-architecture-et-composants-d-un-datacenter/lab-1-1-dimensionner-une-salle-serveur.md) · [Lab 1.2 : Inventorier une machine](ch01-architecture-et-composants-d-un-datacenter/lab-1-2-inventorier-une-machine.md) · [Quiz](ch01-architecture-et-composants-d-un-datacenter/quiz.md) |
    | 2. Principes de la virtualisation (AA1) | Pourquoi virtualiser, techniques, hyperviseurs de type 1 et 2, VM et fichiers, surallocation, conteneurs | [Cours](ch02-principes-de-la-virtualisation/cours.md) · [Lab 2.1 : Préparer le poste pour la virtualisation](ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) · [Quiz](ch02-principes-de-la-virtualisation/quiz.md) |
    | 3. VirtualBox et VMware Workstation (AA2) | Cycle de vie d'une VM, outils invité, snapshots, clones, OVF/OVA, modes réseau de base | [Cours](ch03-virtualbox-et-vmware-workstation/cours.md) · [Lab 3.1 : VirtualBox, une VM Ubuntu Server](ch03-virtualbox-et-vmware-workstation/lab-3-1-virtualbox-une-vm-ubuntu-server.md) · [Lab 3.2 : VMware Workstation, même VM, import et export](ch03-virtualbox-et-vmware-workstation/lab-3-2-vmware-workstation-meme-vm-import-et-export.md) · [Quiz](ch03-virtualbox-et-vmware-workstation/quiz.md) |
    | 4. Hyper-V (AA2) | Architecture, génération des VM, disques VHDX, commutateurs virtuels, checkpoints, export et import | [Cours](ch04-hyper-v/cours.md) · [Lab 4.1 : Activer Hyper-V et créer une VM](ch04-hyper-v/lab-4-1-activer-hyper-v-et-creer-une-vm.md) · [Lab 4.2 : Checkpoints, disques et export](ch04-hyper-v/lab-4-2-checkpoints-disques-et-export.md) · [Quiz](ch04-hyper-v/quiz.md) |
    | 5. Connexion entre VM et réseaux virtuels (AA2) | Commutateurs et modes réseau, adressage, routage, NAT, VLAN, diagnostic | [Cours](ch05-connexion-entre-vm-et-reseaux-virtuels/cours.md) · [Lab 5.1 : Réseaux virtuels sous VirtualBox et VMware](ch05-connexion-entre-vm-et-reseaux-virtuels/lab-5-1-reseaux-virtuels-sous-virtualbox-et-vmware.md) · [Lab 5.2 : Commutateurs virtuels sous Hyper-V](ch05-connexion-entre-vm-et-reseaux-virtuels/lab-5-2-commutateurs-virtuels-sous-hyper-v.md) · [Lab 5.3 : Routage, NAT et VLAN sous Linux](ch05-connexion-entre-vm-et-reseaux-virtuels/lab-5-3-routage-nat-et-vlan-sous-linux.md) · [Quiz](ch05-connexion-entre-vm-et-reseaux-virtuels/quiz.md) |
    | 6. Vagrant : décrire ses environnements (AA2) | Vagrantfile, box, provider, provisioner, plusieurs machines, portabilité entre providers | [Cours](ch06-vagrant-decrire-ses-environnements/cours.md) · [Lab 6.1 : Premier Vagrantfile](ch06-vagrant-decrire-ses-environnements/lab-6-1-premier-vagrantfile.md) · [Lab 6.2 : Environnement multi-machines avec provisioning](ch06-vagrant-decrire-ses-environnements/lab-6-2-environnement-multi-machines-avec-provisioning.md) · [Lab 6.3 : Même environnement sur un autre provider](ch06-vagrant-decrire-ses-environnements/lab-6-3-meme-environnement-sur-un-autre-provider.md) · [Quiz](ch06-vagrant-decrire-ses-environnements/quiz.md) |
    | **Mini-projet 1** | Un mini-datacenter virtuel décrit comme du code | [Énoncé](mini-projet-1.md) |

=== "English"

    | Chapter | Content | Resources |
    |---|---|---|
    | 1. Datacenter Architecture and Components (AA1) | Datacenter families, racks, power and cooling, PUE, redundancy, Tier classes, servers and network | [Lesson](ch01-architecture-et-composants-d-un-datacenter/cours.md) · [Lab 1.1: Sizing a server room](ch01-architecture-et-composants-d-un-datacenter/lab-1-1-dimensionner-une-salle-serveur.md) · [Lab 1.2: Inventorying a machine](ch01-architecture-et-composants-d-un-datacenter/lab-1-2-inventorier-une-machine.md) · [Quiz](ch01-architecture-et-composants-d-un-datacenter/quiz.md) |
    | 2. Principles of Virtualization (AA1) | Why virtualize, techniques, type 1 and type 2 hypervisors, VMs and files, overcommit, containers | [Lesson](ch02-principes-de-la-virtualisation/cours.md) · [Lab 2.1: Preparing the computer for virtualization](ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) · [Quiz](ch02-principes-de-la-virtualisation/quiz.md) |
    | 3. VirtualBox and VMware Workstation (AA2) | VM lifecycle, guest tools, snapshots, clones, OVF/OVA, basic network modes | [Lesson](ch03-virtualbox-et-vmware-workstation/cours.md) · [Lab 3.1: VirtualBox, an Ubuntu Server VM](ch03-virtualbox-et-vmware-workstation/lab-3-1-virtualbox-une-vm-ubuntu-server.md) · [Lab 3.2: VMware Workstation, same VM, import and export](ch03-virtualbox-et-vmware-workstation/lab-3-2-vmware-workstation-meme-vm-import-et-export.md) · [Quiz](ch03-virtualbox-et-vmware-workstation/quiz.md) |
    | 4. Hyper-V (AA2) | Architecture, VM generations, VHDX disks, virtual switches, checkpoints, export and import | [Lesson](ch04-hyper-v/cours.md) · [Lab 4.1: Enabling Hyper-V and creating a VM](ch04-hyper-v/lab-4-1-activer-hyper-v-et-creer-une-vm.md) · [Lab 4.2: Checkpoints, disks and export](ch04-hyper-v/lab-4-2-checkpoints-disques-et-export.md) · [Quiz](ch04-hyper-v/quiz.md) |
    | 5. VM Connectivity and Virtual Networks (AA2) | Switches and network modes, addressing, routing, NAT, VLAN, troubleshooting | [Lesson](ch05-connexion-entre-vm-et-reseaux-virtuels/cours.md) · [Lab 5.1: Virtual networks with VirtualBox and VMware](ch05-connexion-entre-vm-et-reseaux-virtuels/lab-5-1-reseaux-virtuels-sous-virtualbox-et-vmware.md) · [Lab 5.2: Virtual switches with Hyper-V](ch05-connexion-entre-vm-et-reseaux-virtuels/lab-5-2-commutateurs-virtuels-sous-hyper-v.md) · [Lab 5.3: Routing, NAT and VLAN on Linux](ch05-connexion-entre-vm-et-reseaux-virtuels/lab-5-3-routage-nat-et-vlan-sous-linux.md) · [Quiz](ch05-connexion-entre-vm-et-reseaux-virtuels/quiz.md) |
    | 6. Vagrant: Describing Your Environments (AA2) | Vagrantfile, box, provider, provisioner, several machines, portability across providers | [Lesson](ch06-vagrant-decrire-ses-environnements/cours.md) · [Lab 6.1: First Vagrantfile](ch06-vagrant-decrire-ses-environnements/lab-6-1-premier-vagrantfile.md) · [Lab 6.2: Multi-machine environment with provisioning](ch06-vagrant-decrire-ses-environnements/lab-6-2-environnement-multi-machines-avec-provisioning.md) · [Lab 6.3: Same environment on another provider](ch06-vagrant-decrire-ses-environnements/lab-6-3-meme-environnement-sur-un-autre-provider.md) · [Quiz](ch06-vagrant-decrire-ses-environnements/quiz.md) |
    | **Mini-project 1** | A virtual mini-datacenter described as code | [Statement](mini-projet-1.md) |

## Quel état pour l'hyperviseur Windows ? / Which Windows hypervisor state?

=== "Français"

    L'état de l'hyperviseur de Windows change selon les labs (procédure au [Lab 2.1](ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md), avec redémarrage) :

    | État | Labs et projets concernés |
    |---|---|
    | **Désactivé** | Chapitres 1 à 3 ; Labs 5.1, 5.3, 6.1, 6.2 ; mini-projet 1 |
    | **Activé** | Chapitre 4 (Hyper-V) ; Lab 5.2 ; Lab 6.3 |

=== "English"

    The Windows hypervisor state changes depending on the labs (procedure in [Lab 2.1](ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md), with a restart):

    | State | Labs and projects concerned |
    |---|---|
    | **Off** | Chapters 1 to 3; Labs 5.1, 5.3, 6.1, 6.2; mini-project 1 |
    | **On** | Chapter 4 (Hyper-V); Lab 5.2; Lab 6.3 |