# Lab 1.1 : Dimensionner une salle serveur / Sizing a Server Room

| | |
|---|---|
| **Durée / Duration** | 45 à 60 min / 45 to 60 min |
| **Niveau / Level** | Guidé / Guided |
| **Prérequis / Prerequisites** | [Chapitre 1 : cours](cours.md) lu / read |
| **Ressources minimales / Minimum resources** | Aucune machine virtuelle. Un poste quelconque avec un tableur (par exemple LibreOffice Calc, gratuit) ou une calculatrice. / No virtual machine. Any computer with a spreadsheet (for example free LibreOffice Calc) or a calculator. |
| **Acquis visés / Learning outcomes** | AA1 |

=== "Français"

    !!! abstract "Objectifs"
        - Calculer l'espace, la puissance électrique et la charge thermique d'une salle serveur à partir d'un inventaire
        - Déterminer le nombre de baies, la taille de l'onduleur et le nombre d'unités de refroidissement
        - Appliquer une marge d'évolution et un niveau de redondance
        - Relier un objectif de disponibilité à une classe Tier

=== "English"

    !!! abstract "Objectives"
        - Compute the space, electrical power and heat load of a server room from an inventory
        - Determine the number of racks, the UPS size and the number of cooling units
        - Apply a growth margin and a redundancy level
        - Link an availability target to a Tier class

## Contexte / Context

=== "Français"

    Un établissement prépare l'installation d'une petite salle serveur. Vous êtes chargé de produire la **fiche de dimensionnement** qui sera remise à la direction. Ce lab est un exercice de calcul : aucune machine virtuelle n'est nécessaire. Toutes les données sont **fictives** et fournies ci-dessous.

    ```mermaid
    flowchart LR
      I["1. Inventaire"] --> B["2. Baies"]
      B --> P["3. Puissance<br/>et onduleur"]
      P --> C["4. Refroidissement"]
      C --> D["5. Disponibilité"]
      D --> F["6. Fiche de synthèse"]
    ```

=== "English"

    An organization is preparing a small server room. You are in charge of producing the **sizing sheet** that will be handed to management. This lab is a calculation exercise: no virtual machine is needed. All data is **fictional** and provided below.

    ```mermaid
    flowchart LR
      I["1. Inventory"] --> B["2. Racks"]
      B --> P["3. Power<br/>and UPS"]
      P --> C["4. Cooling"]
      C --> D["5. Availability"]
      D --> F["6. Summary sheet"]
    ```

## Données de départ / Starting data

=== "Français"

    **Inventaire des équipements prévus** (la puissance indiquée est la consommation **maximale** de chaque équipement) :

    | Équipement | Quantité | Hauteur unitaire | Puissance unitaire |
    |---|---|---|---|
    | Serveur de calcul | 8 | 2 U | 450 W |
    | Serveur de service | 4 | 1 U | 300 W |
    | Baie de stockage | 2 | 3 U | 600 W |
    | Commutateur réseau | 2 | 1 U | 150 W |
    | Pare-feu | 1 | 1 U | 100 W |

    **Hypothèses de l'étude :**

    | Paramètre | Valeur |
    |---|---|
    | Marge d'évolution | 30 % sur l'espace, 20 % sur la puissance |
    | Hauteur utile d'une baie | 40 U (le reste est réservé aux accessoires et au câblage) |
    | Puissance maximale par baie | 5 kW |
    | PUE supposé du site | 1,6 |
    | Facteur de puissance de l'onduleur | 0,9 (kVA = kW ÷ 0,9) |
    | Calibres d'onduleur disponibles | 6, 10, 15 et 20 kVA |
    | Autonomie visée sur batteries | 10 minutes |
    | Capacité d'une unité de refroidissement | 6 kW de chaleur évacuée |
    | Conversion | 1 W = 3,412 BTU/h |
    | Objectif de la direction | Disponibilité d'au moins 99,9 % |

