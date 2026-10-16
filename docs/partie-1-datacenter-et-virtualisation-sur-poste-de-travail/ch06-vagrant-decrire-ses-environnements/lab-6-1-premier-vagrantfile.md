# Lab 6.1 : Premier Vagrantfile / First Vagrantfile

| | |
|---|---|
| **Durée / Duration** | 60 à 75 min / 60 to 75 min |
| **Niveau / Level** | Guidé / Guided |
| **Prérequis / Prerequisites** | [Chapitre 6 : cours](cours.md) ; VirtualBox installé ([Lab 3.1](../ch03-virtualbox-et-vmware-workstation/lab-3-1-virtualbox-une-vm-ubuntu-server.md)) ; [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) réalisé |
| **Ressources minimales / Minimum resources** | **VM** : 1 vCPU, 1 Go de RAM, disque dynamique selon la box. **Hôte** : au moins 2 Go de RAM libres, 10 Go d'espace disque libre (téléchargement de la box et disque de la VM), accès Internet. L'hyperviseur Windows doit être **désactivé** (provider VirtualBox). / **VM**: 1 vCPU, 1 GB RAM, dynamic disk depending on the box. **Host**: at least 2 GB of free RAM, 10 GB of free disk space (box download and VM disk), Internet access. The Windows hypervisor must be **off** (VirtualBox provider). |
| **Acquis visés / Learning outcomes** | AA2 |

=== "Français"

    !!! abstract "Objectifs"
        - Installer Vagrant et vérifier son fonctionnement avec le provider VirtualBox
        - Écrire un Vagrantfile : box, nom d'hôte, réseau privé, redirection de port, ressources
        - Provisionner la VM avec un script et tester le service obtenu
        - Parcourir le cycle de vie complet : `up`, `ssh`, `provision`, `halt`, `destroy`
        - Constater la reproductibilité en reconstruisant l'environnement

=== "English"

    !!! abstract "Objectives"
        - Install Vagrant and check it works with the VirtualBox provider
        - Write a Vagrantfile: box, hostname, private network, port forwarding, resources
        - Provision the VM with a script and test the resulting service
        - Go through the full lifecycle: `up`, `ssh`, `provision`, `halt`, `destroy`
        - Observe reproducibility by rebuilding the environment

## Contexte et schéma / Context and diagram

=== "Français"

    Vous avez construit la VM de référence à la main (Lab 3.1). Vous allez maintenant obtenir une machine équivalente, avec un **serveur web en plus**, à partir d'un seul fichier. Les adresses suivent le plan de l'annexe [Environnement](../../annexes/environnement.md) (`<IP_VM>`), et la box choisie est notée `<NOM_DE_LA_BOX>`.

    ```mermaid
    flowchart LR
      F["Vagrantfile"] --> V["vagrant up"]
      B["Box<br/>(image de base)"] --> V
      V --> M["VM VirtualBox<br/>+ serveur web"]
      H["Poste hôte"] -- "réseau privé / port 8080" --> M
    ```

    !!! warning "Choix de la box"
        Choisissez une box **Ubuntu Server** compatible VirtualBox, issue d'une **source fiable** sur le catalogue de boxes (par exemple celle indiquée par votre enseignant, ou `bento/ubuntu-24.04` : **à vérifier sur le catalogue avant la séance**). Notez son nom exact et sa version : ils font partie du livrable.

=== "English"

    You built the reference VM by hand (Lab 3.1). You will now get an equivalent machine, with a **web server on top**, from a single file. Addresses follow the plan in the [Environment](../../annexes/environnement.md) annex (`<IP_VM>`), and the chosen box is written `<NOM_DE_LA_BOX>`.

    ```mermaid
    flowchart LR
      F["Vagrantfile"] --> V["vagrant up"]
      B["Box<br/>(base image)"] --> V
      V --> M["VirtualBox VM<br/>+ web server"]
      H["Host computer"] -- "private network / port 8080" --> M
    ```

    !!! warning "Choosing the box"
        Choose an **Ubuntu Server** box compatible with VirtualBox, from a **trusted source** in the box catalog (for example the one given by your instructor, or `bento/ubuntu-24.04`: **to be checked on the catalog before the session**). Note its exact name and version: they are part of the deliverable.

