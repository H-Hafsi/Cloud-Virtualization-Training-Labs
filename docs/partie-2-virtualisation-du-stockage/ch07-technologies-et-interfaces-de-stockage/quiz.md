# Quiz du chapitre 7 : Technologies et interfaces de stockage / Chapter 7 Quiz: Storage Technologies and Interfaces

=== "Français"

    !!! abstract "Consignes"
        Autoévaluation : 10 questions à choix multiples, environ 10 minutes. Une seule réponse est correcte par question. Répondez d'abord de mémoire, puis cliquez sur « Réponse » pour vérifier. Si une réponse vous surprend, relisez la section correspondante du [cours](cours.md).

    **Question 1.** Laquelle de ces interfaces offre le plus grand débit théorique ?

    - A. SATA III
    - B. NVMe sur PCIe 4.0 ×4
    - C. SAS-3
    - D. IDE

    ??? success "Réponse"
        **B.** Environ 8 Go/s, contre environ 600 Mo/s pour SATA III et 12 Gbit/s pour SAS-3. L'IDE est une interface ancienne, bien plus lente.

    **Question 2.** Quelle relation lie le débit, les IOPS et la taille de bloc ?

    - A. Débit = IOPS ÷ taille de bloc
    - B. Débit = latence × IOPS
    - C. Débit = capacité ÷ durée
    - D. Débit = IOPS × taille de bloc

    ??? success "Réponse"
        **D.** Le débit est le nombre d'opérations par seconde multiplié par la quantité de données de chaque opération.

    **Question 3.** Un SSD réalise 20 000 IOPS en blocs de 4 Kio. Quel débit cela représente-t-il environ ?

    - A. Environ 80 Mo/s
    - B. Environ 800 Mo/s
    - C. Environ 8 Mo/s
    - D. Environ 20 Mo/s

    ??? success "Réponse"
        **A.** 20 000 × 4 Kio = 80 000 Kio/s, soit environ 80 Mo/s. Beaucoup d'IOPS en petits blocs ne donnent donc pas un grand débit.

    **Question 4.** Quel est le principal point faible d'un disque dur (HDD) par rapport à un SSD ?

    - A. Sa capacité
    - B. Son coût par Go
    - C. La latence des accès aléatoires, due au déplacement de la tête et à la rotation
    - D. Sa compatibilité avec les serveurs

    ??? success "Réponse"
        **C.** Chaque accès aléatoire exige de déplacer la tête et d'attendre la rotation du plateau, d'où une latence de l'ordre de la milliseconde. En revanche, le HDD est avantageux en capacité et en coût par Go.

    **Question 5.** À quoi sert la commande TRIM ?

    - A. À effacer définitivement des fichiers
    - B. À informer le SSD des blocs qui ne servent plus, pour préserver ses performances
    - C. À chiffrer le disque
    - D. À défragmenter le SSD

    ??? success "Réponse"
        **B.** Le SSD peut alors réutiliser ces blocs sans les recopier inutilement. La défragmentation n'a pas d'intérêt sur un SSD.

    **Question 6.** Quelle est la caractéristique du format de disque raw ?

    - A. Il offre l'allocation dynamique et les snapshots
    - B. Il n'existe que sous VMware
    - C. Il est propre à Hyper-V
    - D. C'est une image brute sans structure : performances maximales, mais ni snapshots ni allocation dynamique propres au format

    ??? success "Réponse"
        **D.** Les fonctions avancées (allocation dynamique, snapshots, chaînage) appartiennent à des formats comme qcow2, VMDK ou VHDX.

    **Question 7.** Quelle est une conséquence du provisionnement dynamique (*thin*) d'un disque virtuel ?

    - A. La première écriture doit allouer de l'espace, ce qui la ralentit un peu ; l'hôte peut manquer d'espace si les disques grandissent
    - B. Les accès sont toujours plus rapides qu'avec un disque fixe
    - C. Tout l'espace est réservé à la création
    - D. Il n'y a aucun risque pour l'hôte

    ??? success "Réponse"
        **A.** Le dynamique économise de la place mais est moins prévisible. La proposition C décrit le provisionnement fixe.

    **Question 8.** Quelles précautions limitent l'effet des caches lors d'une mesure avec `fio` ?

    - A. Utiliser un fichier de test très petit
    - B. Ne pas préciser de paramètres
    - C. Utiliser `--direct=1` et une taille de test supérieure à la mémoire
    - D. Ne faire qu'une seule mesure

    ??? success "Réponse"
        **C.** `--direct=1` contourne le cache du système ; une taille supérieure à la mémoire empêche de tout servir depuis le cache. Il faut aussi répéter la mesure.

    **Question 9.** Quelle précaution est indispensable avant un test d'écriture avec `fio` sur un périphérique brut ?

    - A. Le lancer en tant que root
    - B. S'assurer que le disque est vide et dédié au test, jamais le disque système
    - C. Utiliser l'option `--runtime`
    - D. Désactiver le cache du disque

    ??? success "Réponse"
        **B.** Le test écrit sur tout le disque sans tenir compte du système de fichiers : il détruit son contenu.

    **Question 10.** Pourquoi un disque NVMe virtuel n'est-il pas forcément beaucoup plus rapide qu'un disque SCSI virtuel ?

    - A. Le support physique de l'hôte reste le facteur limitant, quel que soit le contrôleur virtuel
    - B. Un NVMe virtuel est toujours plus lent
    - C. `fio` ne sait pas mesurer le NVMe
    - D. L'invité interdit le NVMe

    ??? success "Réponse"
        **A.** Le disque virtuel est un fichier sur le disque physique de l'hôte. Le contrôleur virtuel peut réduire le surcoût logiciel mais ne rend pas le support plus rapide.

