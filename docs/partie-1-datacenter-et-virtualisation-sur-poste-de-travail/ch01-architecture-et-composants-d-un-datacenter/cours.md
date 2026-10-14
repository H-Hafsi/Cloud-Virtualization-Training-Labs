# Chapitre 1 : Architecture et composants d'un datacenter / Datacenter Architecture and Components

=== "Français"

    !!! abstract "Objectifs du chapitre"
        - Définir ce qu'est un datacenter et distinguer ses grandes familles
        - Décrire les sous-systèmes d'un datacenter : baies, serveurs, réseau, stockage, énergie, refroidissement, sécurité
        - Expliquer la chaîne d'alimentation électrique et la notion de PUE
        - Comparer les niveaux de redondance (N, N+1, 2N) et les classes Tier
        - Calculer une disponibilité et en déduire une durée d'indisponibilité annuelle
        - Acquis d'apprentissage visé : **AA1**

        Matériel requis pour ce cours : aucun. Les labs associés indiquent leurs propres ressources.

=== "English"

    !!! abstract "Chapter objectives"
        - Define what a datacenter is and tell its main families apart
        - Describe the datacenter subsystems: racks, servers, network, storage, power, cooling, security
        - Explain the electrical power chain and the PUE metric
        - Compare redundancy levels (N, N+1, 2N) and Tier classes
        - Compute an availability figure and derive the yearly downtime from it
        - Target learning outcome: **AA1**

        Required equipment for this lesson: none. The associated labs state their own resources.

## 1. Qu'est-ce qu'un datacenter ? / What is a datacenter?

=== "Français"

    Un **datacenter** (centre de données) est un site, ou une partie de bâtiment, qui regroupe des serveurs, du stockage et des équipements réseau, avec tout ce qui permet de les faire fonctionner en continu : alimentation électrique, refroidissement, sécurité physique et supervision. Il héberge les services informatiques d'une organisation (messagerie, applications, bases de données, sites web) ou ceux de ses clients.

    On distingue plusieurs familles :

    | Famille | Principe | Exemple d'usage |
    |---|---|---|
    | **Salle serveur d'entreprise** | Local interne, souvent de petite taille | Services d'un établissement ou d'une PME |
    | **Datacenter d'entreprise** | Site dédié, exploité par l'organisation elle-même | Banque, opérateur, grande administration |
    | **Colocation** | L'exploitant loue des baies, de l'énergie et de la connectivité ; le client apporte ses équipements | Hébergement de serveurs sans construire de site |
    | **Datacenter de cloud public (hyperscale)** | Très grands sites exploités par un fournisseur de cloud | Services à la demande (chapitre 17) |
    | **Edge** | Petits sites proches des utilisateurs ou des capteurs | Traitement local, faible latence |

    !!! info "Pourquoi ce chapitre dans un cours de virtualisation ?"
        La virtualisation et le cloud reposent sur du matériel bien réel. Comprendre ses contraintes (énergie, chaleur, pannes, espace) permet de comprendre **pourquoi** on mutualise les ressources, et de dimensionner correctement une infrastructure virtualisée.

=== "English"

    A **datacenter** is a site, or part of a building, that groups servers, storage and network equipment, together with everything needed to keep them running continuously: electrical power, cooling, physical security and monitoring. It hosts the IT services of an organization (email, applications, databases, websites) or those of its customers.

    Several families exist:

    | Family | Principle | Typical use |
    |---|---|---|
    | **Enterprise server room** | Internal room, often small | Services of a school or a small business |
    | **Enterprise datacenter** | Dedicated site, operated by the organization itself | Bank, operator, large administration |
    | **Colocation** | The operator rents racks, power and connectivity; the customer brings its own equipment | Hosting servers without building a site |
    | **Public cloud datacenter (hyperscale)** | Very large sites run by a cloud provider | On-demand services (chapter 17) |
    | **Edge** | Small sites close to users or sensors | Local processing, low latency |

    !!! info "Why this chapter in a virtualization course?"
        Virtualization and cloud computing run on very real hardware. Understanding its constraints (power, heat, failures, space) explains **why** resources are shared, and helps size a virtualized infrastructure correctly.

## 2. Les sous-systèmes d'un datacenter / Datacenter subsystems

