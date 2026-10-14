# Quiz du chapitre 1 : Architecture et composants d'un datacenter / Chapter 1 Quiz: Datacenter Architecture and Components

=== "Français"

    !!! abstract "Consignes"
        Autoévaluation : 10 questions à choix multiples, environ 10 minutes. Une seule réponse est correcte par question. Répondez d'abord de mémoire, puis cliquez sur « Réponse » pour vérifier. Si une réponse vous surprend, relisez la section correspondante du [cours](cours.md).

    **Question 1.** Dans quel type de datacenter l'exploitant loue-t-il des baies, de l'énergie et de la connectivité, le client apportant ses propres équipements ?

    - A. Edge
    - B. Salle serveur d'entreprise
    - C. Colocation
    - D. Datacenter de cloud public (hyperscale)

    ??? success "Réponse"
        **C.** En colocation, le client installe son matériel dans l'espace loué. Un site *edge* est un petit site proche des utilisateurs ; un site hyperscale est exploité par le fournisseur de cloud lui-même.

    **Question 2.** Quelle est la hauteur d'une unité de rack (1 U) ?

    - A. 25,4 mm
    - B. 44,45 mm
    - C. 482,6 mm
    - D. 600 mm

    ??? success "Réponse"
        **B.** 1 U = 44,45 mm. La valeur de 482,6 mm correspond à la **largeur** de 19 pouces d'une baie normalisée, pas à sa hauteur unitaire.

    **Question 3.** Quelles sont les trois limites qui déterminent le matériel qu'une baie peut accueillir ?

    - A. L'espace, la puissance électrique et le refroidissement
    - B. L'espace, le nombre de ports réseau et le prix
    - C. La puissance électrique, le système d'exploitation et la hauteur
    - D. Le refroidissement, la licence logicielle et la longueur des câbles

    ??? success "Réponse"
        **A.** Ce sont les trois contraintes physiques d'une baie ; c'est la plus exigeante des trois qui décide du nombre d'équipements installables.

    **Question 4.** Un équipement informatique consomme 1 kW d'électricité. Quelle quantité de chaleur doit-on environ évacuer ?

    - A. Environ 0,1 kW
    - B. Environ 0,5 kW
    - C. Aucune : l'énergie est stockée dans l'équipement
    - D. Environ 1 kW

    ??? success "Réponse"
        **D.** La quasi-totalité de l'énergie électrique consommée est dissipée sous forme de chaleur. Le refroidissement se dimensionne donc d'après la puissance installée.

    **Question 5.** Un datacenter consomme 600 kW au total, dont 400 kW pour les équipements informatiques. Quel est son PUE ?

    - A. 0,67
    - B. 1,5
    - C. 2,0
    - D. 0,5

    ??? success "Réponse"
        **B.** PUE = énergie totale ÷ énergie informatique = 600 ÷ 400 = 1,5. Le PUE est toujours supérieur ou égal à 1 ; la valeur 0,67 correspond à l'inverse du rapport.

    **Question 6.** Lors d'une coupure du réseau électrique, quel élément prend le relais immédiatement, le temps que le groupe électrogène démarre ?

    - A. Le groupe électrogène lui-même
    - B. Le commutateur de source (ATS)
    - C. Les batteries de l'onduleur
    - D. La distribution en baie (PDU)

    ??? success "Réponse"
        **C.** Les batteries de l'onduleur assurent la continuité de l'alimentation pendant le démarrage du groupe. Le commutateur de source bascule ensuite sur le groupe, et le PDU ne fait que distribuer l'énergie dans la baie.

    **Question 7.** Que désigne une redondance « 2N » ?

    - A. Deux chaînes complètes et indépendantes
    - B. Un seul élément de secours pour l'ensemble
    - C. Juste ce qu'il faut pour le besoin, sans marge
    - D. Deux serveurs identiques dans chaque baie

    ??? success "Réponse"
        **A.** Avec 2N, la perte d'une chaîne entière, y compris pendant une maintenance sur l'autre, n'interrompt pas le service. La proposition B décrit N+1 et la proposition C décrit N.

    **Question 8.** Une disponibilité de 99,99 % correspond à quelle indisponibilité annuelle maximale ?

    - A. Environ 8,76 heures
    - B. Environ 5,26 minutes
    - C. Environ 87,6 heures
    - D. Environ 52,6 minutes

    ??? success "Réponse"
        **D.** Une année compte 8 760 heures : 0,01 % de 8 760 h = 0,876 h, soit environ 52,6 minutes. Les valeurs de 8,76 h et de 5,26 min correspondent à 99,9 % et à 99,999 %.

    **Question 9.** Qu'est-ce qu'un point de défaillance unique (SPOF) ?

    - A. Un serveur particulièrement puissant
    - B. Un élément dont la panne arrête tout le service
    - C. Une baie complète
    - D. Un site de secours

    ??? success "Réponse"
        **B.** Un SPOF est un élément non redondé dont la panne interrompt le service. Le but de la redondance est de les supprimer.

    **Question 10.** À quoi sert le contrôleur de gestion (BMC) d'un serveur ?

    - A. Augmenter la fréquence du processeur
    - B. Chiffrer le stockage
    - C. Allumer, éteindre et surveiller le serveur à distance, même lorsqu'il est éteint
    - D. Remplacer le système d'exploitation

    ??? success "Réponse"
        **C.** Le BMC est un petit système autonome qui permet l'administration à distance d'un serveur, y compris lorsque celui-ci est éteint.

