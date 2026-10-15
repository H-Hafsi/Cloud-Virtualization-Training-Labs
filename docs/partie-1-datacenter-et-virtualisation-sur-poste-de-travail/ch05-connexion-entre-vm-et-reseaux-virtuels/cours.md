# Chapitre 5 : Connexion entre VM et réseaux virtuels / VM Connectivity and Virtual Networks

=== "Français"

    !!! abstract "Objectifs du chapitre"
        - Décrire les composants d'un réseau virtuel : carte virtuelle, commutateur virtuel, NAT, serveur DHCP
        - Comparer les modes réseau de VirtualBox, VMware Workstation et Hyper-V
        - Construire un plan d'adressage et calculer un sous-réseau
        - Expliquer le routage, la redirection d'adresses (NAT) et les VLAN
        - Appliquer une méthode de diagnostic réseau couche par couche
        - Acquis d'apprentissage visé : **AA2**

        Matériel requis pour ce cours : aucun. Les labs associés indiquent leurs propres ressources.

=== "English"

    !!! abstract "Chapter objectives"
        - Describe the components of a virtual network: virtual card, virtual switch, NAT, DHCP server
        - Compare the network modes of VirtualBox, VMware Workstation and Hyper-V
        - Build an addressing plan and compute a subnet
        - Explain routing, network address translation (NAT) and VLANs
        - Apply a layer-by-layer network troubleshooting method
        - Target learning outcome: **AA2**

        Required equipment for this lesson: none. The associated labs state their own resources.

## 1. Les composants d'un réseau virtuel / Components of a virtual network

=== "Français"

    Aux chapitres 3 et 4, vous avez branché des VM sur des réseaux sans entrer dans le détail. Un réseau virtuel reprend, en logiciel, les éléments d'un réseau physique.

    ```mermaid
    flowchart TB
      V1["VM 1<br/>carte virtuelle"] --- S["Commutateur virtuel"]
      V2["VM 2<br/>carte virtuelle"] --- S
      S --- H["Carte virtuelle de l'hôte<br/>(réseau hôte seul / interne)"]
      S --- N["NAT et DHCP<br/>fournis par l'hyperviseur"]
      N --- U["Carte physique de l'hôte<br/>(réseau extérieur)"]
    ```

    | Composant | Rôle |
    |---|---|
    | **Carte réseau virtuelle (vNIC)** | Carte vue par la VM, avec sa propre adresse MAC. Son modèle varie : `virtio`, Intel (e1000), `vmxnet3`, carte « synthétique » d'Hyper-V |
    | **Commutateur virtuel** | Commutateur logiciel de **niveau 2** : il apprend les adresses MAC et transmet les trames entre les cartes branchées dessus |
    | **Carte virtuelle de l'hôte** | Permet à l'hôte de participer au réseau (par exemple `VMware Network Adapter VMnet1`, `vEthernet (gestion-interne)`) |
    | **NAT** | Fait sortir les VM derrière l'adresse de l'hôte ; l'extérieur ne voit que l'hôte |
    | **Serveur DHCP** | Distribue automatiquement adresse, passerelle et DNS ; souvent fourni par l'hyperviseur sur ses réseaux NAT et hôte seul |

    Deux machines sont sur le **même réseau** (même **domaine de diffusion**) lorsqu'elles sont branchées au même commutateur, sans VLAN ni routeur entre elles.