=== "Français"

    ```mermaid
    flowchart TB
      B["Bâtiment et sécurité physique"]
      E["Énergie : alimentation, onduleurs, groupes électrogènes"]
      C["Refroidissement"]
      R["Réseau : commutateurs, routeurs, liaisons"]
      S["Calcul et stockage : serveurs, baies de stockage"]
      L["Logiciels : virtualisation, supervision, exploitation"]
      B --> E --> C --> R --> S --> L
    ```

    | Sous-système | Rôle | Éléments typiques |
    |---|---|---|
    | **Baie (rack)** | Armoire normalisée de 19 pouces qui accueille les équipements | Hauteur exprimée en **U** (1 U = 44,45 mm), souvent 42 U |
    | **Calcul** | Exécute les applications | Serveurs en baie, lames, tours |
    | **Stockage** | Conserve les données | Disques internes, baies de stockage dédiées |
    | **Réseau** | Relie les serveurs entre eux et au monde extérieur | Commutateurs, routeurs, pare-feu, fibre optique |
    | **Énergie** | Alimente et protège les équipements | Arrivées électriques, onduleurs, groupes électrogènes |
    | **Refroidissement** | Évacue la chaleur produite | Climatisation de salle, allées chaudes et froides |
    | **Sécurité** | Protège contre les intrusions et les sinistres | Contrôle d'accès, vidéosurveillance, détection et extinction d'incendie |
    | **Supervision** | Surveille l'état de l'ensemble | Capteurs, alarmes, outils de monitoring |

    Les équipements sont mesurés en **unités de rack** : un serveur « 1U » occupe une unité de hauteur, un serveur « 2U » en occupe deux. Le nombre d'unités disponibles, la puissance électrique et la capacité de refroidissement sont les trois limites qui déterminent combien de matériel une baie peut réellement accueillir.

=== "English"

    ```mermaid
    flowchart TB
      B["Building and physical security"]
      E["Power: supply, UPS, generators"]
      C["Cooling"]
      R["Network: switches, routers, links"]
      S["Compute and storage: servers, storage arrays"]
      L["Software: virtualization, monitoring, operations"]
      B --> E --> C --> R --> S --> L
    ```

    | Subsystem | Role | Typical elements |
    |---|---|---|
    | **Rack** | Standardized 19-inch cabinet housing the equipment | Height expressed in **U** (1 U = 44.45 mm), often 42 U |
    | **Compute** | Runs the applications | Rack servers, blades, towers |
    | **Storage** | Keeps the data | Internal disks, dedicated storage arrays |
    | **Network** | Connects servers together and to the outside world | Switches, routers, firewalls, optical fiber |
    | **Power** | Feeds and protects the equipment | Power feeds, UPS, generators |
    | **Cooling** | Removes the heat produced | Room air conditioning, hot and cold aisles |
    | **Security** | Protects against intrusion and disasters | Access control, video surveillance, fire detection and suppression |
    | **Monitoring** | Watches the state of everything | Sensors, alarms, monitoring tools |

    Equipment is measured in **rack units**: a "1U" server takes one unit of height, a "2U" server takes two. The number of available units, the electrical power and the cooling capacity are the three limits that determine how much hardware a rack can really host.

## 3. Énergie et refroidissement / Power and cooling

=== "Français"

    ### La chaîne d'alimentation

    ```mermaid
    flowchart LR
      R["Réseau électrique"] --> A["Commutateur de source (ATS)"]
      G["Groupe électrogène"] --> A
      A --> U["Onduleur (UPS)<br/>et batteries"]
      U --> P["Distribution (PDU en baie)"]
      P --> S["Serveurs"]
    ```

    En fonctionnement normal, l'énergie vient du réseau public. En cas de coupure, les **batteries de l'onduleur** prennent le relais immédiatement, le temps que le **groupe électrogène** démarre ; le commutateur de source bascule alors sur le groupe. Chaque serveur dispose en général de **deux blocs d'alimentation**, branchés sur deux chaînes distinctes.

    ### La chaleur

    La quasi-totalité de l'énergie électrique consommée par un équipement informatique est dissipée **sous forme de chaleur** : 1 kW consommé représente environ 1 kW de chaleur à évacuer. Le refroidissement est donc dimensionné d'après la puissance installée.

    Pour limiter le mélange d'air chaud et d'air froid, les baies sont alignées en rangées : les **faces avant** (entrée d'air froid) se font face dans une **allée froide**, les **faces arrière** (sortie d'air chaud) dans une **allée chaude**. Le confinement de l'une des deux allées améliore l'efficacité.

    ### Le PUE

    Le **PUE** (Power Usage Effectiveness) mesure l'efficacité énergétique du site :

    **PUE = énergie totale consommée par le datacenter ÷ énergie consommée par les équipements informatiques**

    Un PUE de 1,0 serait idéal (aucune énergie perdue en refroidissement ou en distribution). **Exemple :** si les équipements informatiques consomment 500 kW et que le site consomme 750 kW au total, le PUE est de 750 / 500 = **1,5** : pour 1 kWh utile, le site en consomme 1,5.

