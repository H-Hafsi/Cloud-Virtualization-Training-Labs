# Chapitre 6 : Vagrant, décrire ses environnements / Vagrant: Describing Your Environments

=== "Français"

    !!! abstract "Objectifs du chapitre"
        - Expliquer pourquoi décrire un environnement dans un fichier plutôt que le construire à la main
        - Définir les notions de Vagrantfile, box, provider et provisioner
        - Décrire le cycle de vie d'une machine gérée par Vagrant et ses commandes principales
        - Lire et écrire un Vagrantfile : machine unique, réseau, ressources, provisionnement, plusieurs machines
        - Comparer les providers (VirtualBox, VMware, Hyper-V) et leurs limites
        - Acquis d'apprentissage visé : **AA2**

        Matériel requis pour ce cours : aucun. Les labs associés indiquent leurs propres ressources.

=== "English"

    !!! abstract "Chapter objectives"
        - Explain why an environment is described in a file rather than built by hand
        - Define the notions of Vagrantfile, box, provider and provisioner
        - Describe the lifecycle of a Vagrant-managed machine and its main commands
        - Read and write a Vagrantfile: single machine, network, resources, provisioning, several machines
        - Compare providers (VirtualBox, VMware, Hyper-V) and their limits
        - Target learning outcome: **AA2**

        Required equipment for this lesson: none. The associated labs state their own resources.

## 1. Pourquoi décrire ses environnements ? / Why describe your environments?

=== "Français"

    Aux chapitres 3 à 5, vous avez construit des VM **à la main** : création dans l'interface, installation, réglages réseau, commandes de préparation. Cette méthode a des limites : elle est longue, difficile à reproduire à l'identique, et personne ne sait exactement comment la machine a été obtenue.

    **Vagrant** (HashiCorp) décrit un environnement de machines virtuelles dans un **fichier texte**. Une seule commande le construit, une autre le détruit, et le même fichier produit le même résultat sur un autre poste. C'est une première forme d'**infrastructure décrite comme du code** (*Infrastructure as Code*), que nous retrouverons au chapitre 20.

    | Construction manuelle | Environnement décrit |
    |---|---|
    | Étapes dans l'interface, de mémoire | Un fichier lisible, relu et versionné |
    | Résultat variable d'un poste à l'autre | Résultat reproductible |
    | Difficile à partager | Se partage en copiant un fichier (ou un dépôt Git) |
    | Reconstruire prend des heures | Détruire et reconstruire prend quelques minutes |

    !!! info "Licence et usage"
        Vagrant est publié par HashiCorp sous une licence qui impose des conditions d'usage : vérifiez la licence de la version utilisée avant tout usage hors enseignement (voir l'annexe [Environnement](../../annexes/environnement.md)). Vagrant sert ici à des environnements de laboratoire, pas à la production.

=== "English"

    In chapters 3 to 5, you built VMs **by hand**: creation in the interface, installation, network settings, preparation commands. This method has limits: it is slow, hard to reproduce identically, and nobody knows exactly how the machine was obtained.

    **Vagrant** (HashiCorp) describes a virtual machine environment in a **text file**. One command builds it, another destroys it, and the same file produces the same result on another computer. It is a first form of **infrastructure described as code** (*Infrastructure as Code*), which we will meet again in chapter 20.

    | Manual build | Described environment |
    |---|---|
    | Steps in the interface, from memory | A readable file, reviewed and versioned |
    | Result varies from one computer to another | Reproducible result |
    | Hard to share | Shared by copying a file (or a Git repository) |
    | Rebuilding takes hours | Destroying and rebuilding takes minutes |

    !!! info "License and use"
        Vagrant is published by HashiCorp under a license that sets conditions of use: check the license of the version used before any use outside teaching (see the [Environment](../../annexes/environnement.md) annex). Vagrant is used here for lab environments, not for production.

## 2. Les quatre notions clés / The four key notions