## Étapes / Steps

### Étape 1 : Installer Vagrant / Step 1: Install Vagrant

=== "Français"

    1. Téléchargez Vagrant depuis le **site officiel** de l'éditeur (HashiCorp) et lancez l'installateur avec les options par défaut. Un redémarrage peut être demandé.
    2. Ouvrez un **nouveau** PowerShell et vérifiez :

=== "English"

    1. Download Vagrant from the vendor's **official website** (HashiCorp) and run the installer with default options. A restart may be requested.
    2. Open a **new** PowerShell and check:

```powershell
vagrant --version
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" --version
(Get-CimInstance Win32_ComputerSystem).HypervisorPresent
```

=== "Français"

    Résultat attendu : les versions de Vagrant et de VirtualBox, et `HypervisorPresent` à `False` (hyperviseur Windows désactivé, voir le Lab 2.1).

    !!! success "Point de contrôle"
        - [ ] `vagrant --version` affiche une version
        - [ ] `HypervisorPresent` vaut `False`

=== "English"

    Expected result: the Vagrant and VirtualBox versions, and `HypervisorPresent` at `False` (Windows hypervisor off, see Lab 2.1).

    !!! success "Checkpoint"
        - [ ] `vagrant --version` shows a version
        - [ ] `HypervisorPresent` is `False`

### Étape 2 : Créer le projet / Step 2: Create the project

```powershell
mkdir "<DOSSIER_PROJETS>\vagrant-premier"
cd "<DOSSIER_PROJETS>\vagrant-premier"
vagrant init <NOM_DE_LA_BOX>
vagrant validate
Get-Content .\Vagrantfile | Select-String -Pattern "^\s*[^#\s]"
```

=== "Français"

    `vagrant init` crée un Vagrantfile presque entièrement composé de commentaires. La dernière commande n'affiche que les lignes actives : vous devriez voir `Vagrant.configure("2") do |config|`, la ligne `config.vm.box` et `end`.

    !!! success "Point de contrôle"
        - [ ] Un fichier `Vagrantfile` existe dans le dossier du projet
        - [ ] `vagrant validate` renvoie « Vagrantfile validated successfully »

=== "English"

    `vagrant init` creates a Vagrantfile made almost entirely of comments. The last command only displays active lines: you should see `Vagrant.configure("2") do |config|`, the `config.vm.box` line and `end`.

    !!! success "Checkpoint"
        - [ ] A `Vagrantfile` exists in the project folder
        - [ ] `vagrant validate` returns "Vagrantfile validated successfully"

### Étape 3 : Écrire le Vagrantfile / Step 3: Write the Vagrantfile

=== "Français"

    Remplacez le contenu du Vagrantfile par le suivant (éditeur au choix : Bloc-notes, VS Code…). Remplacez `<NOM_DE_LA_BOX>` et `<IP_VM>` :

=== "English"

    Replace the Vagrantfile content with the following (any editor: Notepad, VS Code...). Replace `<NOM_DE_LA_BOX>` and `<IP_VM>`:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "<NOM_DE_LA_BOX>"
  config.vm.hostname = "srv-vagrant"
  config.vm.network "private_network", ip: "<IP_VM>"
  config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
  config.vm.provider "virtualbox" do |vb|
    vb.name = "srv-vagrant"
    vb.memory = 1024
    vb.cpus = 1
  end
end
```

```powershell
vagrant validate
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] `vagrant validate` ne signale aucune erreur

=== "English"

    !!! success "Checkpoint"
        - [ ] `vagrant validate` reports no error

### Étape 4 : Créer la machine / Step 4: Create the machine

```powershell
$t = Measure-Command { vagrant up }
$t.TotalMinutes
vagrant status
vagrant box list
```