=== "English"

    ### The power chain

    ```mermaid
    flowchart LR
      R["Utility grid"] --> A["Automatic transfer switch (ATS)"]
      G["Generator"] --> A
      A --> U["UPS<br/>and batteries"]
      U --> P["Distribution (rack PDU)"]
      P --> S["Servers"]
    ```

    In normal operation, power comes from the utility grid. During an outage, the **UPS batteries** take over immediately while the **generator** starts; the transfer switch then switches to the generator. Each server usually has **two power supply units**, plugged into two separate chains.

    ### Heat

    Almost all the electrical energy consumed by IT equipment is dissipated **as heat**: 1 kW consumed means about 1 kW of heat to remove. Cooling is therefore sized from the installed power.

    To limit the mixing of hot and cold air, racks are lined up in rows: the **fronts** (cold air intake) face each other across a **cold aisle**, the **backs** (hot air exhaust) across a **hot aisle**. Containing one of the two aisles improves efficiency.

    ### PUE

    **PUE** (Power Usage Effectiveness) measures the energy efficiency of the site:

    **PUE = total energy used by the datacenter ÷ energy used by the IT equipment**

    A PUE of 1.0 would be ideal (no energy lost to cooling or distribution). **Example:** if the IT equipment draws 500 kW and the whole site draws 750 kW, the PUE is 750 / 500 = **1.5**: for 1 kWh of useful energy, the site uses 1.5.

## 4. Disponibilité et redondance / Availability and redundancy

=== "Français"

    ### Niveaux de redondance

    | Notation | Signification | Conséquence |
    |---|---|---|
    | **N** | Juste ce qu'il faut pour le besoin | Aucune marge : une panne interrompt le service |
    | **N+1** | Un élément de secours pour l'ensemble | Résiste à une panne d'un composant |
    | **2N** | Deux chaînes complètes et indépendantes | Résiste à la perte d'une chaîne entière, y compris pendant une maintenance sur l'autre |

    Un **point de défaillance unique** (SPOF, *single point of failure*) est un élément dont la panne arrête tout le service : l'objectif de la redondance est de les supprimer.

    ### Calculer une disponibilité

    La disponibilité s'exprime en pourcentage du temps pendant lequel le service est utilisable. L'indisponibilité annuelle maximale s'en déduit (une année = 8 760 heures) :

    | Disponibilité | Indisponibilité maximale par an |
    |---|---|
    | 99 % | 87,6 h (environ 3,65 jours) |
    | 99,9 % | 8,76 h |
    | 99,99 % | 52,6 min |
    | 99,999 % | 5,26 min |

    ### Les classes Tier

    L'Uptime Institute définit quatre niveaux de classification des datacenters :

    | Tier | Principe | Disponibilité couramment citée | Indisponibilité annuelle correspondante |
    |---|---|---|---|
    | **I** | Capacité de base, sans redondance | 99,671 % | environ 28,8 h |
    | **II** | Composants de capacité redondants | 99,741 % | environ 22,7 h |
    | **III** | Maintenance possible sans arrêt du service | 99,982 % | environ 1,6 h |
    | **IV** | Tolérance aux pannes | 99,995 % | environ 0,4 h (26 min) |

    !!! warning "Attention"
        Les pourcentages ci-dessus sont les valeurs usuellement citées pour chaque Tier. Ils servent à comparer des ordres de grandeur ; la certification d'un site se fonde sur sa conception et son exploitation, pas uniquement sur ce chiffre. De plus, la disponibilité d'un **service** dépend de toute la chaîne (réseau, serveurs, logiciels), pas seulement du bâtiment.

