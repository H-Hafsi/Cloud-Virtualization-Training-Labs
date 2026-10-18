# Lab 8.3 : Choisir un niveau RAID selon un cahier des charges / Choosing a RAID Level from Requirements

| | |
|---|---|
| **Durée / Duration** | 60 à 75 min / 60 to 75 min |
| **Niveau / Level** | Semi-guidé / Semi-guided |
| **Prérequis / Prerequisites** | [Chapitre 8 : cours](cours.md) ; [Lab 8.1](lab-8-1-raid-1-et-raid-5-avec-mdadm.md) et [Lab 8.2](lab-8-2-panne-remplacement-et-reconstruction.md) conseillés |
| **Ressources minimales / Minimum resources** | Aucune machine virtuelle. Une calculatrice ou un tableur (par exemple LibreOffice Calc, gratuit) et le cours du chapitre 8. Exercice individuel ou en binôme. / No virtual machine. A calculator or a spreadsheet (for example free LibreOffice Calc) and the chapter 8 lesson. Individual or pair exercise. |
| **Acquis visés / Learning outcomes** | AA3 |

=== "Français"

    !!! abstract "Objectifs"
        - Traduire un cahier des charges (capacité, tolérance aux pannes, charge, contraintes matérielles) en critères de choix
        - Calculer le nombre de disques et la capacité utile pour chaque niveau RAID candidat
        - Comparer les niveaux et justifier un choix en tenant compte des performances et du risque de reconstruction
        - Identifier ce que le RAID ne couvre pas et ce qu'il faut ajouter

=== "English"

    !!! abstract "Objectives"
        - Translate requirements (capacity, fault tolerance, workload, hardware constraints) into selection criteria
        - Compute the number of disks and the usable capacity for each candidate RAID level
        - Compare levels and justify a choice taking performance and rebuild risk into account
        - Identify what RAID does not cover and what must be added

## Contexte / Context

