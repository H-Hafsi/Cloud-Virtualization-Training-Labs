# Lab 5.3 : Routage, NAT et VLAN sous Linux / Routing, NAT and VLAN on Linux

| | |
|---|---|
| **Durée / Duration** | 120 à 150 min (trois parties indépendantes : routage, NAT, VLAN) / 120 to 150 min (three independent parts: routing, NAT, VLAN) |
| **Niveau / Level** | Semi-guidé / Semi-guided |
| **Prérequis / Prerequisites** | [Chapitre 5 : cours](cours.md) ; [Lab 5.1](lab-5-1-reseaux-virtuels-sous-virtualbox-et-vmware.md) réalisé (partie VMware : VM `srv-a-vmware`, `srv-b-vmware`, segment LAN) |
| **Ressources minimales / Minimum resources** | **VM** : 3 VM Ubuntu Server sous **VMware Workstation** (`srv-a`, `srv-b`, `routeur`), 1 vCPU et 1 Go de RAM chacune, clones liés de `srv1-vmware`. **Hôte** : au moins 5 Go de RAM libres, 6 Go d'espace disque libre. L'hyperviseur Windows doit être **désactivé**. / **VMs**: 3 Ubuntu Server VMs under **VMware Workstation** (`srv-a`, `srv-b`, `routeur`), 1 vCPU and 1 GB RAM each, linked clones of `srv1-vmware`. **Host**: at least 5 GB of free RAM, 6 GB of free disk space. The Windows hypervisor must be **off**. |
| **Acquis visés / Learning outcomes** | AA2 |

=== "Français"

    !!! abstract "Objectifs"
        - Construire un routeur Linux entre deux réseaux et activer le transfert IP
        - Mettre en place un NAT (masquerade) et une redirection de port avec `nftables`
        - Créer des VLAN 802.1Q sous Linux et réaliser un routage inter-VLAN
        - Diagnostiquer chaque étape avec `ip`, `ping`, `tracepath` et `tcpdump`

=== "English"

    !!! abstract "Objectives"
        - Build a Linux router between two networks and enable IP forwarding
        - Set up NAT (masquerade) and port forwarding with `nftables`
        - Create 802.1Q VLANs on Linux and perform inter-VLAN routing
        - Troubleshoot each step with `ip`, `ping`, `tracepath` and `tcpdump`

## Contexte et schéma / Context and diagram

=== "Français"

    Vous construisez vous-même ce que les hyperviseurs font en coulisses : un **routeur**, un **NAT** et des **VLAN**. Trois VM Linux sont utilisées : deux clients, `srv-a` et `srv-b`, et un `routeur` qui relie leurs réseaux et leur donne accès à Internet.

    ```mermaid
    flowchart LR
      A["srv-a"] --- R1["reseau-1"]
      R1 --- RT["routeur<br/>(3 cartes)"]
      RT --- R2["reseau-2"]
      R2 --- B["srv-b"]
      RT --- W["Carte WAN<br/>VMnet8 (NAT)<br/>Internet"]
    ```

    Dans le **niveau semi-guidé**, les objectifs et les commandes clés sont donnés ; vous adaptez les adresses au plan de l'annexe [Environnement](../../annexes/environnement.md) (variables `<...>` du tableau ci-dessous) et vous rédigez vous-même les fichiers de configuration à partir des modèles.

    !!! warning "Allumer les VM une à une pendant la préparation"
        Les clones héritent du nom d'hôte, de l'identifiant de machine et des clés SSH de la VM d'origine. Préparez-les l'une après l'autre.

=== "English"

    You build yourself what hypervisors do behind the scenes: a **router**, a **NAT** and **VLANs**. Three Linux VMs are used: two clients, `srv-a` and `srv-b`, and a `routeur` that links their networks and gives them Internet access.

    ```mermaid
    flowchart LR
      A["srv-a"] --- R1["reseau-1"]
      R1 --- RT["routeur<br/>(3 cards)"]
      RT --- R2["reseau-2"]
      R2 --- B["srv-b"]
      RT --- W["WAN card<br/>VMnet8 (NAT)<br/>Internet"]
    ```

    At the **semi-guided level**, objectives and key commands are given; you adapt the addresses to the plan in the [Environment](../../annexes/environnement.md) annex (the `<...>` variables in the table below) and write the configuration files yourself from the templates.

    !!! warning "Power VMs on one at a time during preparation"
        Clones inherit the hostname, machine identifier and SSH keys of the original VM. Prepare them one after the other.

