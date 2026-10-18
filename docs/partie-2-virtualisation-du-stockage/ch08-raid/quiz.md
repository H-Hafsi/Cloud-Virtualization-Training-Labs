# Quiz du chapitre 8 : RAID / Chapter 8 Quiz: RAID

=== "Français"

    !!! abstract "Consignes"
        Autoévaluation : 10 questions à choix multiples, environ 10 minutes. Une seule réponse est correcte par question. Répondez d'abord de mémoire, puis cliquez sur « Réponse » pour vérifier. Si une réponse vous surprend, relisez la section correspondante du [cours](cours.md).

    **Question 1.** Quelle technique écrit chaque donnée à l'identique sur deux disques ou plus ?

    - A. La répartition (*striping*)
    - B. La parité
    - C. Le miroir (*mirroring*)
    - D. Le mode linéaire (JBOD)

    ??? success "Réponse"
        **C.** Le miroir conserve une copie complète de chaque donnée sur chaque disque. La répartition découpe les données entre les disques, la parité ajoute une information de reconstruction, et le mode linéaire met simplement les disques bout à bout.

    **Question 2.** Combien de pannes de disque un RAID 0 tolère-t-il ?

    - A. Aucune
    - B. Une
    - C. Deux
    - D. n − 1

    ??? success "Réponse"
        **A.** Le RAID 0 répartit les données sans aucune redondance : la perte d'un seul disque détruit tout le volume.

    **Question 3.** Quelle est la capacité utile d'un RAID 5 de cinq disques de 4 To ?

    - A. 20 To
    - B. 12 To
    - C. 8 To
    - D. 16 To

    ??? success "Réponse"
        **D.** (n − 1) × C = (5 − 1) × 4 = 16 To. L'équivalent d'un disque est consacré à la parité.

    **Question 4.** Quelle est la capacité utile d'un RAID 6 de quatre disques de 2 To ?

    - A. 2 To
    - B. 4 To
    - C. 6 To
    - D. 8 To

    ??? success "Réponse"
        **B.** (n − 2) × C = (4 − 2) × 2 = 4 To. Deux disques sont consacrés à la double parité, ce qui permet de supporter deux pannes.

    **Question 5.** Quelle est la capacité utile d'un RAID 10 de six disques de 2 To ?

    - A. 6 To
    - B. 8 To
    - C. 12 To
    - D. 4 To

    ??? success "Réponse"
        **A.** (n ÷ 2) × C = (6 ÷ 2) × 2 = 6 To. La moitié de la capacité brute sert à la copie.

    **Question 6.** Deux blocs de données valent D1 = `1100` et D2 = `1010`. La parité est P = D1 ⊕ D2. D1 est perdu : quelle valeur retrouve-t-on en le recalculant à partir de P et de D2 ?

    - A. `0110`
    - B. `1010`
    - C. `0011`
    - D. `1100`

    ??? success "Réponse"
        **D.** P = `1100` ⊕ `1010` = `0110`. On retrouve D1 = P ⊕ D2 = `0110` ⊕ `1010` = `1100`.

    **Question 7.** Parmi ces événements, lequel le RAID ne protège-t-il pas ?

    - A. La panne d'un disque d'un RAID 1
    - B. La panne d'un disque d'un RAID 6
    - C. La suppression accidentelle d'un fichier
    - D. La panne d'un seul disque d'un RAID 10

    ??? success "Réponse"
        **C.** La suppression est répercutée immédiatement sur tous les disques de la grappe. Le RAID protège de la panne d'un disque, pas des erreurs de données : il faut une sauvegarde indépendante.

    **Question 8.** Que signifie le « mode dégradé » d'une grappe RAID ?

    - A. La grappe est arrêtée
    - B. La grappe fonctionne mais sans (ou avec moins de) redondance, et plus lentement
    - C. La grappe est plus rapide que d'habitude
    - D. Toutes les données sont perdues

    ??? success "Réponse"
        **B.** Les données restent accessibles (copie ou recalcul par la parité), mais une nouvelle panne peut être fatale tant que la reconstruction n'est pas terminée.

    **Question 9.** Quel est l'intérêt d'un disque de secours (*hot spare*) ?

    - A. Il lance automatiquement la reconstruction dès qu'un disque tombe en panne
    - B. Il augmente la capacité utile de la grappe
    - C. Il double la vitesse de la grappe
    - D. Il remplace la sauvegarde

    ??? success "Réponse"
        **A.** La période de vulnérabilité est ainsi réduite au minimum, sans attendre l'intervention d'un administrateur. Le disque de secours n'ajoute pas de capacité utile.

    **Question 10.** Dans `/proc/mdstat`, une grappe de trois disques affiche `[U_U]`. Que cela signifie-t-il ?

    - A. La grappe est saine
    - B. Les trois disques sont en panne
    - C. Un des trois disques est absent ou en panne : la grappe est dégradée
    - D. Une reconstruction est terminée

    ??? success "Réponse"
        **C.** Chaque `U` est un disque actif ; le `_` signale un disque manquant ou défaillant. Une grappe saine affiche `[UUU]`.

