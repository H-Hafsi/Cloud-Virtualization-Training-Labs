# Chapitre 8 : RAID / RAID

=== "Français"

    !!! abstract "Objectifs du chapitre"
        - Expliquer pourquoi et comment on regroupe des disques en grappe RAID
        - Décrire les trois techniques : répartition (*striping*), miroir et parité
        - Comparer les niveaux RAID 0, 1, 5, 6 et 10 : capacité, tolérance aux pannes, performances
        - Calculer la capacité utile d'une grappe et reconstituer une donnée à partir d'une parité
        - Décrire le mode dégradé, la reconstruction, le disque de secours et les risques associés
        - Distinguer RAID matériel et logiciel, et choisir un niveau selon un besoin
        - Acquis d'apprentissage visé : **AA3**

        Matériel requis pour ce cours : aucun. Les labs associés indiquent leurs propres ressources.

=== "English"

    !!! abstract "Chapter objectives"
        - Explain why and how disks are grouped into a RAID array
        - Describe the three techniques: striping, mirroring and parity
        - Compare RAID levels 0, 1, 5, 6 and 10: capacity, fault tolerance, performance
        - Compute an array's usable capacity and rebuild a piece of data from a parity
        - Describe degraded mode, rebuilding, hot spare and the associated risks
        - Tell hardware and software RAID apart, and choose a level according to a need
        - Target learning outcome: **AA3**

        Required equipment for this lesson: none. The associated labs state their own resources.

## 1. Pourquoi le RAID ? / Why RAID?

=== "Français"

    Au chapitre 7, nous avons vu qu'un disque finit toujours par tomber en panne. Dans un serveur qui héberge de nombreuses VM, la panne d'un disque ne doit pas arrêter le service ni faire perdre les données. Le **RAID** (*Redundant Array of Independent Disks*, grappe redondante de disques indépendants) regroupe plusieurs disques en **un seul volume logique** pour atteindre un ou plusieurs de ces objectifs :

    | Objectif | Comment |
    |---|---|
    | **Disponibilité** | Le volume continue de fonctionner malgré la panne d'un (ou plusieurs) disque |
    | **Performance** | Les accès sont répartis sur plusieurs disques travaillant en parallèle |
    | **Capacité** | Plusieurs disques forment un volume plus grand qu'un seul |

    Chaque niveau RAID est un **compromis** entre ces trois objectifs et le coût (nombre de disques nécessaires).

    !!! danger "Le RAID n'est pas une sauvegarde"
        Le RAID protège contre la **panne d'un disque**, pas contre la suppression accidentelle, la corruption, un virus, un incendie ou un vol : toutes ces erreurs sont reproduites sur l'ensemble de la grappe. La sauvegarde est un autre sujet (chapitre 22, règle 3-2-1).

=== "English"

    In chapter 7, we saw that a disk always ends up failing. In a server hosting many VMs, a disk failure must neither stop the service nor lose data. **RAID** (*Redundant Array of Independent Disks*) groups several disks into **a single logical volume** to reach one or more of these goals:

    | Goal | How |
    |---|---|
    | **Availability** | The volume keeps working despite the failure of one (or more) disks |
    | **Performance** | Accesses are spread over several disks working in parallel |
    | **Capacity** | Several disks form a volume larger than a single one |

    Each RAID level is a **compromise** between these three goals and cost (number of disks required).

    !!! danger "RAID is not a backup"
        RAID protects against a **disk failure**, not against accidental deletion, corruption, a virus, a fire or a theft: all these errors are replicated across the whole array. Backup is another subject (chapter 22, 3-2-1 rule).

## 2. Les trois techniques / The three techniques

