# Chapitre 3 : VirtualBox et VMware Workstation / VirtualBox and VMware Workstation

=== "Français"

    !!! abstract "Objectifs du chapitre"
        - Situer VirtualBox et VMware Workstation parmi les hyperviseurs de poste de travail
        - Décrire le cycle de vie d'une machine virtuelle : création, installation, outils invité, snapshot, clone, export
        - Choisir un mode réseau de base (NAT, pont, hôte seul) selon le besoin
        - Comparer les deux outils et savoir passer de l'un à l'autre (OVF/OVA, formats de disque)
        - Appliquer les bonnes pratiques de dimensionnement et de gestion des VM sur un poste
        - Acquis d'apprentissage visé : **AA2**

        Matériel requis pour ce cours : aucun. Les labs associés indiquent leurs propres ressources.

=== "English"

    !!! abstract "Chapter objectives"
        - Place VirtualBox and VMware Workstation among desktop hypervisors
        - Describe the lifecycle of a virtual machine: creation, installation, guest tools, snapshot, clone, export
        - Choose a basic network mode (NAT, bridge, host-only) according to the need
        - Compare the two tools and know how to move from one to the other (OVF/OVA, disk formats)
        - Apply good practices for sizing and managing VMs on a workstation
        - Target learning outcome: **AA2**

        Required equipment for this lesson: none. The associated labs state their own resources.

## 1. Deux hyperviseurs de poste de travail / Two desktop hypervisors

=== "Français"

    Au chapitre 2, nous avons vu qu'un hyperviseur de **type 2** s'exécute comme une application dans un système hôte. **VirtualBox** et **VMware Workstation** en sont les deux représentants les plus utilisés sur poste de travail, et ils sont tous deux utilisables sous Windows. Ils vous permettent de construire sur un seul ordinateur un petit laboratoire de plusieurs machines.

    | | VirtualBox | VMware Workstation |
    |---|---|---|
    | **Éditeur / licence** | Oracle ; cœur sous licence libre (GPL). Le *paquet d'extension* a une licence distincte : à vérifier avant usage | VMware ; licence propriétaire : **vérifier les conditions en vigueur** au moment de la séance |
    | **Formats de disque** | `.vdi` (natif) ; lit aussi `.vmdk` et `.vhd` | `.vmdk` (natif) |
    | **Ligne de commande** | `VBoxManage` | `vmrun` et outils associés |
    | **Réseau avancé** | Gestionnaire de réseau de VirtualBox | *Virtual Network Editor* |
    | **Échange de VM** | Import et export au format OVF/OVA | Import et export au format OVF/OVA |
    | **Rôle dans ce cours** | Alternative libre (chapitres 3, 5, 6) | Hyperviseur principal et base des labs imbriqués |

    !!! warning "Un seul hyperviseur de poste actif à la fois pour les labs lourds"
        Comme vu au [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md), Windows peut lancer son propre hyperviseur. Les deux outils peuvent alors fonctionner via la plateforme de l'hyperviseur Windows, mais avec des performances moindres et sans virtualisation imbriquée. Pour ce chapitre, travaillez de préférence avec l'hyperviseur de Windows **désactivé**.

=== "English"

    In chapter 2 we saw that a **type 2** hypervisor runs as an application inside a host system. **VirtualBox** and **VMware Workstation** are the two most widely used on workstations, and both can be used on Windows. They let you build a small multi-machine lab on a single computer.

    | | VirtualBox | VMware Workstation |
    |---|---|---|
    | **Vendor / license** | Oracle; core under an open source license (GPL). The *extension pack* has a separate license: check before use | VMware; proprietary license: **check the terms in force** at the time of the session |
    | **Disk formats** | `.vdi` (native); also reads `.vmdk` and `.vhd` | `.vmdk` (native) |
    | **Command line** | `VBoxManage` | `vmrun` and related tools |
    | **Advanced networking** | VirtualBox network manager | *Virtual Network Editor* |
    | **VM exchange** | Import and export in OVF/OVA format | Import and export in OVF/OVA format |
    | **Role in this course** | Open source alternative (chapters 3, 5, 6) | Main hypervisor and base of nested labs |

    !!! warning "Only one desktop hypervisor active at a time for heavy labs"
        As seen in [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md), Windows can start its own hypervisor. Both tools can then run through the Windows hypervisor platform, but with lower performance and without nested virtualization. For this chapter, preferably work with the Windows hypervisor **turned off**.