=== "Français"

    | Notion | Définition |
    |---|---|
    | **Vagrantfile** | Fichier (écrit dans une syntaxe Ruby) qui décrit les machines, leur réseau, leurs ressources et leur configuration |
    | **Box** | Image de base empaquetée (disque et métadonnées) dont Vagrant part pour créer une machine ; téléchargée une fois puis réutilisée |
    | **Provider** | Hyperviseur qui exécute réellement la VM : VirtualBox, VMware, Hyper-V, libvirt… (à ne pas confondre avec le *provider* d'OpenTofu, chapitre 20) |
    | **Provisioner** | Mécanisme qui configure la machine après sa création : script shell, copie de fichiers, Ansible (chapitre 20) |

    Une box est prévue pour un ou plusieurs providers : **la même box ne fonctionne pas forcément avec tous**. Le catalogue public de boxes est un service tiers : on choisit des boxes de **sources fiables** et on fixe leur version.

=== "English"

    | Notion | Definition |
    |---|---|
    | **Vagrantfile** | File (written in Ruby syntax) describing the machines, their network, their resources and their configuration |
    | **Box** | Packaged base image (disk and metadata) from which Vagrant creates a machine; downloaded once then reused |
    | **Provider** | Hypervisor that actually runs the VM: VirtualBox, VMware, Hyper-V, libvirt... (not to be confused with OpenTofu's *provider*, chapter 20) |
    | **Provisioner** | Mechanism that configures the machine after creation: shell script, file copy, Ansible (chapter 20) |

    A box is designed for one or more providers: **the same box does not necessarily work with all of them**. The public box catalog is a third-party service: choose boxes from **trusted sources** and pin their version.

## 3. Le cycle de vie et les commandes / The lifecycle and the commands

=== "Français"

    ```mermaid
    flowchart LR
      I["vagrant init"] --> U["vagrant up"]
      U --> S["vagrant ssh"]
      U --> P["vagrant provision"]
      U --> H["vagrant halt"]
      H --> U
      U --> D["vagrant destroy"]
    ```

    | Commande | Effet |
    |---|---|
    | `vagrant init` | Crée un Vagrantfile de départ dans le dossier courant |
    | `vagrant validate` | Vérifie la syntaxe du Vagrantfile |
    | `vagrant up` | Crée (si besoin) et démarre les machines ; provisionne à la première création |
    | `vagrant status` | Affiche l'état des machines du dossier |
    | `vagrant ssh [nom]` | Ouvre une session SSH dans une machine |
    | `vagrant provision` | Rejoue les provisioners sur une machine en marche |
    | `vagrant reload` | Redémarre en relisant le Vagrantfile |
    | `vagrant halt` | Éteint proprement les machines (elles existent toujours) |
    | `vagrant destroy` | **Supprime** les machines et leurs disques |
    | `vagrant box add / list / remove` | Gère les boxes téléchargées |
    | `vagrant global-status` | Liste toutes les machines Vagrant du poste |

    Le dossier contenant le Vagrantfile est le **dossier du projet** ; Vagrant y crée un sous-dossier `.vagrant/` qui mémorise l'état local (identifiants des machines). Il ne doit pas être versionné.

=== "English"

    ```mermaid
    flowchart LR
      I["vagrant init"] --> U["vagrant up"]
      U --> S["vagrant ssh"]
      U --> P["vagrant provision"]
      U --> H["vagrant halt"]
      H --> U
      U --> D["vagrant destroy"]
    ```

    | Command | Effect |
    |---|---|
    | `vagrant init` | Creates a starter Vagrantfile in the current folder |
    | `vagrant validate` | Checks the Vagrantfile syntax |
    | `vagrant up` | Creates (if needed) and starts the machines; provisions on first creation |
    | `vagrant status` | Shows the state of the folder's machines |
    | `vagrant ssh [name]` | Opens an SSH session into a machine |
    | `vagrant provision` | Replays the provisioners on a running machine |
    | `vagrant reload` | Restarts while rereading the Vagrantfile |
    | `vagrant halt` | Cleanly shuts the machines down (they still exist) |
    | `vagrant destroy` | **Deletes** the machines and their disks |
    | `vagrant box add / list / remove` | Manages downloaded boxes |
    | `vagrant global-status` | Lists all Vagrant machines on the computer |

    The folder containing the Vagrantfile is the **project folder**; Vagrant creates a `.vagrant/` subfolder in it that remembers local state (machine identifiers). It must not be versioned.

## 4. Lire et écrire un Vagrantfile / Reading and writing a Vagrantfile

=== "Français"

    **Une machine** : box, nom d'hôte, réseau privé à adresse fixe, ressources et un provisionnement par script.

=== "English"

    **One machine**: box, hostname, private network with a fixed address, resources and a script provisioner.

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "<NOM_DE_LA_BOX>"          # box de départ / base box
  config.vm.hostname = "srv1"
  config.vm.network "private_network", ip: "<IP_VM>"
  config.vm.provider "virtualbox" do |vb|    # ressources / resources
    vb.memory = 1024
    vb.cpus = 1
  end
  config.vm.provision "shell", inline: "apt-get update"
end
```

=== "Français"

    **Plusieurs machines** : chaque machine est déclarée par `config.vm.define`, avec ses propres paramètres.

=== "English"

    **Several machines**: each machine is declared with `config.vm.define`, with its own settings.

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "<NOM_DE_LA_BOX>"
  config.vm.define "web" do |web|
    web.vm.hostname = "web"
    web.vm.network "private_network", ip: "<IP_WEB>"
  end
  config.vm.define "db" do |db|
    db.vm.hostname = "db"
    db.vm.network "private_network", ip: "<IP_DB>"
  end
end
```

=== "Français"

    | Paramètre | Rôle |
    |---|---|
    | `config.vm.box` | Box de départ (on peut fixer sa version avec `config.vm.box_version`) |
    | `config.vm.network "private_network"` | Réseau privé de l'hôte (équivalent d'un réseau hôte seul), adresse fixe ou DHCP |
    | `config.vm.network "forwarded_port"` | Redirection d'un port de l'hôte vers la VM |
    | `config.vm.network "public_network"` | Réseau en pont, sur le réseau local |
    | `config.vm.provider "..." do ... end` | Réglages propres à un provider (mémoire, processeurs) |
    | `config.vm.provision "shell"` | Script en ligne (`inline:`) ou fichier (`path:`) exécuté dans la VM |
    | `config.vm.synced_folder` | Dossier partagé entre l'hôte et la VM |

    Le provisionnement s'exécute à la **première création** de la machine ; pour le rejouer, on utilise `vagrant provision` ou `vagrant up --provision`. Un script de provisionnement bien écrit peut être rejoué sans dommage (on parle d'**idempotence**, étudiée au chapitre 20).