=== "English"

    !!! abstract "Instructions"
        Self-assessment: 10 multiple-choice questions, about 10 minutes. Only one answer is correct per question. Answer from memory first, then click "Answer" to check. If an answer surprises you, reread the matching section of the [lesson](cours.md).

    **Question 1.** Which of these interfaces offers the highest theoretical throughput?

    - A. SATA III
    - B. NVMe over PCIe 4.0 ×4
    - C. SAS-3
    - D. IDE

    ??? success "Answer"
        **B.** About 8 GB/s, against about 600 MB/s for SATA III and 12 Gbit/s for SAS-3. IDE is an old, much slower interface.

    **Question 2.** Which relation links throughput, IOPS and block size?

    - A. Throughput = IOPS ÷ block size
    - B. Throughput = latency × IOPS
    - C. Throughput = capacity ÷ duration
    - D. Throughput = IOPS × block size

    ??? success "Answer"
        **D.** Throughput is the number of operations per second multiplied by the amount of data in each operation.

    **Question 3.** An SSD does 20,000 IOPS with 4 KiB blocks. About what throughput is that?

    - A. About 80 MB/s
    - B. About 800 MB/s
    - C. About 8 MB/s
    - D. About 20 MB/s

    ??? success "Answer"
        **A.** 20,000 × 4 KiB = 80,000 KiB/s, about 80 MB/s. Many IOPS with small blocks therefore do not give high throughput.

    **Question 4.** What is the main weakness of a hard disk (HDD) compared with an SSD?

    - A. Its capacity
    - B. Its cost per GB
    - C. The latency of random accesses, due to head movement and rotation
    - D. Its compatibility with servers

    ??? success "Answer"
        **C.** Each random access requires moving the head and waiting for the platter to rotate, hence a latency in the millisecond range. On the other hand, HDDs are advantageous in capacity and cost per GB.

    **Question 5.** What is the TRIM command used for?

    - A. To permanently erase files
    - B. To tell the SSD which blocks are no longer used, to preserve its performance
    - C. To encrypt the disk
    - D. To defragment the SSD

    ??? success "Answer"
        **B.** The SSD can then reuse these blocks without copying them needlessly. Defragmentation has no benefit on an SSD.

    **Question 6.** What characterizes the raw disk format?

    - A. It offers dynamic allocation and snapshots
    - B. It only exists on VMware
    - C. It is specific to Hyper-V
    - D. It is a raw image without structure: maximum performance, but no format-level snapshots or dynamic allocation

    ??? success "Answer"
        **D.** Advanced features (dynamic allocation, snapshots, chaining) belong to formats like qcow2, VMDK or VHDX.

    **Question 7.** What is a consequence of dynamic (*thin*) provisioning of a virtual disk?

    - A. The first write has to allocate space, which slows it slightly; the host can run out of space if disks grow
    - B. Access is always faster than with a fixed disk
    - C. All the space is reserved at creation
    - D. There is no risk for the host

    ??? success "Answer"
        **A.** Dynamic saves space but is less predictable. Option C describes fixed provisioning.

    **Question 8.** Which precautions limit the effect of caches during a `fio` measurement?

    - A. Using a very small test file
    - B. Not specifying any parameters
    - C. Using `--direct=1` and a test size larger than memory
    - D. Doing only one measurement

    ??? success "Answer"
        **C.** `--direct=1` bypasses the system cache; a size larger than memory prevents everything from being served from cache. The measurement should also be repeated.

    **Question 9.** Which precaution is essential before a `fio` write test on a raw device?

    - A. Running it as root
    - B. Making sure the disk is empty and dedicated to the test, never the system disk
    - C. Using the `--runtime` option
    - D. Disabling the disk cache

    ??? success "Answer"
        **B.** The test writes on the whole disk ignoring the file system: it destroys its content.

    **Question 10.** Why is a virtual NVMe disk not necessarily much faster than a virtual SCSI disk?

    - A. The host's physical medium remains the limiting factor, whatever the virtual controller
    - B. A virtual NVMe is always slower
    - C. `fio` cannot measure NVMe
    - D. The guest forbids NVMe

    ??? success "Answer"
        **A.** The virtual disk is a file on the host's physical disk. The virtual controller can reduce software overhead but does not make the medium faster.