## Étapes / Steps

### Étape 1 : Plan d'adressage et préparation des VM / Step 1: Addressing plan and VM preparation

=== "Français"

    Complétez le plan d'adressage à partir de l'annexe Environnement (réseaux libres, sans chevauchement) :

    | Réseau | Usage | Plage | Adresses |
    |---|---|---|---|
    | `reseau-1` | Clients A | `<RESEAU_1>` | `srv-a` : `<IP_A>` ; routeur : `<IP_R1>` |
    | `reseau-2` | Clients B | `<RESEAU_2>` | `srv-b` : `<IP_B>` ; routeur : `<IP_R2>` |
    | VLAN 10 | Partie 3 | `<RESEAU_V10>` | `srv-a` : `<IP_V10_A>` ; `srv-b` (test) : `<IP_V10_B>` ; routeur : `<IP_V10_R>` |
    | VLAN 20 | Partie 3 | `<RESEAU_V20>` | `srv-b` : `<IP_V20_B>` ; routeur : `<IP_V20_R>` |
    | WAN | Sortie du routeur | VMnet8 (DHCP) | routeur : `<IP_WAN>` |

    **Préparation des VM sous VMware Workstation :**

    1. Réutilisez `srv-a-vmware` et `srv-b-vmware` du Lab 5.1 (ou recréez-les comme clones liés de `srv1-vmware`). Créez un troisième clone lié, `routeur`, et préparez son identité (nom d'hôte `routeur`, `machine-id`, clés SSH : mêmes commandes qu'au Lab 5.1, étape 2).
    2. Créez les segments LAN `reseau-1`, `reseau-2` et `reseau-trunk` (**Network Adapter**, **LAN Segment**, **LAN Segments…**, **Add**).
    3. Réglez les cartes réseau :

    | VM | Carte 1 | Carte 2 | Carte 3 |
    |---|---|---|---|
    | `srv-a` | VMnet8 (NAT) | LAN Segment `reseau-1` | — |
    | `srv-b` | VMnet8 (NAT) | LAN Segment `reseau-2` | — |
    | `routeur` | VMnet8 (NAT), **WAN** | LAN Segment `reseau-1` | LAN Segment `reseau-2` (ajoutez la carte) |

    Relevez les **noms des interfaces** avec `ip -br link` : `<NIC_WAN>`, `<NIC_R1>`, `<NIC_R2>` sur le routeur, `<NOM_INTERFACE>` (carte 2) sur les clients.

    !!! success "Point de contrôle"
        - [ ] Trois VM préparées, noms d'hôte distincts
        - [ ] Les noms d'interfaces sont relevés et le plan d'adressage est complété

=== "English"

    Complete the addressing plan from the Environment annex (free, non-overlapping networks):

    | Network | Use | Range | Addresses |
    |---|---|---|---|
    | `reseau-1` | A clients | `<RESEAU_1>` | `srv-a`: `<IP_A>`; router: `<IP_R1>` |
    | `reseau-2` | B clients | `<RESEAU_2>` | `srv-b`: `<IP_B>`; router: `<IP_R2>` |
    | VLAN 10 | Part 3 | `<RESEAU_V10>` | `srv-a`: `<IP_V10_A>`; `srv-b` (test): `<IP_V10_B>`; router: `<IP_V10_R>` |
    | VLAN 20 | Part 3 | `<RESEAU_V20>` | `srv-b`: `<IP_V20_B>`; router: `<IP_V20_R>` |
    | WAN | Router's outside | VMnet8 (DHCP) | router: `<IP_WAN>` |

    **VM preparation under VMware Workstation:**

    1. Reuse `srv-a-vmware` and `srv-b-vmware` from Lab 5.1 (or recreate them as linked clones of `srv1-vmware`). Create a third linked clone, `routeur`, and prepare its identity (hostname `routeur`, `machine-id`, SSH keys: same commands as Lab 5.1, step 2).
    2. Create the LAN segments `reseau-1`, `reseau-2` and `reseau-trunk` (**Network Adapter**, **LAN Segment**, **LAN Segments…**, **Add**).
    3. Set the network cards:

    | VM | Card 1 | Card 2 | Card 3 |
    |---|---|---|---|
    | `srv-a` | VMnet8 (NAT) | LAN Segment `reseau-1` | — |
    | `srv-b` | VMnet8 (NAT) | LAN Segment `reseau-2` | — |
    | `routeur` | VMnet8 (NAT), **WAN** | LAN Segment `reseau-1` | LAN Segment `reseau-2` (add the card) |

    Note the **interface names** with `ip -br link`: `<NIC_WAN>`, `<NIC_R1>`, `<NIC_R2>` on the router, `<NOM_INTERFACE>` (card 2) on the clients.

    !!! success "Checkpoint"
        - [ ] Three VMs prepared, distinct hostnames
        - [ ] Interface names are noted and the addressing plan is complete