## 2. Le cycle de vie d'une machine virtuelle / The lifecycle of a virtual machine

=== "Français"

    ```mermaid
    flowchart LR
      A["1. Créer le matériel<br/>virtuel"] --> B["2. Attacher<br/>l'ISO"]
      B --> C["3. Installer<br/>le système"]
      C --> D["4. Installer les<br/>outils invité"]
      D --> E["5. Snapshot de<br/>référence"]
      E --> F["6. Utiliser"]
      E --> G["7. Cloner ou<br/>exporter"]
    ```

    | Étape | Ce que l'on fait | Points d'attention |
    |---|---|---|
    | **1. Créer le matériel virtuel** | Choisir le nom, le type de système, le nombre de vCPU, la RAM, la taille et le provisionnement du disque, la ou les cartes réseau | Dimensionner d'après le rôle de la VM (voir la section 5) |
    | **2. Attacher l'ISO** | Insérer l'image d'installation dans le lecteur virtuel | Vérifier l'empreinte de l'ISO (Lab 2.1) |
    | **3. Installer le système** | Suivre l'installateur comme sur une machine physique | Un nom d'hôte explicite, un compte administrateur |
    | **4. Installer les outils invité** | Pilotes et services qui améliorent l'intégration (affichage, horloge, arrêt propre, fichiers) | **VirtualBox Guest Additions** ou **VMware Tools** (sous Linux, `open-vm-tools`) |
    | **5. Snapshot de référence** | Figer l'état « propre » après installation et mises à jour | Le nommer clairement (par exemple `etat-propre`) |
    | **6. Utiliser** | Travailler, tester, casser | Revenir au snapshot en cas d'erreur |
    | **7. Cloner ou exporter** | Dupliquer la VM sur le même poste, ou l'empaqueter pour un autre | Voir ci-dessous |

    ### Cloner, exporter, importer

    | Opération | Résultat | Usage |
    |---|---|---|
    | **Clone** | Copie de la VM sur le même hyperviseur (complète ou liée) | Obtenir plusieurs machines identiques à partir d'une VM de référence |
    | **Export OVF/OVA** | Fichier(s) décrivant la VM et son disque, lisible par plusieurs hyperviseurs | Transférer une VM vers un autre poste ou un autre outil |
    | **Import OVF/OVA** | Création d'une VM à partir d'un tel fichier | Récupérer une VM fournie par l'enseignant ou exportée d'un autre outil |

    Un clone complet est indépendant ; un clone lié reste dépendant de la VM d'origine (chapitre 2). Après un clone ou un import, **changez le nom d'hôte et vérifiez l'adresse réseau** : deux machines identiques sur un même réseau entrent en conflit.