=== "English"

    In chapters 3 and 4, you connected VMs to networks without going into detail. A virtual network reproduces, in software, the elements of a physical network.

    ```mermaid
    flowchart TB
      V1["VM 1<br/>virtual card"] --- S["Virtual switch"]
      V2["VM 2<br/>virtual card"] --- S
      S --- H["Host virtual card<br/>(host-only / internal network)"]
      S --- N["NAT and DHCP<br/>provided by the hypervisor"]
      N --- U["Host physical card<br/>(outside network)"]
    ```

    | Component | Role |
    |---|---|
    | **Virtual network interface card (vNIC)** | Card seen by the VM, with its own MAC address. Its model varies: `virtio`, Intel (e1000), `vmxnet3`, Hyper-V "synthetic" card |
    | **Virtual switch** | **Layer 2** software switch: it learns MAC addresses and forwards frames between the cards plugged into it |
    | **Host virtual card** | Lets the host take part in the network (for example `VMware Network Adapter VMnet1`, `vEthernet (gestion-interne)`) |
    | **NAT** | Makes VMs go out behind the host's address; the outside only sees the host |
    | **DHCP server** | Automatically hands out address, gateway and DNS; often provided by the hypervisor on its NAT and host-only networks |

    Two machines are on the **same network** (same **broadcast domain**) when they are plugged into the same switch, with no VLAN or router between them.

## 2. Les modes réseau selon l'hyperviseur / Network modes by hypervisor

=== "Français"

    | Fonction | VirtualBox | VMware Workstation | Hyper-V |
    |---|---|---|---|
    | **NAT** | NAT *(chaque VM isolée)* ou **NAT Network** *(VM du même réseau NAT visibles entre elles)* | NAT (`VMnet8`) | *Default Switch* |
    | **Pont** | Bridged Adapter | Bridged (`VMnet0`) | Commutateur **externe** |
    | **Hôte seul** | Host-only Adapter | Host-only (`VMnet1`) | Commutateur **interne** |
    | **Isolé entre VM** | Internal Network | LAN Segment | Commutateur **privé** |

    Que peut faire une VM selon le mode ? (« ✓ » : oui, « ✗ » : non)

    | Mode | VM ↔ VM (même réseau) | VM ↔ hôte | VM → Internet | Extérieur → VM |
    |---|---|---|---|---|
    | **NAT** (VMware, Hyper-V, VirtualBox NAT Network) | ✓ | ✓ | ✓ | ✗ (sans redirection de ports) |
    | **NAT** de VirtualBox (simple) | ✗ (VM isolées) | Limité (par l'adresse de l'hôte vue depuis la VM) | ✓ | ✗ |
    | **Pont** | ✓ | ✓ (en général) | ✓ | ✓ (depuis le réseau local) |
    | **Hôte seul / Interne Hyper-V** | ✓ | ✓ | ✗ | ✗ |
    | **Isolé** (Internal Network, LAN Segment, Privé) | ✓ | ✗ | ✗ | ✗ |

    !!! warning "Mode pont et Wi-Fi"
        Le mode pont fonctionne bien avec une carte Ethernet filaire. Avec une carte Wi-Fi, il est parfois limité ou instable selon la carte et le pilote. Dans ce cours, on privilégie NAT et hôte seul/interne, qui fonctionnent partout.

=== "English"

    | Function | VirtualBox | VMware Workstation | Hyper-V |
    |---|---|---|---|
    | **NAT** | NAT *(each VM isolated)* or **NAT Network** *(VMs on the same NAT network see each other)* | NAT (`VMnet8`) | *Default Switch* |
    | **Bridge** | Bridged Adapter | Bridged (`VMnet0`) | **External** switch |
    | **Host-only** | Host-only Adapter | Host-only (`VMnet1`) | **Internal** switch |
    | **Isolated between VMs** | Internal Network | LAN Segment | **Private** switch |

    What can a VM do depending on the mode? ("✓": yes, "✗": no)

    | Mode | VM ↔ VM (same network) | VM ↔ host | VM → Internet | Outside → VM |
    |---|---|---|---|---|
    | **NAT** (VMware, Hyper-V, VirtualBox NAT Network) | ✓ | ✓ | ✓ | ✗ (without port forwarding) |
    | **NAT** of VirtualBox (simple) | ✗ (isolated VMs) | Limited (through the host address seen from the VM) | ✓ | ✗ |
    | **Bridge** | ✓ | ✓ (generally) | ✓ | ✓ (from the local network) |
    | **Host-only / Hyper-V internal** | ✓ | ✓ | ✗ | ✗ |
    | **Isolated** (Internal Network, LAN Segment, Private) | ✓ | ✗ | ✗ | ✗ |

    !!! warning "Bridge mode and Wi-Fi"
        Bridge mode works well with a wired Ethernet card. With a Wi-Fi card, it is sometimes limited or unstable depending on the card and driver. In this course, NAT and host-only/internal are preferred, as they work everywhere.