=== "English"

    | Setting | Role |
    |---|---|
    | `config.vm.box` | Starting box (its version can be pinned with `config.vm.box_version`) |
    | `config.vm.network "private_network"` | Host private network (equivalent to a host-only network), fixed address or DHCP |
    | `config.vm.network "forwarded_port"` | Redirects a host port to the VM |
    | `config.vm.network "public_network"` | Bridged network, on the local network |
    | `config.vm.provider "..." do ... end` | Provider-specific settings (memory, processors) |
    | `config.vm.provision "shell"` | Inline script (`inline:`) or file (`path:`) run inside the VM |
    | `config.vm.synced_folder` | Folder shared between the host and the VM |

    Provisioning runs at the machine's **first creation**; to replay it, use `vagrant provision` or `vagrant up --provision`. A well-written provisioning script can be replayed without harm (this is called **idempotence**, studied in chapter 20).

## 5. Providers et portabilité / Providers and portability

=== "Français"

    Le même Vagrantfile peut cibler plusieurs providers, mais **chaque provider a ses particularités** :

    | Provider | Particularités |
    |---|---|
    | **VirtualBox** | Intégré à Vagrant ; `private_network` et `forwarded_port` pris en charge ; clones liés possibles (`vb.linked_clone = true`) |
    | **VMware (Workstation)** | Demande un module complémentaire et un utilitaire installés avec Vagrant ; boxes dédiées ; réglages via le fichier de configuration de la VM |
    | **Hyper-V** | Intégré à Vagrant ; nécessite une session **administrateur** ; la configuration réseau est limitée (le choix du commutateur se fait à l'exécution) ; dossiers partagés par SMB |
    | **libvirt (KVM)** | Module complémentaire, sous Linux (chapitre 12) |

    On choisit le provider à l'exécution : `vagrant up --provider=virtualbox` (ou la variable d'environnement `VAGRANT_DEFAULT_PROVIDER`). Les réglages propres à un provider se mettent dans un bloc `config.vm.provider` distinct par provider, ce qui permet d'avoir un seul Vagrantfile pour plusieurs outils.

    !!! warning "Un seul hyperviseur actif à la fois"
        Les règles des chapitres précédents s'appliquent : l'état de l'hyperviseur Windows (activé pour Hyper-V, désactivé pour VirtualBox et VMware) détermine le provider utilisable. Voir le [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md).