## Partie 1 : routage / Part 1: routing

### Étape 2 : Configurer les adresses et les routes / Step 2: Configure addresses and routes

=== "Français"

    **Sur le routeur**, créez `/etc/netplan/99-routeur.yaml` : adresses `<IP_R1>/24` sur `<NIC_R1>` et `<IP_R2>/24` sur `<NIC_R2>` (la carte WAN reste en DHCP). **Sur `srv-a`**, réécrivez `/etc/netplan/99-gestion.yaml` : adresse `<IP_A>/24` sur `<NOM_INTERFACE>` et une **route** vers `<RESEAU_2>/24` passant par `<IP_R1>`. **Sur `srv-b`**, l'équivalent : adresse `<IP_B>/24` et route vers `<RESEAU_1>/24` via `<IP_R2>`. Modèles :

=== "English"

    **On the router**, create `/etc/netplan/99-routeur.yaml`: addresses `<IP_R1>/24` on `<NIC_R1>` and `<IP_R2>/24` on `<NIC_R2>` (the WAN card stays on DHCP). **On `srv-a`**, rewrite `/etc/netplan/99-gestion.yaml`: address `<IP_A>/24` on `<NOM_INTERFACE>` and a **route** to `<RESEAU_2>/24` via `<IP_R1>`. **On `srv-b`**, the equivalent: address `<IP_B>/24` and route to `<RESEAU_1>/24` via `<IP_R2>`. Templates:

```yaml
# Routeur : /etc/netplan/99-routeur.yaml
network:
  version: 2
  ethernets:
    <NIC_R1>:
      dhcp4: false
      addresses:
        - <IP_R1>/24
    <NIC_R2>:
      dhcp4: false
      addresses:
        - <IP_R2>/24
```

```yaml
# Client srv-a : /etc/netplan/99-gestion.yaml
network:
  version: 2
  ethernets:
    <NOM_INTERFACE>:
      dhcp4: false
      addresses:
        - <IP_A>/24
      routes:
        - to: <RESEAU_2>/24
          via: <IP_R1>
```

