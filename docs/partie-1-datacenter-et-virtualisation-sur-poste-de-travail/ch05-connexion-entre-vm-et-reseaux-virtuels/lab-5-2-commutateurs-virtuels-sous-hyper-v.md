# Lab 5.2 : Commutateurs virtuels sous Hyper-V / Virtual Switches with Hyper-V

| | |
|---|---|
| **Durée / Duration** | 90 à 120 min / 90 to 120 min |
| **Niveau / Level** | Guidé / Guided |
| **Prérequis / Prerequisites** | [Chapitre 5 : cours](cours.md) ; [Lab 4.1](../ch04-hyper-v/lab-4-1-activer-hyper-v-et-creer-une-vm.md) et [Lab 4.2](../ch04-hyper-v/lab-4-2-checkpoints-disques-et-export.md) réalisés (VM `srv1-hv` éteinte, sans checkpoint ; commutateur `gestion-interne`) ; [Lab 5.1](lab-5-1-reseaux-virtuels-sous-virtualbox-et-vmware.md) conseillé |
| **Ressources minimales / Minimum resources** | **VM** : 2 VM (`srv-a-hv`, `srv-b-hv`), 1 vCPU et 1 Go de RAM chacune, disques différentiels (peu d'espace réel). **Hôte** : Windows Pro, Enterprise ou Éducation avec Hyper-V actif, au moins 4 Go de RAM libres, 15 Go d'espace disque libre. L'hyperviseur Windows doit être **activé**. / **VMs**: 2 VMs (`srv-a-hv`, `srv-b-hv`), 1 vCPU and 1 GB RAM each, differencing disks (little real space). **Host**: Windows Pro, Enterprise or Education with Hyper-V on, at least 4 GB of free RAM, 15 GB of free disk space. The Windows hypervisor must be **on**. |
| **Acquis visés / Learning outcomes** | AA2 |

=== "Français"

    !!! abstract "Objectifs"
        - Créer des commutateurs virtuels interne, privé et externe, et un réseau NAT personnalisé
        - Vérifier par des tests ce que chaque type de commutateur autorise
        - Mettre en place une redirection de port sur un NAT Hyper-V
        - Isoler deux VM d'un même commutateur avec des VLAN
        - Comparer avec les modes de VirtualBox et de VMware (Lab 5.1)

=== "English"

    !!! abstract "Objectives"
        - Create internal, private and external virtual switches, and a custom NAT network
        - Verify through tests what each switch type allows
        - Set up port forwarding on a Hyper-V NAT
        - Isolate two VMs on the same switch with VLANs
        - Compare with the VirtualBox and VMware modes (Lab 5.1)

## Contexte et schéma / Context and diagram

=== "Français"

    Vous reprenez l'expérience du Lab 5.1 avec Hyper-V. Deux VM, `srv-a-hv` et `srv-b-hv`, sont créées à partir d'un disque de base en lecture seule (disques **différentiels**, voir le chapitre 4). Chacune a deux cartes : la carte 1 reste sur le **Default Switch** (Internet, mises à jour), la carte 2 est branchée au commutateur testé.

    ```mermaid
    flowchart LR
      H["Poste hôte"] --- I["Commutateur interne<br/>gestion-interne"]
      A["srv-a-hv"] --- I
      B["srv-b-hv"] --- I
      A --- P["Commutateur privé<br/>sw-prive (+ VLAN)"]
      B --- P
      A --- N["Commutateur interne + NAT<br/>sw-nat"]
      B --- N
      H --- N
      N --- X["Internet"]
    ```

    Les adresses et plages suivent le plan de l'annexe [Environnement](../../annexes/environnement.md) : `<IP_HOTE>` (réseau du commutateur `gestion-interne`, déjà configuré au Lab 4.1), `<IP_A1>` / `<IP_A2>` (réseau A), `<IP_B1>` / `<IP_B2>` (réseau B, privé), `<RESEAU_C>`, `<IP_PASSERELLE_C>`, `<IP_C1>` / `<IP_C2>` (réseau C, NAT). Les commandes se lancent dans **PowerShell en administrateur**.

    !!! warning "Ne démarrez pas `srv1-hv` pendant ce lab"
        Les disques des deux VM dépendent d'une copie de son disque. Ne modifiez jamais cette copie de base : elle est mise en lecture seule à l'étape 2.

=== "English"

    You repeat the Lab 5.1 experiment with Hyper-V. Two VMs, `srv-a-hv` and `srv-b-hv`, are created from a read-only base disk (**differencing** disks, see chapter 4). Each has two cards: card 1 stays on the **Default Switch** (Internet, updates), card 2 is plugged into the switch under test.

    ```mermaid
    flowchart LR
      H["Host computer"] --- I["Internal switch<br/>gestion-interne"]
      A["srv-a-hv"] --- I
      B["srv-b-hv"] --- I
      A --- P["Private switch<br/>sw-prive (+ VLAN)"]
      B --- P
      A --- N["Internal switch + NAT<br/>sw-nat"]
      B --- N
      H --- N
      N --- X["Internet"]
    ```

    Addresses and ranges follow the plan of the [Environment](../../annexes/environnement.md) annex: `<IP_HOTE>` (network of the `gestion-interne` switch, already configured in Lab 4.1), `<IP_A1>` / `<IP_A2>` (network A), `<IP_B1>` / `<IP_B2>` (network B, private), `<RESEAU_C>`, `<IP_PASSERELLE_C>`, `<IP_C1>` / `<IP_C2>` (network C, NAT). Commands are run in **PowerShell as administrator**.

    !!! warning "Do not start `srv1-hv` during this lab"
        The two VMs' disks depend on a copy of its disk. Never modify this base copy: it is set to read-only in step 2.

## Étapes / Steps

### Étape 1 : Vérifier l'état de départ / Step 1: Check the starting state

```powershell
(Get-CimInstance Win32_ComputerSystem).HypervisorPresent
Get-VMSwitch | Select-Object Name, SwitchType
Get-VM -Name "srv1-hv" | Select-Object Name, State
Get-VMCheckpoint -VMName "srv1-hv"
```

=== "Français"

    Résultat attendu : `HypervisorPresent` à `True` (sinon, réactivez l'hyperviseur comme au [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md)) ; les commutateurs `Default Switch` et `gestion-interne` ; `srv1-hv` à l'état `Off` ; aucun checkpoint.

    Complétez le plan d'adressage (il fait partie du livrable) :

    | Réseau | Commutateur | Plage | `srv-a-hv` | `srv-b-hv` | Hôte |
    |---|---|---|---|---|---|
    | A | `gestion-interne` (interne) | … | `<IP_A1>` | `<IP_A2>` | `<IP_HOTE>` |
    | B | `sw-prive` (privé) | … | `<IP_B1>` | `<IP_B2>` | sans objet |
    | C | `sw-nat` (interne + NAT) | `<RESEAU_C>` | `<IP_C1>` | `<IP_C2>` | `<IP_PASSERELLE_C>` |

=== "English"

    Expected result: `HypervisorPresent` is `True` (otherwise, turn the hypervisor back on as in [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md)); the `Default Switch` and `gestion-interne` switches; `srv1-hv` in the `Off` state; no checkpoint.

    Complete the addressing plan (it is part of the deliverable):

    | Network | Switch | Range | `srv-a-hv` | `srv-b-hv` | Host |
    |---|---|---|---|---|---|
    | A | `gestion-interne` (internal) | … | `<IP_A1>` | `<IP_A2>` | `<IP_HOTE>` |
    | B | `sw-prive` (private) | … | `<IP_B1>` | `<IP_B2>` | not applicable |
    | C | `sw-nat` (internal + NAT) | `<RESEAU_C>` | `<IP_C1>` | `<IP_C2>` | `<IP_PASSERELLE_C>` |

### Étape 2 : Créer le disque de base et les deux VM / Step 2: Create the base disk and the two VMs

=== "Français"

    Copiez le disque de `srv1-hv` en un disque de base **en lecture seule**, puis créez deux VM dont les disques sont des **différentiels** de ce disque. Remplacez `<DOSSIER_VM>` par votre dossier des VM :

=== "English"

    Copy `srv1-hv`'s disk into a **read-only** base disk, then create two VMs whose disks are **differencing** disks of that disk. Replace `<DOSSIER_VM>` with your VM folder:

```powershell
$base = "<DOSSIER_VM>\base\base-ubuntu.vhdx"
New-Item -ItemType Directory -Path "<DOSSIER_VM>\base" -Force | Out-Null
Copy-Item "<DOSSIER_VM>\srv1-hv\srv1-hv.vhdx" $base
Set-ItemProperty -Path $base -Name IsReadOnly -Value $true

foreach ($n in "srv-a-hv","srv-b-hv") {
  New-Item -ItemType Directory -Path "<DOSSIER_VM>\$n" -Force | Out-Null
  New-VHD -Path "<DOSSIER_VM>\$n\$n.vhdx" -ParentPath $base -Differencing | Out-Null
  New-VM -Name $n -Generation 2 -MemoryStartupBytes 1GB -VHDPath "<DOSSIER_VM>\$n\$n.vhdx" -SwitchName "Default Switch" | Out-Null
  Set-VMProcessor -VMName $n -Count 1
  Set-VMMemory -VMName $n -DynamicMemoryEnabled $false
  Set-VMFirmware -VMName $n -SecureBootTemplate MicrosoftUEFICertificateAuthority
  Add-VMNetworkAdapter -VMName $n -SwitchName "gestion-interne"
}
Get-VM | Select-Object Name, State
```

=== "Français"

    Chaque VM a deux cartes : *Default Switch* et `gestion-interne`.

    **Préparez l'identité de chaque VM, l'une après l'autre** (elles partagent le nom d'hôte, l'identifiant de machine et les clés SSH de `srv1-hv`). Pour `srv-a-hv` : démarrez-la, ouvrez la console (`vmconnect.exe localhost "srv-a-hv"`), puis :

=== "English"

    Each VM has two cards: *Default Switch* and `gestion-interne`.

    **Prepare each VM's identity, one after the other** (they share `srv1-hv`'s hostname, machine identifier and SSH keys). For `srv-a-hv`: start it, open the console (`vmconnect.exe localhost "srv-a-hv"`), then:

```bash
sudo hostnamectl set-hostname srv-a        # srv-b pour l'autre VM
sudo truncate -s 0 /etc/machine-id
sudo rm -f /var/lib/dbus/machine-id
sudo ln -s /etc/machine-id /var/lib/dbus/machine-id
sudo rm -f /etc/ssh/ssh_host_*
sudo dpkg-reconfigure openssh-server
ip -br link
sudo nano /etc/netplan/99-gestion.yaml     # adresse de la carte 2 : <IP_A1> (srv-a) ou <IP_A2> (srv-b)
sudo netplan apply
sudo poweroff
```

=== "Français"

    Notez le **nom de la carte 2** (`<NOM_INTERFACE>`) relevé par `ip -br link`. Si le nom d'interface a changé par rapport au fichier Netplan, corrigez-le dans `99-gestion.yaml`. Recommencez pour `srv-b-hv`.

    !!! success "Point de contrôle"
        - [ ] La copie de base est en lecture seule ; `Get-VHD` sur un disque de VM indique le type `Differencing`
        - [ ] Deux VM éteintes, avec un nom d'hôte distinct

=== "English"

    Note the **name of card 2** (`<NOM_INTERFACE>`) shown by `ip -br link`. If the interface name changed compared to the Netplan file, fix it in `99-gestion.yaml`. Repeat for `srv-b-hv`.

    !!! success "Checkpoint"
        - [ ] The base copy is read-only; `Get-VHD` on a VM disk shows type `Differencing`
        - [ ] Two powered-off VMs, with distinct hostnames

### Étape 3 : Scénario A, commutateur interne / Step 3: Scenario A, internal switch

=== "Français"

    La carte 2 des deux VM est déjà sur `gestion-interne`. Démarrez les deux VM. Les tests se font comme au Lab 5.1 :

=== "English"

    Card 2 of both VMs is already on `gestion-interne`. Start both VMs. Tests are done as in Lab 5.1:

```powershell
Start-VM -Name "srv-a-hv","srv-b-hv"
```

```bash
ping -c 2 <IP_A2>                      # depuis srv-a
ping -c 2 <IP_HOTE>
ping -c 2 -I <NOM_INTERFACE> 1.1.1.1
```

=== "Français"

    Depuis **PowerShell** sur l'hôte : `ping <IP_A1>`. Notez les résultats dans la matrice de l'étape 8.

    !!! success "Point de contrôle"
        - [ ] Les VM se voient entre elles et avec l'hôte ; l'accès à Internet par cette carte échoue (pas de passerelle)

=== "English"

    From **PowerShell** on the host: `ping <IP_A1>`. Note the results in the step 8 matrix.

    !!! success "Checkpoint"
        - [ ] The VMs see each other and the host; Internet access through this card fails (no gateway)

### Étape 4 : Scénario B, commutateur privé / Step 4: Scenario B, private switch

=== "Français"

    Créez un commutateur privé et déplacez la carte 2 des deux VM dessus :

=== "English"

    Create a private switch and move card 2 of both VMs onto it:

```powershell
New-VMSwitch -Name "sw-prive" -SwitchType Private
foreach ($n in "srv-a-hv","srv-b-hv") {
  Get-VMNetworkAdapter -VMName $n | Where-Object SwitchName -eq "gestion-interne" | Connect-VMNetworkAdapter -SwitchName "sw-prive"
}
Get-VMNetworkAdapter -VMName "srv-a-hv","srv-b-hv" | Select-Object VMName, SwitchName
```

=== "Français"

    Dans chaque VM, changez l'adresse de la carte 2 pour le réseau B (`<IP_B1>` pour `srv-a`, `<IP_B2>` pour `srv-b`) dans `/etc/netplan/99-gestion.yaml`, puis `sudo netplan apply`. Refaites les tests : ping de l'autre VM, de l'hôte (aucune adresse sur ce réseau), d'une adresse Internet par la carte 2, et depuis l'hôte (`ping <IP_B1>`).

    !!! success "Point de contrôle"
        - [ ] Les deux VM communiquent entre elles ; l'hôte et Internet ne répondent pas

=== "English"

    In each VM, change card 2's address to network B (`<IP_B1>` for `srv-a`, `<IP_B2>` for `srv-b`) in `/etc/netplan/99-gestion.yaml`, then `sudo netplan apply`. Redo the tests: ping the other VM, the host (no address on this network), an Internet address through card 2, and from the host (`ping <IP_B1>`).

    !!! success "Checkpoint"
        - [ ] The two VMs communicate with each other; the host and the Internet do not answer

### Étape 5 : Scénario C, un NAT personnalisé / Step 5: Scenario C, a custom NAT

=== "Français"

    Un commutateur **interne** devient un réseau NAT lorsque l'hôte en est la passerelle et que Windows traduit les adresses. Créez le commutateur, donnez à l'hôte l'adresse de passerelle, puis déclarez le NAT :

=== "English"

    An **internal** switch becomes a NAT network when the host is its gateway and Windows translates the addresses. Create the switch, give the host the gateway address, then declare the NAT:

```powershell
New-VMSwitch -Name "sw-nat" -SwitchType Internal
New-NetIPAddress -InterfaceAlias "vEthernet (sw-nat)" -IPAddress <IP_PASSERELLE_C> -PrefixLength 24
New-NetNat -Name "nat-lab" -InternalIPInterfaceAddressPrefix <RESEAU_C>/24
Get-NetNat
foreach ($n in "srv-a-hv","srv-b-hv") {
  Get-VMNetworkAdapter -VMName $n | Where-Object SwitchName -eq "sw-prive" | Connect-VMNetworkAdapter -SwitchName "sw-nat"
}
```

=== "Français"

    Dans chaque VM, configurez la carte 2 avec une adresse du réseau C, la passerelle de l'hôte et un DNS (`<IP_C1>` pour `srv-a`, `<IP_C2>` pour `srv-b`). La route par défaut reçoit une métrique élevée pour ne pas remplacer celle de la carte 1 :

=== "English"

    In each VM, configure card 2 with a network C address, the host's gateway and a DNS (`<IP_C1>` for `srv-a`, `<IP_C2>` for `srv-b`). The default route gets a high metric so as not to replace card 1's:

```yaml
network:
  version: 2
  ethernets:
    <NOM_INTERFACE>:
      dhcp4: false
      addresses:
        - <IP_C1>/24
      routes:
        - to: default
          via: <IP_PASSERELLE_C>
          metric: 200
      nameservers:
        addresses: [1.1.1.1]
```

```bash
sudo netplan apply
ip route
ping -c 2 <IP_PASSERELLE_C>
ping -c 2 -I <NOM_INTERFACE> 1.1.1.1
```

=== "Français"

    **Redirection de port (facultatif).** Exposez le SSH de `srv-a-hv` sur le port 2222 de l'hôte :

=== "English"

    **Port forwarding (optional).** Expose `srv-a-hv`'s SSH on the host's port 2222:

```powershell
Add-NetNatStaticMapping -NatName "nat-lab" -Protocol TCP -ExternalIPAddress 0.0.0.0 -ExternalPort 2222 -InternalIPAddress <IP_C1> -InternalPort 22
Get-NetNatStaticMapping
```

=== "Français"

    Testez avec l'adresse IP de la carte physique de votre poste, idéalement depuis **un autre poste du réseau** : `ssh -p 2222 utilisateur@<ADRESSE_HOTE>`. La boucle locale (`127.0.0.1`) peut ne pas être redirigée par le NAT de Windows. Le pare-feu Windows peut bloquer le port : l'autoriser en entrée si le test depuis un autre poste échoue.

    !!! success "Point de contrôle"
        - [ ] Les VM joignent la passerelle (`<IP_PASSERELLE_C>`) et sortent vers Internet par la carte 2
        - [ ] `Get-NetNat` liste `nat-lab`
        - [ ] (Facultatif) La connexion SSH redirigée atteint `srv-a-hv`

=== "English"

    Test with the IP address of your computer's physical card, ideally from **another computer on the network**: `ssh -p 2222 user@<ADRESSE_HOTE>`. The loopback address (`127.0.0.1`) may not be redirected by Windows NAT. The Windows firewall may block the port: allow it inbound if the test from another computer fails.

    !!! success "Checkpoint"
        - [ ] The VMs reach the gateway (`<IP_PASSERELLE_C>`) and go out to the Internet through card 2
        - [ ] `Get-NetNat` lists `nat-lab`
        - [ ] (Optional) The forwarded SSH connection reaches `srv-a-hv`

### Étape 6 : Scénario D, VLAN sur un même commutateur / Step 6: Scenario D, VLANs on a single switch

=== "Français"

    Replacez la carte 2 des deux VM sur le commutateur privé, avec les adresses du réseau B (voir l'étape 4). Mettez ensuite chaque carte dans un VLAN **différent** :

=== "English"

    Put card 2 of both VMs back on the private switch, with the network B addresses (see step 4). Then put each card in a **different** VLAN:

```powershell
foreach ($n in "srv-a-hv","srv-b-hv") {
  Get-VMNetworkAdapter -VMName $n | Where-Object SwitchName -eq "sw-nat" | Connect-VMNetworkAdapter -SwitchName "sw-prive"
}
Get-VMNetworkAdapter -VMName "srv-a-hv" | Where-Object SwitchName -eq "sw-prive" | Set-VMNetworkAdapterVlan -Access -VlanId 10
Get-VMNetworkAdapter -VMName "srv-b-hv" | Where-Object SwitchName -eq "sw-prive" | Set-VMNetworkAdapterVlan -Access -VlanId 20
Get-VMNetworkAdapterVlan -VMName "srv-a-hv","srv-b-hv"
```

=== "Français"

    Dans la VM, `ping <IP_B2>` depuis `srv-a` (adresse du réseau B) : il **échoue** malgré le même commutateur et le même réseau IP, car les deux cartes sont dans des VLAN différents. Mettez maintenant `srv-b-hv` dans le même VLAN et refaites le test :

=== "English"

    In the VM, `ping <IP_B2>` from `srv-a` (a network B address): it **fails** despite the same switch and the same IP network, because the two cards are in different VLANs. Now put `srv-b-hv` in the same VLAN and redo the test:

```powershell
Get-VMNetworkAdapter -VMName "srv-b-hv" | Where-Object SwitchName -eq "sw-prive" | Set-VMNetworkAdapterVlan -Access -VlanId 10
```

=== "Français"

    Le ping **réussit** de nouveau.

    !!! success "Point de contrôle"
        - [ ] Ping impossible avec des VLAN différents, possible avec le même VLAN
        - [ ] Vous savez expliquer pourquoi un routeur serait nécessaire entre deux VLAN (cours, section 5)

=== "English"

    The ping **succeeds** again.

    !!! success "Checkpoint"
        - [ ] No ping with different VLANs, ping works with the same VLAN
        - [ ] You can explain why a router would be needed between two VLANs (lesson, section 5)

### Étape 7 : Commutateur externe (facultatif) / Step 7: External switch (optional)

=== "Français"

    **Uniquement si votre poste est relié en Ethernet filaire** (la liaison Wi-Fi est parfois problématique) : cette étape interrompt brièvement la connexion réseau de l'hôte. Repérez le nom de la carte physique, créez le commutateur externe et branchez-y la carte 2 de `srv-a-hv` :

=== "English"

    **Only if your computer is connected by wired Ethernet** (a Wi-Fi link is sometimes problematic): this step briefly interrupts the host's network connection. Find the physical card name, create the external switch and plug `srv-a-hv`'s card 2 into it:

```powershell
Get-NetAdapter -Physical | Select-Object Name, Status
New-VMSwitch -Name "sw-externe" -NetAdapterName "<NOM_CARTE_PHYSIQUE>" -AllowManagementOS $true
Get-VMNetworkAdapter -VMName "srv-a-hv" | Where-Object SwitchName -eq "sw-prive" | Set-VMNetworkAdapterVlan -Untagged
Get-VMNetworkAdapter -VMName "srv-a-hv" | Where-Object SwitchName -eq "sw-prive" | Connect-VMNetworkAdapter -SwitchName "sw-externe"
```

=== "Français"

    Dans la VM, passez la carte 2 en DHCP (modèle de l'étape 5 avec `dhcp4: true`, sans adresse ni route), appliquez, puis relevez l'adresse obtenue : elle appartient au **réseau local de l'établissement**. Testez un ping de la VM vers la passerelle du réseau local, et depuis un autre poste du réseau vers la VM.

    !!! success "Point de contrôle"
        - [ ] La VM a une adresse du réseau local et joint la passerelle du réseau local
        - [ ] Un autre poste du réseau joint la VM (selon la politique du réseau)

=== "English"

    In the VM, switch card 2 to DHCP (step 5 template with `dhcp4: true`, without address or route), apply, then note the address obtained: it belongs to the **institution's local network**. Test a ping from the VM to the local network's gateway, and from another computer on the network to the VM.

    !!! success "Checkpoint"
        - [ ] The VM has a local network address and reaches the local network's gateway
        - [ ] Another computer on the network reaches the VM (depending on network policy)

### Étape 8 : Matrice de résultats / Step 8: Results matrix

=== "Français"

    Remplissez la matrice avec ✓ (ça répond) ou ✗ (ça ne répond pas) : c'est le **livrable** du lab. Complétez avec une ligne pour chaque commutateur testé.

    | Scénario | VM ↔ VM | VM ↔ hôte | VM → Internet (carte 2) | Hôte → VM | Redirection de port |
    |---|---|---|---|---|---|
    | Interne (`gestion-interne`) | … | … | … | … | — |
    | Privé (`sw-prive`) | … | … | … | … | — |
    | Interne + NAT (`sw-nat`) | … | … | … | … | … |
    | Privé avec VLAN différents | … | sans objet | sans objet | sans objet | — |
    | Externe (facultatif) | … | … | … | … | — |

    **Comparaison.** Complétez la correspondance observée avec le Lab 5.1 :

    | Hyper-V | VirtualBox | VMware Workstation |
    |---|---|---|
    | Commutateur interne | … | … |
    | Commutateur privé | … | … |
    | Interne + NAT | … | … |
    | Commutateur externe | … | … |

=== "English"

    Fill in the matrix with ✓ (it answers) or ✗ (it does not answer): this is the lab **deliverable**. Add one row for each switch tested.

    | Scenario | VM ↔ VM | VM ↔ host | VM → Internet (card 2) | Host → VM | Port forwarding |
    |---|---|---|---|---|---|
    | Internal (`gestion-interne`) | … | … | … | … | — |
    | Private (`sw-prive`) | … | … | … | … | — |
    | Internal + NAT (`sw-nat`) | … | … | … | … | … |
    | Private with different VLANs | … | not applicable | not applicable | not applicable | — |
    | External (optional) | … | … | … | … | — |

    **Comparison.** Complete the correspondence observed with Lab 5.1:

    | Hyper-V | VirtualBox | VMware Workstation |
    |---|---|---|
    | Internal switch | … | … |
    | Private switch | … | … |
    | Internal + NAT | … | … |
    | External switch | … | … |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Quelle différence y a-t-il entre un commutateur interne simple et un commutateur interne avec NAT ?"
        Le commutateur interne simple ne donne aucun accès au réseau extérieur : l'hôte n'est qu'un participant du réseau. Avec le NAT, l'hôte joue le rôle de passerelle et traduit les adresses des VM, qui peuvent alors sortir vers Internet derrière l'adresse de l'hôte.

    ??? question "Pourquoi deux VM sur le même commutateur et le même réseau IP ne se voient-elles pas lorsqu'elles sont dans des VLAN différents ?"
        Le VLAN sépare les machines au niveau 2 : les trames d'un VLAN ne sont pas transmises aux cartes d'un autre VLAN, même sur le même commutateur. Les adresses IP identiques ne changent rien tant qu'aucun routeur ne relie les deux VLAN.

    ??? question "Quel est le risque du commutateur externe, et comment le réduire ?"
        Les VM apparaissent comme des machines du réseau local : elles sont exposées aux autres postes (et à leurs attaques), et peuvent perturber le réseau (adresses en conflit, serveurs DHCP intrus). On limite ce risque en n'ouvrant que les services nécessaires, en respectant la politique du réseau et en privilégiant NAT ou interne.

=== "English"

    ??? question "What is the difference between a plain internal switch and an internal switch with NAT?"
        The plain internal switch gives no access to the outside network: the host is just a participant in the network. With NAT, the host acts as a gateway and translates the VMs' addresses, so they can go out to the Internet behind the host's address.

    ??? question "Why do two VMs on the same switch and the same IP network not see each other when they are in different VLANs?"
        A VLAN separates machines at layer 2: frames of one VLAN are not forwarded to the cards of another VLAN, even on the same switch. Identical IP networks change nothing as long as no router links the two VLANs.

    ??? question "What is the risk of the external switch, and how can it be reduced?"
        The VMs appear as machines on the local network: they are exposed to other computers (and their attacks), and can disturb the network (conflicting addresses, rogue DHCP servers). The risk is limited by opening only necessary services, respecting the network policy and preferring NAT or internal.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | `New-VM -VHDPath` signale que le disque est utilisé ou introuvable | Vérifier le chemin et que le dossier de la VM existe ; ne pas partager un même disque différentiel entre deux VM |
    | Une VM ne démarre pas (démarrage sécurisé) | Vérifier `Set-VMFirmware -SecureBootTemplate MicrosoftUEFICertificateAuthority` (Lab 4.1) |
    | Impossible de définir l'adresse sur `vEthernet (sw-nat)` | Vérifier le nom exact de la carte (`Get-NetAdapter`) et qu'elle n'a pas déjà une adresse |
    | `New-NetNat` échoue | Un seul NAT par plage ; supprimer un NAT existant en conflit (`Get-NetNat`, `Remove-NetNat`) ; vérifier que la plage ne chevauche pas un réseau existant |
    | Pas de sortie vers Internet par le NAT | Passerelle et DNS dans Netplan ; transfert IP de l'hôte ; pare-feu ; plage du NAT identique à celle de la carte de l'hôte |
    | Deux VM ont la même adresse ou le même nom | Identité non préparée (étape 2) |
    | Le VLAN ne semble pas appliqué | Vérifier avec `Get-VMNetworkAdapterVlan` que la bonne carte (celle du commutateur testé) a été modifiée |
    | `New-VMSwitch` externe coupe la connexion | Normal quelques secondes ; en Wi-Fi, éviter ce scénario |
    | Un hyperviseur de type 2 ne démarre plus | Hyper-V actif : performances réduites ou refus ; voir la bascule du Lab 2.1 |

=== "English"

    | Symptom | Hints |
    |---|---|
    | `New-VM -VHDPath` reports the disk is in use or not found | Check the path and that the VM folder exists; do not share a single differencing disk between two VMs |
    | A VM does not start (Secure Boot) | Check `Set-VMFirmware -SecureBootTemplate MicrosoftUEFICertificateAuthority` (Lab 4.1) |
    | Cannot set the address on `vEthernet (sw-nat)` | Check the exact adapter name (`Get-NetAdapter`) and that it does not already have an address |
    | `New-NetNat` fails | One NAT per range; remove a conflicting existing NAT (`Get-NetNat`, `Remove-NetNat`); check the range does not overlap an existing network |
    | No Internet exit through the NAT | Gateway and DNS in Netplan; host IP forwarding; firewall; NAT range identical to the host adapter's |
    | Two VMs have the same address or name | Identity not prepared (step 2) |
    | The VLAN does not seem applied | Check with `Get-VMNetworkAdapterVlan` that the right card (the one on the switch under test) was modified |
    | The external `New-VMSwitch` cuts the connection | Normal for a few seconds; avoid this scenario on Wi-Fi |
    | A type 2 hypervisor no longer starts | Hyper-V active: reduced performance or refusal; see the switching procedure in Lab 2.1 |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    Éteignez les VM. Pour tout supprimer : `Remove-VM -Name "srv-a-hv","srv-b-hv" -Force`, supprimez leurs dossiers, puis `Remove-NetNat -Name "nat-lab"` et `Remove-VMSwitch -Name "sw-prive","sw-nat"` (et `sw-externe` s'il a été créé). Conservez `gestion-interne` et `srv1-hv`. La copie de base (`base-ubuntu.vhdx`, en lecture seule) peut être supprimée après les VM qui en dépendent. Vérifiez dans l'énoncé du [Lab 5.3](lab-5-3-routage-nat-et-vlan-sous-linux.md) l'état d'hyperviseur Windows qu'il exige avant de poursuivre ; pour revenir aux labs de VirtualBox ou de VMware, désactivez l'hyperviseur Windows (Lab 2.1).

=== "English"

    Shut the VMs down. To delete everything: `Remove-VM -Name "srv-a-hv","srv-b-hv" -Force`, delete their folders, then `Remove-NetNat -Name "nat-lab"` and `Remove-VMSwitch -Name "sw-prive","sw-nat"` (and `sw-externe` if created). Keep `gestion-interne` and `srv1-hv`. The base copy (`base-ubuntu.vhdx`, read-only) can be deleted after the VMs depending on it. Check in [Lab 5.3](lab-5-3-routage-nat-et-vlan-sous-linux.md)'s statement which Windows hypervisor state it requires before continuing; to return to the VirtualBox or VMware labs, turn the Windows hypervisor off (Lab 2.1).

[Retour au chapitre 5 / Back to chapter 5](cours.md)