=== "English"

    The same Vagrantfile can target several providers, but **each provider has its own peculiarities**:

    | Provider | Peculiarities |
    |---|---|
    | **VirtualBox** | Built into Vagrant; `private_network` and `forwarded_port` supported; linked clones possible (`vb.linked_clone = true`) |
    | **VMware (Workstation)** | Requires an additional plugin and utility installed alongside Vagrant; dedicated boxes; settings through the VM's configuration file |
    | **Hyper-V** | Built into Vagrant; requires an **administrator** session; limited network configuration (the switch is chosen at runtime); shared folders over SMB |
    | **libvirt (KVM)** | Additional plugin, on Linux (chapter 12) |

    The provider is chosen at runtime: `vagrant up --provider=virtualbox` (or the `VAGRANT_DEFAULT_PROVIDER` environment variable). Provider-specific settings go in a separate `config.vm.provider` block per provider, allowing a single Vagrantfile for several tools.

    !!! warning "Only one hypervisor active at a time"
        The rules from previous chapters apply: the Windows hypervisor state (on for Hyper-V, off for VirtualBox and VMware) determines the usable provider. See [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md).

## 6. Bonnes pratiques et limites / Good practices and limits

=== "Français"

    - **Versionnez le Vagrantfile** (et les scripts de provisionnement) dans Git ; **excluez `.vagrant/`** avec un `.gitignore`.
    - **Fixez la version** de la box pour obtenir le même résultat dans le temps.
    - **N'y mettez pas de secrets** (mots de passe, clés) : le fichier est destiné à être partagé.
    - **Ne faites confiance qu'aux boxes de sources fiables** : une box est un système complet que vous allez exécuter.
    - **Surveillez l'espace disque** : les boxes et les disques de VM s'accumulent (`vagrant box list`, `vagrant global-status`).
    - **Gardez Vagrant pour le laboratoire** : il convient au développement et à l'apprentissage, pas à l'exploitation d'une infrastructure.

    !!! tip "À retenir"
        - Décrire un environnement dans un fichier le rend reproductible, partageable et versionnable.
        - Quatre notions : Vagrantfile, box, provider, provisioner.
        - Cycle : `init`, `up`, `ssh`, `provision`, `halt`, `destroy` ; `destroy` supprime les machines.
        - `config.vm.define` déclare plusieurs machines dans un même fichier.
        - Un même Vagrantfile peut viser plusieurs providers, chacun avec ses limites.
        - Versionner le Vagrantfile, exclure `.vagrant/`, fixer la version de la box, ne pas y mettre de secrets.