```bash
sudo chmod 600 /etc/netplan/99-*.yaml
sudo netplan apply
ip -br addr
ip route
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Chaque machine a son adresse ; `ip route` montre la route vers l'autre réseau sur les clients
        - [ ] `srv-a` joint `<IP_R1>` et `srv-b` joint `<IP_R2>` (`ping`)

=== "English"

    !!! success "Checkpoint"
        - [ ] Each machine has its address; `ip route` shows the route to the other network on the clients
        - [ ] `srv-a` reaches `<IP_R1>` and `srv-b` reaches `<IP_R2>` (`ping`)

### Étape 3 : Activer le transfert IP / Step 3: Enable IP forwarding

=== "Français"

    Depuis `srv-a`, essayez de joindre `srv-b` : le `ping` **échoue**. Sur le routeur, vérifiez que le transfert IP est désactivé, activez-le, puis refaites le test :

=== "English"

    From `srv-a`, try to reach `srv-b`: the `ping` **fails**. On the router, check that IP forwarding is off, enable it, then redo the test:

```bash
ping -c 2 <IP_B>                       # depuis srv-a : échec attendu
cat /proc/sys/net/ipv4/ip_forward      # sur le routeur : 0
sudo sysctl -w net.ipv4.ip_forward=1
ping -c 2 <IP_B>                       # depuis srv-a : succès attendu
tracepath -n <IP_B>                    # depuis srv-a : le routeur apparaît
```

=== "Français"

    Rendez le réglage **permanent**, puis observez le trafic sur le routeur pendant qu'un `ping` tourne depuis `srv-a` :

=== "English"

    Make the setting **permanent**, then observe traffic on the router while a `ping` runs from `srv-a`:

```bash
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-routage.conf
sudo sysctl --system
sudo tcpdump -ni <NIC_R2> icmp
```

=== "Français"

    Résultat attendu : les paquets ICMP de `srv-a` traversent le routeur et sortent par `<NIC_R2>` ; la valeur de `ip_forward` vaut `1` après un redémarrage du routeur.

    !!! success "Point de contrôle"
        - [ ] Le ping `srv-a` vers `srv-b` échoue avant l'activation et réussit après
        - [ ] `tracepath` fait apparaître le routeur ; `tcpdump` montre les paquets sur la seconde carte

=== "English"

    Expected result: ICMP packets from `srv-a` go through the router and out of `<NIC_R2>`; the `ip_forward` value is `1` after a router restart.

    !!! success "Checkpoint"
        - [ ] The `srv-a` to `srv-b` ping fails before enabling and succeeds after
        - [ ] `tracepath` shows the router; `tcpdump` shows the packets on the second card

## Partie 2 : NAT / Part 2: NAT

### Étape 4 : Sortie vers Internet par le routeur (masquerade) / Step 4: Internet exit through the router (masquerade)

=== "Français"

    1. **Déconnectez la carte 1** de `srv-a` et `srv-b` (**VM**, **Removable Devices**, **Network Adapter**, **Disconnect**) : elles ne doivent plus joindre Internet que par le routeur.
    2. Sur chaque client, remplacez la route spécifique par une **route par défaut** via le routeur (avec une métrique basse) et un DNS, dans `/etc/netplan/99-gestion.yaml` (`<IP_R1>` pour `srv-a`, `<IP_R2>` pour `srv-b`) :

=== "English"

    1. **Disconnect card 1** of `srv-a` and `srv-b` (**VM**, **Removable Devices**, **Network Adapter**, **Disconnect**): they must now reach the Internet only through the router.
    2. On each client, replace the specific route with a **default route** via the router (with a low metric) and a DNS, in `/etc/netplan/99-gestion.yaml` (`<IP_R1>` for `srv-a`, `<IP_R2>` for `srv-b`):

```yaml
      routes:
        - to: default
          via: <IP_R1>
          metric: 50
      nameservers:
        addresses: [1.1.1.1]
```

```bash
sudo netplan apply
ip route
ping -c 2 <IP_R1>
ping -c 2 1.1.1.1
```

=== "Français"

    Le dernier `ping` **échoue** : le routeur transmet le paquet, mais l'adresse source privée de `srv-a` n'est pas routable au-delà du réseau de VMware, qui ne sait pas où renvoyer la réponse. Sur le **routeur**, créez la règle de **masquerade** sur la carte WAN avec `nftables` :

=== "English"

    The last `ping` **fails**: the router forwards the packet, but `srv-a`'s private source address is not routable beyond the VMware network, which does not know where to send the reply. On the **router**, create the **masquerade** rule on the WAN card with `nftables`:

```bash
sudo nft add table ip nat
sudo nft 'add chain ip nat postrouting { type nat hook postrouting priority 100 ; }'
sudo nft add rule ip nat postrouting oifname "<NIC_WAN>" masquerade
sudo nft list ruleset
```

=== "Français"

    Refaites `ping -c 2 1.1.1.1` depuis `srv-a`, puis testez la résolution de noms : `getent hosts ubuntu.com`. Pendant un `ping`, observez sur le routeur l'adresse source avant et après la traduction :

=== "English"

    Redo `ping -c 2 1.1.1.1` from `srv-a`, then test name resolution: `getent hosts ubuntu.com`. During a `ping`, observe on the router the source address before and after translation:

```bash
sudo tcpdump -ni <NIC_R1> icmp     # adresse source de srv-a
sudo tcpdump -ni <NIC_WAN> icmp    # adresse source du routeur (<IP_WAN>)
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Avant la règle, l'accès à Internet échoue ; après, il réussit depuis les deux clients
        - [ ] Sur la carte WAN, l'adresse source observée est celle du routeur

=== "English"

    !!! success "Checkpoint"
        - [ ] Before the rule, Internet access fails; after it, it succeeds from both clients
        - [ ] On the WAN card, the source address observed is the router's