=== "English"

    ### Redundancy levels

    | Notation | Meaning | Consequence |
    |---|---|---|
    | **N** | Just what the need requires | No margin: one failure interrupts the service |
    | **N+1** | One spare element for the whole | Survives the failure of one component |
    | **2N** | Two complete, independent chains | Survives the loss of an entire chain, even during maintenance on the other |

    A **single point of failure** (SPOF) is an element whose failure stops the whole service: the purpose of redundancy is to remove them.

    ### Computing availability

    Availability is expressed as the percentage of time during which the service is usable. The maximum yearly downtime follows from it (one year = 8,760 hours):

    | Availability | Maximum downtime per year |
    |---|---|
    | 99% | 87.6 h (about 3.65 days) |
    | 99.9% | 8.76 h |
    | 99.99% | 52.6 min |
    | 99.999% | 5.26 min |

    ### Tier classes

    The Uptime Institute defines four datacenter classification levels:

    | Tier | Principle | Commonly quoted availability | Corresponding yearly downtime |
    |---|---|---|---|
    | **I** | Basic capacity, no redundancy | 99.671% | about 28.8 h |
    | **II** | Redundant capacity components | 99.741% | about 22.7 h |
    | **III** | Maintenance possible without stopping the service | 99.982% | about 1.6 h |
    | **IV** | Fault tolerance | 99.995% | about 0.4 h (26 min) |

    !!! warning "Caution"
        The percentages above are the values usually quoted for each Tier. They are meant to compare orders of magnitude; certifying a site is based on its design and operation, not only on this figure. Moreover, the availability of a **service** depends on the whole chain (network, servers, software), not only on the building.

## 5. Serveurs, réseau et administration à distance / Servers, network and remote management

=== "Français"

    ### Le serveur

    Un serveur est un ordinateur conçu pour fonctionner en continu et se monter en baie. Ses composants principaux :

    | Composant | Rôle |
    |---|---|
    | **Processeur (CPU)** | Exécute les calculs ; les serveurs ont souvent plusieurs processeurs et de nombreux cœurs |
    | **Mémoire (RAM)** | Contient les programmes et données en cours d'utilisation |
    | **Stockage** | Disques durs (HDD), SSD ou NVMe, internes ou externes |
    | **Cartes réseau (NIC)** | Relient le serveur au réseau, souvent en plusieurs exemplaires |
    | **Alimentations** | Généralement deux, redondantes |
    | **Contrôleur de gestion (BMC)** | Petit système autonome permettant d'allumer, d'éteindre et de surveiller le serveur à distance, même lorsqu'il est éteint |

    On rencontre trois formats : **serveur en baie** (1U, 2U…), **serveur lame** (module inséré dans un châssis partagé) et **serveur tour**.

    ### Le réseau

    Dans la topologie classique, chaque baie contient un **commutateur de baie** (*top of rack*) relié à des commutateurs d'agrégation, puis à un cœur de réseau. Les architectures récentes privilégient la topologie **leaf-spine** : chaque commutateur de baie (*leaf*) est relié à tous les commutateurs de dorsale (*spine*), ce qui donne des chemins courts et prévisibles entre deux serveurs quelconques.

    ### Du matériel au logiciel

    Un serveur physique n'exploite souvent qu'une partie de ses ressources, alors qu'il consomme de l'énergie et occupe de la place en permanence. Cette constatation motive la **virtualisation**, qui permet de faire cohabiter plusieurs systèmes isolés sur une même machine : c'est l'objet du chapitre 2.

    !!! tip "À retenir"
        - Un datacenter = calcul, stockage, réseau + énergie, refroidissement, sécurité et supervision.
        - Les trois limites d'une baie : espace (U), puissance électrique, refroidissement.
        - Presque toute l'énergie consommée devient de la chaleur à évacuer.
        - PUE = énergie totale / énergie informatique ; plus il est proche de 1, mieux c'est.
        - Redondance : N < N+1 < 2N ; le but est de supprimer les points de défaillance uniques.
        - Disponibilité de 99,99 % ≈ 52,6 minutes d'indisponibilité par an.
        - Les classes Tier (I à IV) comparent la résilience des sites.