## 3. Adressage et sous-réseaux / Addressing and subnets

=== "Français"

    Une adresse IPv4 s'écrit avec son **préfixe** : `10.10.10.5/24` signifie que les 24 premiers bits identifient le **réseau** et les 8 derniers la **machine**.

    | Notion | Définition | Exemple (`10.10.10.0/24`) |
    |---|---|---|
    | **Adresse de réseau** | Premier bloc, tous les bits machine à 0 | `10.10.10.0` |
    | **Adresse de diffusion** | Dernier bloc, tous les bits machine à 1 | `10.10.10.255` |
    | **Machines utilisables** | 2^(32 − préfixe) − 2 | 2^8 − 2 = **254** |
    | **Passerelle** | Adresse du routeur qui permet de sortir du réseau | par exemple `10.10.10.1` |

    **Autre exemple : `10.10.10.0/26`.** Il reste 6 bits pour les machines : 2^6 = 64 adresses, soit **62** utilisables ; adresse de réseau `10.10.10.0`, adresse de diffusion `10.10.10.63`. Le sous-réseau suivant commence à `10.10.10.64`.

    **Plages privées** (RFC 1918), réservées aux réseaux internes : `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.

    Règles pratiques pour votre plan d'adressage :

    - Un **réseau par usage** (administration, stockage, etc.), avec un préfixe identique et cohérent.
    - **Aucun chevauchement** entre les réseaux virtuels, ni avec le réseau physique de votre poste : le routage ne sait pas choisir entre deux réseaux identiques.
    - Des adresses **statiques** pour les serveurs, documentées dans un tableau ; DHCP pour les clients et les essais.
    - Respecter le plan de l'annexe [Environnement](../../annexes/environnement.md).

=== "English"

    An IPv4 address is written with its **prefix**: `10.10.10.5/24` means the first 24 bits identify the **network** and the last 8 the **host**.

    | Notion | Definition | Example (`10.10.10.0/24`) |
    |---|---|---|
    | **Network address** | First block, all host bits at 0 | `10.10.10.0` |
    | **Broadcast address** | Last block, all host bits at 1 | `10.10.10.255` |
    | **Usable hosts** | 2^(32 − prefix) − 2 | 2^8 − 2 = **254** |
    | **Gateway** | Address of the router that allows leaving the network | for example `10.10.10.1` |

    **Another example: `10.10.10.0/26`.** 6 bits remain for hosts: 2^6 = 64 addresses, so **62** usable; network address `10.10.10.0`, broadcast address `10.10.10.63`. The next subnet starts at `10.10.10.64`.

    **Private ranges** (RFC 1918), reserved for internal networks: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.

    Practical rules for your addressing plan:

    - **One network per purpose** (administration, storage, etc.), with a consistent prefix.
    - **No overlap** between virtual networks, or with your computer's physical network: routing cannot choose between two identical networks.
    - **Static** addresses for servers, documented in a table; DHCP for clients and tests.
    - Follow the plan in the [Environment](../../annexes/environnement.md) annex.

## 4. Routage et NAT / Routing and NAT

=== "Français"

    Pour que deux réseaux différents communiquent, il faut un **routeur** : une machine reliée aux deux, qui transmet les paquets de l'un à l'autre. Dans un laboratoire virtuel, un routeur est souvent une **VM Linux à deux cartes** dont le **transfert IP** (*IP forwarding*) est activé.

    ```mermaid
    flowchart LR
      A["VM A<br/>10.10.10.10/24<br/>passerelle 10.10.10.1"] --- R1["10.10.10.1"]
      subgraph RT["Routeur (VM Linux, transfert IP activé)"]
        R1
        R2["10.10.20.1"]
      end
      R2 --- B["VM B<br/>10.10.20.10/24<br/>passerelle 10.10.20.1"]
    ```

    - Chaque machine consulte sa **table de routage** : si la destination n'est pas sur un de ses réseaux, elle envoie le paquet à sa **passerelle par défaut**.
    - Un routeur lit l'adresse de destination, consulte sa table et transmet le paquet vers le bon réseau.

    ### Le NAT

    Le **NAT** (traduction d'adresses réseau) réécrit les adresses d'un paquet au passage d'un routeur :

    | Forme | Principe | Usage |
    |---|---|---|
    | **Source NAT / masquerade** | L'adresse source privée est remplacée par celle de la sortie du routeur | Faire accéder des machines privées à Internet |
    | **Destination NAT / redirection de port** | Les paquets reçus sur un port du routeur sont redirigés vers une machine privée | Exposer un service d'une VM depuis l'extérieur |

    Le routeur mémorise les échanges pour renvoyer les réponses à la bonne machine. C'est ce que font, en coulisses, les modes NAT des hyperviseurs. Au Lab 5.3, vous réaliserez vous-même un routage et un NAT avec des VM Linux.

=== "English"

    For two different networks to communicate, a **router** is needed: a machine connected to both, that forwards packets from one to the other. In a virtual lab, a router is often a **Linux VM with two cards** with **IP forwarding** enabled.

    ```mermaid
    flowchart LR
      A["VM A<br/>10.10.10.10/24<br/>gateway 10.10.10.1"] --- R1["10.10.10.1"]
      subgraph RT["Router (Linux VM, IP forwarding on)"]
        R1
        R2["10.10.20.1"]
      end
      R2 --- B["VM B<br/>10.10.20.10/24<br/>gateway 10.10.20.1"]
    ```

    - Each machine consults its **routing table**: if the destination is not on one of its networks, it sends the packet to its **default gateway**.
    - A router reads the destination address, consults its table and forwards the packet to the right network.

    ### NAT

    **NAT** (network address translation) rewrites a packet's addresses as it passes through a router:

    | Form | Principle | Use |
    |---|---|---|
    | **Source NAT / masquerade** | The private source address is replaced by that of the router's outgoing interface | Let private machines reach the Internet |
    | **Destination NAT / port forwarding** | Packets received on a router port are redirected to a private machine | Expose a VM's service from outside |

    The router remembers exchanges to send replies to the right machine. This is what hypervisor NAT modes do behind the scenes. In Lab 5.3, you will build routing and NAT yourself with Linux VMs.

## 5. Les VLAN / VLANs

=== "Français"

    Un **VLAN** (réseau local virtuel) découpe logiquement un même commutateur en plusieurs réseaux séparés : les machines de deux VLAN différents ne se voient pas au niveau 2, même branchées au même équipement. Les VLAN réduisent les domaines de diffusion et isolent les usages (administration, stockage, invités…).

    | Notion | Définition |
    |---|---|
    | **Identifiant de VLAN** | Nombre de 12 bits (norme 802.1Q) ; les valeurs utilisables vont de 1 à 4094 |
    | **Port d'accès (access)** | Port appartenant à un seul VLAN : les trames n'y portent pas d'étiquette |
    | **Port de jonction (trunk)** | Port transportant plusieurs VLAN : chaque trame porte une **étiquette** (tag) 802.1Q qui indique son VLAN |
    | **Routage inter-VLAN** | Pour que deux VLAN communiquent, il faut un **routeur** (niveau 3) : c'est un réseau IP distinct par VLAN |

    Dans le monde virtuel, on retrouve ces notions : un **groupe de ports** VMware peut porter un identifiant de VLAN, une carte Hyper-V peut se voir attribuer un VLAN, et Linux sait créer des interfaces étiquetées (`eth0.10` pour le VLAN 10). Le Lab 5.3 vous fait créer deux VLAN sous Linux.

=== "English"

    A **VLAN** (virtual local area network) logically splits a single switch into several separate networks: machines in two different VLANs do not see each other at layer 2, even when plugged into the same device. VLANs reduce broadcast domains and isolate uses (administration, storage, guests...).

    | Notion | Definition |
    |---|---|
    | **VLAN ID** | 12-bit number (802.1Q standard); usable values range from 1 to 4094 |
    | **Access port** | Port belonging to a single VLAN: frames carry no tag |
    | **Trunk port** | Port carrying several VLANs: each frame carries an 802.1Q **tag** indicating its VLAN |
    | **Inter-VLAN routing** | For two VLANs to communicate, a **router** (layer 3) is needed: each VLAN is a distinct IP network |

    In the virtual world, these notions are found again: a VMware **port group** can carry a VLAN ID, a Hyper-V card can be assigned a VLAN, and Linux can create tagged interfaces (`eth0.10` for VLAN 10). Lab 5.3 has you create two VLANs under Linux.

## 6. Diagnostiquer un problème réseau / Troubleshooting a network problem

=== "Français"

    Procédez **couche par couche**, du plus proche au plus lointain, et arrêtez-vous à la première étape qui échoue.

    | Étape | Question | Outil (Linux) | Outil (Windows) |
    |---|---|---|---|
    | 1. Interface | La carte est-elle active, avec la bonne adresse ? | `ip -br link`, `ip -br addr` | `ipconfig` |
    | 2. Voisin | La machine voisine du même réseau répond-elle ? | `ping <adresse du voisin>` | `ping` |
    | 3. Passerelle | La passerelle répond-elle ? La route par défaut est-elle correcte ? | `ping <passerelle>`, `ip route` | `Test-NetConnection <passerelle>` |
    | 4. Extérieur | Une adresse extérieure répond-elle ? | `ping <adresse IP publique>` | `ping` |
    | 5. Noms | Le DNS résout-il les noms ? | `getent hosts <nom>`, `resolvectl status` | `Resolve-DnsName <nom>` |
    | 6. Service | Le service écoute-t-il, et est-il accessible ? | `ss -tlnp`, `curl <URL>` | `Test-NetConnection <hôte> -Port <port>`, `curl.exe` |
    | 7. Trafic | Que voit-on réellement sur le réseau ? | `sudo tcpdump -i <interface>` | — |

    !!! tip "À retenir"
        - Un réseau virtuel reprend en logiciel : cartes virtuelles, commutateurs, NAT, DHCP.
        - Modes courants : NAT (Internet), pont (réseau local), hôte seul / interne (administration), isolé (entre VM).
        - Adresse = réseau + machine ; machines utilisables = 2^(32 − préfixe) − 2.
        - Pas de chevauchement entre réseaux ; adresses statiques documentées pour les serveurs.
        - Un routeur relie deux réseaux ; la passerelle par défaut est la sortie du réseau.
        - NAT source (accès sortant) et NAT de destination (redirection de port).
        - VLAN : séparation logique au niveau 2 (802.1Q) ; routeur nécessaire entre VLAN.
        - Diagnostic couche par couche : interface, voisin, passerelle, extérieur, DNS, service, trafic.

=== "English"

    Proceed **layer by layer**, from the nearest to the farthest, and stop at the first step that fails.

    | Step | Question | Tool (Linux) | Tool (Windows) |
    |---|---|---|---|
    | 1. Interface | Is the card up, with the right address? | `ip -br link`, `ip -br addr` | `ipconfig` |
    | 2. Neighbor | Does the neighboring machine on the same network answer? | `ping <neighbor address>` | `ping` |
    | 3. Gateway | Does the gateway answer? Is the default route correct? | `ping <gateway>`, `ip route` | `Test-NetConnection <gateway>` |
    | 4. Outside | Does an outside address answer? | `ping <public IP address>` | `ping` |
    | 5. Names | Does DNS resolve names? | `getent hosts <name>`, `resolvectl status` | `Resolve-DnsName <name>` |
    | 6. Service | Is the service listening, and reachable? | `ss -tlnp`, `curl <URL>` | `Test-NetConnection <host> -Port <port>`, `curl.exe` |
    | 7. Traffic | What is actually seen on the network? | `sudo tcpdump -i <interface>` | — |

    !!! tip "Key takeaways"
        - A virtual network reproduces in software: virtual cards, switches, NAT, DHCP.
        - Common modes: NAT (Internet), bridge (local network), host-only / internal (administration), isolated (between VMs).
        - Address = network + host; usable hosts = 2^(32 − prefix) − 2.
        - No overlap between networks; documented static addresses for servers.
        - A router links two networks; the default gateway is the way out of the network.
        - Source NAT (outbound access) and destination NAT (port forwarding).
        - VLAN: logical separation at layer 2 (802.1Q); a router is needed between VLANs.
        - Layer-by-layer troubleshooting: interface, neighbor, gateway, outside, DNS, service, traffic.

## Pour vérifier votre compréhension / Check your understanding

=== "Français"

    ??? question "Combien de machines peut-on adresser dans un réseau `/26` ? Quelle est l'adresse de diffusion de `10.10.10.0/26` ?"
        Il reste 6 bits pour les machines : 2^6 = 64 adresses, dont 2 réservées (réseau et diffusion), soit **62** machines utilisables. L'adresse de diffusion de `10.10.10.0/26` est **`10.10.10.63`**.

    ??? question "Un collègue sur le réseau local doit accéder à un serveur web hébergé dans votre VM. Quel mode réseau choisissez-vous et quelle précaution prenez-vous ?"
        Le mode **pont** : la VM apparaît comme une machine du réseau local, avec une adresse de ce réseau. Il faut vérifier que la carte de l'hôte le permet (le Wi-Fi est parfois limité) et se souvenir que la VM est alors **exposée** sur le réseau : n'ouvrez que les services nécessaires. Une alternative est le NAT avec redirection de port.

    ??? question "Deux VM branchées au même commutateur sont dans deux VLAN différents. Peuvent-elles communiquer ? Que faut-il ajouter pour cela ?"
        Non, pas directement : un VLAN sépare les machines au niveau 2. Il faut un **routeur** (niveau 3) relié aux deux VLAN, chacun étant un réseau IP distinct avec sa passerelle.

=== "English"

    ??? question "How many hosts can be addressed in a `/26` network? What is the broadcast address of `10.10.10.0/26`?"
        6 bits remain for hosts: 2^6 = 64 addresses, 2 of them reserved (network and broadcast), so **62** usable hosts. The broadcast address of `10.10.10.0/26` is **`10.10.10.63`**.

    ??? question "A colleague on the local network must reach a web server hosted in your VM. Which network mode do you choose and what precaution do you take?"
        **Bridge** mode: the VM appears as a machine on the local network, with an address from that network. You must check that the host's card allows it (Wi-Fi is sometimes limited) and remember that the VM is then **exposed** on the network: open only the necessary services. An alternative is NAT with port forwarding.

    ??? question "Two VMs plugged into the same switch are in two different VLANs. Can they communicate? What must be added?"
        Not directly: a VLAN separates machines at layer 2. A **router** (layer 3) connected to both VLANs is needed, each being a distinct IP network with its own gateway.

## Ressources du chapitre / Chapter resources

- [Lab 5.1 : Réseaux virtuels sous VirtualBox et VMware / Virtual networks with VirtualBox and VMware](lab-5-1-reseaux-virtuels-sous-virtualbox-et-vmware.md)
- [Lab 5.2 : Commutateurs virtuels sous Hyper-V / Virtual switches with Hyper-V](lab-5-2-commutateurs-virtuels-sous-hyper-v.md)
- [Lab 5.3 : Routage, NAT et VLAN sous Linux / Routing, NAT and VLAN on Linux](lab-5-3-routage-nat-et-vlan-sous-linux.md)
- [Quiz du chapitre / Chapter quiz](quiz.md)
- [Glossaire / Glossary](../../annexes/glossaire.md)
- [Environnement de TP / Lab environment](../../annexes/environnement.md)