=== "Français"

    ### Répartition (*striping*)

    Les données sont découpées en **blocs** (*chunks*) écrits successivement sur chaque disque. Les disques travaillent en parallèle : la performance augmente, mais **sans redondance**, la perte d'un disque détruit tout le volume.

    ### Miroir (*mirroring*)

    Chaque donnée est écrite **à l'identique sur deux disques (ou plus)**. Si un disque tombe, l'autre contient toutes les données. Le coût est l'espace : la capacité utile n'est que celle d'un disque.

    ### Parité

    Pour un ensemble de blocs de données, on calcule un bloc de **parité** par un OU exclusif (XOR, noté ⊕). Si un bloc de données est perdu, on le reconstitue à partir des blocs restants et de la parité.

    **Exemple.** Trois blocs de 4 bits : D1 = `1011`, D2 = `0110`, D3 = `1100`.

    - Parité : P = D1 ⊕ D2 ⊕ D3 = `0001`.
    - Si D2 est perdu : D2 = P ⊕ D1 ⊕ D3 = `0001` ⊕ `1011` ⊕ `1100` = `0110`. Le bloc est retrouvé.

    Dans le RAID 5, la parité n'est pas sur un disque dédié : elle **tourne** d'un disque à l'autre pour répartir la charge d'écriture.

    | Bande | Disque 1 | Disque 2 | Disque 3 |
    |---|---|---|---|
    | 1 | A1 | A2 | **Pa** |
    | 2 | B1 | **Pb** | B2 |
    | 3 | **Pc** | C1 | C2 |

=== "English"

    ### Striping

    Data is split into **chunks** written successively on each disk. Disks work in parallel: performance increases, but **without redundancy**, losing one disk destroys the whole volume.

    ### Mirroring

    Each piece of data is written **identically on two disks (or more)**. If one disk fails, the other holds all the data. The cost is space: the usable capacity is only that of one disk.

    ### Parity

    For a set of data blocks, a **parity** block is computed with an exclusive OR (XOR, written ⊕). If a data block is lost, it is rebuilt from the remaining blocks and the parity.

    **Example.** Three 4-bit blocks: D1 = `1011`, D2 = `0110`, D3 = `1100`.

    - Parity: P = D1 ⊕ D2 ⊕ D3 = `0001`.
    - If D2 is lost: D2 = P ⊕ D1 ⊕ D3 = `0001` ⊕ `1011` ⊕ `1100` = `0110`. The block is recovered.

    In RAID 5, parity is not on a dedicated disk: it **rotates** from one disk to another to spread the write load.

    | Stripe | Disk 1 | Disk 2 | Disk 3 |
    |---|---|---|---|
    | 1 | A1 | A2 | **Pa** |
    | 2 | B1 | **Pb** | B2 |
    | 3 | **Pc** | C1 | C2 |

## 3. Les niveaux RAID / RAID levels