=== "English"

    ```mermaid
    flowchart LR
      A["1. Create the virtual<br/>hardware"] --> B["2. Attach<br/>the ISO"]
      B --> C["3. Install<br/>the system"]
      C --> D["4. Install<br/>guest tools"]
      D --> E["5. Reference<br/>snapshot"]
      E --> F["6. Use"]
      E --> G["7. Clone or<br/>export"]
    ```

    | Step | What you do | Points of attention |
    |---|---|---|
    | **1. Create the virtual hardware** | Choose the name, OS type, number of vCPUs, RAM, disk size and provisioning, network card(s) | Size according to the VM's role (see section 5) |
    | **2. Attach the ISO** | Insert the installation image in the virtual drive | Verify the ISO hash (Lab 2.1) |
    | **3. Install the system** | Follow the installer as on a physical machine | An explicit hostname, an administrator account |
    | **4. Install guest tools** | Drivers and services that improve integration (display, clock, clean shutdown, files) | **VirtualBox Guest Additions** or **VMware Tools** (on Linux, `open-vm-tools`) |
    | **5. Reference snapshot** | Freeze the "clean" state after installation and updates | Name it clearly (for example `etat-propre`) |
    | **6. Use** | Work, test, break things | Return to the snapshot after a mistake |
    | **7. Clone or export** | Duplicate the VM on the same computer, or package it for another | See below |

    ### Cloning, exporting, importing

    | Operation | Result | Use |
    |---|---|---|
    | **Clone** | Copy of the VM on the same hypervisor (full or linked) | Get several identical machines from a reference VM |
    | **OVF/OVA export** | File(s) describing the VM and its disk, readable by several hypervisors | Move a VM to another computer or another tool |
    | **OVF/OVA import** | Creating a VM from such a file | Retrieve a VM provided by the instructor or exported from another tool |

    A full clone is independent; a linked clone stays dependent on the original VM (chapter 2). After a clone or import, **change the hostname and check the network address**: two identical machines on the same network conflict.

## 3. Le réseau virtuel de base / Basic virtual networking