### Étape 5 : Redirection de port (DNAT) / Step 5: Port forwarding (DNAT)

=== "Français"

    Sur `srv-b`, lancez un petit serveur web :

=== "English"

    On `srv-b`, start a small web server:

```bash
python3 -m http.server 8000
```

=== "Français"

    Sur le **routeur**, redirigez le port 8080 de la carte WAN vers ce serveur :

=== "English"

    On the **router**, redirect port 8080 of the WAN card to this server:

```bash
sudo nft 'add chain ip nat prerouting { type nat hook prerouting priority -100 ; }'
sudo nft add rule ip nat prerouting iifname "<NIC_WAN>" tcp dport 8080 dnat to <IP_B>:8000
sudo nft list ruleset
```

=== "Français"

    Depuis **PowerShell** sur l'hôte (qui joint le routeur par son adresse WAN sur VMnet8) :

=== "English"

    From **PowerShell** on the host (which reaches the router through its WAN address on VMnet8):

```powershell
curl.exe http://<IP_WAN>:8080
```

=== "Français"

    Résultat attendu : la page de listage du répertoire servie par `srv-b`. Arrêtez le serveur (++ctrl+c++).

    **Persistance (facultatif).** Les règles `nft` saisies à la main sont perdues au redémarrage. Pour les conserver : ajoutez-les à `/etc/nftables.conf` puis `sudo systemctl enable --now nftables`. Testez après un redémarrage du routeur.

    !!! success "Point de contrôle"
        - [ ] La requête vers `<IP_WAN>:8080` atteint le serveur de `srv-b`
        - [ ] Vous savez expliquer la différence entre la règle `masquerade` et la règle `dnat`

=== "English"

    Expected result: the directory listing page served by `srv-b`. Stop the server (++ctrl+c++).

    **Persistence (optional).** `nft` rules typed by hand are lost at reboot. To keep them: add them to `/etc/nftables.conf` then `sudo systemctl enable --now nftables`. Test after a router restart.

    !!! success "Checkpoint"
        - [ ] The request to `<IP_WAN>:8080` reaches `srv-b`'s server
        - [ ] You can explain the difference between the `masquerade` rule and the `dnat` rule

## Partie 3 : VLAN / Part 3: VLANs

### Étape 6 : Préparer le segment commun / Step 6: Prepare the shared segment

=== "Français"

    1. Éteignez les trois VM. Branchez la **carte 2** de `srv-a`, `srv-b` et du routeur sur le segment LAN **`reseau-trunk`**. Rallumez-les.
    2. Les VLAN de cette partie sont créés avec `ip` (non permanents). Sur chaque machine, retirez l'ancienne adresse de la carte 2 (`<NOM_INTERFACE>` sur les clients, `<NIC_R1>` sur le routeur) et activez la carte :

=== "English"

    1. Shut the three VMs down. Plug **card 2** of `srv-a`, `srv-b` and the router into the LAN segment **`reseau-trunk`**. Power them back on.
    2. The VLANs in this part are created with `ip` (not permanent). On each machine, remove the old address from card 2 (`<NOM_INTERFACE>` on clients, `<NIC_R1>` on the router) and bring the card up:

```bash
sudo ip addr flush dev <NOM_INTERFACE>
sudo ip link set <NOM_INTERFACE> up
sudo modprobe 8021q
lsmod | grep 8021q
```

=== "Français"

    !!! warning "Ne redémarrez pas pendant cette partie"
        Les fichiers Netplan des parties précédentes referaient ressortir les anciennes adresses au démarrage.

=== "English"

    !!! warning "Do not restart during this part"
        The Netplan files from the previous parts would bring the old addresses back at boot.

### Étape 7 : VLAN différents, même sous-réseau / Step 7: Different VLANs, same subnet

=== "Français"

    Sur `srv-a`, créez une interface dans le **VLAN 10** :

=== "English"

    On `srv-a`, create an interface in **VLAN 10**:

```bash
sudo ip link add link <NOM_INTERFACE> name <NOM_INTERFACE>.10 type vlan id 10
sudo ip addr add <IP_V10_A>/24 dev <NOM_INTERFACE>.10
sudo ip link set <NOM_INTERFACE>.10 up
ip -d link show <NOM_INTERFACE>.10
```

=== "Français"

    Sur `srv-b`, créez volontairement une interface dans le **VLAN 20** avec une adresse du **sous-réseau du VLAN 10** :