=== "Français"

    Pour **n disques identiques** de capacité C :

    | Niveau | Principe | Disques minimum | Capacité utile | Pannes tolérées |
    |---|---|---|---|---|
    | **RAID 0** | Répartition | 2 | n × C | **0** |
    | **RAID 1** | Miroir | 2 | C | n − 1 |
    | **RAID 5** | Répartition + parité répartie | 3 | (n − 1) × C | 1 |
    | **RAID 6** | Répartition + double parité | 4 | (n − 2) × C | 2 |
    | **RAID 10** | Miroirs répartis (répartition de paires en miroir) | 4 | (n ÷ 2) × C | Au moins 1 (jusqu'à n ÷ 2 si les pannes touchent des paires différentes) |

    **Exemple : 4 disques de 4 To.** RAID 0 : 16 To (aucune tolérance) ; RAID 5 : 12 To ; RAID 6 : 8 To ; RAID 10 : 8 To ; RAID 1 sur les 4 disques : 4 To.

    **Performances comparées**

    | Niveau | Lecture | Écriture | Remarque |
    |---|---|---|---|
    | **RAID 0** | Excellente | Excellente | Aucune sécurité |
    | **RAID 1** | Bonne (lectures réparties sur les copies) | Celle d'un disque | Simple et robuste |
    | **RAID 5** | Bonne | Pénalisée par le calcul de parité | Une petite écriture coûte environ 4 opérations disque |
    | **RAID 6** | Bonne | Plus pénalisée encore | Environ 6 opérations disque par petite écriture |
    | **RAID 10** | Très bonne | Bonne | Coût en capacité (50 %), mais pas de calcul de parité |

    ```mermaid
    flowchart TB
      subgraph R10["RAID 10 (4 disques)"]
        direction LR
        subgraph M1["Miroir 1"]
          D1["Disque 1"] --- D2["Disque 2"]
        end
        subgraph M2["Miroir 2"]
          D3["Disque 3"] --- D4["Disque 4"]
        end
      end
      S["Données réparties sur les deux miroirs"] --> R10
    ```

    Il existe aussi le mode **JBOD** (*just a bunch of disks*) ou « linéaire » : les disques sont simplement mis bout à bout, **sans aucune redondance** ni gain de performance.

=== "English"

    For **n identical disks** of capacity C:

    | Level | Principle | Minimum disks | Usable capacity | Failures tolerated |
    |---|---|---|---|---|
    | **RAID 0** | Striping | 2 | n × C | **0** |
    | **RAID 1** | Mirroring | 2 | C | n − 1 |
    | **RAID 5** | Striping + distributed parity | 3 | (n − 1) × C | 1 |
    | **RAID 6** | Striping + double parity | 4 | (n − 2) × C | 2 |
    | **RAID 10** | Striped mirrors (striping of mirrored pairs) | 4 | (n ÷ 2) × C | At least 1 (up to n ÷ 2 if failures hit different pairs) |

    **Example: 4 disks of 4 TB.** RAID 0: 16 TB (no tolerance); RAID 5: 12 TB; RAID 6: 8 TB; RAID 10: 8 TB; RAID 1 on all 4 disks: 4 TB.

    **Performance compared**

    | Level | Read | Write | Remark |
    |---|---|---|---|
    | **RAID 0** | Excellent | Excellent | No safety |
    | **RAID 1** | Good (reads spread over the copies) | That of one disk | Simple and robust |
    | **RAID 5** | Good | Penalized by parity computation | A small write costs about 4 disk operations |
    | **RAID 6** | Good | Penalized even more | About 6 disk operations per small write |
    | **RAID 10** | Very good | Good | Capacity cost (50%), but no parity computation |

    ```mermaid
    flowchart TB
      subgraph R10["RAID 10 (4 disks)"]
        direction LR
        subgraph M1["Mirror 1"]
          D1["Disk 1"] --- D2["Disk 2"]
        end
        subgraph M2["Mirror 2"]
          D3["Disk 3"] --- D4["Disk 4"]
        end
      end
      S["Data striped across both mirrors"] --> R10
    ```

    There is also the **JBOD** (*just a bunch of disks*) or "linear" mode: disks are simply placed end to end, **with no redundancy** and no performance gain.

## 4. Panne, mode dégradé et reconstruction / Failure, degraded mode and rebuild

=== "Français"

    Quand un disque tombe en panne dans une grappe redondante :

    1. La grappe passe en **mode dégradé** : elle fonctionne encore, mais **sans (ou avec moins de) redondance**, et plus lentement (les données manquantes sont recalculées ou lues sur les copies).
    2. On **remplace** le disque défaillant (ou un **disque de secours** (*hot spare*) prend automatiquement sa place).
    3. La **reconstruction** (*rebuild*) recopie ou recalcule les données du disque perdu sur le nouveau disque. Pendant cette phase, la grappe est **vulnérable** et plus lente.

    **Les risques de la reconstruction**

    - Une **deuxième panne** pendant la reconstruction fait perdre les données d'un RAID 5 ou d'un RAID 1 à deux disques.
    - La reconstruction lit **tous les disques restants** : une erreur de lecture irrécupérable sur l'un d'eux peut empêcher de reconstruire. Le risque croît avec la **capacité** des disques et la **durée** de la reconstruction.
    - C'est pourquoi on préfère le **RAID 6** ou le **RAID 10** pour les disques de grande capacité.

    **Surveiller et vérifier.** Une grappe doit être surveillée (alertes de panne) et sa cohérence contrôlée régulièrement par un **balayage** (*scrub* ou *check*), qui relit tous les blocs pour détecter les incohérences avant qu'une panne ne les révèle. Un cas particulier du RAID 5 est le « trou d'écriture » (*write hole*) : une coupure de courant au milieu d'une écriture peut laisser données et parité incohérentes.

=== "English"

    When a disk fails in a redundant array:

    1. The array goes into **degraded mode**: it still works, but **without (or with less) redundancy**, and more slowly (missing data is recomputed or read from copies).
    2. The failed disk is **replaced** (or a **hot spare** automatically takes its place).
    3. The **rebuild** copies or recomputes the lost disk's data onto the new disk. During this phase, the array is **vulnerable** and slower.

    **The risks of rebuilding**

    - A **second failure** during the rebuild loses the data of a RAID 5 or a two-disk RAID 1.
    - The rebuild reads **all the remaining disks**: an unrecoverable read error on one of them can prevent the rebuild. The risk grows with disk **capacity** and rebuild **duration**.
    - This is why **RAID 6** or **RAID 10** is preferred for large-capacity disks.

    **Monitor and verify.** An array must be monitored (failure alerts) and its consistency checked regularly by a **scrub** (or *check*), which rereads all blocks to detect inconsistencies before a failure reveals them. A special case of RAID 5 is the "write hole": a power cut in the middle of a write can leave data and parity inconsistent.

## 5. RAID matériel ou logiciel ? Choisir un niveau / Hardware or software RAID? Choosing a level

=== "Français"

    | | RAID matériel | RAID logiciel |
    |---|---|---|
    | **Où** | Carte contrôleur dédiée dans le serveur | Système d'exploitation (par exemple `mdadm` sous Linux) |
    | **Atouts** | Cache avec batterie ou mémoire flash de protection, soulage le processeur, géré avant le démarrage du système | Pas de matériel spécifique, portable d'un serveur à l'autre, souple, gratuit |
    | **Limites** | Coût ; dépend du modèle du contrôleur (une carte en panne peut exiger la même référence) | Utilise le processeur de l'hôte ; dépend du système |

    Un « RAID » proposé dans le BIOS d'une carte mère grand public est en réalité du RAID **logiciel** piloté par un pilote : il offre peu d'avantages sur `mdadm`. Dans ce cours, nous utilisons le **RAID logiciel Linux** (`mdadm`, lecture de `/proc/mdstat`).

    !!! warning "RAID sur des disques virtuels"
        Dans une VM, un RAID construit sur plusieurs disques virtuels stockés **sur le même disque physique** n'apporte **aucune** vraie tolérance aux pannes : si le disque physique tombe, tous les disques virtuels sont perdus ensemble. Les labs l'utilisent pour **apprendre les mécanismes**, pas pour protéger des données.

    **Choisir un niveau selon le besoin**

    | Besoin | Niveau adapté | Raison |
    |---|---|---|
    | Données temporaires, vitesse maximale, perte acceptable | RAID 0 | Performances sans redondance |
    | Système d'exploitation, petit serveur | RAID 1 | Simple, robuste, reconstruction rapide |
    | Beaucoup de lectures, capacité importante | RAID 5 (disques de capacité raisonnable) | Bon équilibre capacité-sécurité |
    | Gros disques, archives, sécurité renforcée | RAID 6 | Supporte deux pannes |
    | Bases de données, VM, forte charge d'écriture | RAID 10 | Bonnes performances en écriture, reconstruction simple |

    !!! tip "À retenir"
        - RAID = plusieurs disques en un volume logique ; ce n'est **pas** une sauvegarde.
        - Trois techniques : répartition (performance), miroir (copie), parité (XOR, reconstitution).
        - RAID 0 : aucune tolérance ; RAID 1 : miroir ; RAID 5 : 1 panne ; RAID 6 : 2 pannes ; RAID 10 : miroirs répartis.
        - Capacité utile : RAID 5 = (n − 1) × C ; RAID 6 = (n − 2) × C ; RAID 10 = (n ÷ 2) × C ; RAID 1 = C.
        - Panne : mode dégradé, remplacement ou disque de secours, reconstruction (phase vulnérable).
        - Gros disques : préférer RAID 6 ou RAID 10 ; surveiller et vérifier la grappe régulièrement.
        - RAID matériel (contrôleur dédié) ou logiciel (`mdadm`) ; un RAID sur disques virtuels d'un même disque physique n'est pas tolérant aux pannes.

=== "English"

    | | Hardware RAID | Software RAID |
    |---|---|---|
    | **Where** | Dedicated controller card in the server | Operating system (for example `mdadm` on Linux) |
    | **Strengths** | Cache with battery or flash protection, offloads the processor, managed before the system boots | No specific hardware, portable from one server to another, flexible, free |
    | **Limits** | Cost; depends on the controller model (a failed card may require the same reference) | Uses the host's processor; depends on the system |

    A "RAID" offered in a consumer motherboard's BIOS is actually **software** RAID driven by a driver: it offers few advantages over `mdadm`. In this course, we use **Linux software RAID** (`mdadm`, reading `/proc/mdstat`).

    !!! warning "RAID on virtual disks"
        In a VM, a RAID built on several virtual disks stored **on the same physical disk** provides **no** real fault tolerance: if the physical disk fails, all virtual disks are lost together. The labs use it to **learn the mechanisms**, not to protect data.

    **Choosing a level according to the need**

    | Need | Suitable level | Reason |
    |---|---|---|
    | Temporary data, maximum speed, loss acceptable | RAID 0 | Performance without redundancy |
    | Operating system, small server | RAID 1 | Simple, robust, fast rebuild |
    | Many reads, large capacity | RAID 5 (disks of reasonable capacity) | Good capacity-safety balance |
    | Large disks, archives, stronger safety | RAID 6 | Withstands two failures |
    | Databases, VMs, heavy write load | RAID 10 | Good write performance, simple rebuild |

    !!! tip "Key takeaways"
        - RAID = several disks as one logical volume; it is **not** a backup.
        - Three techniques: striping (performance), mirroring (copy), parity (XOR, reconstruction).
        - RAID 0: no tolerance; RAID 1: mirror; RAID 5: 1 failure; RAID 6: 2 failures; RAID 10: striped mirrors.
        - Usable capacity: RAID 5 = (n − 1) × C; RAID 6 = (n − 2) × C; RAID 10 = (n ÷ 2) × C; RAID 1 = C.
        - Failure: degraded mode, replacement or hot spare, rebuild (vulnerable phase).
        - Large disks: prefer RAID 6 or RAID 10; monitor and verify the array regularly.
        - Hardware RAID (dedicated controller) or software (`mdadm`); RAID on virtual disks of the same physical disk is not fault tolerant.

## Pour vérifier votre compréhension / Check your understanding

=== "Français"

    ??? question "Six disques de 2 To : quelle est la capacité utile en RAID 5, en RAID 6 et en RAID 10 ?"
        RAID 5 : (6 − 1) × 2 = **10 To** ; RAID 6 : (6 − 2) × 2 = **8 To** ; RAID 10 : (6 ÷ 2) × 2 = **6 To**. Plus la tolérance aux pannes est élevée (ou plus la copie est complète), plus la capacité utile diminue.

    ??? question "Pourquoi un volume en RAID 1 n'est-il pas une sauvegarde ?"
        Le miroir recopie **immédiatement toute modification**, y compris une suppression accidentelle, une corruption ou un chiffrement par un virus : l'erreur est présente sur les deux disques. Le RAID protège de la panne d'un disque, pas des erreurs de données, d'un sinistre ou d'un vol : il faut une sauvegarde indépendante.

    ??? question "Pourquoi la reconstruction d'un RAID 5 de gros disques est-elle une période à risque ?"
        La reconstruction lit tous les disques restants et peut durer longtemps. Pendant ce temps, la grappe n'a plus de redondance : une seconde panne, ou une erreur de lecture irrécupérable sur un disque restant, entraîne la perte de données. Le RAID 6 (ou le RAID 10) réduit ce risque.

=== "English"

    ??? question "Six disks of 2 TB: what is the usable capacity in RAID 5, RAID 6 and RAID 10?"
        RAID 5: (6 − 1) × 2 = **10 TB**; RAID 6: (6 − 2) × 2 = **8 TB**; RAID 10: (6 ÷ 2) × 2 = **6 TB**. The higher the fault tolerance (or the more complete the copy), the lower the usable capacity.

    ??? question "Why is a RAID 1 volume not a backup?"
        The mirror copies **every change immediately**, including accidental deletion, corruption or encryption by a virus: the error is present on both disks. RAID protects against a disk failure, not against data errors, a disaster or a theft: an independent backup is needed.

    ??? question "Why is rebuilding a RAID 5 of large disks a risky period?"
        The rebuild reads all the remaining disks and can last a long time. During that time, the array has no redundancy left: a second failure, or an unrecoverable read error on a remaining disk, causes data loss. RAID 6 (or RAID 10) reduces this risk.

## Ressources du chapitre / Chapter resources

- [Lab 8.1 : RAID 1 et RAID 5 avec mdadm / RAID 1 and RAID 5 with mdadm](lab-8-1-raid-1-et-raid-5-avec-mdadm.md)
- [Lab 8.2 : Panne, remplacement et reconstruction / Failure, replacement and rebuild](lab-8-2-panne-remplacement-et-reconstruction.md)
- [Lab 8.3 : Choisir un niveau RAID selon un cahier des charges / Choosing a RAID level from requirements](lab-8-3-choisir-un-niveau-raid-selon-un-cahier-des-charges.md)
- [Quiz du chapitre / Chapter quiz](quiz.md)
- [Glossaire / Glossary](../../annexes/glossaire.md)
- [Environnement de TP / Lab environment](../../annexes/environnement.md)