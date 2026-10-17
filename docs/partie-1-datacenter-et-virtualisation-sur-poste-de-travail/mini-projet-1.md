# Mini-projet 1 : Un mini-datacenter virtuel décrit comme du code / Mini-Project 1: A Virtual Mini-Datacenter Described as Code

## 1. Fiche du mini-projet / Project sheet

=== "Français"

    | Rubrique | Description |
    |---|---|
    | Cours | Cloud et virtualisation : concevoir et exploiter une infrastructure virtualisée |
    | Position | Fin de la Partie 1 : Datacenter et virtualisation sur poste de travail (chapitres 1 à 6) |
    | Modalité | En binôme, sur le poste de l'un des deux étudiants |
    | Durée | 3 h en séance, plus un compte rendu à rendre sous une semaine |
    | Prérequis | Chapitres 1 à 6 terminés ; Labs 1.1, 5.3, 6.1, 6.2 et 6.3 réalisés |
    | Environnement | Poste Windows, VirtualBox, Vagrant, hyperviseur Windows **désactivé** (voir l'annexe [Environnement](../annexes/environnement.md)) |
    | Ressources minimales | Au moins 5 Go de RAM libres, 10 Go d'espace disque libre, accès Internet ; 3 VM de 1 vCPU et 1 Go de RAM |
    | Acquis visés | AA1 (dimensionner une infrastructure), AA2 (construire, relier et décrire des machines virtuelles) |

=== "English"

    | Item | Description |
    |---|---|
    | Course | Cloud and Virtualization: designing and operating a virtualized infrastructure |
    | Position | End of Part 1: Datacenter and virtualization on a workstation (chapters 1 to 6) |
    | Format | In pairs, on one of the two students' computers |
    | Duration | 3 h in class, plus a report due within a week |
    | Prerequisites | Chapters 1 to 6 completed; Labs 1.1, 5.3, 6.1, 6.2 and 6.3 done |
    | Environment | Windows computer, VirtualBox, Vagrant, Windows hypervisor **off** (see the [Environment](../annexes/environnement.md) annex) |
    | Minimum resources | At least 5 GB of free RAM, 10 GB of free disk space, Internet access; 3 VMs of 1 vCPU and 1 GB RAM |
    | Learning outcomes | AA1 (size an infrastructure), AA2 (build, connect and describe virtual machines) |

## 2. Contexte / Context

=== "Français"

    Une petite structure veut tester sur un poste de travail l'architecture d'un service web avant de louer du matériel. Vous êtes l'équipe infrastructure. Votre mission : décrire **entièrement en fichiers** un mini-datacenter virtuel composé de trois machines, puis démontrer qu'il fonctionne, qu'il se **reconstruit à l'identique** et qu'on sait **diagnostiquer une panne**.

    L'application est volontairement minimale : l'évaluation porte sur l'**exploitation de l'infrastructure** (dimensionnement, réseau, description, reproductibilité, diagnostic) et non sur la qualité du code de l'application, qui vous est fournie sous forme de scripts à intégrer.

=== "English"

    A small organization wants to test the architecture of a web service on a workstation before renting hardware. You are the infrastructure team. Your mission: describe **entirely in files** a virtual mini-datacenter made of three machines, then demonstrate that it works, that it **rebuilds identically** and that you can **troubleshoot a failure**.

    The application is deliberately minimal: evaluation is about **operating the infrastructure** (sizing, network, description, reproducibility, troubleshooting), not the quality of the application code, which is provided to you as scripts to integrate.

## 3. Objectifs pédagogiques / Learning objectives

=== "Français"

    À l'issue du mini-projet, vous serez capables de :

    1. dimensionner un petit environnement virtualisé à partir des ressources d'un poste ;
    2. construire un plan d'adressage cohérent pour plusieurs réseaux virtuels ;
    3. décrire plusieurs machines et leurs réseaux dans un Vagrantfile, avec un provisionnement rejouable ;
    4. mettre en place un routeur Linux avec transfert IP et NAT ;
    5. vérifier le chemin réel des paquets et diagnostiquer une panne méthodiquement.

=== "English"

    By the end of the mini-project, you will be able to:

    1. size a small virtualized environment from a computer's resources;
    2. build a consistent addressing plan for several virtual networks;
    3. describe several machines and their networks in a Vagrantfile, with replayable provisioning;
    4. set up a Linux router with IP forwarding and NAT;
    5. check the real path of packets and troubleshoot a failure methodically.

## 4. Architecture cible / Target architecture

=== "Français"

    ```mermaid
    flowchart LR
      H["Poste hôte<br/>127.0.0.1:8080"] --> W["web<br/>nginx"]
      W --- F["reseau-front<br/>(réseau interne)"]
      F --- R["routeur<br/>(transfert IP + NAT)"]
      R --- B["reseau-back<br/>(réseau interne)"]
      B --- A["app<br/>service port 8000"]
      R --- I["Internet<br/>(NAT de l'hyperviseur)"]
    ```

    - `web` et `app` sont sur **deux réseaux internes différents** ; ils ne communiquent que **par le routeur**.
    - Le routeur joint Internet par la carte NAT créée par Vagrant et assure le NAT pour `web` et `app`.
    - L'hôte n'atteint `web` que par la redirection de port 8080.

=== "English"

    ```mermaid
    flowchart LR
      H["Host computer<br/>127.0.0.1:8080"] --> W["web<br/>nginx"]
      W --- F["reseau-front<br/>(internal network)"]
      F --- R["routeur<br/>(IP forwarding + NAT)"]
      R --- B["reseau-back<br/>(internal network)"]
      B --- A["app<br/>service port 8000"]
      R --- I["Internet<br/>(hypervisor NAT)"]
    ```

    - `web` and `app` are on **two different internal networks**; they only communicate **through the router**.
    - The router reaches the Internet through the NAT card created by Vagrant and performs NAT for `web` and `app`.
    - The host only reaches `web` through port forwarding on 8080.

## 5. Cahier des charges / Requirements

=== "Français"

    | N° | Exigence |
    |---|---|
    | E1 | Une **fiche de dimensionnement** compare les ressources demandées (vCPU, RAM, disque des trois VM) à celles du poste et justifie la faisabilité. |
    | E2 | Un **plan d'adressage** documenté définit les réseaux `reseau-front` et `reseau-back` et les adresses des trois machines, sans chevauchement, conformément à l'annexe Environnement. |
    | E3 | Un **Vagrantfile unique** décrit les trois machines `routeur`, `web` et `app`, avec une box dont la **version est fixée**. |
    | E4 | `web` n'est reliée qu'à `reseau-front`, `app` qu'à `reseau-back`, et `routeur` aux deux. |
    | E5 | Le routeur **transmet les paquets** entre les deux réseaux, y compris après un redémarrage. |
    | E6 | `web` et `app` ont le routeur pour **passerelle par défaut** : leur trafic vers Internet traverse le routeur, qui assure le **NAT**. |
    | E7 | `app` sert une page sur le port 8000 ; `web` la relaie sur le port 80 vers l'adresse de `app`. |
    | E8 | Depuis l'hôte, `http://127.0.0.1:8080` renvoie la page produite par `app`. |
    | E9 | L'environnement se **reconstruit de zéro** par `vagrant destroy -f` puis `vagrant up`, **sans intervention manuelle**. |
    | E10 | Une **panne volontaire** est diagnostiquée de façon méthodique et expliquée par écrit. |
    | E11 | Le dossier du projet contient le Vagrantfile, les scripts, un `README.md` et un `.gitignore` ; **aucun secret** n'y figure. |

=== "English"

    | No. | Requirement |
    |---|---|
    | E1 | A **sizing sheet** compares the requested resources (vCPU, RAM, disk of the three VMs) with the computer's and justifies feasibility. |
    | E2 | A documented **addressing plan** defines the `reseau-front` and `reseau-back` networks and the addresses of the three machines, with no overlap, in line with the Environment annex. |
    | E3 | A **single Vagrantfile** describes the three machines `routeur`, `web` and `app`, with a box whose **version is pinned**. |
    | E4 | `web` is only connected to `reseau-front`, `app` only to `reseau-back`, and `routeur` to both. |
    | E5 | The router **forwards packets** between the two networks, including after a restart. |
    | E6 | `web` and `app` have the router as **default gateway**: their Internet traffic goes through the router, which performs **NAT**. |
    | E7 | `app` serves a page on port 8000; `web` relays it on port 80 to `app`'s address. |
    | E8 | From the host, `http://127.0.0.1:8080` returns the page produced by `app`. |
    | E9 | The environment **rebuilds from scratch** with `vagrant destroy -f` then `vagrant up`, **with no manual intervention**. |
    | E10 | A **deliberate failure** is troubleshot methodically and explained in writing. |
    | E11 | The project folder contains the Vagrantfile, the scripts, a `README.md` and a `.gitignore`; **no secret** appears in it. |

## 6. Démarche guidée en phases / Guided approach in phases

### Phase 0 : Dimensionner et planifier (20 min) / Phase 0: Size and plan (20 min)

=== "Français"

    - **But :** savoir ce que coûte l'environnement et où chaque machine se place avant d'écrire une ligne.
    - **Indices :** relevez les ressources de votre poste (fiche du Lab 1.2) ; additionnez les besoins des trois VM (vCPU, RAM, disque dynamique) ; comparez-les à votre profil de poste (annexe Environnement) en tenant compte de la marge à laisser à Windows. Choisissez deux réseaux libres dans l'annexe et notez les adresses de `routeur` (deux), `web` et `app`.
    - **Point de contrôle :** une fiche de dimensionnement et un plan d'adressage (tableau et schéma) sont validés par l'enseignant.
    - **Question de réflexion :** quelle contrainte (RAM, vCPU, disque) limiterait le plus vite l'ajout de machines sur votre poste ?

=== "English"

    - **Goal:** know what the environment costs and where each machine sits before writing a single line.
    - **Hints:** collect your computer's resources (Lab 1.2 sheet); add up the needs of the three VMs (vCPU, RAM, dynamic disk); compare them with your computer profile (Environment annex) allowing for the margin to leave to Windows. Choose two free networks in the annex and note the addresses of `routeur` (two), `web` and `app`.
    - **Checkpoint:** a sizing sheet and an addressing plan (table and diagram) are validated by the instructor.
    - **Reflection question:** which constraint (RAM, vCPU, disk) would limit adding more machines on your computer first?

### Phase 1 : Décrire les machines et les réseaux (30 min) / Phase 1: Describe machines and networks (30 min)

=== "Français"

    - **But :** obtenir les trois machines, chacune sur les bons réseaux, sans provisionnement.
    - **Indices :** partez du squelette ci-dessous. L'option `virtualbox__intnet` nomme un réseau interne VirtualBox ; Vagrant ajoute toujours sa propre carte NAT (carte 1). Fixez la version de la box. Utilisez `vagrant validate`, puis `vagrant up` et `vagrant ssh <machine> -c "ip -br addr"`.
    - **Point de contrôle :** `vagrant status` montre trois machines ; chaque machine porte les adresses prévues au plan.
    - **Question de réflexion :** pourquoi la carte NAT créée par Vagrant existe-t-elle, et en quoi risque-t-elle de fausser vos tests ?

=== "English"

    - **Goal:** get the three machines, each on the right networks, without provisioning.
    - **Hints:** start from the skeleton below. The `virtualbox__intnet` option names a VirtualBox internal network; Vagrant always adds its own NAT card (card 1). Pin the box version. Use `vagrant validate`, then `vagrant up` and `vagrant ssh <machine> -c "ip -br addr"`.
    - **Checkpoint:** `vagrant status` shows three machines; each machine carries the addresses from the plan.
    - **Reflection question:** why does the NAT card created by Vagrant exist, and how could it skew your tests?

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "<NOM_DE_LA_BOX>"
  config.vm.box_version = "<VERSION>"

  config.vm.define "routeur" do |r|
    r.vm.hostname = "routeur"
    r.vm.network "private_network", ip: "<IP_ROUTEUR_FRONT>", virtualbox__intnet: "reseau-front"
    r.vm.network "private_network", ip: "<IP_ROUTEUR_BACK>",  virtualbox__intnet: "reseau-back"
    # ... ressources (virtualbox), provisionnement provision/routeur.sh
  end

  config.vm.define "app" do |a|
    # ... nom d'hôte, réseau reseau-back, ressources, provisionnement provision/app.sh
  end

  config.vm.define "web" do |w|
    # ... nom d'hôte, réseau reseau-front, redirection du port 8080, ressources, provisionnement provision/web.sh
  end
end
```

### Phase 2 : Le routeur (30 min) / Phase 2: The router (30 min)

=== "Français"

    - **But :** `routeur` transmet les paquets entre les deux réseaux et assure le NAT vers Internet.
    - **Indices :** reprenez les notions du [Lab 5.3](ch05-connexion-entre-vm-et-reseaux-virtuels/lab-5-3-routage-nat-et-vlan-sous-linux.md) : transfert IP permanent (`sysctl`), règle `masquerade` sur la **carte de sortie** (la carte NAT de Vagrant : repérez-la par `ip route show default`), et rendez la configuration **permanente** ou rejouable. Le script `provision/routeur.sh` doit pouvoir être rejoué sans erreur.
    - **Point de contrôle :** `sysctl net.ipv4.ip_forward` vaut 1 sur `routeur`, y compris après `vagrant reload routeur` ; `nft list ruleset` montre la règle de NAT.
    - **Question de réflexion :** que se passerait-il si la règle `masquerade` était posée sur la mauvaise carte ?

=== "English"

    - **Goal:** `routeur` forwards packets between the two networks and performs NAT to the Internet.
    - **Hints:** reuse the notions from [Lab 5.3](ch05-connexion-entre-vm-et-reseaux-virtuels/lab-5-3-routage-nat-et-vlan-sous-linux.md): permanent IP forwarding (`sysctl`), a `masquerade` rule on the **outgoing card** (Vagrant's NAT card: find it with `ip route show default`), and make the configuration **permanent** or replayable. The `provision/routeur.sh` script must be replayable without errors.
    - **Checkpoint:** `sysctl net.ipv4.ip_forward` is 1 on `routeur`, including after `vagrant reload routeur`; `nft list ruleset` shows the NAT rule.
    - **Reflection question:** what would happen if the `masquerade` rule were set on the wrong card?

### Phase 3 : L'application et les routes (30 min) / Phase 3: The application and routes (30 min)

=== "Français"

    - **But :** `app` sert la page, `web` la relaie, et les deux passent par le routeur.
    - **Indices :** réutilisez les scripts du [Lab 6.2](ch06-vagrant-decrire-ses-environnements/lab-6-2-environnement-multi-machines-avec-provisioning.md), en passant l'adresse de `app` en argument à `web`. Sur `web` et `app`, ajoutez une **route par défaut** via l'adresse du routeur sur leur réseau, avec une **métrique plus basse** que celle de la carte NAT de Vagrant, et rendez-la persistante (par exemple avec un fichier Netplan, comme au Lab 5.3).
    - **Point de contrôle :** `curl.exe http://127.0.0.1:8080` renvoie la page de `app` ; `vagrant ssh web -c "ip route"` montre la route par défaut via le routeur.
    - **Question de réflexion :** pourquoi `web` ne peut-il pas joindre `app` directement, sans passer par le routeur ?

=== "English"

    - **Goal:** `app` serves the page, `web` relays it, and both go through the router.
    - **Hints:** reuse the scripts from [Lab 6.2](ch06-vagrant-decrire-ses-environnements/lab-6-2-environnement-multi-machines-avec-provisioning.md), passing `app`'s address as an argument to `web`. On `web` and `app`, add a **default route** via the router's address on their network, with a **lower metric** than Vagrant's NAT card, and make it persistent (for example with a Netplan file, as in Lab 5.3).
    - **Checkpoint:** `curl.exe http://127.0.0.1:8080` returns `app`'s page; `vagrant ssh web -c "ip route"` shows the default route via the router.
    - **Reflection question:** why can `web` not reach `app` directly, without going through the router?

### Phase 4 : Tests, preuves et panne (30 min) / Phase 4: Tests, evidence and failure (30 min)

=== "Français"

    - **But :** prouver que le chemin passe bien par le routeur et diagnostiquer une panne.
    - **Indices :** depuis `web`, `tracepath -n <IP_APP>` et `tracepath -n 1.1.1.1` doivent faire apparaître le routeur au premier saut ; sur `routeur`, `sudo tcpdump -ni <interface> icmp` montre les paquets en transit. **Panne volontaire :** votre binôme voisin (ou l'enseignant) désactive en secret un élément (le transfert IP, la règle de NAT ou une route) ; vous suivez la méthode couche par couche du cours (interface, voisin, passerelle, extérieur, DNS, service) et notez l'ordre de vos investigations.
    - **Point de contrôle :** la panne est localisée, corrigée, et la preuve du retour à la normale est notée.
    - **Question de réflexion :** quel symptôme vous a orienté vers la cause, et quelle commande l'a confirmée ?

=== "English"

    - **Goal:** prove that the path really goes through the router and troubleshoot a failure.
    - **Hints:** from `web`, `tracepath -n <IP_APP>` and `tracepath -n 1.1.1.1` must show the router at the first hop; on `routeur`, `sudo tcpdump -ni <interface> icmp` shows the packets in transit. **Deliberate failure:** your neighboring pair (or the instructor) secretly disables an element (IP forwarding, the NAT rule or a route); you follow the course's layer-by-layer method (interface, neighbor, gateway, outside, DNS, service) and note the order of your investigations.
    - **Checkpoint:** the failure is located, fixed, and the evidence of return to normal is noted.
    - **Reflection question:** which symptom guided you towards the cause, and which command confirmed it?

### Phase 5 : Reproductibilité et clôture (20 min) / Phase 5: Reproducibility and wrap-up (20 min)

=== "Français"

    - **But :** démontrer qu'on reconstruit tout sans intervention et documenter le projet.
    - **Indices :** `vagrant destroy -f`, puis `vagrant up` en mesurant la durée ; refaites les tests de la phase 3. Rédigez le `README.md` (but, prérequis, commandes de construction et de test, plan d'adressage). Créez le `.gitignore` (`.vagrant/`). Vérifiez qu'aucun mot de passe ou clé ne figure dans les fichiers.
    - **Point de contrôle :** après reconstruction, les exigences E7 et E8 sont toujours satisfaites sans aucune commande manuelle.
    - **Question de réflexion :** qu'est-ce qui, dans votre description, a **failli** empêcher une reconstruction à l'identique ?

=== "English"

    - **Goal:** demonstrate that everything is rebuilt without intervention and document the project.
    - **Hints:** `vagrant destroy -f`, then `vagrant up` while measuring the duration; redo the phase 3 tests. Write the `README.md` (purpose, prerequisites, build and test commands, addressing plan). Create the `.gitignore` (`.vagrant/`). Check that no password or key appears in the files.
    - **Checkpoint:** after rebuilding, requirements E7 and E8 are still satisfied with no manual command.
    - **Reflection question:** what in your description **almost** prevented an identical rebuild?

## 7. Livrables / Deliverables

=== "Français"

    1. **Dossier du projet** : `Vagrantfile`, dossier `provision/` (scripts), `README.md`, `.gitignore` (sans `.vagrant/`, sans secret).
    2. **Compte rendu** (3 à 5 pages) comprenant :
        - la fiche de dimensionnement et le plan d'adressage (tableau et schéma) ;
        - la preuve de chaque exigence (extraits de sorties de commandes : `ip -br addr`, `ip route`, `sysctl`, `nft list ruleset`, `tracepath`, `curl`) ;
        - le récit du diagnostic de la panne (symptômes, hypothèses, ordre des vérifications, cause, correction) ;
        - les réponses aux questions de réflexion ;
        - un court paragraphe « difficultés rencontrées et enseignements ».
    3. **Démonstration orale** de 5 minutes : reconstruction, accès à la page par l'hôte, tracé du chemin, panne et diagnostic.

=== "English"

    1. **Project folder**: `Vagrantfile`, `provision/` folder (scripts), `README.md`, `.gitignore` (without `.vagrant/`, without secrets).
    2. **Report** (3 to 5 pages) including:
        - the sizing sheet and the addressing plan (table and diagram);
        - evidence for each requirement (command output excerpts: `ip -br addr`, `ip route`, `sysctl`, `nft list ruleset`, `tracepath`, `curl`);
        - the failure troubleshooting story (symptoms, hypotheses, order of checks, cause, fix);
        - answers to the reflection questions;
        - a short paragraph "difficulties met and lessons learned".
    3. **Oral demonstration** of 5 minutes: rebuild, access to the page from the host, path trace, failure and troubleshooting.

## 8. Grille d'évaluation (/20) / Evaluation grid (/20)

=== "Français"

    | Critère | Points | AA |
    |---|---|---|
    | Fiche de dimensionnement et plan d'adressage (E1, E2) | 3 | AA1 |
    | Vagrantfile : trois machines, réseaux, box fixée, ressources (E3, E4) | 4 | AA2 |
    | Routage et NAT : transfert IP, passerelles, chemin des paquets (E5, E6) | 5 | AA2 |
    | Application de bout en bout, accès depuis l'hôte (E7, E8) | 3 | AA2 |
    | Reconstruction sans intervention et qualité du dossier (E9, E11) | 2 | AA2 |
    | Diagnostic de la panne : méthode et explication (E10) | 3 | AA1 |

=== "English"

    | Criterion | Points | AA |
    |---|---|---|
    | Sizing sheet and addressing plan (E1, E2) | 3 | AA1 |
    | Vagrantfile: three machines, networks, pinned box, resources (E3, E4) | 4 | AA2 |
    | Routing and NAT: IP forwarding, gateways, packet path (E5, E6) | 5 | AA2 |
    | End-to-end application, access from the host (E7, E8) | 3 | AA2 |
    | Rebuild without intervention and project quality (E9, E11) | 2 | AA2 |
    | Failure troubleshooting: method and explanation (E10) | 3 | AA1 |

## 9. Erreurs fréquentes et conseils de diagnostic / Common mistakes and troubleshooting tips

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | `web` ou `app` joint Internet sans passer par le routeur | La carte NAT de Vagrant sert de sortie : route par défaut manquante ou métrique trop haute ; vérifier avec `ip route` et `tracepath` |
    | `web` n'atteint pas `app` | Transfert IP désactivé sur `routeur` ; route par défaut absente sur un des deux ; noms de réseaux internes différents d'une machine à l'autre (`virtualbox__intnet`) |
    | La règle de NAT ne s'applique pas | Mauvaise interface de sortie ; règle perdue au redémarrage (non persistante) ; table `nft` non créée |
    | La route disparaît après `vagrant reload` | Route posée à la main : la persister (Netplan) ou la poser dans le provisionnement rejoué |
    | `curl.exe` sur le port 8080 ne répond pas | `web` non provisionnée ; port déjà utilisé sur l'hôte ; `nginx -t` en erreur |
    | `vagrant up` échoue sur le réseau privé | Adresse hors de la plage autorisée ou en conflit avec un autre réseau de l'hôte |
    | Scripts qui échouent à la reconstruction | Script non rejouable (fichier ou règle déjà existant) ; fins de ligne Windows ; dépendance à une action manuelle |
    | VM qui ne démarrent pas | Virtualisation désactivée ou hyperviseur Windows actif (voir le [Lab 2.1](ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md)) |

=== "English"

    | Symptom | Hints |
    |---|---|
    | `web` or `app` reaches the Internet without going through the router | Vagrant's NAT card serves as exit: default route missing or metric too high; check with `ip route` and `tracepath` |
    | `web` cannot reach `app` | IP forwarding disabled on `routeur`; default route missing on one of the two; internal network names differ between machines (`virtualbox__intnet`) |
    | The NAT rule does not apply | Wrong outgoing interface; rule lost at restart (not persistent); `nft` table not created |
    | The route disappears after `vagrant reload` | Route set by hand: make it persistent (Netplan) or set it in the replayed provisioning |
    | `curl.exe` on port 8080 gets no answer | `web` not provisioned; port already in use on the host; `nginx -t` in error |
    | `vagrant up` fails on the private network | Address outside the allowed range or conflicting with another host network |
    | Scripts fail on rebuild | Script not replayable (file or rule already exists); Windows line endings; dependence on a manual action |
    | VMs do not start | Virtualization disabled or Windows hypervisor active (see [Lab 2.1](ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md)) |

## 10. Pour aller plus loin (facultatif) / Going further (optional)

=== "Français"

    - Ajouter un **deuxième service** sur `app` et une seconde route de `web` (chemin `/api`).
    - Séparer le trafic en **deux VLAN** sur un même réseau interne, comme au Lab 5.3.
    - Adapter le Vagrantfile à **Hyper-V** (Lab 6.3) et lister ce qui ne se transpose pas.
    - Ajouter un snapshot (`vagrant snapshot save`) avant la panne et le restaurer ensuite.

=== "English"

    - Add a **second service** on `app` and a second route on `web` (`/api` path).
    - Split the traffic into **two VLANs** on a single internal network, as in Lab 5.3.
    - Adapt the Vagrantfile to **Hyper-V** (Lab 6.3) and list what does not carry over.
    - Add a snapshot (`vagrant snapshot save`) before the failure and restore it afterwards.

## 11. Aide-mémoire / Cheat sheet

| Besoin / Need | Commande / Command |
|---|---|
| Valider le Vagrantfile / Validate | `vagrant validate` |
| Construire / Build | `vagrant up` |
| Rejouer le provisionnement / Replay provisioning | `vagrant provision <machine>` |
| Commande dans une machine / Run a command in a machine | `vagrant ssh <machine> -c "<commande>"` |
| Redémarrer / Restart | `vagrant reload <machine>` |
| Tout détruire / Destroy all | `vagrant destroy -f` |
| Adresses et routes / Addresses and routes | `ip -br addr`, `ip route` |
| Transfert IP / IP forwarding | `sysctl net.ipv4.ip_forward` |
| Règles de NAT / NAT rules | `sudo nft list ruleset` |
| Chemin des paquets / Packet path | `tracepath -n <adresse>` |
| Capture / Capture | `sudo tcpdump -ni <interface> icmp` |