# Quiz du chapitre 5 : Connexion entre VM et réseaux virtuels / Chapter 5 Quiz: VM Connectivity and Virtual Networks

=== "Français"

    !!! abstract "Consignes"
        Autoévaluation : 10 questions à choix multiples, environ 10 minutes. Une seule réponse est correcte par question. Répondez d'abord de mémoire, puis cliquez sur « Réponse » pour vérifier. Si une réponse vous surprend, relisez la section correspondante du [cours](cours.md).

    **Question 1.** À quel niveau fonctionne un commutateur virtuel ?

    - A. Niveau 1 (signaux physiques)
    - B. Niveau 2 (trames et adresses MAC)
    - C. Niveau 3 (paquets IP et routage)
    - D. Niveau 7 (applications)

    ??? success "Réponse"
        **B.** Un commutateur apprend les adresses MAC et transmet les trames entre les cartes branchées dessus. Le routage entre réseaux relève du niveau 3.

    **Question 2.** Quel mode réseau permet aux VM de communiquer entre elles et avec l'hôte, mais pas avec Internet ?

    - A. NAT
    - B. Pont (bridge)
    - C. Réseau isolé (réseau interne, segment LAN, commutateur privé)
    - D. Hôte seul (host-only)

    ??? success "Réponse"
        **D.** En hôte seul, l'hôte participe au réseau mais aucune sortie n'est fournie. Un réseau isolé n'inclut pas l'hôte ; le NAT et le pont donnent accès à Internet.

    **Question 3.** Quel type de commutateur Hyper-V correspond au mode « hôte seul » de VirtualBox et de VMware ?

    - A. Interne
    - B. Privé
    - C. Externe
    - D. Le Default Switch

    ??? success "Réponse"
        **A.** Le commutateur interne relie les VM et l'hôte. Le privé correspond au réseau isolé (réseau interne, segment LAN), l'externe au pont.

    **Question 4.** Combien de machines utilisables comporte un réseau IPv4 en `/26` ?

    - A. 64
    - B. 63
    - C. 62
    - D. 30

    ??? success "Réponse"
        **C.** Il reste 6 bits pour les machines : 2^6 = 64 adresses, dont 2 réservées (réseau et diffusion), soit 62 utilisables.

    **Question 5.** Quelle est l'adresse de diffusion du réseau `10.10.10.0/26` ?

    - A. `10.10.10.255`
    - B. `10.10.10.63`
    - C. `10.10.10.64`
    - D. `10.10.10.62`

    ??? success "Réponse"
        **B.** Le bloc va de `10.10.10.0` à `10.10.10.63` ; la dernière adresse est l'adresse de diffusion. `10.10.10.64` est le début du sous-réseau suivant.

    **Question 6.** Pourquoi deux réseaux virtuels ne doivent-ils pas avoir la même plage d'adresses ?

    - A. Le routage ne peut pas choisir entre deux réseaux identiques
    - B. Le serveur DHCP cesserait de fonctionner
    - C. Les VM perdraient une partie de leur RAM
    - D. Le NAT serait désactivé

    ??? success "Réponse"
        **A.** Avec des plages identiques, une machine reliée aux deux réseaux ne sait pas par quelle interface joindre une adresse donnée.

    **Question 7.** Qu'est-ce qui permet à une VM Linux de transmettre des paquets entre deux réseaux, comme un routeur ?

    - A. La commande `ip addr`
    - B. Le serveur DHCP
    - C. Le DNS
    - D. L'activation du transfert IP (`net.ipv4.ip_forward=1`)

    ??? success "Réponse"
        **D.** Par défaut, Linux ignore les paquets qui ne lui sont pas destinés. Avec le transfert IP, il les achemine vers le bon réseau d'après sa table de routage.

    **Question 8.** Quelle forme de NAT permet à des machines privées d'accéder à Internet derrière l'adresse du routeur ?

    - A. Le NAT de destination (DNAT)
    - B. La redirection de port
    - C. Le NAT source (masquerade)
    - D. Le marquage VLAN

    ??? success "Réponse"
        **C.** Le masquerade remplace l'adresse source privée par celle de la sortie du routeur. Le DNAT et la redirection de port servent à atteindre une machine privée depuis l'extérieur.

    **Question 9.** Deux VM sont branchées au même commutateur, avec des adresses du même sous-réseau, mais dans des VLAN différents. Que se passe-t-il ?

    - A. Elles communiquent normalement
    - B. Elles ne communiquent pas au niveau 2 ; un routeur serait nécessaire entre les VLAN
    - C. Elles communiquent grâce au NAT
    - D. Elles communiquent seulement si leurs adresses MAC sont identiques

    ??? success "Réponse"
        **B.** Un VLAN est un domaine de diffusion distinct : les trames d'un VLAN ne sont pas transmises à un autre. Un routeur, avec un réseau IP par VLAN, est nécessaire pour les relier.

    **Question 10.** Quelle est la première étape d'un diagnostic réseau couche par couche ?

    - A. Vérifier l'interface et son adresse
    - B. Capturer le trafic avec `tcpdump`
    - C. Réinstaller la VM
    - D. Changer le serveur DNS

    ??? success "Réponse"
        **A.** On commence par le plus proche : l'interface est-elle active, avec la bonne adresse ? On remonte ensuite vers le voisin, la passerelle, l'extérieur, le DNS, le service, et on capture le trafic en dernier recours.