=== "English"

    On `srv-b`, deliberately create an interface in **VLAN 20** with an address from the **VLAN 10 subnet**:

```bash
sudo ip link add link <NOM_INTERFACE> name <NOM_INTERFACE>.20 type vlan id 20
sudo ip addr add <IP_V10_B>/24 dev <NOM_INTERFACE>.20
sudo ip link set <NOM_INTERFACE>.20 up
ping -c 2 <IP_V10_A>
```

=== "Français"

    Le `ping` **échoue** : même segment, même sous-réseau, mais VLAN différents. Corrigez : supprimez l'interface du VLAN 20 de `srv-b` et recréez-la dans le **VLAN 10** :

=== "English"

    The `ping` **fails**: same segment, same subnet, but different VLANs. Fix it: delete `srv-b`'s VLAN 20 interface and recreate it in **VLAN 10**:

```bash
sudo ip link del <NOM_INTERFACE>.20
sudo ip link add link <NOM_INTERFACE> name <NOM_INTERFACE>.10 type vlan id 10
sudo ip addr add <IP_V10_B>/24 dev <NOM_INTERFACE>.10
sudo ip link set <NOM_INTERFACE>.10 up
ping -c 2 <IP_V10_A>
```

=== "Français"

    Le `ping` **réussit**. Observez les **étiquettes 802.1Q** sur le routeur, sur l'interface physique, pendant un `ping` :

=== "English"

    The `ping` **succeeds**. Observe the **802.1Q tags** on the router, on the physical interface, during a `ping`:

```bash
sudo tcpdump -ni <NIC_R1> -e vlan
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Ping impossible entre VLAN 10 et VLAN 20 sur le même sous-réseau, possible dans le même VLAN
        - [ ] `ip -d link show` indique le type `vlan` et l'identifiant

=== "English"

    !!! success "Checkpoint"
        - [ ] No ping between VLAN 10 and VLAN 20 on the same subnet, ping works within the same VLAN
        - [ ] `ip -d link show` shows the `vlan` type and the identifier

### Étape 8 : Routage inter-VLAN / Step 8: Inter-VLAN routing

=== "Français"

    Remettez `srv-b` dans le **VLAN 20** avec son adresse du réseau VLAN 20, et créez sur le **routeur** une interface dans chaque VLAN, qui jouera le rôle de passerelle (« routeur sur un bras ») :

=== "English"

    Put `srv-b` back in **VLAN 20** with its VLAN 20 network address, and create on the **router** an interface in each VLAN, which will act as the gateway ("router on a stick"):

```bash
# srv-b : retirer l'interface VLAN 10, créer le VLAN 20
sudo ip link del <NOM_INTERFACE>.10
sudo ip link add link <NOM_INTERFACE> name <NOM_INTERFACE>.20 type vlan id 20
sudo ip addr add <IP_V20_B>/24 dev <NOM_INTERFACE>.20
sudo ip link set <NOM_INTERFACE>.20 up
sudo ip route add <RESEAU_V10>/24 via <IP_V20_R>

# srv-a : route vers le VLAN 20
sudo ip route add <RESEAU_V20>/24 via <IP_V10_R>