=== "Français"

    Vous êtes l'équipe infrastructure d'un établissement. Quatre demandes vous parviennent. Pour chacune, vous devez **choisir et justifier** un niveau RAID. Toutes les données sont **fictives**. Il n'y a pas de manipulation sur machine : c'est un exercice de raisonnement, dont la démarche compte autant que le résultat.

    **Rappel des formules du cours** pour *n* disques de capacité *C* :

    | Niveau | Capacité utile | Pannes garanties | Disques minimum |
    |---|---|---|---|
    | RAID 0 | n × C | 0 | 2 |
    | RAID 1 | C | n − 1 | 2 |
    | RAID 5 | (n − 1) × C | 1 | 3 |
    | RAID 6 | (n − 2) × C | 2 | 4 |
    | RAID 10 | (n ÷ 2) × C | 1 (garantie) | 4 |

    **Nombre de disques nécessaires** pour une capacité utile *U* (en arrondissant *U ÷ C* à l'entier supérieur, noté *k*) : RAID 0 : *k* ; RAID 1 : 2 (si U ≤ C) ; RAID 5 : *k* + 1 ; RAID 6 : *k* + 2 ; RAID 10 : 2 × *k*.

    **Rendement** = capacité utile ÷ capacité brute (n × C).

=== "English"

    You are an institution's infrastructure team. Four requests arrive. For each, you must **choose and justify** a RAID level. All data is **fictional**. There is no hands-on work on a machine: this is a reasoning exercise, where the approach matters as much as the result.

    **Reminder of the lesson's formulas** for *n* disks of capacity *C*:

    | Level | Usable capacity | Guaranteed failures | Minimum disks |
    |---|---|---|---|
    | RAID 0 | n × C | 0 | 2 |
    | RAID 1 | C | n − 1 | 2 |
    | RAID 5 | (n − 1) × C | 1 | 3 |
    | RAID 6 | (n − 2) × C | 2 | 4 |
    | RAID 10 | (n ÷ 2) × C | 1 (guaranteed) | 4 |

    **Number of disks needed** for a usable capacity *U* (rounding *U ÷ C* up to the next integer, written *k*): RAID 0: *k*; RAID 1: 2 (if U ≤ C); RAID 5: *k* + 1; RAID 6: *k* + 2; RAID 10: 2 × *k*.

    **Efficiency** = usable capacity ÷ raw capacity (n × C).

## Les quatre demandes / The four requests

=== "Français"

    | Demande | Besoin | Charge | Tolérance exigée | Contraintes matérielles |
    |---|---|---|---|---|
    | **S1** Serveur de fichiers du département | **12 To** utiles | Surtout des lectures (environ 90 %) | **1 panne** | Disques de 4 To ; 6 baies au maximum |
    | **S2** Bases de données et VM d'un atelier | **2 To** utiles | Beaucoup d'écritures aléatoires (environ 50 %) ; reconstruction courte souhaitée | **1 panne** | SSD de 1 To ; 8 baies au maximum |
    | **S3** Archivage de sauvegardes réglementaire | **40 To** utiles | Écritures séquentielles, lectures rares | **2 pannes** (exigence réglementaire) | Disques de 10 To ; 8 baies au maximum |
    | **S4** Espace temporaire de montage vidéo | **8 To** utiles | Débit maximal ; les données sont **recopiables** depuis les sources | Aucune exigence | Disques de 4 To ; 4 baies au maximum |

=== "English"

    | Request | Need | Workload | Required tolerance | Hardware constraints |
    |---|---|---|---|---|
    | **S1** Department file server | **12 TB** usable | Mostly reads (about 90%) | **1 failure** | 4 TB disks; 6 bays maximum |
    | **S2** Workshop databases and VMs | **2 TB** usable | Many random writes (about 50%); short rebuild desired | **1 failure** | 1 TB SSDs; 8 bays maximum |
    | **S3** Regulatory backup archive | **40 TB** usable | Sequential writes, rare reads | **2 failures** (regulatory requirement) | 10 TB disks; 8 bays maximum |
    | **S4** Temporary video editing space | **8 TB** usable | Maximum throughput; data is **recopiable** from the sources | No requirement | 4 TB disks; 4 bays maximum |

## Étapes / Steps

### Étape 1 : Lister les niveaux candidats pour chaque demande / Step 1: List the candidate levels for each request

=== "Français"

    Pour chaque demande, remplissez un tableau comme celui-ci pour **chaque niveau envisageable** (RAID 0, 1, 5, 6, 10) :

    | Niveau | Disques nécessaires *n* | Capacité utile obtenue | Pannes garanties | Rendement | Dans la limite des baies ? | Répond à la tolérance exigée ? |
    |---|---|---|---|---|---|---|
    | … | … | … | … | … | … | … |

    **Indices :** calculez *k* = U ÷ C arrondi au supérieur ; appliquez les formules du rappel ; éliminez les niveaux qui dépassent les baies disponibles ou ne garantissent pas la tolérance exigée. Attention : un RAID 10 ne garantit qu'**une** panne, même s'il peut en supporter davantage selon les disques touchés.

    !!! success "Point de contrôle"
        - [ ] Un tableau par demande, avec le calcul de *n* visible
        - [ ] Les niveaux qui ne conviennent pas sont **éliminés avec la raison** (baies, tolérance, capacité)

=== "English"

    For each request, fill in a table like this one for **each conceivable level** (RAID 0, 1, 5, 6, 10):

    | Level | Disks needed *n* | Usable capacity obtained | Guaranteed failures | Efficiency | Within bay limit? | Meets required tolerance? |
    |---|---|---|---|---|---|---|
    | … | … | … | … | … | … | … |

    **Hints:** compute *k* = U ÷ C rounded up; apply the reminder formulas; eliminate levels that exceed the available bays or do not guarantee the required tolerance. Beware: a RAID 10 only guarantees **one** failure, even though it may withstand more depending on which disks are hit.

    !!! success "Checkpoint"
        - [ ] One table per request, with the computation of *n* visible
        - [ ] Unsuitable levels are **eliminated with the reason** (bays, tolerance, capacity)

### Étape 2 : Comparer les niveaux restants / Step 2: Compare the remaining levels

=== "Français"

    Parmi les niveaux qui satisfont la capacité, la tolérance et les baies, comparez selon les critères du cours :

    | Critère | À examiner |
    |---|---|
    | **Performances** | Adéquation du niveau à la charge : lectures majoritaires, écritures aléatoires (pénalité d'écriture : environ 2 pour RAID 1 et 10, 4 pour RAID 5, 6 pour RAID 6), écritures séquentielles |
    | **Reconstruction** | Durée (taille des disques) et risque d'une seconde panne ou d'une erreur de lecture pendant la reconstruction |
    | **Rendement** | Part de la capacité brute réellement utilisable |
    | **Marge** | Possibilité d'ajouter un disque de secours dans les baies restantes |

    **Indice pour la reconstruction :** plus les disques sont gros et plus la grappe repose sur la parité, plus la fenêtre de vulnérabilité est longue et risquée (chapitre 8, section 4).

    !!! success "Point de contrôle"
        - [ ] Pour chaque demande, au moins deux niveaux restants sont comparés selon les quatre critères

=== "English"

    Among the levels that satisfy capacity, tolerance and bays, compare using the lesson's criteria:

    | Criterion | To examine |
    |---|---|
    | **Performance** | Fit of the level to the workload: mostly reads, random writes (write penalty: about 2 for RAID 1 and 10, 4 for RAID 5, 6 for RAID 6), sequential writes |
    | **Rebuild** | Duration (disk size) and risk of a second failure or a read error during the rebuild |
    | **Efficiency** | Share of raw capacity actually usable |
    | **Margin** | Possibility of adding a hot spare in the remaining bays |

    **Hint for rebuild:** the larger the disks and the more the array relies on parity, the longer and riskier the vulnerability window (chapter 8, section 4).

    !!! success "Checkpoint"
        - [ ] For each request, at least two remaining levels are compared on the four criteria

### Étape 3 : Décider et justifier / Step 3: Decide and justify

=== "Français"

    Pour chaque demande, rédigez une **décision** en cinq lignes au plus :

    1. le niveau retenu, le nombre de disques et la capacité utile obtenue ;
    2. les **deux raisons principales** du choix, reliées au cahier des charges ;
    3. le niveau qui arrivait en deuxième position et pourquoi il n'a pas été retenu ;
    4. la présence ou non d'un **disque de secours** dans les baies restantes ;
    5. les **risques résiduels**.

    !!! success "Point de contrôle"
        - [ ] Quatre décisions rédigées, chacune justifiée par des éléments chiffrés du cahier des charges

=== "English"

    For each request, write a **decision** in five lines at most:

    1. the chosen level, the number of disks and the usable capacity obtained;
    2. the **two main reasons** for the choice, tied to the requirements;
    3. the level that came second and why it was not chosen;
    4. the presence or not of a **hot spare** in the remaining bays;
    5. the **residual risks**.

    !!! success "Checkpoint"
        - [ ] Four decisions written, each justified by figures from the requirements

### Étape 4 : Ce que le RAID ne couvre pas / Step 4: What RAID does not cover

=== "Français"

    Pour **chaque** demande, indiquez ce qui doit **compléter** le RAID :

    | Risque | Réponse à prévoir |
    |---|---|
    | Suppression accidentelle, corruption, virus | … |
    | Sinistre sur le site (incendie, vol) | … |
    | Panne du contrôleur ou de la machine | … |
    | Panne non détectée | … |

    **Indices :** sauvegarde indépendante et hors site (règle 3-2-1, chapitre 22), surveillance et alertes, contrôle de cohérence régulier, pièces de rechange. Pour S4, expliquez pourquoi l'absence de redondance est acceptable **ici** et dans quelles conditions elle ne le serait plus.

    !!! success "Point de contrôle"
        - [ ] Un complément précis est proposé pour chaque risque et chaque demande

=== "English"

    For **each** request, state what must **complement** RAID:

    | Risk | Response to provide |
    |---|---|
    | Accidental deletion, corruption, virus | … |
    | Site disaster (fire, theft) | … |
    | Controller or machine failure | … |
    | Undetected failure | … |

    **Hints:** independent off-site backup (3-2-1 rule, chapter 22), monitoring and alerts, regular consistency check, spare parts. For S4, explain why the lack of redundancy is acceptable **here** and under which conditions it would no longer be.

    !!! success "Checkpoint"
        - [ ] A precise complement is proposed for each risk and each request

### Étape 5 : Vérifier un choix par l'expérience (facultatif) / Step 5: Check a choice through experiment (optional)

=== "Français"

    Si vous disposez de la VM du Lab 8.1, reproduisez **en petit** l'une de vos décisions (par exemple le niveau retenu pour S2 ou S3) avec des disques de 2 Go : créez la grappe avec le nombre de disques calculé, vérifiez que la **capacité observée** est celle attendue par la formule, simulez une panne et vérifiez la tolérance (Lab 8.2). Notez tout écart entre le calcul et l'observation.

=== "English"

    If you have the Lab 8.1 VM, reproduce **in small** one of your decisions (for example the level chosen for S2 or S3) with 2 GB disks: create the array with the computed number of disks, check that the **observed capacity** matches what the formula expects, simulate a failure and check the tolerance (Lab 8.2). Note any difference between computation and observation.

### Étape 6 : Livrable / Step 6: Deliverable

=== "Français"

    Rendez un document de décision (une page par demande, ou un tableau de synthèse) :

    | Demande | Niveau retenu | Disques et capacité utile | Tolérance garantie | Disque de secours | Complément au RAID |
    |---|---|---|---|---|---|
    | S1 | … | … | … | … | … |
    | S2 | … | … | … | … | … |
    | S3 | … | … | … | … | … |
    | S4 | … | … | … | … | … |

    Joignez les tableaux de calcul des étapes 1 et 2 et les cinq lignes de justification de l'étape 3.

=== "English"

    Hand in a decision document (one page per request, or a summary table):

    | Request | Chosen level | Disks and usable capacity | Guaranteed tolerance | Hot spare | Complement to RAID |
    |---|---|---|---|---|---|
    | S1 | … | … | … | … | … |
    | S2 | … | … | … | … | … |
    | S3 | … | … | … | … | … |
    | S4 | … | … | … | … | … |

    Attach the computation tables from steps 1 and 2 and the five justification lines from step 3.

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi déconseille-t-on souvent le RAID 5 pour de très gros disques ?"
        La reconstruction lit tous les disques restants et dure longtemps : pendant ce temps, la grappe n'a plus de redondance, et une seconde panne ou une erreur de lecture irrécupérable fait perdre les données. Le risque augmente avec la taille des disques. Un RAID 6 ou un RAID 10 réduit ce risque.

    ??? question "Pourquoi le RAID 10 est-il souvent préféré pour des bases de données ou des machines virtuelles ?"
        Il n'a pas de pénalité de parité pour les petites écritures aléatoires, et sa reconstruction consiste à recopier un seul disque miroir, ce qui est rapide et peu risqué. Le prix en est un rendement de 50 % de la capacité brute.

    ??? question "Une grappe en RAID 6 est-elle une protection suffisante pour des données importantes ?"
        Non : elle protège de deux pannes de disque, mais pas d'une suppression, d'un virus, d'un sinistre ou d'une erreur d'administration. Une sauvegarde indépendante et hors site reste indispensable.

=== "English"

    ??? question "Why is RAID 5 often discouraged for very large disks?"
        The rebuild reads all remaining disks and takes a long time: during that time, the array has no redundancy left, and a second failure or an unrecoverable read error loses the data. The risk grows with disk size. RAID 6 or RAID 10 reduces this risk.

    ??? question "Why is RAID 10 often preferred for databases or virtual machines?"
        It has no parity penalty for small random writes, and its rebuild consists in copying a single mirror disk, which is fast and low-risk. The price is an efficiency of 50% of raw capacity.

    ??? question "Is a RAID 6 array enough protection for important data?"
        No: it protects against two disk failures, but not against deletion, a virus, a disaster or an administration error. An independent off-site backup remains essential.

## Erreurs fréquentes / Common mistakes

=== "Français"

    | Erreur | Conséquence |
    |---|---|
    | Confondre capacité **brute** (tous les disques) et capacité **utile** | Grappe trop petite ou surdimensionnée |
    | Oublier le ou les disques de parité dans le calcul du nombre de disques | Capacité utile inférieure au besoin |
    | Croire qu'un RAID 10 supporte **toujours** deux pannes | Perte de données si les deux pannes frappent la même paire miroir |
    | Dépasser le nombre de baies disponibles | Solution irréalisable |
    | Arrondir *U ÷ C* vers le bas | Capacité insuffisante |
    | Considérer que le RAID remplace la sauvegarde | Perte de données irrécupérable en cas d'erreur ou de sinistre |
    | Choisir un niveau sans l'argumenter par la charge | Décision peu défendable |

=== "English"

    | Mistake | Consequence |
    |---|---|
    | Mixing up **raw** capacity (all disks) and **usable** capacity | Array too small or oversized |
    | Forgetting the parity disk(s) when counting disks | Usable capacity below the need |
    | Believing a RAID 10 **always** withstands two failures | Data loss if both failures hit the same mirror pair |
    | Exceeding the number of available bays | Unfeasible solution |
    | Rounding *U ÷ C* down | Insufficient capacity |
    | Considering that RAID replaces backup | Unrecoverable data loss on error or disaster |
    | Choosing a level without arguing it from the workload | Hard-to-defend decision |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    Cet exercice ne modifie aucune machine : il n'y a rien à nettoyer. Si vous avez fait l'étape 5, arrêtez les grappes et effacez les métadonnées des disques comme au Lab 8.1. Conservez votre document de décision : la grille de choix (capacité, tolérance, charge, reconstruction) resservira pour le stockage en réseau et le stockage défini par logiciel (chapitres 10 et 11).

=== "English"

    This exercise changes no machine: there is nothing to clean up. If you did step 5, stop the arrays and erase the disks' metadata as in Lab 8.1. Keep your decision document: the selection grid (capacity, tolerance, workload, rebuild) will be used again for network storage and software-defined storage (chapters 10 and 11).

[Retour au chapitre 8 / Back to chapter 8](cours.md)