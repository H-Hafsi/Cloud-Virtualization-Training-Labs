# Quiz du chapitre 2 : Principes de la virtualisation / Chapter 2 Quiz: Principles of Virtualization

=== "Français"

    !!! abstract "Consignes"
        Autoévaluation : 10 questions à choix multiples, environ 10 minutes. Une seule réponse est correcte par question. Répondez d'abord de mémoire, puis cliquez sur « Réponse » pour vérifier. Si une réponse vous surprend, relisez la section correspondante du [cours](cours.md).

    **Question 1.** Quel est le principal risque lié au regroupement de nombreuses VM sur un seul hôte ?

    - A. Les VM perdent automatiquement leur isolation
    - B. La panne de l'hôte arrête toutes ses VM
    - C. Les licences logicielles deviennent obligatoires
    - D. Le disque de l'hôte ne peut plus être lu

    ??? success "Réponse"
        **B.** L'hôte devient un point de défaillance unique tant qu'aucune redondance n'est prévue. L'isolation entre VM est au contraire l'un des bénéfices de la virtualisation.

    **Question 2.** Dans quelle technique le système invité sait-il qu'il est virtualisé et utilise-t-il des pilotes adaptés (par exemple `virtio`) ?

    - A. L'émulation
    - B. La virtualisation complète
    - C. L'assistance matérielle
    - D. La paravirtualisation

    ??? success "Réponse"
        **D.** En paravirtualisation, l'invité coopère avec l'hyperviseur grâce à des pilotes dédiés, surtout efficaces pour les disques et le réseau. En virtualisation complète, l'invité n'est pas modifié et ignore qu'il est virtualisé.

    **Question 3.** Que sont Intel VT-x et AMD-V ?

    - A. Des extensions matérielles du processeur qui facilitent la virtualisation
    - B. Deux hyperviseurs de type 2
    - C. Des formats de disque virtuel
    - D. Des protocoles de réseau virtuel

    ??? success "Réponse"
        **A.** Ce sont des fonctions des processeurs, qu'il faut activer dans le micrologiciel (BIOS/UEFI) pour que les hyperviseurs en profitent.

    **Question 4.** Lequel de ces logiciels est un hyperviseur de type 1 ?

    - A. VirtualBox
    - B. VMware Workstation
    - C. VMware ESXi
    - D. Docker

    ??? success "Réponse"
        **C.** ESXi s'installe directement sur le matériel. VirtualBox et VMware Workstation sont des hyperviseurs de type 2 ; Docker est un moteur de conteneurs, pas un hyperviseur.

    **Question 5.** Dans VMware, quel fichier contient le contenu du disque d'une VM ?

    - A. Le fichier `.vmx`
    - B. Le fichier `.vmdk`
    - C. Le fichier `.vbox`
    - D. Le fichier `.iso`

    ??? success "Réponse"
        **B.** Le `.vmdk` est le disque virtuel. Le `.vmx` est le fichier de configuration de la VM ; le `.vbox` est le fichier de configuration de VirtualBox ; un `.iso` est une image de support d'installation.

    **Question 6.** Qu'est-ce que le *thin provisioning* d'un disque virtuel ?

    - A. Le fichier grandit au fur et à mesure de l'utilisation, jusqu'à la taille maximale déclarée
    - B. Tout l'espace du disque est réservé dès la création
    - C. Le disque est automatiquement compressé
    - D. Le disque est copié sur un second hôte

    ??? success "Réponse"
        **A.** Le *thin provisioning* économise de la place, mais l'hôte peut manquer d'espace si les disques grandissent plus vite que prévu. La proposition B décrit le *thick provisioning*.

    **Question 7.** Laquelle de ces affirmations sur les snapshots est vraie ?

    - A. Un snapshot est une sauvegarde complète et indépendante de la VM
    - B. Un snapshot ne consomme jamais d'espace disque
    - C. Un snapshot permet de restaurer la VM même si l'hôte est perdu
    - D. Un snapshot n'est pas une sauvegarde : il dépend du disque d'origine

    ??? success "Réponse"
        **D.** Le snapshot est stocké avec la VM et dépend de son disque : si l'hôte ou le disque est perdu, le snapshot l'est aussi. Il consomme de l'espace, et une longue chaîne de snapshots ralentit la VM.

    **Question 8.** Un hôte de 4 cœurs exécute 6 VM de 2 vCPU chacune. Quel est le rapport vCPU par cœur ?

    - A. 0,33
    - B. 2
    - C. 3
    - D. 12

    ??? success "Réponse"
        **C.** 6 × 2 = 12 vCPU pour 4 cœurs, soit 12 ÷ 4 = 3 vCPU par cœur. La valeur 12 est le nombre total de vCPU, pas le rapport.

    **Question 9.** Quelle affirmation décrit correctement un conteneur ?

    - A. Il embarque son propre noyau, comme une VM
    - B. Il partage le noyau de l'hôte et s'appuie sur les namespaces et les cgroups
    - C. Il nécessite obligatoirement un hyperviseur de type 1
    - D. Il émule entièrement le matériel d'un ordinateur

    ??? success "Réponse"
        **B.** Un conteneur est un processus isolé par le noyau de l'hôte : les namespaces donnent sa vue du système, les cgroups limitent ses ressources. C'est ce qui le rend léger, mais moins isolé qu'une VM.

    **Question 10.** Vous avez exécuté `bcdedit /set hypervisorlaunchtype off`, puis redémarré, et rien d'autre ne maintient l'hyperviseur Windows actif. Quelle valeur attendez-vous pour `HypervisorPresent` ?

    - A. `False`
    - B. `True`
    - C. `Disabled`
    - D. Une erreur « accès refusé »

    ??? success "Réponse"
        **A.** L'hyperviseur de Windows ne démarre plus : `HypervisorPresent` vaut `False`. S'il reste à `True`, une autre fonctionnalité (plateforme de machine virtuelle, WSL 2, intégrité de la mémoire…) le maintient actif.