=== "Français"

    Au premier lancement, Vagrant **télécharge la box** (quelques centaines de Mo à quelques Go, selon la box), importe la machine dans VirtualBox, configure le réseau puis démarre. Observez les messages affichés. Notez la durée (`TotalMinutes`) : elle sert de référence pour l'étape 8. Ouvrez aussi **VirtualBox** : la VM `srv-vagrant` y apparaît.

    !!! success "Point de contrôle"
        - [ ] `vagrant status` indique `running`
        - [ ] `vagrant box list` affiche la box avec sa version

=== "English"

    On first run, Vagrant **downloads the box** (a few hundred MB to a few GB, depending on the box), imports the machine into VirtualBox, configures the network then starts. Watch the messages displayed. Note the duration (`TotalMinutes`): it serves as a reference for step 8. Also open **VirtualBox**: the `srv-vagrant` VM appears there.

    !!! success "Checkpoint"
        - [ ] `vagrant status` shows `running`
        - [ ] `vagrant box list` shows the box with its version

### Étape 5 : Explorer la machine / Step 5: Explore the machine

```powershell
vagrant ssh
```

```bash
hostname
cat /etc/os-release | head -3
ip -br addr
nproc
free -h
ls /vagrant
exit
```

=== "Français"

    Résultat attendu : le nom d'hôte `srv-vagrant`, la version d'Ubuntu, deux interfaces dont une portant `<IP_VM>`, 1 processeur, environ 1 Go de mémoire. Le dossier `/vagrant` est un **dossier partagé** : il contient les fichiers de votre dossier de projet, dont le Vagrantfile.

    !!! success "Point de contrôle"
        - [ ] Les caractéristiques de la VM correspondent au Vagrantfile
        - [ ] `ls /vagrant` montre le `Vagrantfile`

=== "English"

    Expected result: hostname `srv-vagrant`, the Ubuntu version, two interfaces one of which carries `<IP_VM>`, 1 processor, about 1 GB of memory. The `/vagrant` folder is a **shared folder**: it contains the files of your project folder, including the Vagrantfile.

    !!! success "Checkpoint"
        - [ ] The VM's characteristics match the Vagrantfile
        - [ ] `ls /vagrant` shows the `Vagrantfile`

### Étape 6 : Provisionner un serveur web / Step 6: Provision a web server

=== "Français"

    Ajoutez le bloc de provisionnement suivant dans le Vagrantfile, juste avant le dernier `end` :

=== "English"

    Add the following provisioning block to the Vagrantfile, just before the last `end`:

```ruby
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y nginx
  SHELL
```

```powershell
vagrant validate
vagrant provision
```

=== "Français"

    Le provisionnement ne s'exécute pas tout seul sur une machine déjà créée : `vagrant provision` le rejoue. Testez ensuite le service depuis l'hôte, par le **réseau privé** et par la **redirection de port** :

=== "English"

    Provisioning does not run by itself on an already created machine: `vagrant provision` replays it. Then test the service from the host, through the **private network** and through **port forwarding**:

```powershell
curl.exe http://<IP_VM>
curl.exe http://127.0.0.1:8080
```

=== "Français"

    Résultat attendu : dans les deux cas, la page d'accueil du serveur web.

    !!! success "Point de contrôle"
        - [ ] Les deux requêtes renvoient la page d'accueil
        - [ ] Vous savez expliquer la différence entre les deux chemins d'accès

=== "English"

    Expected result: in both cases, the web server's welcome page.

    !!! success "Checkpoint"
        - [ ] Both requests return the welcome page
        - [ ] You can explain the difference between the two access paths

### Étape 7 : Éteindre, redémarrer, détruire / Step 7: Halt, restart, destroy

```powershell
vagrant halt
vagrant status
vagrant up
vagrant status
```

=== "Français"

    `vagrant up` sur une machine existante ne rejoue **pas** le provisionnement : le serveur web est déjà installé. Vérifiez avec `curl.exe http://<IP_VM>`. Puis **détruisez** la machine :