=== "English"

    !!! abstract "Instructions"
        Self-assessment: 10 multiple-choice questions, about 10 minutes. Only one answer is correct per question. Answer from memory first, then click "Answer" to check. If an answer surprises you, reread the matching section of the [lesson](cours.md).

    **Question 1.** At which layer does a virtual switch work?

    - A. Layer 1 (physical signals)
    - B. Layer 2 (frames and MAC addresses)
    - C. Layer 3 (IP packets and routing)
    - D. Layer 7 (applications)

    ??? success "Answer"
        **B.** A switch learns MAC addresses and forwards frames between the cards plugged into it. Routing between networks belongs to layer 3.

    **Question 2.** Which network mode lets VMs communicate with each other and with the host, but not with the Internet?

    - A. NAT
    - B. Bridge
    - C. Isolated network (internal network, LAN segment, private switch)
    - D. Host-only

    ??? success "Answer"
        **D.** In host-only mode, the host takes part in the network but no exit is provided. An isolated network does not include the host; NAT and bridge give Internet access.

    **Question 3.** Which Hyper-V switch type matches the "host-only" mode of VirtualBox and VMware?

    - A. Internal
    - B. Private
    - C. External
    - D. The Default Switch

    ??? success "Answer"
        **A.** The internal switch connects VMs and the host. Private matches the isolated network (internal network, LAN segment), external matches bridge.

    **Question 4.** How many usable hosts does an IPv4 `/26` network have?

    - A. 64
    - B. 63
    - C. 62
    - D. 30

    ??? success "Answer"
        **C.** 6 bits remain for hosts: 2^6 = 64 addresses, 2 of them reserved (network and broadcast), so 62 usable.

    **Question 5.** What is the broadcast address of the `10.10.10.0/26` network?

    - A. `10.10.10.255`
    - B. `10.10.10.63`
    - C. `10.10.10.64`
    - D. `10.10.10.62`

    ??? success "Answer"
        **B.** The block runs from `10.10.10.0` to `10.10.10.63`; the last address is the broadcast address. `10.10.10.64` is the start of the next subnet.

    **Question 6.** Why must two virtual networks not have the same address range?

    - A. Routing cannot choose between two identical networks
    - B. The DHCP server would stop working
    - C. VMs would lose part of their RAM
    - D. NAT would be disabled

    ??? success "Answer"
        **A.** With identical ranges, a machine connected to both networks does not know through which interface to reach a given address.

    **Question 7.** What allows a Linux VM to forward packets between two networks, like a router?

    - A. The `ip addr` command
    - B. The DHCP server
    - C. DNS
    - D. Enabling IP forwarding (`net.ipv4.ip_forward=1`)

    ??? success "Answer"
        **D.** By default, Linux ignores packets not addressed to it. With IP forwarding, it forwards them to the right network according to its routing table.

    **Question 8.** Which form of NAT lets private machines reach the Internet behind the router's address?

    - A. Destination NAT (DNAT)
    - B. Port forwarding
    - C. Source NAT (masquerade)
    - D. VLAN tagging

    ??? success "Answer"
        **C.** Masquerade replaces the private source address with that of the router's outgoing interface. DNAT and port forwarding are used to reach a private machine from outside.

    **Question 9.** Two VMs are plugged into the same switch, with addresses from the same subnet, but in different VLANs. What happens?

    - A. They communicate normally
    - B. They do not communicate at layer 2; a router would be needed between the VLANs
    - C. They communicate thanks to NAT
    - D. They communicate only if their MAC addresses are identical

    ??? success "Answer"
        **B.** A VLAN is a distinct broadcast domain: frames of one VLAN are not forwarded to another. A router, with one IP network per VLAN, is needed to link them.

    **Question 10.** What is the first step of a layer-by-layer network diagnosis?

    - A. Check the interface and its address
    - B. Capture traffic with `tcpdump`
    - C. Reinstall the VM
    - D. Change the DNS server

    ??? success "Answer"
        **A.** Start with the nearest: is the interface up, with the right address? Then move on to the neighbor, gateway, outside, DNS, service, and capture traffic as a last resort.