=== "English"

    **Planned equipment inventory** (the stated power is each device's **maximum** consumption):

    | Equipment | Quantity | Unit height | Unit power |
    |---|---|---|---|
    | Compute server | 8 | 2 U | 450 W |
    | Service server | 4 | 1 U | 300 W |
    | Storage array | 2 | 3 U | 600 W |
    | Network switch | 2 | 1 U | 150 W |
    | Firewall | 1 | 1 U | 100 W |

    **Study assumptions:**

    | Parameter | Value |
    |---|---|
    | Growth margin | 30% on space, 20% on power |
    | Usable rack height | 40 U (the rest is reserved for accessories and cabling) |
    | Maximum power per rack | 5 kW |
    | Assumed site PUE | 1.6 |
    | UPS power factor | 0.9 (kVA = kW ÷ 0.9) |
    | Available UPS ratings | 6, 10, 15 and 20 kVA |
    | Target battery autonomy | 10 minutes |
    | Capacity of one cooling unit | 6 kW of heat removed |
    | Conversion | 1 W = 3.412 BTU/h |
    | Management target | Availability of at least 99.9% |

## Étapes / Steps

### Étape 1 : Totaliser l'inventaire / Step 1: Total the inventory

=== "Français"

    Créez un tableur avec une ligne par équipement et les colonnes suivantes :

    | A | B | C | D | E | F |
    |---|---|---|---|---|---|
    | Équipement | Quantité | U unitaire | W unitaire | U total (`=B2*C2`) | W total (`=B2*D2`) |

    Ajoutez une ligne **Total** avec `=SUM(E2:E6)` et `=SUM(F2:F6)` (selon votre version et votre langue, le séparateur d'arguments est `,` ou `;`).

    **Exemple de méthode** (avec d'autres données) : 3 serveurs de 1 U et 200 W donnent 3 × 1 = 3 U et 3 × 200 = 600 W.

    !!! success "Point de contrôle"
        - [ ] Vous avez un total en **U** et un total en **W** (puis le même en **kW**, en divisant par 1 000)
        - [ ] Vous avez recalculé un total « à la main » pour vérifier le tableur

=== "English"

    Create a spreadsheet with one row per device and the following columns:

    | A | B | C | D | E | F |
    |---|---|---|---|---|---|
    | Equipment | Quantity | Unit U | Unit W | Total U (`=B2*C2`) | Total W (`=B2*D2`) |

    Add a **Total** row with `=SUM(E2:E6)` and `=SUM(F2:F6)` (depending on your version and language, the argument separator is `,` or `;`).

    **Method example** (with other data): 3 servers of 1 U and 200 W give 3 × 1 = 3 U and 3 × 200 = 600 W.

    !!! success "Checkpoint"
        - [ ] You have a total in **U** and a total in **W** (then the same in **kW**, dividing by 1,000)
        - [ ] You recomputed one total by hand to check the spreadsheet

### Étape 2 : Déterminer le nombre de baies / Step 2: Determine the number of racks

=== "Français"

    Le nombre de baies doit satisfaire **deux contraintes** ; on retient la plus exigeante.

    1. **Contrainte d'espace** : multipliez le total en U par 1,3 (marge de 30 %), puis divisez par 40 U utiles. Arrondissez **à l'entier supérieur**.
    2. **Contrainte de puissance** : multipliez la puissance totale par 1,2 (marge de 20 %), puis divisez par 5 kW. Arrondissez **à l'entier supérieur**.

    Nombre de baies = la plus grande des deux valeurs.

    !!! success "Point de contrôle"
        - [ ] Vous avez obtenu deux nombres de baies (espace, puissance) et retenu le plus grand
        - [ ] Vous savez dire quelle contrainte est **limitante** ici

=== "English"

    The number of racks must satisfy **two constraints**; keep the more demanding one.

    1. **Space constraint**: multiply the total U by 1.3 (30% margin), then divide by 40 usable U. Round **up** to the next integer.
    2. **Power constraint**: multiply the total power by 1.2 (20% margin), then divide by 5 kW. Round **up** to the next integer.

    Number of racks = the larger of the two values.

    !!! success "Checkpoint"
        - [ ] You obtained two rack counts (space, power) and kept the larger one
        - [ ] You can tell which constraint is **limiting** here

### Étape 3 : Puissance, onduleur et batteries / Step 3: Power, UPS and batteries

=== "Français"

    Calculez, dans l'ordre :

    1. la **charge informatique avec marge** (kW) : puissance totale × 1,2 ;
    2. la **puissance totale du site** (kW) : charge informatique avec marge × PUE ;
    3. la **puissance apparente de l'onduleur** (kVA) : charge informatique avec marge ÷ 0,9 ;
    4. le **calibre d'onduleur** retenu : le plus petit calibre disponible supérieur ou égal au résultat précédent ;
    5. l'**énergie des batteries** nécessaire pour 10 minutes d'autonomie (kWh) : charge informatique avec marge × (10 ÷ 60).

    !!! info "Simplification"
        Le calcul de l'énergie des batteries ignore le rendement de l'onduleur et le vieillissement des batteries : en pratique, on prend une marge supplémentaire. Le but ici est de comprendre la démarche.

    !!! success "Point de contrôle"
        - [ ] Vos résultats ont les bonnes unités (kW, kVA, kWh)
        - [ ] Le calibre d'onduleur retenu est supérieur ou égal à la puissance apparente calculée

=== "English"

    Compute, in order:

    1. the **IT load with margin** (kW): total power × 1.2;
    2. the **total site power** (kW): IT load with margin × PUE;
    3. the **UPS apparent power** (kVA): IT load with margin ÷ 0.9;
    4. the selected **UPS rating**: the smallest available rating greater than or equal to the previous result;
    5. the **battery energy** needed for 10 minutes of autonomy (kWh): IT load with margin × (10 ÷ 60).

    !!! info "Simplification"
        The battery energy calculation ignores UPS efficiency and battery aging: in practice, an additional margin is taken. The goal here is to understand the approach.

    !!! success "Checkpoint"
        - [ ] Your results have the right units (kW, kVA, kWh)
        - [ ] The selected UPS rating is greater than or equal to the computed apparent power

### Étape 4 : Refroidissement / Step 4: Cooling

=== "Français"

    La chaleur à évacuer est prise égale à la **charge informatique avec marge** (voir le cours : la puissance consommée est quasi entièrement dissipée en chaleur).

    1. Exprimez cette charge thermique en **kW** puis en **BTU/h** (× 1 000 × 3,412).
    2. Calculez le nombre d'unités de refroidissement **nécessaires** (charge ÷ 6 kW, arrondi à l'entier supérieur).
    3. Ajoutez **une unité de secours** pour obtenir une configuration **N+1**.

    !!! success "Point de contrôle"
        - [ ] Vous avez un nombre d'unités en configuration N et en configuration N+1
        - [ ] Vous pouvez expliquer pourquoi l'unité supplémentaire n'augmente pas la capacité « utile »

=== "English"

    The heat to remove is taken equal to the **IT load with margin** (see the lesson: consumed power is almost entirely dissipated as heat).

    1. Express this heat load in **kW** then in **BTU/h** (× 1,000 × 3.412).
    2. Compute the number of cooling units **needed** (load ÷ 6 kW, rounded up).
    3. Add **one spare unit** to obtain an **N+1** configuration.

    !!! success "Checkpoint"
        - [ ] You have a number of units in N and in N+1 configuration
        - [ ] You can explain why the extra unit does not increase the "useful" capacity

### Étape 5 : Disponibilité et classe Tier / Step 5: Availability and Tier class

=== "Français"

    1. Calculez l'**indisponibilité annuelle maximale** correspondant à 99,9 % (une année = 8 760 heures).
    2. Comparez-la aux indisponibilités annuelles des classes Tier I à IV données dans le cours (section 4).
    3. Indiquez la **classe minimale** compatible avec l'objectif de la direction.
    4. Proposez **trois mesures** de conception qui découlent de ce choix (par exemple pour l'alimentation, le refroidissement ou le réseau).

    !!! success "Point de contrôle"
        - [ ] Vous avez une durée d'indisponibilité en heures
        - [ ] Votre classe Tier est justifiée par une comparaison chiffrée
        - [ ] Vos trois mesures utilisent des notions du cours (N+1, 2N, point de défaillance unique)

=== "English"

    1. Compute the **maximum yearly downtime** corresponding to 99.9% (one year = 8,760 hours).
    2. Compare it with the yearly downtimes of Tier classes I to IV given in the lesson (section 4).
    3. State the **minimum class** compatible with management's target.
    4. Propose **three design measures** that follow from this choice (for example for power, cooling or network).

    !!! success "Checkpoint"
        - [ ] You have a downtime figure in hours
        - [ ] Your Tier class is justified by a numerical comparison
        - [ ] Your three measures use concepts from the lesson (N+1, 2N, single point of failure)

### Étape 6 : Fiche de dimensionnement / Step 6: Sizing sheet

=== "Français"

    Remplissez la fiche ci-dessous : c'est le **livrable** du lab.

    | Rubrique | Valeur | Unité |
    |---|---|---|
    | Total de l'inventaire : espace | … | U |
    | Total de l'inventaire : puissance | … | kW |
    | Nombre de baies (espace / puissance / retenu) | … / … / … | baies |
    | Charge informatique avec marge | … | kW |
    | Puissance totale du site (PUE 1,6) | … | kW |
    | Onduleur (puissance apparente / calibre retenu) | … / … | kVA |
    | Énergie des batteries pour 10 min | … | kWh |
    | Charge thermique | … | kW et BTU/h |
    | Unités de refroidissement (N / N+1) | … / … | unités |
    | Indisponibilité annuelle maximale pour 99,9 % | … | h |
    | Classe Tier minimale retenue | … | |

=== "English"

    Fill in the sheet below: this is the lab **deliverable**.

    | Item | Value | Unit |
    |---|---|---|
    | Inventory total: space | … | U |
    | Inventory total: power | … | kW |
    | Number of racks (space / power / selected) | … / … / … | racks |
    | IT load with margin | … | kW |
    | Total site power (PUE 1.6) | … | kW |
    | UPS (apparent power / selected rating) | … / … | kVA |
    | Battery energy for 10 min | … | kWh |
    | Heat load | … | kW and BTU/h |
    | Cooling units (N / N+1) | … / … | units |
    | Maximum yearly downtime for 99.9% | … | h |
    | Selected minimum Tier class | … | |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi, dans cette étude, la puissance détermine-t-elle le nombre de baies plutôt que l'espace ?"
        L'espace nécessaire tient dans une seule baie, mais la puissance totale dépasse la limite admissible par baie. Les trois limites d'une baie (espace, puissance, refroidissement) ne sont presque jamais atteintes en même temps : c'est la plus contraignante qui décide.

    ??? question "Pourquoi dimensionne-t-on un onduleur en kVA et non seulement en kW ?"
        L'onduleur doit fournir la **puissance apparente** demandée par les équipements, qui est supérieure à la puissance active lorsque le facteur de puissance est inférieur à 1. Un onduleur dimensionné seulement d'après les kW pourrait être surchargé.

    ??? question "Que se passe-t-il si le refroidissement tombe en panne alors que l'onduleur fonctionne ?"
        L'onduleur maintient l'alimentation, mais il n'évacue pas la chaleur : la température monte rapidement et les serveurs finissent par s'arrêter ou s'endommager. Le refroidissement doit donc, comme l'énergie, être redondant et supervisé.

=== "English"

    ??? question "Why, in this study, does power determine the number of racks rather than space?"
        The required space fits in a single rack, but the total power exceeds the allowed limit per rack. The three limits of a rack (space, power, cooling) are almost never reached at the same time: the most constraining one decides.

    ??? question "Why is a UPS sized in kVA and not only in kW?"
        The UPS must supply the **apparent power** demanded by the equipment, which is higher than the active power when the power factor is below 1. A UPS sized only from kW could be overloaded.

    ??? question "What happens if cooling fails while the UPS keeps running?"
        The UPS keeps the power on, but it does not remove heat: temperature rises quickly and the servers eventually shut down or get damaged. Cooling must therefore, like power, be redundant and monitored.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | Résultats incohérents (valeurs 1 000 fois trop grandes ou trop petites) | Confusion entre **W** et **kW** : convertir avant de comparer à la limite par baie ou aux calibres |
    | Nombre de baies ou d'unités non entier | Arrondir **toujours à l'entier supérieur** : on ne peut pas installer une fraction de baie |
    | Calibre d'onduleur inférieur au besoin | Comparer des **kVA** avec des **kVA** (diviser les kW par 0,9) |
    | Marge oubliée | Relire les hypothèses : 30 % sur l'espace, 20 % sur la puissance |
    | Formule du tableur qui renvoie une erreur | Vérifier le séparateur d'arguments (`,` ou `;`) selon la langue du tableur |

=== "English"

    | Symptom | Hints |
    |---|---|
    | Inconsistent results (values 1,000 times too large or too small) | Mix-up between **W** and **kW**: convert before comparing with the per-rack limit or the ratings |
    | Number of racks or units is not an integer | **Always round up**: a fraction of a rack cannot be installed |
    | UPS rating below the need | Compare **kVA** with **kVA** (divide kW by 0.9) |
    | Forgotten margin | Reread the assumptions: 30% on space, 20% on power |
    | Spreadsheet formula returns an error | Check the argument separator (`,` or `;`) according to the spreadsheet language |

## Clôture / Wrap-up

=== "Français"

    Ce lab ne modifie aucune machine : il n'y a rien à nettoyer. Conservez votre fiche de dimensionnement. Le [Lab 1.2](lab-1-2-inventorier-une-machine.md) vous fait passer du dimensionnement à l'inventaire d'une machine réelle.

=== "English"

    This lab changes no machine: there is nothing to clean up. Keep your sizing sheet. [Lab 1.2](lab-1-2-inventorier-une-machine.md) takes you from sizing to inventorying a real machine.

[Retour au chapitre 1 / Back to chapter 1](cours.md)