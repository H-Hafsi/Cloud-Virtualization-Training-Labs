# Lab 5.1 : Réseaux virtuels sous VirtualBox et VMware / Virtual Networks with VirtualBox and VMware

| | |
|---|---|
| **Durée / Duration** | 90 à 120 min (partie A VirtualBox : environ 45 min ; partie B VMware : environ 45 min) / 90 to 120 min (part A VirtualBox: about 45 min; part B VMware: about 45 min) |
| **Niveau / Level** | Guidé / Guided |
| **Prérequis / Prerequisites** | [Chapitre 5 : cours](cours.md) ; [Lab 3.1](../ch03-virtualbox-et-vmware-workstation/lab-3-1-virtualbox-une-vm-ubuntu-server.md) et [Lab 3.2](../ch03-virtualbox-et-vmware-workstation/lab-3-2-vmware-workstation-meme-vm-import-et-export.md) réalisés (VM `srv1` sous VirtualBox, `srv1-vmware` sous VMware, snapshots `etat-propre`) |
| **Ressources minimales / Minimum resources** | **VM** : 2 clones liés de la VM de référence, 1 vCPU et 1 Go de RAM chacun, disques de 25 Go dynamiques (peu d'espace réel). **Hôte** : au moins 4 Go de RAM libres pour deux VM allumées, 10 Go d'espace disque libre. L'hyperviseur Windows doit être **désactivé**. / **VMs**: 2 linked clones of the reference VM, 1 vCPU and 1 GB RAM each, 25 GB dynamic disks (little real space). **Host**: at least 4 GB of free RAM for two running VMs, 10 GB of free disk space. The Windows hypervisor must be **off**. |
| **Acquis visés / Learning outcomes** | AA2 |

=== "Français"

    !!! abstract "Objectifs"
        - Mettre en place les réseaux hôte seul, isolé et NAT sous VirtualBox, puis sous VMware Workstation
        - Vérifier par des tests ce que chaque mode autorise (VM ↔ VM, VM ↔ hôte, VM → Internet, extérieur → VM)
        - Configurer une redirection de port NAT pour atteindre une VM depuis l'hôte
        - Documenter un plan d'adressage cohérent

=== "English"

    !!! abstract "Objectives"
        - Set up host-only, isolated and NAT networks under VirtualBox, then under VMware Workstation
        - Verify through tests what each mode allows (VM ↔ VM, VM ↔ host, VM → Internet, outside → VM)
        - Configure a NAT port forwarding rule to reach a VM from the host
        - Document a consistent addressing plan

## Contexte et schéma / Context and diagram

=== "Français"

    Vous allez observer, par l'expérience, le tableau du cours (section 2). Deux VM, `srv-a` et `srv-b`, sont branchées successivement sur trois types de réseaux. Pour chaque réseau, vous relèverez ce qui fonctionne et ce qui ne fonctionne pas.

    ```mermaid
    flowchart LR
      H["Poste hôte"]
      A["srv-a"]
      B["srv-b"]
      A --- R1["Réseau hôte seul"]
      B --- R1
      H --- R1
      A --- R2["Réseau isolé"]
      B --- R2
      A --- R3["Réseau NAT"]
      B --- R3
      R3 --- I["Internet"]
    ```

    Chaque VM garde une carte en **NAT** (carte 1, pour les mises à jour) et reçoit sur la **carte 2** le réseau à tester. Les adresses et les noms de réseaux suivent le plan de l'annexe [Environnement](../../annexes/environnement.md) : elles sont notées `<IP_A1>`, `<IP_A2>` (réseau hôte seul), `<IP_B1>`, `<IP_B2>` (réseau isolé), `<IP_HOTE>` (adresse de l'hôte sur le réseau hôte seul).

    !!! warning "Allumer les VM une à une pendant la préparation"
        Les deux clones héritent de la même adresse, du même nom d'hôte et des mêmes clés SSH que la VM d'origine. Préparez-les **l'une après l'autre**, avant de les allumer ensemble.