=== "English"

    !!! abstract "Instructions"
        Self-assessment: 10 multiple-choice questions, about 10 minutes. Only one answer is correct per question. Answer from memory first, then click "Answer" to check. If an answer surprises you, reread the matching section of the [lesson](cours.md).

    **Question 1.** In which type of datacenter does the operator rent racks, power and connectivity, with the customer bringing its own equipment?

    - A. Edge
    - B. Enterprise server room
    - C. Colocation
    - D. Public cloud datacenter (hyperscale)

    ??? success "Answer"
        **C.** In colocation, the customer installs its hardware in the rented space. An *edge* site is a small site close to users; a hyperscale site is run by the cloud provider itself.

    **Question 2.** What is the height of one rack unit (1 U)?

    - A. 25.4 mm
    - B. 44.45 mm
    - C. 482.6 mm
    - D. 600 mm

    ??? success "Answer"
        **B.** 1 U = 44.45 mm. The 482.6 mm value is the 19-inch **width** of a standard rack, not its unit height.

    **Question 3.** Which three limits determine the hardware a rack can host?

    - A. Space, electrical power and cooling
    - B. Space, number of network ports and price
    - C. Electrical power, operating system and height
    - D. Cooling, software license and cable length

    ??? success "Answer"
        **A.** These are the three physical constraints of a rack; the most demanding one decides how much equipment can be installed.

    **Question 4.** An IT device consumes 1 kW of electricity. About how much heat must be removed?

    - A. About 0.1 kW
    - B. About 0.5 kW
    - C. None: the energy is stored in the device
    - D. About 1 kW

    ??? success "Answer"
        **D.** Almost all the consumed electrical energy is dissipated as heat. Cooling is therefore sized from the installed power.

    **Question 5.** A datacenter draws 600 kW in total, of which 400 kW goes to the IT equipment. What is its PUE?

    - A. 0.67
    - B. 1.5
    - C. 2.0
    - D. 0.5

    ??? success "Answer"
        **B.** PUE = total energy ÷ IT energy = 600 ÷ 400 = 1.5. PUE is always greater than or equal to 1; the 0.67 value is the inverse of the ratio.

    **Question 6.** During a utility power outage, which element takes over immediately while the generator starts?

    - A. The generator itself
    - B. The automatic transfer switch (ATS)
    - C. The UPS batteries
    - D. The rack distribution (PDU)

    ??? success "Answer"
        **C.** The UPS batteries keep the power continuous while the generator starts. The transfer switch then switches to the generator, and the PDU only distributes power inside the rack.

    **Question 7.** What does a "2N" redundancy mean?

    - A. Two complete, independent chains
    - B. A single spare element for the whole
    - C. Just what the need requires, with no margin
    - D. Two identical servers in each rack

    ??? success "Answer"
        **A.** With 2N, losing an entire chain, even during maintenance on the other, does not interrupt the service. Option B describes N+1 and option C describes N.

    **Question 8.** An availability of 99.99% corresponds to which maximum yearly downtime?

    - A. About 8.76 hours
    - B. About 5.26 minutes
    - C. About 87.6 hours
    - D. About 52.6 minutes

    ??? success "Answer"
        **D.** A year has 8,760 hours: 0.01% of 8,760 h = 0.876 h, about 52.6 minutes. The 8.76 h and 5.26 min values correspond to 99.9% and 99.999%.

    **Question 9.** What is a single point of failure (SPOF)?

    - A. A particularly powerful server
    - B. An element whose failure stops the whole service
    - C. A complete rack
    - D. A backup site

    ??? success "Answer"
        **B.** A SPOF is a non-redundant element whose failure interrupts the service. The purpose of redundancy is to remove them.

    **Question 10.** What is a server's management controller (BMC) used for?

    - A. Increasing the processor frequency
    - B. Encrypting the storage
    - C. Powering the server on and off and monitoring it remotely, even when it is off
    - D. Replacing the operating system

    ??? success "Answer"
        **C.** The BMC is a small autonomous system that allows remote administration of a server, including when it is powered off.