=== "English"

    `vagrant up` on an existing machine does **not** replay provisioning: the web server is already installed. Check with `curl.exe http://<IP_VM>`. Then **destroy** the machine:

```powershell
vagrant destroy -f
vagrant status
vagrant global-status
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Après `halt`, l'état est `poweroff` ; après `up`, il est `running`
        - [ ] Après `destroy`, la VM a disparu de VirtualBox

=== "English"

    !!! success "Checkpoint"
        - [ ] After `halt`, the state is `poweroff`; after `up`, it is `running`
        - [ ] After `destroy`, the VM has disappeared from VirtualBox

### Étape 8 : Reconstruire à l'identique / Step 8: Rebuild identically

```powershell
$t2 = Measure-Command { vagrant up }
$t2.TotalMinutes
curl.exe http://<IP_VM>
```

=== "Français"

    La box est déjà téléchargée : la reconstruction est plus courte que la première création, **sans aucune intervention manuelle**, et le serveur web est à nouveau installé grâce au provisionnement. Comparez les deux durées.

=== "English"

    The box is already downloaded: the rebuild is shorter than the first creation, **without any manual intervention**, and the web server is installed again thanks to provisioning. Compare the two durations.

### Étape 9 : Préparer le dossier à être partagé / Step 9: Prepare the folder to be shared

=== "Français"

    Créez un fichier `.gitignore` qui exclut l'état local de Vagrant, puis vérifiez le contenu du dossier :

=== "English"

    Create a `.gitignore` file that excludes Vagrant's local state, then check the folder content:

```powershell
Set-Content -Path .gitignore -Value ".vagrant/"
Get-ChildItem -Force | Select-Object Name
```

=== "Français"

    Seuls le `Vagrantfile` et le `.gitignore` seraient à versionner ; `.vagrant/` est propre à votre poste.

### Étape 10 : Fiche de synthèse / Step 10: Summary sheet

=== "Français"

    Remplissez la fiche ci-dessous : c'est le **livrable** du lab. Joignez votre Vagrantfile.

    | Rubrique | Valeur |
    |---|---|
    | Versions de Vagrant et de VirtualBox | … / … |
    | Nom et version de la box (`vagrant box list`) | … |
    | Durée du premier `vagrant up` / de la reconstruction (min) | … / … |
    | Caractéristiques observées dans la VM (nom d'hôte, vCPU, mémoire) | … |
    | Adresse du réseau privé | … |
    | Résultat de `curl.exe` par le réseau privé / par le port 8080 | … / … |
    | `vagrant up` sur machine existante : provisionnement rejoué ? | oui / non |
    | Fichiers à versionner / à exclure | … / … |

=== "English"

    Fill in the sheet below: this is the lab **deliverable**. Attach your Vagrantfile.

    | Item | Value |
    |---|---|
    | Vagrant and VirtualBox versions | … / … |
    | Box name and version (`vagrant box list`) | … |
    | Duration of the first `vagrant up` / of the rebuild (min) | … / … |
    | Characteristics observed in the VM (hostname, vCPU, memory) | … |
    | Private network address | … |
    | `curl.exe` result through the private network / through port 8080 | … / … |
    | `vagrant up` on an existing machine: provisioning replayed? | yes / no |
    | Files to version / to exclude | … / … |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi la reconstruction est-elle plus rapide que la première création ?"
        La box a déjà été téléchargée et se trouve dans le cache local de Vagrant : seule la création de la machine à partir de cette image, le réseau et le provisionnement sont à refaire.

    ??? question "Pourquoi le provisionnement ne s'est-il pas rejoué lors du second `vagrant up` sur la machine existante ?"
        Le provisionnement est exécuté à la **première création** de la machine. Sur une machine déjà créée, il faut le demander explicitement (`vagrant provision` ou `vagrant up --provision`). Après un `destroy`, la création suivante est une première création.

    ??? question "Que gagne-t-on, par rapport au Lab 3.1, à décrire la VM dans un fichier ?"
        La reproductibilité (même résultat à chaque fois), le partage (copier le fichier suffit), la traçabilité (le fichier documente la configuration) et la rapidité de reconstruction (détruire et recréer sans effort).

=== "English"

    ??? question "Why is the rebuild faster than the first creation?"
        The box has already been downloaded and sits in Vagrant's local cache: only the creation of the machine from that image, the network and provisioning have to be redone.

    ??? question "Why was provisioning not replayed at the second `vagrant up` on the existing machine?"
        Provisioning runs at the machine's **first creation**. On an already created machine, it must be requested explicitly (`vagrant provision` or `vagrant up --provision`). After a `destroy`, the next creation is a first creation.

    ??? question "Compared with Lab 3.1, what is gained by describing the VM in a file?"
        Reproducibility (same result every time), sharing (copying the file is enough), traceability (the file documents the configuration) and fast rebuilds (destroy and recreate with no effort).

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | `vagrant` : commande introuvable | Ouvrir un **nouveau** PowerShell après l'installation ; redémarrer si demandé |
    | Le téléchargement de la box échoue | Nom ou version de la box incorrects ; accès Internet ; proxy de l'établissement ; box non fournie pour le provider VirtualBox |
    | « VT-x is not available » ou VM qui ne démarre pas | Virtualisation désactivée ou hyperviseur Windows actif : voir le [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) |
    | « Timed out while waiting for the machine to boot » | Démarrage lent ou VM bloquée : relancer ; vérifier la virtualisation ; ouvrir la VM dans VirtualBox pour voir l'écran |
    | Erreur sur le réseau privé (adresse refusée) | Adresse déjà utilisée ou hors de la plage autorisée pour le réseau hôte seul de VirtualBox : choisir une autre adresse du plan |
    | Le port 8080 est déjà utilisé sur l'hôte | Choisir un autre port hôte dans `forwarded_port` |
    | `curl.exe` n'obtient pas de réponse | Provisionnement non rejoué (`vagrant provision`) ; pare-feu Windows ; mauvaise adresse |
    | Le dossier `/vagrant` est vide ou absent | Dossier partagé non monté : les outils invité de la box peuvent manquer ; voir les messages de `vagrant up` |

=== "English"

    | Symptom | Hints |
    |---|---|
    | `vagrant`: command not found | Open a **new** PowerShell after installation; restart if requested |
    | The box download fails | Wrong box name or version; Internet access; institution proxy; box not provided for the VirtualBox provider |
    | "VT-x is not available" or VM does not start | Virtualization disabled or Windows hypervisor active: see [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) |
    | "Timed out while waiting for the machine to boot" | Slow boot or stuck VM: retry; check virtualization; open the VM in VirtualBox to see the screen |
    | Private network error (address rejected) | Address already used or outside the range allowed for VirtualBox's host-only network: choose another address from the plan |
    | Port 8080 is already in use on the host | Choose another host port in `forwarded_port` |
    | `curl.exe` gets no answer | Provisioning not replayed (`vagrant provision`); Windows firewall; wrong address |
    | The `/vagrant` folder is empty or missing | Shared folder not mounted: the box's guest tools may be missing; see the `vagrant up` messages |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    Gardez ce dossier de projet : le [Lab 6.2](lab-6-2-environnement-multi-machines-avec-provisioning.md) s'en inspire. Pour libérer de l'espace : `vagrant destroy -f` dans le dossier, puis `vagrant box remove <NOM_DE_LA_BOX>` si vous ne réutilisez pas la box. `vagrant global-status --prune` retire de la liste les machines qui n'existent plus.

=== "English"

    Keep this project folder: [Lab 6.2](lab-6-2-environnement-multi-machines-avec-provisioning.md) builds on it. To free up space: `vagrant destroy -f` in the folder, then `vagrant box remove <NOM_DE_LA_BOX>` if you do not reuse the box. `vagrant global-status --prune` removes machines that no longer exist from the list.

[Retour au chapitre 6 / Back to chapter 6](cours.md)