=== "English"

    You will observe, through experience, the lesson's table (section 2). Two VMs, `srv-a` and `srv-b`, are plugged successively into three types of networks. For each network, you will note what works and what does not.

    ```mermaid
    flowchart LR
      H["Host computer"]
      A["srv-a"]
      B["srv-b"]
      A --- R1["Host-only network"]
      B --- R1
      H --- R1
      A --- R2["Isolated network"]
      B --- R2
      A --- R3["NAT network"]
      B --- R3
      R3 --- I["Internet"]
    ```

    Each VM keeps a **NAT** card (card 1, for updates) and gets the network under test on **card 2**. Addresses and network names follow the plan in the [Environment](../../annexes/environnement.md) annex: they are written `<IP_A1>`, `<IP_A2>` (host-only network), `<IP_B1>`, `<IP_B2>` (isolated network), `<IP_HOTE>` (the host's address on the host-only network).

    !!! warning "Power VMs on one at a time during preparation"
        Both clones inherit the same address, hostname and SSH keys as the original VM. Prepare them **one after the other**, before powering them on together.

## Étapes / Steps

### Étape 1 : Plan d'adressage / Step 1: Addressing plan

=== "Français"

    Choisissez dans l'annexe Environnement trois réseaux **libres** et sans chevauchement, puis remplissez le tableau (il fait partie du livrable) :

    | Réseau | Mode | Plage (adresse/préfixe) | `srv-a` | `srv-b` | Adresse de l'hôte |
    |---|---|---|---|---|---|
    | A | Hôte seul | … | `<IP_A1>` | `<IP_A2>` | `<IP_HOTE>` |
    | B | Isolé (réseau interne / segment LAN) | … | `<IP_B1>` | `<IP_B2>` | sans objet |
    | C | NAT | … (géré par l'hyperviseur) | par DHCP | par DHCP | sans objet |

=== "English"

    Choose from the Environment annex three **free**, non-overlapping networks, then fill in the table (it is part of the deliverable):

    | Network | Mode | Range (address/prefix) | `srv-a` | `srv-b` | Host address |
    |---|---|---|---|---|---|
    | A | Host-only | … | `<IP_A1>` | `<IP_A2>` | `<IP_HOTE>` |
    | B | Isolated (internal network / LAN segment) | … | `<IP_B1>` | `<IP_B2>` | not applicable |
    | C | NAT | … (managed by the hypervisor) | by DHCP | by DHCP | not applicable |

## Partie A : VirtualBox / Part A: VirtualBox

### Étape 2 : Cloner la VM de référence / Step 2: Clone the reference VM

=== "Français"

    Dans VirtualBox, sélectionnez la VM `srv1` (éteinte), faites **Machine**, **Clone** et créez deux clones : `srv-a` puis `srv-b`.

    | Option de l'assistant | Valeur |
    |---|---|
    | MAC Address Policy | **Generate new MAC addresses for all network adapters** |
    | Clone type | **Linked clone** (utilise le snapshot `etat-propre` ; économise de l'espace) |

    Pour chaque clone, ouvrez **Settings**, **System** et réglez **1 Go de mémoire** et **1 processeur**.

    Démarrez **`srv-a` seul**, ouvrez sa console et changez son identité (le nom d'hôte, l'identifiant de machine, les clés SSH), puis éteignez-la. Recommencez avec `srv-b` :

=== "English"

    In VirtualBox, select the `srv1` VM (powered off), choose **Machine**, **Clone** and create two clones: `srv-a` then `srv-b`.

    | Wizard option | Value |
    |---|---|
    | MAC Address Policy | **Generate new MAC addresses for all network adapters** |
    | Clone type | **Linked clone** (uses the `etat-propre` snapshot; saves space) |

    For each clone, open **Settings**, **System** and set **1 GB of memory** and **1 processor**.

    Start **`srv-a` alone**, open its console and change its identity (hostname, machine identifier, SSH keys), then shut it down. Repeat with `srv-b`:

```bash
sudo hostnamectl set-hostname srv-a        # srv-b pour l'autre clone
sudo truncate -s 0 /etc/machine-id
sudo rm -f /var/lib/dbus/machine-id
sudo ln -s /etc/machine-id /var/lib/dbus/machine-id
sudo rm -f /etc/ssh/ssh_host_*
sudo dpkg-reconfigure openssh-server
sudo poweroff
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Deux clones `srv-a` et `srv-b`, adresses MAC régénérées, nom d'hôte distinct
        - [ ] Les deux VM sont éteintes

=== "English"

    !!! success "Checkpoint"
        - [ ] Two clones `srv-a` and `srv-b`, MAC addresses regenerated, distinct hostname
        - [ ] Both VMs are powered off

### Étape 3 : Scénario A, réseau hôte seul / Step 3: Scenario A, host-only network

=== "Français"

    1. Dans **Tools**, **Network**, vérifiez que le réseau hôte seul utilise la plage du réseau A (voir le Lab 3.1) et notez l'adresse de l'hôte (`<IP_HOTE>`).
    2. Pour `srv-a` et `srv-b` : **Settings**, **Network**, **Adapter 1** reste en **NAT** ; **Adapter 2** : *Attached to* **Host-only Adapter**, réseau A.
    3. Démarrez les deux VM. Dans chacune, remplacez dans `/etc/netplan/99-gestion.yaml` l'adresse par `<IP_A1>` (`srv-a`) ou `<IP_A2>` (`srv-b`), puis appliquez :

=== "English"

    1. In **Tools**, **Network**, check that the host-only network uses the network A range (see Lab 3.1) and note the host's address (`<IP_HOTE>`).
    2. For `srv-a` and `srv-b`: **Settings**, **Network**, **Adapter 1** stays on **NAT**; **Adapter 2**: *Attached to* **Host-only Adapter**, network A.
    3. Start both VMs. In each, replace the address in `/etc/netplan/99-gestion.yaml` with `<IP_A1>` (`srv-a`) or `<IP_A2>` (`srv-b`), then apply:

```bash
sudo nano /etc/netplan/99-gestion.yaml
sudo netplan apply
ip -br addr
```

=== "Français"

    **Tests.** Depuis `srv-a` (remplacez `<NOM_INTERFACE>` par le nom de la **carte 2**, souvent `enp0s8`) :

=== "English"

    **Tests.** From `srv-a` (replace `<NOM_INTERFACE>` with the name of **card 2**, often `enp0s8`):

```bash
ping -c 2 <IP_A2>
ping -c 2 <IP_HOTE>
ping -c 2 -I <NOM_INTERFACE> 1.1.1.1
```

=== "Français"

    Depuis **PowerShell** sur l'hôte : `ping <IP_A1>`. Notez chaque résultat (✓ ou ✗) dans la matrice de l'étape 8 : VM ↔ VM, VM ↔ hôte, VM → Internet (la troisième commande force la sortie par la carte 2), hôte → VM.

    !!! success "Point de contrôle"
        - [ ] Les quatre tests sont réalisés et notés

=== "English"

    From **PowerShell** on the host: `ping <IP_A1>`. Write down each result (✓ or ✗) in the step 8 matrix: VM ↔ VM, VM ↔ host, VM → Internet (the third command forces output through card 2), host → VM.

    !!! success "Checkpoint"
        - [ ] The four tests are done and noted

### Étape 4 : Scénario B, réseau interne isolé / Step 4: Scenario B, isolated internal network

=== "Français"

    1. Éteignez les VM. Pour chacune : **Settings**, **Network**, **Adapter 2** : *Attached to* **Internal Network**, nom `reseau-isole`.
    2. Redémarrez-les et réglez dans `/etc/netplan/99-gestion.yaml` les adresses du réseau B : `<IP_B1>` pour `srv-a`, `<IP_B2>` pour `srv-b`. Appliquez (`sudo netplan apply`).
    3. Refaites les tests de l'étape 3 : ping de l'autre VM (`<IP_B2>` depuis `srv-a`), de l'hôte, d'une adresse Internet en forçant la carte 2, et depuis l'hôte (`ping <IP_B1>`).

    !!! success "Point de contrôle"
        - [ ] Les VM communiquent entre elles mais ni l'hôte ni Internet ne répondent par ce réseau

=== "English"

    1. Shut the VMs down. For each: **Settings**, **Network**, **Adapter 2**: *Attached to* **Internal Network**, name `reseau-isole`.
    2. Restart them and set the network B addresses in `/etc/netplan/99-gestion.yaml`: `<IP_B1>` for `srv-a`, `<IP_B2>` for `srv-b`. Apply (`sudo netplan apply`).
    3. Redo the step 3 tests: ping the other VM (`<IP_B2>` from `srv-a`), the host, an Internet address forcing card 2, and from the host (`ping <IP_B1>`).

    !!! success "Checkpoint"
        - [ ] The VMs communicate with each other but neither the host nor the Internet answers over this network

### Étape 5 : Scénario C, NAT Network et redirection de port / Step 5: Scenario C, NAT Network and port forwarding

=== "Français"

    1. Dans **Tools**, **Network**, onglet **NAT Networks**, créez un réseau (par exemple `reseau-nat`) avec la plage du réseau C, **DHCP activé**.
    2. Dans ce réseau, ouvrez **Port Forwarding** et ajoutez une règle : *Protocol* TCP, *Host IP* `127.0.0.1`, *Host Port* `2222`, *Guest IP* l'adresse de `srv-a` (voir ci-dessous), *Guest Port* `22`.
    3. Éteignez les VM. Pour chacune : **Adapter 2** : *Attached to* **NAT Network**, `reseau-nat`.
    4. Dans `/etc/netplan/99-gestion.yaml`, passez la carte 2 en DHCP :

=== "English"

    1. In **Tools**, **Network**, **NAT Networks** tab, create a network (for example `reseau-nat`) with the network C range, **DHCP enabled**.
    2. In this network, open **Port Forwarding** and add a rule: *Protocol* TCP, *Host IP* `127.0.0.1`, *Host Port* `2222`, *Guest IP* the address of `srv-a` (see below), *Guest Port* `22`.
    3. Shut the VMs down. For each: **Adapter 2**: *Attached to* **NAT Network**, `reseau-nat`.
    4. In `/etc/netplan/99-gestion.yaml`, switch card 2 to DHCP:

```yaml
network:
  version: 2
  ethernets:
    <NOM_INTERFACE>:
      dhcp4: true
```

```bash
sudo netplan apply
ip -br addr
ip route
```

=== "Français"

    Relevez l'adresse obtenue par `srv-a` : reportez-la comme *Guest IP* de la règle de redirection (modifiez la règle si besoin). Refaites les tests de ping (VM ↔ VM, VM ↔ hôte, Internet en forçant la carte 2). Puis, depuis **PowerShell** sur l'hôte, testez la redirection :

=== "English"

    Note the address obtained by `srv-a`: use it as the *Guest IP* of the forwarding rule (edit the rule if needed). Redo the ping tests (VM ↔ VM, VM ↔ host, Internet forcing card 2). Then, from **PowerShell** on the host, test the forwarding:

```powershell
ssh -p 2222 utilisateur@127.0.0.1
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Les VM obtiennent une adresse du réseau C par DHCP et sortent vers Internet
        - [ ] La connexion SSH vers `127.0.0.1:2222` atteint `srv-a`

=== "English"

    !!! success "Checkpoint"
        - [ ] The VMs get a network C address by DHCP and reach the Internet
        - [ ] The SSH connection to `127.0.0.1:2222` reaches `srv-a`

## Partie B : VMware Workstation / Part B: VMware Workstation

### Étape 6 : Cloner la VM sous VMware et préparer les réseaux / Step 6: Clone the VM under VMware and prepare networks

=== "Français"

    1. Éteignez toutes les VM VirtualBox. Dans VMware, sélectionnez `srv1-vmware` (éteinte), faites **VM**, **Manage**, **Clone**, choisissez le snapshot `etat-propre`, puis **Create a linked clone**. Créez `srv-a-vmware` et `srv-b-vmware`. Réglez 1 Go de mémoire et 1 processeur pour chacun.
    2. Préparez l'identité de chaque clone (une VM à la fois), avec les mêmes commandes qu'à l'étape 2 (noms `srv-a` et `srv-b`).
    3. Dans **Edit**, **Virtual Network Editor** (**Change Settings**), vérifiez : **VMnet1** (hôte seul) sur la plage du réseau A ; **VMnet8** (NAT) sur la plage du réseau C.

    !!! success "Point de contrôle"
        - [ ] Deux clones préparés, éteints
        - [ ] VMnet1 et VMnet8 configurés selon le plan

=== "English"

    1. Shut down all VirtualBox VMs. In VMware, select `srv1-vmware` (powered off), choose **VM**, **Manage**, **Clone**, pick the `etat-propre` snapshot, then **Create a linked clone**. Create `srv-a-vmware` and `srv-b-vmware`. Set 1 GB of memory and 1 processor for each.
    2. Prepare each clone's identity (one VM at a time), with the same commands as in step 2 (names `srv-a` and `srv-b`).
    3. In **Edit**, **Virtual Network Editor** (**Change Settings**), check: **VMnet1** (host-only) on the network A range; **VMnet8** (NAT) on the network C range.

    !!! success "Checkpoint"
        - [ ] Two clones prepared, powered off
        - [ ] VMnet1 and VMnet8 configured according to the plan

### Étape 7 : Les trois scénarios sous VMware / Step 7: The three scenarios under VMware

=== "Français"

    Pour chaque scénario, éteignez les VM, réglez le **Network Adapter 2** (**Edit virtual machine settings**), adaptez `/etc/netplan/99-gestion.yaml` (le **nom de la carte 2** peut différer de VirtualBox : relevez-le avec `ip -br link`) et refaites les tests de l'étape 3.

    | Scénario | Réglage de la carte 2 | Adresses | Tests particuliers |
    |---|---|---|---|
    | **A : hôte seul** | **Custom : VMnet1 (Host-only)** | `<IP_A1>` et `<IP_A2>` statiques | Ping de l'hôte (`<IP_HOTE>` : adresse de l'hôte sur VMnet1, vue par `ipconfig`) |
    | **B : isolé** | **LAN Segment** : cliquez sur **LAN Segments…**, **Add**, nom `reseau-isole`, puis sélectionnez-le | `<IP_B1>` et `<IP_B2>` statiques | L'hôte ne répond pas |
    | **C : NAT** | **Custom : VMnet8 (NAT)** | DHCP de VMware (`dhcp4: true`) | Redirection de port : voir ci-dessous |

    **Redirection de port sous VMware.** Dans le **Virtual Network Editor**, sélectionnez **VMnet8**, cliquez sur **NAT Settings…**, **Add** dans *Port Forwarding* : *Host port* `2222`, *Type* TCP, *Virtual machine IP address* l'adresse de `srv-a-vmware`, *Virtual machine port* `22`. Testez depuis **PowerShell** : `ssh -p 2222 utilisateur@127.0.0.1`.

    !!! success "Point de contrôle"
        - [ ] Les trois scénarios sont testés sous VMware et les résultats notés
        - [ ] La redirection de port vers `srv-a-vmware` fonctionne

=== "English"

    For each scenario, shut the VMs down, set **Network Adapter 2** (**Edit virtual machine settings**), adapt `/etc/netplan/99-gestion.yaml` (the **name of card 2** may differ from VirtualBox: note it with `ip -br link`) and redo the step 3 tests.

    | Scenario | Card 2 setting | Addresses | Special tests |
    |---|---|---|---|
    | **A: host-only** | **Custom: VMnet1 (Host-only)** | `<IP_A1>` and `<IP_A2>` static | Ping the host (`<IP_HOTE>`: the host's address on VMnet1, seen with `ipconfig`) |
    | **B: isolated** | **LAN Segment**: click **LAN Segments…**, **Add**, name `reseau-isole`, then select it | `<IP_B1>` and `<IP_B2>` static | The host does not answer |
    | **C: NAT** | **Custom: VMnet8 (NAT)** | VMware DHCP (`dhcp4: true`) | Port forwarding: see below |

    **Port forwarding under VMware.** In the **Virtual Network Editor**, select **VMnet8**, click **NAT Settings…**, **Add** under *Port Forwarding*: *Host port* `2222`, *Type* TCP, *Virtual machine IP address* the address of `srv-a-vmware`, *Virtual machine port* `22`. Test from **PowerShell**: `ssh -p 2222 user@127.0.0.1`.

    !!! success "Checkpoint"
        - [ ] The three scenarios are tested under VMware and the results noted
        - [ ] Port forwarding to `srv-a-vmware` works

### Étape 8 : Matrice de résultats / Step 8: Results matrix

=== "Français"

    Remplissez la matrice avec ✓ (ça répond) ou ✗ (ça ne répond pas) : c'est le **livrable** du lab. Comparez-la au tableau du cours : tout écart doit être expliqué.

    | Scénario | VM ↔ VM | VM ↔ hôte | VM → Internet (carte 2) | Hôte → VM | Redirection de port |
    |---|---|---|---|---|---|
    | VirtualBox, hôte seul | … | … | … | … | — |
    | VirtualBox, réseau interne | … | … | … | … | — |
    | VirtualBox, NAT Network | … | … | … | … | … |
    | VMware, hôte seul (VMnet1) | … | … | … | … | — |
    | VMware, segment LAN | … | … | … | … | — |
    | VMware, NAT (VMnet8) | … | … | … | … | … |

=== "English"

    Fill in the matrix with ✓ (it answers) or ✗ (it does not answer): this is the lab **deliverable**. Compare it with the lesson's table: any difference must be explained.

    | Scenario | VM ↔ VM | VM ↔ host | VM → Internet (card 2) | Host → VM | Port forwarding |
    |---|---|---|---|---|---|
    | VirtualBox, host-only | … | … | … | … | — |
    | VirtualBox, internal network | … | … | … | … | — |
    | VirtualBox, NAT Network | … | … | … | … | … |
    | VMware, host-only (VMnet1) | … | … | … | … | — |
    | VMware, LAN segment | … | … | … | … | — |
    | VMware, NAT (VMnet8) | … | … | … | … | … |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi forcer la sortie par la carte 2 avec `ping -I` pour tester l'accès à Internet ?"
        Les VM ont aussi la carte 1 en NAT, qui leur donne toujours accès à Internet. Sans forcer la carte 2, le test réussirait à cause de la carte 1 et ne dirait rien sur le mode réseau étudié.

    ??? question "Pourquoi une VM sur un réseau NAT n'est-elle pas joignable depuis l'extérieur sans règle de redirection de port ?"
        Le NAT remplace l'adresse source des paquets sortants et ne mémorise que les échanges initiés depuis l'intérieur. Un paquet arrivant de l'extérieur sans échange préalable n'a pas de destination connue : la redirection de port précise vers quelle VM l'envoyer.

    ??? question "Pourquoi les deux réseaux (A et B) ne doivent-ils pas avoir la même plage d'adresses ?"
        Avec des plages identiques, une VM branchée sur les deux réseaux aurait deux interfaces dans le même sous-réseau, et le routage ne saurait pas choisir celle à utiliser : les tests deviendraient ambigus.

=== "English"

    ??? question "Why force output through card 2 with `ping -I` to test Internet access?"
        The VMs also have card 1 on NAT, which always gives them Internet access. Without forcing card 2, the test would succeed because of card 1 and would say nothing about the network mode under study.

    ??? question "Why is a VM on a NAT network not reachable from outside without a port forwarding rule?"
        NAT replaces the source address of outgoing packets and only remembers exchanges initiated from inside. A packet arriving from outside with no prior exchange has no known destination: port forwarding specifies which VM to send it to.

    ??? question "Why must networks A and B not have the same address range?"
        With identical ranges, a VM plugged into both networks would have two interfaces in the same subnet, and routing could not choose which one to use: the tests would become ambiguous.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | Deux VM ont la même adresse ou le même nom | Une VM clonée n'a pas été préparée (étape 2) ; les préparer une par une |
    | La carte 2 n'a pas d'adresse | Nom d'interface différent dans le fichier Netplan (`ip -br link`) ; indentation YAML ; `sudo netplan apply` |
    | `ping` de l'hôte vers la VM échoue (hôte seul) | Adresse de la VM hors de la plage du réseau hôte seul ; carte de l'hôte désactivée (`VMware Network Adapter VMnet1`, VirtualBox Host-Only) ; pare-feu Windows |
    | La VM n'obtient pas d'adresse en DHCP (NAT) | NAT Network sans DHCP activé ; service DHCP de VMnet8 arrêté ; mauvais réseau sélectionné |
    | `ping -I` renvoie « Network is unreachable » | Attendu sur un réseau sans passerelle (hôte seul, isolé) : c'est le résultat à noter |
    | La redirection de port ne répond pas | Mauvaise adresse de la VM dans la règle ; le serveur SSH ne tourne pas dans la VM ; port 2222 déjà utilisé sur l'hôte |
    | Les VM VirtualBox ne démarrent pas pendant que VMware tourne | Éteindre les VM de l'autre hyperviseur ; vérifier que l'hyperviseur Windows est désactivé (Lab 2.1) |

=== "English"

    | Symptom | Hints |
    |---|---|
    | Two VMs have the same address or name | A cloned VM was not prepared (step 2); prepare them one by one |
    | Card 2 has no address | Different interface name in the Netplan file (`ip -br link`); YAML indentation; `sudo netplan apply` |
    | `ping` from the host to the VM fails (host-only) | VM address outside the host-only network range; host adapter disabled (`VMware Network Adapter VMnet1`, VirtualBox Host-Only); Windows firewall |
    | The VM gets no DHCP address (NAT) | NAT Network without DHCP enabled; VMnet8 DHCP service stopped; wrong network selected |
    | `ping -I` returns "Network is unreachable" | Expected on a network without a gateway (host-only, isolated): this is the result to note |
    | Port forwarding does not answer | Wrong VM address in the rule; SSH server not running in the VM; port 2222 already in use on the host |
    | VirtualBox VMs do not start while VMware is running | Shut down the other hypervisor's VMs; check that the Windows hypervisor is off (Lab 2.1) |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    Éteignez toutes les VM. Les clones liés sont légers : conservez-les pour le [Lab 5.3](lab-5-3-routage-nat-et-vlan-sous-linux.md) ou supprimez-les (**Remove**, **Delete all files**). Les VM de référence `srv1` et `srv1-vmware` ne sont pas modifiées. Supprimez, si vous ne les réutilisez pas, la redirection de port, le NAT Network et le segment LAN créés.

=== "English"

    Shut down all VMs. Linked clones are light: keep them for [Lab 5.3](lab-5-3-routage-nat-et-vlan-sous-linux.md) or delete them (**Remove**, **Delete all files**). The `srv1` and `srv1-vmware` reference VMs are not modified. If you do not reuse them, delete the port forwarding rule, the NAT Network and the LAN segment you created.

[Retour au chapitre 5 / Back to chapter 5](cours.md)