=== "Français"

    Chaque carte réseau d'une VM est rattachée à un **mode réseau**. Les principaux modes sont présentés ici ; le chapitre 5 les approfondit.

    | Mode | La VM peut… | Elle est visible… | Usage typique |
    |---|---|---|---|
    | **NAT** | Accéder à Internet et au réseau extérieur | Pas depuis l'extérieur (sans redirection de ports) | Mises à jour, téléchargements |
    | **Pont (bridge)** | Se comporter comme une machine du réseau physique | Du réseau local | Exposer un service sur le réseau |
    | **Hôte seul (host-only)** | Communiquer avec l'hôte et les autres VM du même réseau | De l'hôte et des VM de ce réseau uniquement | Administration (SSH depuis l'hôte), labos isolés |
    | **Réseau interne** (VirtualBox) ou **segment LAN** (VMware) | Communiquer uniquement avec les VM du même réseau privé | Des seules VM concernées | Réseaux isolés entre VM |

    Dans ce cours, une VM de référence possède en général **deux cartes** : une en **NAT** pour l'accès à Internet, une en **hôte seul** pour l'administration depuis votre poste. L'annexe [Environnement](../../annexes/environnement.md) précise les réseaux et les plages d'adresses utilisés.

=== "English"

    Each network card of a VM is attached to a **network mode**. The main modes are presented here; chapter 5 goes deeper.

    | Mode | The VM can... | It is visible... | Typical use |
    |---|---|---|---|
    | **NAT** | Reach the Internet and the outside network | Not from outside (without port forwarding) | Updates, downloads |
    | **Bridge** | Behave like a machine on the physical network | From the local network | Exposing a service on the network |
    | **Host-only** | Communicate with the host and other VMs on the same network | Only from the host and VMs on that network | Administration (SSH from the host), isolated labs |
    | **Internal network** (VirtualBox) or **LAN segment** (VMware) | Communicate only with VMs on the same private network | Only to the VMs concerned | Isolated networks between VMs |

    In this course, a reference VM usually has **two cards**: one in **NAT** for Internet access, one in **host-only** for administration from your computer. The [Environment](../../annexes/environnement.md) annex specifies the networks and address ranges used.

## 4. Choisir et combiner les outils / Choosing and combining the tools

=== "Français"

    Les deux outils créent des VM Linux de la même façon ; les différences portent surtout sur les **fonctions avancées**, les **formats** et la **licence**.

    | Besoin | Piste |
    |---|---|
    | Un outil libre, sans formalité de licence | VirtualBox (vérifier la licence du paquet d'extension si vous l'utilisez) |
    | Meilleure intégration pour la virtualisation imbriquée et les labs des Parties 3 et 4 | VMware Workstation (activer l'option de virtualisation matérielle exposée à la VM) |
    | Passer une VM d'un outil à l'autre | Export OVF/OVA, puis import ; ou conversion du disque (`.vdi` vers `.vmdk`) |
    | Automatiser la création de VM | Ligne de commande (`VBoxManage`, `vmrun`) puis Vagrant (chapitre 6) |

    **Virtualisation imbriquée.** Pour qu'une VM puisse elle-même exécuter un hyperviseur (chapitres 12 à 16 et 18), il faut exposer les extensions de virtualisation du processeur à cette VM : l'option se nomme *Virtualize Intel VT-x/EPT or AMD-V/RVI* dans VMware Workstation et *Enable Nested VT-x/AMD-V* dans VirtualBox. Elle dépend du processeur et de la configuration de l'hôte : on la teste dans la VM (voir l'annexe Environnement).

    !!! tip "À retenir"
        - VirtualBox et VMware Workstation sont des hyperviseurs de type 2, utilisables sous Windows.
        - Cycle de vie : créer, attacher l'ISO, installer, outils invité, snapshot, utiliser, cloner ou exporter.
        - Modes réseau de base : NAT (Internet), pont (réseau local), hôte seul (administration), réseau interne.
        - Une VM de référence : une carte NAT + une carte hôte seul.
        - OVF/OVA permet d'échanger des VM entre outils ; après clone ou import, changer le nom d'hôte et l'adresse.
        - Un snapshot de référence se prend après installation et mises à jour, avant les manipulations risquées.
        - Un seul hyperviseur de poste actif à la fois ; hyperviseur Windows désactivé pour les labs lourds.

=== "English"

    Both tools create Linux VMs in the same way; the differences mostly concern **advanced features**, **formats** and **licensing**.

    | Need | Hint |
    |---|---|
    | An open source tool, no license formalities | VirtualBox (check the extension pack license if you use it) |
    | Best integration for nested virtualization and Part 3 and Part 4 labs | VMware Workstation (enable the option exposing hardware virtualization to the VM) |
    | Moving a VM from one tool to the other | OVF/OVA export, then import; or disk conversion (`.vdi` to `.vmdk`) |
    | Automating VM creation | Command line (`VBoxManage`, `vmrun`) then Vagrant (chapter 6) |

    **Nested virtualization.** For a VM to run a hypervisor itself (chapters 12 to 16 and 18), the processor's virtualization extensions must be exposed to that VM: the option is named *Virtualize Intel VT-x/EPT or AMD-V/RVI* in VMware Workstation and *Enable Nested VT-x/AMD-V* in VirtualBox. It depends on the processor and the host configuration: it is tested inside the VM (see the Environment annex).

    !!! tip "Key takeaways"
        - VirtualBox and VMware Workstation are type 2 hypervisors, usable on Windows.
        - Lifecycle: create, attach ISO, install, guest tools, snapshot, use, clone or export.
        - Basic network modes: NAT (Internet), bridge (local network), host-only (administration), internal network.
        - A reference VM: one NAT card + one host-only card.
        - OVF/OVA lets you exchange VMs between tools; after a clone or import, change the hostname and the address.
        - A reference snapshot is taken after installation and updates, before risky operations.
        - Only one desktop hypervisor active at a time; Windows hypervisor off for heavy labs.

## 5. Dimensionner et gérer les VM sur un poste / Sizing and managing VMs on a workstation

=== "Français"

    - **Laissez de la mémoire à l'hôte.** La somme de la RAM des VM **en fonctionnement** doit rester nettement inférieure à la RAM du poste, sinon Windows et les VM ralentissent ensemble.
    - **Dimensionnez par rôle.** Un serveur sans interface graphique n'a pas besoin de beaucoup : les gabarits S, M et L de l'annexe Environnement donnent des points de départ.
    - **Provisionnez en *thin* (dynamique)** pour économiser de l'espace, en surveillant l'espace libre de l'hôte.
    - **Utilisez un SSD** et placez les VM sur un disque ayant assez d'espace libre.
    - **Éteignez proprement** les VM inutilisées plutôt que de les laisser en veille : la veille conserve la mémoire sur le disque.
    - **Nommez** les VM, les snapshots et les réseaux de façon explicite et cohérente.
    - **Supprimez les snapshots devenus inutiles** : ils consomment de l'espace et ralentissent la VM.
    - **Administrez à distance** : une VM Linux sans interface graphique se pilote par SSH depuis l'hôte, via le réseau « hôte seul ».

=== "English"

    - **Leave memory to the host.** The total RAM of **running** VMs must remain well below the computer's RAM, otherwise Windows and the VMs slow down together.
    - **Size by role.** A server without a graphical interface does not need much: the S, M and L templates in the Environment annex give starting points.
    - **Provision as thin (dynamic)** to save space, while watching the host's free space.
    - **Use an SSD** and place VMs on a disk with enough free space.
    - **Shut down cleanly** unused VMs rather than suspending them: suspending keeps memory on disk.
    - **Name** VMs, snapshots and networks explicitly and consistently.
    - **Delete snapshots that are no longer useful**: they use space and slow the VM down.
    - **Administer remotely**: a Linux VM without a graphical interface is driven over SSH from the host, through the "host-only" network.

## Pour vérifier votre compréhension / Check your understanding

=== "Français"

    ??? question "À quel moment faut-il prendre le snapshot de référence d'une VM, et pourquoi ?"
        Après l'installation du système, des outils invité et des mises à jour, mais **avant** les manipulations risquées ou propres à un lab. Il fournit un état « propre » auquel revenir en quelques secondes en cas d'erreur, ou pour repartir de zéro avant une séance.

    ??? question "Quelle différence y a-t-il entre cloner une VM et l'exporter en OVA ?"
        Le clone crée une copie **sur le même hyperviseur**, utilisable immédiatement. L'export en OVA produit un **fichier portable** qui décrit la VM et son disque, que l'on peut transférer sur un autre poste ou importer dans un autre outil.

    ??? question "Une VM doit accéder à Internet pour ses mises à jour et être administrable en SSH depuis votre poste. Quelle configuration réseau proposez-vous ?"
        Deux cartes réseau : une en **NAT** pour l'accès à Internet, une en **hôte seul** pour l'administration depuis l'hôte (adresse statique ou fixe sur le réseau hôte seul). C'est la configuration de référence du cours.

=== "English"

    ??? question "When should a VM's reference snapshot be taken, and why?"
        After installing the system, the guest tools and the updates, but **before** risky or lab-specific operations. It provides a "clean" state to return to in seconds after a mistake, or to start over before a session.

    ??? question "What is the difference between cloning a VM and exporting it as an OVA?"
        A clone creates a copy **on the same hypervisor**, immediately usable. An OVA export produces a **portable file** describing the VM and its disk, which can be moved to another computer or imported into another tool.

    ??? question "A VM must reach the Internet for updates and be administrable over SSH from your computer. Which network configuration do you suggest?"
        Two network cards: one in **NAT** for Internet access, one in **host-only** for administration from the host (static or fixed address on the host-only network). This is the course's reference configuration.

## Ressources du chapitre / Chapter resources

- [Lab 3.1 : VirtualBox, une VM Ubuntu Server / VirtualBox: an Ubuntu Server VM](lab-3-1-virtualbox-une-vm-ubuntu-server.md)
- [Lab 3.2 : VMware Workstation, même VM, import et export / VMware Workstation: same VM, import and export](lab-3-2-vmware-workstation-meme-vm-import-et-export.md)
- [Quiz du chapitre / Chapter quiz](quiz.md)
- [Glossaire / Glossary](../../annexes/glossaire.md)
- [Environnement de TP / Lab environment](../../annexes/environnement.md)