=== "English"

    !!! abstract "Instructions"
        Self-assessment: 10 multiple-choice questions, about 10 minutes. Only one answer is correct per question. Answer from memory first, then click "Answer" to check. If an answer surprises you, reread the matching section of the [lesson](cours.md).

    **Question 1.** What is the main risk of grouping many VMs on a single host?

    - A. The VMs automatically lose their isolation
    - B. The failure of the host stops all its VMs
    - C. Software licenses become mandatory
    - D. The host disk can no longer be read

    ??? success "Answer"
        **B.** The host becomes a single point of failure unless redundancy is provided. Isolation between VMs is, on the contrary, one of the benefits of virtualization.

    **Question 2.** In which technique does the guest system know it is virtualized and use adapted drivers (for example `virtio`)?

    - A. Emulation
    - B. Full virtualization
    - C. Hardware assistance
    - D. Paravirtualization

    ??? success "Answer"
        **D.** In paravirtualization, the guest cooperates with the hypervisor through dedicated drivers, most effective for disks and network. In full virtualization, the guest is unmodified and unaware that it is virtualized.

    **Question 3.** What are Intel VT-x and AMD-V?

    - A. Processor hardware extensions that make virtualization easier
    - B. Two type 2 hypervisors
    - C. Virtual disk formats
    - D. Virtual network protocols

    ??? success "Answer"
        **A.** They are processor features that must be enabled in the firmware (BIOS/UEFI) so that hypervisors can use them.

    **Question 4.** Which of these is a type 1 hypervisor?

    - A. VirtualBox
    - B. VMware Workstation
    - C. VMware ESXi
    - D. Docker

    ??? success "Answer"
        **C.** ESXi installs directly on the hardware. VirtualBox and VMware Workstation are type 2 hypervisors; Docker is a container engine, not a hypervisor.

    **Question 5.** In VMware, which file holds the content of a VM's disk?

    - A. The `.vmx` file
    - B. The `.vmdk` file
    - C. The `.vbox` file
    - D. The `.iso` file

    ??? success "Answer"
        **B.** The `.vmdk` is the virtual disk. The `.vmx` is the VM configuration file; the `.vbox` is VirtualBox's configuration file; an `.iso` is an installation media image.

    **Question 6.** What is thin provisioning of a virtual disk?

    - A. The file grows as it is used, up to the declared maximum size
    - B. All the disk space is reserved at creation
    - C. The disk is automatically compressed
    - D. The disk is copied to a second host

    ??? success "Answer"
        **A.** Thin provisioning saves space, but the host can run out of space if disks grow faster than planned. Option B describes thick provisioning.

    **Question 7.** Which of these statements about snapshots is true?

    - A. A snapshot is a complete, independent backup of the VM
    - B. A snapshot never uses disk space
    - C. A snapshot allows restoring the VM even if the host is lost
    - D. A snapshot is not a backup: it depends on the original disk

    ??? success "Answer"
        **D.** The snapshot is stored with the VM and depends on its disk: if the host or the disk is lost, so is the snapshot. It uses space, and a long snapshot chain slows the VM down.

    **Question 8.** A 4-core host runs 6 VMs of 2 vCPUs each. What is the vCPU-per-core ratio?

    - A. 0.33
    - B. 2
    - C. 3
    - D. 12

    ??? success "Answer"
        **C.** 6 × 2 = 12 vCPUs for 4 cores, so 12 ÷ 4 = 3 vCPUs per core. The value 12 is the total number of vCPUs, not the ratio.

    **Question 9.** Which statement correctly describes a container?

    - A. It ships its own kernel, like a VM
    - B. It shares the host kernel and relies on namespaces and cgroups
    - C. It necessarily requires a type 1 hypervisor
    - D. It fully emulates a computer's hardware

    ??? success "Answer"
        **B.** A container is a process isolated by the host kernel: namespaces give it its view of the system, cgroups limit its resources. This makes it light, but less isolated than a VM.

    **Question 10.** You ran `bcdedit /set hypervisorlaunchtype off`, then restarted, and nothing else keeps the Windows hypervisor active. What value do you expect for `HypervisorPresent`?

    - A. `False`
    - B. `True`
    - C. `Disabled`
    - D. An "access denied" error

    ??? success "Answer"
        **A.** The Windows hypervisor no longer starts: `HypervisorPresent` is `False`. If it stays `True`, another feature (virtual machine platform, WSL 2, memory integrity...) keeps it active.