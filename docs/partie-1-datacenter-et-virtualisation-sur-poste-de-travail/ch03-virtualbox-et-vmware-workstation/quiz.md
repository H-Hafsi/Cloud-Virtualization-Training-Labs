# Quiz du chapitre 3 : VirtualBox et VMware Workstation / Chapter 3 Quiz: VirtualBox and VMware Workstation

=== "Français"

    !!! abstract "Consignes"
        Autoévaluation : 10 questions à choix multiples, environ 10 minutes. Une seule réponse est correcte par question. Répondez d'abord de mémoire, puis cliquez sur « Réponse » pour vérifier. Si une réponse vous surprend, relisez la section correspondante du [cours](cours.md).

    **Question 1.** Quel mode réseau permet à une VM d'accéder à Internet sans être joignable directement depuis le réseau extérieur (sans redirection de ports) ?

    - A. Pont (bridge)
    - B. NAT
    - C. Hôte seul (host-only)
    - D. Réseau interne

    ??? success "Réponse"
        **B.** En NAT, la VM sort derrière l'adresse de l'hôte. Le mode pont l'expose sur le réseau local ; l'hôte seul et le réseau interne ne donnent pas accès à Internet.

    **Question 2.** Quelle est la configuration réseau de référence d'une VM dans ce cours ?

    - A. Deux cartes en NAT
    - B. Deux cartes en pont
    - C. Une seule carte en hôte seul
    - D. Une carte en NAT et une carte en hôte seul

    ??? success "Réponse"
        **D.** La carte NAT donne l'accès à Internet (mises à jour) ; la carte hôte seul permet l'administration en SSH depuis le poste, avec une adresse stable.

    **Question 3.** À quel moment prend-on le snapshot de référence d'une VM ?

    - A. Après l'installation, les outils invité et les mises à jour, avant les manipulations risquées
    - B. Avant d'installer le système
    - C. Jamais : le snapshot remplace la sauvegarde
    - D. Uniquement lorsque la VM est en panne

    ??? success "Réponse"
        **A.** Il fige un état « propre » auquel revenir. Un snapshot ne remplace pas une sauvegarde, et le prendre une fois la VM en panne serait trop tard.

    **Question 4.** Qu'est-ce qu'un fichier OVA ?

    - A. Un format de disque propre à VirtualBox
    - B. Un snapshot
    - C. Une archive portable qui décrit une VM et son disque, échangeable entre hyperviseurs
    - D. Un type de réseau virtuel

    ??? success "Réponse"
        **C.** OVF décrit la VM en plusieurs fichiers ; OVA regroupe le tout dans une seule archive. Les deux permettent d'importer ou d'exporter une VM d'un outil à l'autre.

    **Question 5.** Que doit-on modifier sur une VM tout juste clonée ?

    - A. Rien : un clone est prêt à l'emploi
    - B. Le nom d'hôte et l'adresse réseau (ainsi que l'identifiant de machine et les clés SSH)
    - C. Uniquement la quantité de RAM
    - D. L'image ISO d'installation

    ??? success "Réponse"
        **B.** Un clone est une copie parfaite : sans modification, il partage nom d'hôte, adresse, `machine-id` et clés SSH avec l'original, ce qui provoque des conflits et affaiblit la sécurité.

    **Question 6.** Pourquoi les noms des interfaces réseau d'une VM peuvent-ils changer après un import dans un autre hyperviseur ?

    - A. Parce que l'archive OVA est corrompue
    - B. Parce que Netplan est défaillant
    - C. Parce qu'Ubuntu change les noms au hasard à chaque démarrage
    - D. Parce que le matériel virtuel présenté à la VM est différent, et que Linux nomme les interfaces d'après leur emplacement matériel

    ??? success "Réponse"
        **D.** Un autre hyperviseur présente d'autres cartes réseau, à d'autres emplacements. Les fichiers de configuration qui citent les anciens noms doivent alors être mis à jour.

    **Question 7.** Dans une VM, la commande `grep -Ec '(vmx|svm)' /proc/cpuinfo` renvoie 4. Que cela signifie-t-il ?

    - A. La VM voit les extensions de virtualisation du processeur : elle pourra exécuter un hyperviseur
    - B. Le poste hôte ne supporte pas la virtualisation
    - C. La VM utilise 4 Go de RAM
    - D. Le disque virtuel est en provisionnement dynamique

    ??? success "Réponse"
        **A.** Un résultat supérieur à 0 indique que les extensions `vmx` (Intel) ou `svm` (AMD) sont visibles : la virtualisation imbriquée est disponible. Un résultat de 0 signale qu'elle ne l'est pas.

    **Question 8.** Un disque virtuel dynamique de 25 Go vient de recevoir une installation d'Ubuntu Server. Quelle est la taille attendue du fichier de disque sur l'hôte ?

    - A. Exactement 25 Go
    - B. 0 octet
    - C. Nettement inférieure à 25 Go
    - D. Supérieure à 25 Go

    ??? success "Réponse"
        **C.** Le fichier ne grossit qu'avec les données réellement écrites. Il faut surveiller l'espace libre de l'hôte, car il peut croître jusqu'à 25 Go.

    **Question 9.** Quel outil en ligne de commande pilote VirtualBox ?

    - A. `vmrun`
    - B. `VBoxManage`
    - C. `virsh`
    - D. `bcdedit`

    ??? success "Réponse"
        **B.** `VBoxManage` est l'outil de VirtualBox. `vmrun` appartient à VMware, `virsh` à libvirt (KVM) et `bcdedit` configure le démarrage de Windows.

    **Question 10.** Quel état de l'hyperviseur de Windows est recommandé pour les labs de VirtualBox et de VMware Workstation ?

    - A. Désactivé : meilleures performances et virtualisation imbriquée disponible
    - B. Activé, obligatoirement
    - C. Sans importance
    - D. Remplacé par Hyper-V, obligatoirement

    ??? success "Réponse"
        **A.** Lorsque l'hyperviseur Windows est actif, les hyperviseurs de poste passent par lui avec des performances moindres et sans virtualisation imbriquée. Il reste nécessaire seulement pour les labs Hyper-V.

=== "English"

    !!! abstract "Instructions"
        Self-assessment: 10 multiple-choice questions, about 10 minutes. Only one answer is correct per question. Answer from memory first, then click "Answer" to check. If an answer surprises you, reread the matching section of the [lesson](cours.md).

    **Question 1.** Which network mode lets a VM reach the Internet without being directly reachable from the outside network (without port forwarding)?

    - A. Bridge
    - B. NAT
    - C. Host-only
    - D. Internal network

    ??? success "Answer"
        **B.** With NAT, the VM goes out behind the host's address. Bridge mode exposes it on the local network; host-only and internal network give no Internet access.

    **Question 2.** What is a VM's reference network configuration in this course?

    - A. Two NAT cards
    - B. Two bridged cards
    - C. A single host-only card
    - D. One NAT card and one host-only card

    ??? success "Answer"
        **D.** The NAT card provides Internet access (updates); the host-only card allows SSH administration from the computer, with a stable address.

    **Question 3.** When is a VM's reference snapshot taken?

    - A. After installation, guest tools and updates, before risky operations
    - B. Before installing the system
    - C. Never: the snapshot replaces the backup
    - D. Only when the VM is broken

    ??? success "Answer"
        **A.** It freezes a "clean" state to return to. A snapshot does not replace a backup, and taking it once the VM is broken would be too late.

    **Question 4.** What is an OVA file?

    - A. A VirtualBox-specific disk format
    - B. A snapshot
    - C. A portable archive describing a VM and its disk, exchangeable between hypervisors
    - D. A type of virtual network

    ??? success "Answer"
        **C.** OVF describes the VM in several files; OVA packs everything into a single archive. Both allow importing or exporting a VM from one tool to the other.

    **Question 5.** What must be changed on a freshly cloned VM?

    - A. Nothing: a clone is ready to use
    - B. The hostname and network address (as well as the machine identifier and SSH keys)
    - C. Only the amount of RAM
    - D. The installation ISO image

    ??? success "Answer"
        **B.** A clone is a perfect copy: unchanged, it shares hostname, address, `machine-id` and SSH keys with the original, which causes conflicts and weakens security.

    **Question 6.** Why can a VM's network interface names change after an import into another hypervisor?

    - A. Because the OVA archive is corrupted
    - B. Because Netplan is faulty
    - C. Because Ubuntu changes names randomly at each boot
    - D. Because the virtual hardware presented to the VM is different, and Linux names interfaces after their hardware location

    ??? success "Answer"
        **D.** Another hypervisor presents other network cards, at other locations. Configuration files citing the old names must then be updated.

    **Question 7.** Inside a VM, the command `grep -Ec '(vmx|svm)' /proc/cpuinfo` returns 4. What does that mean?

    - A. The VM sees the processor's virtualization extensions: it will be able to run a hypervisor
    - B. The host computer does not support virtualization
    - C. The VM uses 4 GB of RAM
    - D. The virtual disk uses dynamic provisioning

    ??? success "Answer"
        **A.** A result greater than 0 means the `vmx` (Intel) or `svm` (AMD) extensions are visible: nested virtualization is available. A result of 0 means it is not.

    **Question 8.** A 25 GB dynamic virtual disk has just received an Ubuntu Server installation. What size do you expect for the disk file on the host?

    - A. Exactly 25 GB
    - B. 0 bytes
    - C. Well below 25 GB
    - D. Above 25 GB

    ??? success "Answer"
        **C.** The file only grows with the data actually written. The host's free space must be watched, as it can grow up to 25 GB.

    **Question 9.** Which command-line tool drives VirtualBox?

    - A. `vmrun`
    - B. `VBoxManage`
    - C. `virsh`
    - D. `bcdedit`

    ??? success "Answer"
        **B.** `VBoxManage` is VirtualBox's tool. `vmrun` belongs to VMware, `virsh` to libvirt (KVM) and `bcdedit` configures Windows boot.

    **Question 10.** Which Windows hypervisor state is recommended for the VirtualBox and VMware Workstation labs?

    - A. Off: better performance and nested virtualization available
    - B. On, mandatory
    - C. Irrelevant
    - D. Replaced by Hyper-V, mandatory

    ??? success "Answer"
        **A.** When the Windows hypervisor is active, desktop hypervisors go through it with lower performance and without nested virtualization. It remains necessary only for the Hyper-V labs.