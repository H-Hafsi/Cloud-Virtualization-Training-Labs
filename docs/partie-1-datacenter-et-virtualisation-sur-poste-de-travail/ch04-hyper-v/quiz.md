# Quiz du chapitre 4 : Hyper-V / Chapter 4 Quiz: Hyper-V

=== "Français"

    !!! abstract "Consignes"
        Autoévaluation : 10 questions à choix multiples, environ 10 minutes. Une seule réponse est correcte par question. Répondez d'abord de mémoire, puis cliquez sur « Réponse » pour vérifier. Si une réponse vous surprend, relisez la section correspondante du [cours](cours.md).

    **Question 1.** Dans quelles éditions de Windows 10 et 11 Hyper-V est-il disponible ?

    - A. Famille uniquement
    - B. Toutes les éditions
    - C. Pro, Enterprise et Éducation
    - D. Aucune : il n'existe que sous Windows Server

    ??? success "Réponse"
        **C.** Hyper-V est une fonctionnalité des éditions Pro, Enterprise et Éducation ; il n'existe pas en édition Famille.

    **Question 2.** Pourquoi Hyper-V est-il considéré comme un hyperviseur de type 1, même sur un poste Windows ?

    - A. Une fois activé, il démarre avant Windows, qui devient la partition racine
    - B. Parce qu'il est gratuit
    - C. Parce qu'il émule entièrement le matériel
    - D. Parce qu'il s'exécute comme une simple application de Windows

    ??? success "Réponse"
        **A.** L'hyperviseur prend le contrôle du matériel au démarrage ; Windows s'exécute dans une partition privilégiée, comme les VM s'exécutent dans des partitions enfants.

    **Question 3.** Quelles sont les caractéristiques d'une VM Hyper-V de génération 2 ?

    - A. BIOS et contrôleur IDE émulé
    - B. Réservée aux systèmes 32 bits
    - C. Pas de démarrage sécurisé
    - D. UEFI, démarrage sécurisé et périphériques synthétiques (contrôleur SCSI)

    ??? success "Réponse"
        **D.** La génération 1 correspond au BIOS et aux périphériques hérités émulés ; la génération 2 convient aux systèmes 64 bits récents.

    **Question 4.** Une VM Ubuntu Server de génération 2 ne démarre pas sur l'ISO avec les réglages par défaut. Quelle est la cause probable ?

    - A. La RAM est insuffisante
    - B. Le modèle de certificats du démarrage sécurisé est prévu pour Windows et refuse le chargeur d'Ubuntu
    - C. Le disque VHD est trop petit
    - D. Le Default Switch est absent

    ??? success "Réponse"
        **B.** On choisit le modèle « Microsoft UEFI Certificate Authority » (`Set-VMFirmware -SecureBootTemplate`) ou on désactive le démarrage sécurisé pour cette VM.

    **Question 5.** Quel format de disque virtuel Hyper-V est recommandé ?

    - A. VHD
    - B. VMDK
    - C. VHDX
    - D. VDI

    ??? success "Réponse"
        **C.** Le VHDX gère de plus grandes tailles (jusqu'à 64 To) et résiste mieux aux coupures. Le VHD est l'ancien format ; VMDK et VDI sont ceux de VMware et de VirtualBox.

    **Question 6.** Quel type de commutateur permet aux VM de communiquer entre elles et avec l'hôte, sans accès au réseau physique ?

    - A. Interne
    - B. Externe
    - C. Privé
    - D. Le Default Switch

    ??? success "Réponse"
        **A.** Le commutateur interne relie les VM et l'hôte. Le privé n'autorise que les VM entre elles, l'externe est lié à une carte du réseau physique et le Default Switch fournit un accès à Internet par NAT.

    **Question 7.** Que fournit le Default Switch d'Hyper-V sous Windows 10 et 11 ?

    - A. Un réseau modifiable, à adresses fixes
    - B. Un pont direct sur le réseau local
    - C. Aucun accès à Internet
    - D. Un accès à Internet par NAT, avec un sous-réseau qui peut changer d'un redémarrage à l'autre

    ??? success "Réponse"
        **D.** Il convient aux essais rapides, pas à un plan d'adressage fixe. C'est pourquoi le lab utilise un commutateur interne pour l'administration.

    **Question 8.** Quelle est la différence entre un checkpoint de production et un checkpoint standard ?

    - A. Le checkpoint de production contient la mémoire de la VM
    - B. Le checkpoint de production capture l'état du disque sans la mémoire ; le standard capture aussi la mémoire
    - C. Il n'y a aucune différence
    - D. Le checkpoint standard est une sauvegarde indépendante

    ??? success "Réponse"
        **B.** Restaurer un checkpoint de production revient à un démarrage propre ; restaurer un checkpoint standard reprend la VM « en fonctionnement ». Aucun des deux n'est une sauvegarde.

    **Question 9.** Après avoir pris un checkpoint, quel est le fichier de disque actif de la VM et son type ?

    - A. Un `.vhdx` de taille fixe
    - B. Un `.vhdx` dynamique
    - C. Un `.avhdx` de type différentiel
    - D. Un fichier de mémoire

    ??? success "Réponse"
        **C.** Le disque d'origine est figé ; les écritures suivantes vont dans un disque différentiel (`.avhdx`) qui le référence comme parent.

    **Question 10.** Vous voulez dupliquer une VM exportée sur le même hôte alors que l'original existe encore. Quel mode d'import choisissez-vous ?

    - A. Copier, avec un nouvel identifiant
    - B. Enregistrer sur place
    - C. Restaurer, en conservant l'identifiant
    - D. N'importe lequel : le résultat est identique

    ??? success "Réponse"
        **A.** Deux VM de même identifiant ne peuvent pas coexister sur un hôte. Le mode « copier » duplique les fichiers et génère un nouvel identifiant (`-Copy -GenerateNewId`).

=== "English"

    !!! abstract "Instructions"
        Self-assessment: 10 multiple-choice questions, about 10 minutes. Only one answer is correct per question. Answer from memory first, then click "Answer" to check. If an answer surprises you, reread the matching section of the [lesson](cours.md).

    **Question 1.** In which editions of Windows 10 and 11 is Hyper-V available?

    - A. Home only
    - B. All editions
    - C. Pro, Enterprise and Education
    - D. None: it only exists on Windows Server

    ??? success "Answer"
        **C.** Hyper-V is a feature of the Pro, Enterprise and Education editions; it is not available in Home edition.

    **Question 2.** Why is Hyper-V considered a type 1 hypervisor, even on a Windows computer?

    - A. Once enabled, it starts before Windows, which becomes the root partition
    - B. Because it is free
    - C. Because it fully emulates the hardware
    - D. Because it runs as a simple Windows application

    ??? success "Answer"
        **A.** The hypervisor takes control of the hardware at boot; Windows runs in a privileged partition, as VMs run in child partitions.

    **Question 3.** What are the characteristics of a generation 2 Hyper-V VM?

    - A. BIOS and emulated IDE controller
    - B. Reserved for 32-bit systems
    - C. No Secure Boot
    - D. UEFI, Secure Boot and synthetic devices (SCSI controller)

    ??? success "Answer"
        **D.** Generation 1 corresponds to BIOS and emulated legacy devices; generation 2 suits recent 64-bit systems.

    **Question 4.** A generation 2 Ubuntu Server VM does not boot from the ISO with default settings. What is the probable cause?

    - A. Not enough RAM
    - B. The Secure Boot certificate template is designed for Windows and rejects Ubuntu's boot loader
    - C. The VHD disk is too small
    - D. The Default Switch is missing

    ??? success "Answer"
        **B.** Choose the "Microsoft UEFI Certificate Authority" template (`Set-VMFirmware -SecureBootTemplate`) or disable Secure Boot for that VM.

    **Question 5.** Which Hyper-V virtual disk format is recommended?

    - A. VHD
    - B. VMDK
    - C. VHDX
    - D. VDI

    ??? success "Answer"
        **C.** VHDX supports larger sizes (up to 64 TB) and is more resilient to power loss. VHD is the legacy format; VMDK and VDI are those of VMware and VirtualBox.

    **Question 6.** Which switch type lets VMs communicate with each other and with the host, without access to the physical network?

    - A. Internal
    - B. External
    - C. Private
    - D. The Default Switch

    ??? success "Answer"
        **A.** The internal switch connects VMs and the host. Private only allows VMs to talk to each other, external is bound to a physical network card, and the Default Switch provides Internet access through NAT.

    **Question 7.** What does Hyper-V's Default Switch provide on Windows 10 and 11?

    - A. An editable network with fixed addresses
    - B. A direct bridge onto the local network
    - C. No Internet access
    - D. Internet access through NAT, with a subnet that may change from one restart to the next

    ??? success "Answer"
        **D.** It suits quick tests, not a fixed addressing plan. This is why the lab uses an internal switch for administration.

    **Question 8.** What is the difference between a production checkpoint and a standard checkpoint?

    - A. The production checkpoint contains the VM's memory
    - B. The production checkpoint captures the disk state without memory; the standard one also captures memory
    - C. There is no difference
    - D. The standard checkpoint is an independent backup

    ??? success "Answer"
        **B.** Restoring a production checkpoint amounts to a clean boot; restoring a standard checkpoint resumes the VM "as running". Neither is a backup.

    **Question 9.** After taking a checkpoint, what is the VM's active disk file and its type?

    - A. A fixed-size `.vhdx`
    - B. A dynamic `.vhdx`
    - C. An `.avhdx` of differencing type
    - D. A memory file

    ??? success "Answer"
        **C.** The original disk is frozen; subsequent writes go to a differencing disk (`.avhdx`) that references it as its parent.

    **Question 10.** You want to duplicate an exported VM on the same host while the original still exists. Which import mode do you choose?

    - A. Copy, with a new identifier
    - B. Register in place
    - C. Restore, keeping the identifier
    - D. Any: the result is identical

    ??? success "Answer"
        **A.** Two VMs with the same identifier cannot coexist on a host. The "copy" mode duplicates the files and generates a new identifier (`-Copy -GenerateNewId`).