# routeur : une interface par VLAN sur la carte du segment commun
sudo ip addr flush dev <NIC_R1>
sudo ip link set <NIC_R1> up
sudo ip link add link <NIC_R1> name <NIC_R1>.10 type vlan id 10
sudo ip link add link <NIC_R1> name <NIC_R1>.20 type vlan id 20
sudo ip addr add <IP_V10_R>/24 dev <NIC_R1>.10
sudo ip addr add <IP_V20_R>/24 dev <NIC_R1>.20
sudo ip link set <NIC_R1>.10 up
sudo ip link set <NIC_R1>.20 up
sysctl net.ipv4.ip_forward
```

=== "Français"

    Testez depuis `srv-a` : `ping -c 2 <IP_V10_R>`, puis `ping -c 2 <IP_V20_B>` et `tracepath -n <IP_V20_B>`. Sur le routeur, capturez avec `sudo tcpdump -ni <NIC_R1> -e vlan` : le paquet arrive étiqueté VLAN 10 et repart étiqueté VLAN 20.

    !!! success "Point de contrôle"
        - [ ] `srv-a` (VLAN 10) joint `srv-b` (VLAN 20) à travers le routeur
        - [ ] La capture montre l'étiquette VLAN 10 à l'aller et l'étiquette VLAN 20 au passage vers `srv-b`

=== "English"

    Test from `srv-a`: `ping -c 2 <IP_V10_R>`, then `ping -c 2 <IP_V20_B>` and `tracepath -n <IP_V20_B>`. On the router, capture with `sudo tcpdump -ni <NIC_R1> -e vlan`: the packet arrives tagged VLAN 10 and leaves tagged VLAN 20.

    !!! success "Checkpoint"
        - [ ] `srv-a` (VLAN 10) reaches `srv-b` (VLAN 20) through the router
        - [ ] The capture shows the VLAN 10 tag on the way in and the VLAN 20 tag on the way to `srv-b`

### Étape 9 : Fiche de synthèse / Step 9: Summary sheet

=== "Français"

    Remplissez la fiche ci-dessous : c'est le **livrable** du lab. Joignez le plan d'adressage complété (étape 1) et les extraits de configuration (fichiers Netplan, règles `nft`).

    | Rubrique | Valeur / observation |
    |---|---|
    | Valeur de `ip_forward` avant / après activation | … / … |
    | Résultat du ping `srv-a` vers `srv-b` avant / après | … / … |
    | Sortie Internet des clients avant / après la règle masquerade | … / … |
    | Adresse source vue sur la carte WAN pendant un ping | … |
    | Règle de redirection (port WAN, destination) et résultat du test | … |
    | Ping entre VLAN 10 et VLAN 20 sur le même sous-réseau | … |
    | Étiquettes VLAN observées avec `tcpdump` | … |
    | Ping inter-VLAN à travers le routeur | … |

=== "English"

    Fill in the sheet below: this is the lab **deliverable**. Attach the completed addressing plan (step 1) and the configuration excerpts (Netplan files, `nft` rules).

    | Item | Value / observation |
    |---|---|
    | `ip_forward` value before / after enabling | … / … |
    | `srv-a` to `srv-b` ping result before / after | … / … |
    | Clients' Internet exit before / after the masquerade rule | … / … |
    | Source address seen on the WAN card during a ping | … |
    | Forwarding rule (WAN port, destination) and test result | … |
    | Ping between VLAN 10 and VLAN 20 on the same subnet | … |
    | VLAN tags observed with `tcpdump` | … |
    | Inter-VLAN ping through the router | … |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi le ping de `srv-a` vers `srv-b` échouait-il avant l'activation du transfert IP, alors que les routes étaient correctes ?"
        Un système Linux ne transmet pas par défaut les paquets qui ne lui sont pas destinés. Tant que `ip_forward` vaut 0, le routeur reçoit le paquet mais l'ignore : les routes des clients étaient bonnes, le maillon manquant était le routeur lui-même.

    ??? question "Quelle est la différence entre la règle `masquerade` et la règle `dnat` ?"
        Le `masquerade` modifie l'adresse **source** des paquets sortants (les clients privés sortent derrière l'adresse du routeur). Le `dnat` modifie l'adresse et le port de **destination** des paquets entrants (une connexion arrivant sur le routeur est redirigée vers un serveur privé).

    ??? question "Pourquoi un routeur est-il indispensable entre deux VLAN, même sur le même segment physique ?"
        Un VLAN est un domaine de diffusion distinct : les trames d'un VLAN ne sont pas transmises à un autre au niveau 2. Chaque VLAN étant un réseau IP séparé, seul un routeur (niveau 3), relié aux deux, peut acheminer les paquets de l'un à l'autre.

=== "English"

    ??? question "Why did the `srv-a` to `srv-b` ping fail before enabling IP forwarding, although the routes were correct?"
        A Linux system does not forward packets not addressed to it by default. As long as `ip_forward` is 0, the router receives the packet but ignores it: the clients' routes were right, the missing link was the router itself.

    ??? question "What is the difference between the `masquerade` rule and the `dnat` rule?"
        `masquerade` modifies the **source** address of outgoing packets (private clients go out behind the router's address). `dnat` modifies the **destination** address and port of incoming packets (a connection arriving at the router is redirected to a private server).

    ??? question "Why is a router essential between two VLANs, even on the same physical segment?"
        A VLAN is a distinct broadcast domain: frames of one VLAN are not forwarded to another at layer 2. As each VLAN is a separate IP network, only a router (layer 3), connected to both, can carry packets from one to the other.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | Les clients ne joignent pas le routeur | Noms d'interfaces ou segments LAN erronés (étape 1) ; indentation YAML ; `sudo netplan apply` ; `ip -br addr` |
    | Ping inter-réseaux impossible malgré `ip_forward=1` | Route manquante ou fausse sur un client (`ip route`) ; passerelle ne correspondant pas à l'adresse du routeur sur ce réseau ; pare-feu (`sudo ufw status`) |
    | Pas d'accès à Internet par le routeur | Carte WAN sans adresse (VMnet8) ; règle `masquerade` sur la mauvaise interface ; DNS manquant ; carte 1 des clients encore connectée (résultat faussé) |
    | `nft` : erreur de syntaxe | Guillemets simples autour des accolades pour la création de chaîne ; respecter les espaces ; `sudo nft list ruleset` pour voir l'état |
    | La redirection de port ne répond pas | Mauvaise adresse WAN ; serveur de `srv-b` arrêté ; `srv-b` n'a pas le routeur pour passerelle par défaut ; règle sur la mauvaise interface (`iifname`) |
    | `ping` échoue entre VLAN pourtant identiques | Module `8021q` absent (`sudo modprobe 8021q`) ; carte parente non activée ; l'hyperviseur filtre les trames étiquetées (à vérifier sur ce segment) |
    | Les VLAN disparaissent après redémarrage | Normal : créés avec `ip` ; pour les rendre permanents, utiliser la section `vlans:` de Netplan |
    | Une VM VirtualBox ou Hyper-V ne démarre plus | Un seul hyperviseur actif à la fois ; vérifier l'état de l'hyperviseur Windows (Lab 2.1) |

=== "English"

    | Symptom | Hints |
    |---|---|
    | Clients cannot reach the router | Wrong interface names or LAN segments (step 1); YAML indentation; `sudo netplan apply`; `ip -br addr` |
    | Inter-network ping impossible despite `ip_forward=1` | Missing or wrong route on a client (`ip route`); gateway not matching the router's address on that network; firewall (`sudo ufw status`) |
    | No Internet access through the router | WAN card without address (VMnet8); `masquerade` rule on the wrong interface; missing DNS; clients' card 1 still connected (skews the result) |
    | `nft`: syntax error | Single quotes around braces when creating a chain; respect spaces; `sudo nft list ruleset` to see the state |
    | Port forwarding does not answer | Wrong WAN address; `srv-b` server stopped; `srv-b` does not have the router as default gateway; rule on the wrong interface (`iifname`) |
    | `ping` fails between identical VLANs | `8021q` module missing (`sudo modprobe 8021q`); parent card not up; hypervisor filters tagged frames (to be checked on this segment) |
    | VLANs disappear after a restart | Normal: created with `ip`; to make them permanent, use Netplan's `vlans:` section |
    | A VirtualBox or Hyper-V VM no longer starts | Only one hypervisor active at a time; check the Windows hypervisor state (Lab 2.1) |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    Pour effacer la configuration du routeur sans redémarrer : `sudo nft flush ruleset`, `sudo rm /etc/sysctl.d/99-routage.conf` et `sudo sysctl -w net.ipv4.ip_forward=0`. Supprimez les fichiers `99-routeur.yaml` et les réglages de `99-gestion.yaml` ajoutés dans ce lab si vous réutilisez les VM, ou supprimez les VM clonées (**VM**, **Manage**, **Delete from Disk**) : les VM de référence ne sont pas modifiées. Reconnectez la carte 1 des clients si vous les conservez. Pour revenir à un état propre, restaurez le snapshot `etat-propre`.

=== "English"

    To clear the router's configuration without restarting: `sudo nft flush ruleset`, `sudo rm /etc/sysctl.d/99-routage.conf` and `sudo sysctl -w net.ipv4.ip_forward=0`. Delete the `99-routeur.yaml` files and the `99-gestion.yaml` settings added in this lab if you reuse the VMs, or delete the cloned VMs (**VM**, **Manage**, **Delete from Disk**): the reference VMs are not modified. Reconnect the clients' card 1 if you keep them. To return to a clean state, restore the `etat-propre` snapshot.

[Retour au chapitre 5 / Back to chapter 5](cours.md)