=== "English"

    ### The server

    A server is a computer designed to run continuously and to be mounted in a rack. Its main components:

    | Component | Role |
    |---|---|
    | **Processor (CPU)** | Performs computations; servers often have several processors and many cores |
    | **Memory (RAM)** | Holds the programs and data currently in use |
    | **Storage** | Hard disks (HDD), SSDs or NVMe, internal or external |
    | **Network cards (NIC)** | Connect the server to the network, often several of them |
    | **Power supplies** | Usually two, redundant |
    | **Management controller (BMC)** | Small autonomous system that allows powering the server on and off and monitoring it remotely, even when it is off |

    Three form factors exist: **rack server** (1U, 2U...), **blade server** (module inserted into a shared chassis) and **tower server**.

    ### The network

    In the classic topology, each rack contains a **top-of-rack switch** connected to aggregation switches, then to a network core. Recent architectures favor the **leaf-spine** topology: each top-of-rack switch (*leaf*) is connected to all backbone switches (*spine*), which gives short, predictable paths between any two servers.

    ### From hardware to software

    A physical server often uses only part of its resources, while it permanently consumes power and takes up space. This observation motivates **virtualization**, which lets several isolated systems coexist on the same machine: this is the subject of chapter 2.

    !!! tip "Key takeaways"
        - A datacenter = compute, storage, network + power, cooling, security and monitoring.
        - The three limits of a rack: space (U), electrical power, cooling.
        - Almost all consumed energy becomes heat to be removed.
        - PUE = total energy / IT energy; the closer to 1, the better.
        - Redundancy: N < N+1 < 2N; the goal is to remove single points of failure.
        - 99.99% availability ≈ 52.6 minutes of downtime per year.
        - Tier classes (I to IV) compare site resilience.

## Pour vérifier votre compréhension / Check your understanding

=== "Français"

    ??? question "Une baie de 42 U contient vingt serveurs 2U. Peut-elle accueillir vingt serveurs 1U supplémentaires ? Quelle autre limite doit-on vérifier ?"
        Non pour l'espace : vingt serveurs 2U occupent 40 U, il ne reste que 2 U. Même avec de la place, il faudrait vérifier la **puissance électrique** disponible dans la baie et la **capacité de refroidissement**.

    ??? question "Un site consomme 900 kW au total, dont 600 kW pour les équipements informatiques. Quel est son PUE ? Que signifie ce résultat ?"
        PUE = 900 / 600 = **1,5**. Pour 1 kWh consommé par l'informatique, le site en consomme 1,5 au total : 0,5 kWh sert au refroidissement, à la distribution électrique et aux autres besoins du site.

    ??? question "Pourquoi une alimentation 2N est-elle plus sûre qu'une alimentation N+1 lors d'une maintenance ?"
        Avec 2N, deux chaînes complètes et indépendantes existent : on peut mettre l'une hors service pour maintenance tout en gardant une chaîne entière en fonctionnement. Avec N+1, un seul élément de secours est disponible pour l'ensemble, ce qui laisse moins de marge si une panne survient pendant l'intervention.

=== "English"

    ??? question "A 42 U rack holds twenty 2U servers. Can it host twenty more 1U servers? Which other limit must be checked?"
        Not in terms of space: twenty 2U servers take 40 U, only 2 U remain. Even with free space, the **electrical power** available in the rack and the **cooling capacity** would have to be checked.

    ??? question "A site draws 900 kW in total, of which 600 kW goes to the IT equipment. What is its PUE? What does this result mean?"
        PUE = 900 / 600 = **1.5**. For every 1 kWh used by IT, the site uses 1.5 in total: 0.5 kWh goes to cooling, power distribution and other site needs.

    ??? question "Why is a 2N power supply safer than an N+1 one during maintenance?"
        With 2N, two complete, independent chains exist: one can be taken out of service for maintenance while a full chain keeps running. With N+1, only one spare element is available for the whole, leaving less margin if a failure occurs during the intervention.

## Ressources du chapitre / Chapter resources

- [Lab 1.1 : Dimensionner une salle serveur / Sizing a server room](lab-1-1-dimensionner-une-salle-serveur.md)
- [Lab 1.2 : Inventorier une machine / Inventorying a machine](lab-1-2-inventorier-une-machine.md)
- [Quiz du chapitre / Chapter quiz](quiz.md)
- [Glossaire / Glossary](../../annexes/glossaire.md)
- [Environnement de TP / Lab environment](../../annexes/environnement.md)