=== "English"

    - **Version the Vagrantfile** (and provisioning scripts) in Git; **exclude `.vagrant/`** with a `.gitignore`.
    - **Pin the box version** to get the same result over time.
    - **Put no secrets in it** (passwords, keys): the file is meant to be shared.
    - **Only trust boxes from reliable sources**: a box is a complete system that you are going to run.
    - **Watch disk space**: boxes and VM disks pile up (`vagrant box list`, `vagrant global-status`).
    - **Keep Vagrant for the lab**: it suits development and learning, not operating an infrastructure.

    !!! tip "Key takeaways"
        - Describing an environment in a file makes it reproducible, shareable and versionable.
        - Four notions: Vagrantfile, box, provider, provisioner.
        - Lifecycle: `init`, `up`, `ssh`, `provision`, `halt`, `destroy`; `destroy` deletes the machines.
        - `config.vm.define` declares several machines in a single file.
        - The same Vagrantfile can target several providers, each with its limits.
        - Version the Vagrantfile, exclude `.vagrant/`, pin the box version, put no secrets in it.

## Pour vérifier votre compréhension / Check your understanding

=== "Français"

    ??? question "Quelle est la différence entre une box et une machine virtuelle créée par Vagrant ?"
        La box est une **image de base** empaquetée, téléchargée une fois et réutilisable. La machine virtuelle est une **instance** créée à partir de cette box, avec ses propres disques, son réseau et ses réglages, et qui peut être détruite sans toucher à la box.

    ??? question "Quelle différence y a-t-il entre `vagrant halt` et `vagrant destroy` ?"
        `vagrant halt` éteint proprement les machines, qui existent toujours et redémarrent avec `vagrant up`. `vagrant destroy` **supprime** les machines et leurs disques : la prochaine `vagrant up` repart de la box et rejoue le provisionnement.

    ??? question "Pourquoi versionne-t-on le Vagrantfile mais pas le dossier `.vagrant/` ?"
        Le Vagrantfile décrit l'environnement : le partager permet de le reproduire. Le dossier `.vagrant/` ne contient que l'état **local** (identifiants des machines sur ce poste) : il n'a aucun sens ailleurs et ferait des conflits.

=== "English"

    ??? question "What is the difference between a box and a virtual machine created by Vagrant?"
        The box is a packaged **base image**, downloaded once and reusable. The virtual machine is an **instance** created from this box, with its own disks, network and settings, and can be destroyed without touching the box.

    ??? question "What is the difference between `vagrant halt` and `vagrant destroy`?"
        `vagrant halt` cleanly shuts the machines down, which still exist and restart with `vagrant up`. `vagrant destroy` **deletes** the machines and their disks: the next `vagrant up` starts again from the box and replays provisioning.

    ??? question "Why version the Vagrantfile but not the `.vagrant/` folder?"
        The Vagrantfile describes the environment: sharing it lets others reproduce it. The `.vagrant/` folder only holds **local** state (machine identifiers on this computer): it makes no sense elsewhere and would cause conflicts.

## Ressources du chapitre / Chapter resources

- [Lab 6.1 : Premier Vagrantfile / First Vagrantfile](lab-6-1-premier-vagrantfile.md)
- [Lab 6.2 : Environnement multi-machines avec provisioning / Multi-machine environment with provisioning](lab-6-2-environnement-multi-machines-avec-provisioning.md)
- [Lab 6.3 : Même environnement sur un autre provider / Same environment on another provider](lab-6-3-meme-environnement-sur-un-autre-provider.md)
- [Quiz du chapitre / Chapter quiz](quiz.md)
- [Glossaire / Glossary](../../annexes/glossaire.md)
- [Environnement de TP / Lab environment](../../annexes/environnement.md)