=== "English"

    !!! abstract "Instructions"
        Self-assessment: 10 multiple-choice questions, about 10 minutes. Only one answer is correct per question. Answer from memory first, then click "Answer" to check. If an answer surprises you, reread the matching section of the [lesson](cours.md).

    **Question 1.** Which technique writes each piece of data identically on two or more disks?

    - A. Striping
    - B. Parity
    - C. Mirroring
    - D. Linear mode (JBOD)

    ??? success "Answer"
        **C.** Mirroring keeps a complete copy of each piece of data on each disk. Striping splits data between disks, parity adds reconstruction information, and linear mode simply places disks end to end.

    **Question 2.** How many disk failures does a RAID 0 tolerate?

    - A. None
    - B. One
    - C. Two
    - D. n − 1

    ??? success "Answer"
        **A.** RAID 0 stripes data without any redundancy: losing a single disk destroys the whole volume.

    **Question 3.** What is the usable capacity of a RAID 5 of five 4 TB disks?

    - A. 20 TB
    - B. 12 TB
    - C. 8 TB
    - D. 16 TB

    ??? success "Answer"
        **D.** (n − 1) × C = (5 − 1) × 4 = 16 TB. The equivalent of one disk is devoted to parity.

    **Question 4.** What is the usable capacity of a RAID 6 of four 2 TB disks?

    - A. 2 TB
    - B. 4 TB
    - C. 6 TB
    - D. 8 TB

    ??? success "Answer"
        **B.** (n − 2) × C = (4 − 2) × 2 = 4 TB. Two disks are devoted to double parity, which allows two failures to be withstood.

    **Question 5.** What is the usable capacity of a RAID 10 of six 2 TB disks?

    - A. 6 TB
    - B. 8 TB
    - C. 12 TB
    - D. 4 TB

    ??? success "Answer"
        **A.** (n ÷ 2) × C = (6 ÷ 2) × 2 = 6 TB. Half of the raw capacity goes to the copy.

    **Question 6.** Two data blocks are D1 = `1100` and D2 = `1010`. Parity is P = D1 ⊕ D2. D1 is lost: what value is found by recomputing it from P and D2?

    - A. `0110`
    - B. `1010`
    - C. `0011`
    - D. `1100`

    ??? success "Answer"
        **D.** P = `1100` ⊕ `1010` = `0110`. D1 = P ⊕ D2 = `0110` ⊕ `1010` = `1100`.

    **Question 7.** Among these events, which one does RAID not protect against?

    - A. The failure of one disk in a RAID 1
    - B. The failure of one disk in a RAID 6
    - C. Accidental deletion of a file
    - D. The failure of a single disk in a RAID 10

    ??? success "Answer"
        **C.** Deletion is immediately replicated onto all disks of the array. RAID protects against a disk failure, not against data errors: an independent backup is needed.

    **Question 8.** What does the "degraded mode" of a RAID array mean?

    - A. The array is stopped
    - B. The array works but without (or with less) redundancy, and more slowly
    - C. The array is faster than usual
    - D. All data is lost

    ??? success "Answer"
        **B.** Data remains accessible (copy or recomputation through parity), but a new failure can be fatal until the rebuild is finished.

    **Question 9.** What is the benefit of a hot spare disk?

    - A. It automatically starts the rebuild as soon as a disk fails
    - B. It increases the array's usable capacity
    - C. It doubles the array's speed
    - D. It replaces backup

    ??? success "Answer"
        **A.** The vulnerability period is thus reduced to a minimum, without waiting for an administrator to step in. The spare disk adds no usable capacity.

    **Question 10.** In `/proc/mdstat`, a three-disk array shows `[U_U]`. What does it mean?

    - A. The array is healthy
    - B. All three disks have failed
    - C. One of the three disks is missing or failed: the array is degraded
    - D. A rebuild has finished

    ??? success "Answer"
        **C.** Each `U` is an active disk; the `_` flags a missing or failed disk. A healthy array shows `[UUU]`.