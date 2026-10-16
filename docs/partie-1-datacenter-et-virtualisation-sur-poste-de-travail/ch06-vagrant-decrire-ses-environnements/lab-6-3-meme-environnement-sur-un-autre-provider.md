# Lab 6.3 : Même environnement sur un autre provider / Same Environment on Another Provider

| | |
|---|---|
| **Durée / Duration** | 75 à 90 min / 75 to 90 min |
| **Niveau / Level** | Semi-guidé / Semi-guided |
| **Prérequis / Prerequisites** | [Chapitre 6 : cours](cours.md) ; [Lab 6.2](lab-6-2-environnement-multi-machines-avec-provisioning.md) réalisé ; Hyper-V installé ([Lab 4.1](../ch04-hyper-v/lab-4-1-activer-hyper-v-et-creer-une-vm.md)) |
| **Ressources minimales / Minimum resources** | **VM** : 2 VM (`app`, `web`), 1 vCPU et 1 Go de RAM chacune. **Hôte** : Windows Pro, Enterprise ou Éducation avec Hyper-V, compte administrateur, au moins 4 Go de RAM libres, 15 Go d'espace disque libre (nouvelle box), accès Internet, un redémarrage. L'hyperviseur Windows doit être **activé**. / **VMs**: 2 VMs (`app`, `web`), 1 vCPU and 1 GB RAM each. **Host**: Windows Pro, Enterprise or Education with Hyper-V, administrator account, at least 4 GB of free RAM, 15 GB of free disk space (new box), Internet access, one restart. The Windows hypervisor must be **on**. |
| **Acquis visés / Learning outcomes** | AA2 |

=== "Français"

    !!! abstract "Objectifs"
        - Exécuter un même projet Vagrant avec un second provider (Hyper-V)
        - Repérer ce qui change d'un provider à l'autre : box, réseau, redirection de port, dossiers partagés
        - Adapter un Vagrantfile pour qu'il fonctionne avec les deux providers
        - Mesurer la portabilité réelle d'un environnement décrit comme du code

=== "English"

    !!! abstract "Objectives"
        - Run the same Vagrant project with a second provider (Hyper-V)
        - Spot what changes from one provider to the other: box, network, port forwarding, shared folders
        - Adapt a Vagrantfile so that it works with both providers
        - Measure the real portability of an environment described as code

## Contexte et schéma / Context and diagram

=== "Français"

    Vous reprenez l'environnement du Lab 6.2 (machines `app` et `web`) et vous le faites tourner sur **Hyper-V**. L'objectif n'est pas seulement que cela marche : c'est de découvrir **ce qui ne se transpose pas tel quel**. Les adresses sont distribuées par le *Default Switch* (DHCP), ce qui change la façon dont `web` trouve `app`.

    ```mermaid
    flowchart LR
      P["Projet Vagrant<br/>(Vagrantfile + scripts)"] --> VB["Provider VirtualBox<br/>(Lab 6.2)"]
      P --> HV["Provider Hyper-V<br/>(ce lab)"]
    ```

    !!! warning "Ordre des opérations"
        VirtualBox et Hyper-V ne cohabitent pas bien (chapitre 2). **Détruisez les machines VirtualBox du Lab 6.2 avant** de réactiver l'hyperviseur Windows (étape 1). Toutes les commandes Vagrant de ce lab se lancent depuis **PowerShell en administrateur**, obligatoire avec le provider Hyper-V.

=== "English"

    You take the Lab 6.2 environment (machines `app` and `web`) and run it on **Hyper-V**. The goal is not only to make it work: it is to discover **what does not carry over as is**. Addresses are handed out by the *Default Switch* (DHCP), which changes how `web` finds `app`.

    ```mermaid
    flowchart LR
      P["Vagrant project<br/>(Vagrantfile + scripts)"] --> VB["VirtualBox provider<br/>(Lab 6.2)"]
      P --> HV["Hyper-V provider<br/>(this lab)"]
    ```

    !!! warning "Order of operations"
        VirtualBox and Hyper-V do not coexist well (chapter 2). **Destroy the VirtualBox machines from Lab 6.2 before** turning the Windows hypervisor back on (step 1). All Vagrant commands in this lab are run from **PowerShell as administrator**, which is mandatory with the Hyper-V provider.

## Étapes / Steps

### Étape 1 : Préparer le terrain / Step 1: Prepare the ground

=== "Français"

    1. Dans le dossier du Lab 6.2, détruisez les machines VirtualBox (hyperviseur Windows encore désactivé) :
    2. Copiez le projet (sans l'état local) dans un nouveau dossier, qui servira à ce lab :

=== "English"

    1. In the Lab 6.2 folder, destroy the VirtualBox machines (Windows hypervisor still off):
    2. Copy the project (without local state) into a new folder, which will be used for this lab:

```powershell
cd "<DOSSIER_PROJETS>\vagrant-multi"
vagrant destroy -f
mkdir "<DOSSIER_PROJETS>\vagrant-multi-hv\provision"
Copy-Item .\Vagrantfile, .gitignore "<DOSSIER_PROJETS>\vagrant-multi-hv\"
Copy-Item .\provision\* "<DOSSIER_PROJETS>\vagrant-multi-hv\provision\"
```

=== "Français"

    3. Réactivez l'hyperviseur Windows comme au [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) (`bcdedit /set hypervisorlaunchtype auto`, puis redémarrage), ouvrez **PowerShell en administrateur** et vérifiez :

=== "English"

    3. Turn the Windows hypervisor back on as in [Lab 2.1](../ch02-principes-de-la-virtualisation/lab-2-1-preparer-le-poste-pour-la-virtualisation.md) (`bcdedit /set hypervisorlaunchtype auto`, then restart), open **PowerShell as administrator** and check:

```powershell
(Get-CimInstance Win32_ComputerSystem).HypervisorPresent
Get-Service vmms
Get-VMSwitch -Name "Default Switch"
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Les machines du Lab 6.2 sont détruites ; le nouveau dossier contient le Vagrantfile, `.gitignore` et `provision/`
        - [ ] `HypervisorPresent` vaut `True`, `vmms` est `Running`, le *Default Switch* existe

=== "English"

    !!! success "Checkpoint"
        - [ ] The Lab 6.2 machines are destroyed; the new folder contains the Vagrantfile, `.gitignore` and `provision/`
        - [ ] `HypervisorPresent` is `True`, `vmms` is `Running`, the *Default Switch* exists

### Étape 2 : Trouver et télécharger une box Hyper-V / Step 2: Find and download a Hyper-V box

=== "Français"

    La box du Lab 6.1 est prévue pour VirtualBox : **elle ne fonctionne pas avec Hyper-V**. Cherchez sur le catalogue de boxes une box **Ubuntu Server compatible Hyper-V**, d'une source fiable (celle indiquée par votre enseignant en priorité ; sinon une box dont la page affiche le provider `hyperv`). Notez-la `<NOM_BOX_HYPERV>`, puis téléchargez-la :

=== "English"

    The Lab 6.1 box is designed for VirtualBox: **it does not work with Hyper-V**. Look in the box catalog for an **Ubuntu Server box compatible with Hyper-V**, from a trusted source (the one given by your instructor first; otherwise a box whose page lists the `hyperv` provider). Call it `<NOM_BOX_HYPERV>`, then download it:

```powershell
vagrant box add <NOM_BOX_HYPERV> --provider hyperv
vagrant box list
```

=== "Français"

    `vagrant box list` indique, entre parenthèses, le provider de chaque box : vous devez voir `hyperv` pour la nouvelle box.

    !!! success "Point de contrôle"
        - [ ] La box Hyper-V est téléchargée et listée avec le provider `hyperv`

=== "English"

    `vagrant box list` shows each box's provider in parentheses: you should see `hyperv` for the new box.

    !!! success "Checkpoint"
        - [ ] The Hyper-V box is downloaded and listed with the `hyperv` provider

### Étape 3 : Prévoir ce qui va changer / Step 3: Predict what will change

=== "Français"

    **Avant d'exécuter quoi que ce soit**, consultez la documentation du provider Hyper-V de Vagrant (site de l'éditeur) et complétez ce tableau de prévisions. Vous le comparerez aux observations à l'étape 6.

    | Aspect | VirtualBox (Lab 6.2) | Hyper-V : votre prévision |
    |---|---|---|
    | Réseau privé avec adresse fixe (`private_network`) | pris en charge | … |
    | Redirection de port (`forwarded_port`) | pris en charge | … |
    | Dossier partagé `/vagrant` | partage VirtualBox | … |
    | Droits nécessaires pour `vagrant up` | utilisateur | … |
    | Nom d'une machine pour les autres machines | adresse fixe du plan | … |

=== "English"

    **Before running anything**, read the Vagrant Hyper-V provider documentation (vendor's site) and complete this prediction table. You will compare it with observations in step 6.

    | Aspect | VirtualBox (Lab 6.2) | Hyper-V: your prediction |
    |---|---|---|
    | Private network with fixed address (`private_network`) | supported | … |
    | Port forwarding (`forwarded_port`) | supported | … |
    | Shared folder `/vagrant` | VirtualBox sharing | … |
    | Rights needed for `vagrant up` | user | … |
    | A machine's name seen by the other machines | fixed address from the plan | … |

### Étape 4 : Adapter le Vagrantfile / Step 4: Adapt the Vagrantfile

=== "Français"

    Réécrivez le `Vagrantfile` du nouveau dossier pour qu'il serve **les deux providers**. Le choix se fait par la variable d'environnement `VAGRANT_DEFAULT_PROVIDER` (que vous définirez à l'étape 5). Ce que vous devez obtenir :

    - une **box différente** selon le provider ;
    - avec Hyper-V : **pas d'adresse fixe, pas de redirection de port**, le réseau est le *Default Switch* choisi dans le fichier (`public_network` avec `bridge: "Default Switch"`) ; les dossiers partagés (SMB, qui demande des identifiants Windows) sont **désactivés** ;
    - avec Hyper-V, `web` joint `app` par son **nom** (`app.mshome.net`, le domaine fourni par le *Default Switch*) au lieu d'une adresse ;
    - les réglages de ressources déclarés **pour chaque provider** (bloc `hyperv`).

    Squelette à compléter (les zones `...` sont à écrire) :

=== "English"

    Rewrite the new folder's `Vagrantfile` so that it serves **both providers**. The choice is made through the `VAGRANT_DEFAULT_PROVIDER` environment variable (which you will set in step 5). What you must get:

    - a **different box** depending on the provider;
    - with Hyper-V: **no fixed address, no port forwarding**, the network is the *Default Switch* chosen in the file (`public_network` with `bridge: "Default Switch"`); shared folders (SMB, which asks for Windows credentials) are **disabled**;
    - with Hyper-V, `web` reaches `app` by its **name** (`app.mshome.net`, the domain provided by the *Default Switch*) instead of an address;
    - resource settings declared **for each provider** (`hyperv` block).

    Skeleton to complete (the `...` areas are yours to write):

```ruby
HYPERV = (ENV["VAGRANT_DEFAULT_PROVIDER"] == "hyperv")

MACHINES = {
  "app" => { ip: "<IP_APP>", script: "provision/app.sh" },
  "web" => { ip: "<IP_WEB>", script: "provision/web.sh" }
}

BOX      = HYPERV ? "<NOM_BOX_HYPERV>" : "<NOM_DE_LA_BOX>"
APP_ADDR = HYPERV ? "app.mshome.net" : MACHINES["app"][:ip]

Vagrant.configure("2") do |config|
  config.vm.box = BOX
  config.vm.synced_folder ".", "/vagrant", disabled: true

  MACHINES.each do |name, cfg|
    config.vm.define name do |m|
      m.vm.hostname = name

      if HYPERV
        # ... réseau : public_network avec bridge: "Default Switch"
      else
        # ... réseau : private_network + forwarded_port (web uniquement)
      end

      m.vm.provider "virtualbox" do |vb|
        # ... nom, mémoire, processeurs
      end
      m.vm.provider "hyperv" do |h|
        # ... vmname, memory, cpus
      end

      m.vm.provision "file", source: "provision/a-propos.txt", destination: "/home/vagrant/a-propos.txt"
      m.vm.provision "shell", path: cfg[:script], args: [APP_ADDR]
    end
  end
end
```

=== "Français"

    Indices : le bloc `hyperv` accepte `vmname`, `memory` et `cpus` ; l'option `bridge:` de `public_network` nomme le commutateur ; la redirection de port n'existe pas avec ce provider. Validez :

=== "English"

    Hints: the `hyperv` block accepts `vmname`, `memory` and `cpus`; the `bridge:` option of `public_network` names the switch; port forwarding does not exist with this provider. Validate:

```powershell
vagrant validate
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] `vagrant validate` ne signale aucune erreur
        - [ ] Chaque ligne spécifique à un provider est dans une branche ou un bloc dédié

=== "English"

    !!! success "Checkpoint"
        - [ ] `vagrant validate` reports no error
        - [ ] Each provider-specific line is in a dedicated branch or block

### Étape 5 : Construire avec Hyper-V / Step 5: Build with Hyper-V

```powershell
$env:VAGRANT_DEFAULT_PROVIDER = "hyperv"
$t = Measure-Command { vagrant up }
$t.TotalMinutes
vagrant status
Get-VM | Where-Object Name -like "tp63-*" | Select-Object Name, State
Get-VMNetworkAdapter -VMName "tp63-web","tp63-app" | Select-Object VMName, SwitchName, IPAddresses
```

=== "Français"

    Lisez attentivement les **messages** de Vagrant (avertissements sur le réseau, les dossiers, éventuels échecs) : ils vous diront ce que le provider ne prend pas en charge. Les deux VM apparaissent dans le **Gestionnaire Hyper-V** ; leurs adresses viennent du *Default Switch*. Notez l'adresse de `web` (`<IP_WEB_HV>`) et vérifiez que `web` voit `app` par son nom :

=== "English"

    Read Vagrant's **messages** carefully (warnings about network, folders, possible failures): they will tell you what the provider does not support. The two VMs appear in **Hyper-V Manager**; their addresses come from the *Default Switch*. Note `web`'s address (`<IP_WEB_HV>`) and check that `web` sees `app` by name:

```powershell
vagrant ssh web -c "getent hosts app.mshome.net"
curl.exe http://<IP_WEB_HV>
```

=== "Français"

    Résultat attendu : la résolution du nom renvoie l'adresse de `app`, et la requête vers `web` renvoie la page produite par `app`. Si la résolution de nom échoue, notez le message et passez au dépannage ci-dessous.

    !!! success "Point de contrôle"
        - [ ] Les deux machines sont `running` dans Hyper-V
        - [ ] `web` relaie bien la page de `app`

=== "English"

    Expected result: name resolution returns `app`'s address, and the request to `web` returns the page produced by `app`. If name resolution fails, note the message and go to the troubleshooting below.

    !!! success "Checkpoint"
        - [ ] Both machines are `running` in Hyper-V
        - [ ] `web` does relay `app`'s page

### Étape 6 : Comparer prévisions et observations / Step 6: Compare predictions and observations

=== "Français"

    Complétez le tableau comparatif : c'est le **livrable** du lab. Joignez le Vagrantfile adapté.

    | Aspect | Prévu (étape 3) | Observé (Hyper-V) | VirtualBox (Lab 6.2) |
    |---|---|---|---|
    | Réseau privé à adresse fixe | … | … | pris en charge |
    | Redirection de port | … | … | pris en charge |
    | Dossier partagé | … | … | … |
    | Droits nécessaires | … | … | … |
    | Résolution de `app` par `web` | … | … | adresse fixe |
    | Durée du premier `vagrant up` (min) | — | … | … |
    | Messages d'avertissement de Vagrant | — | … | … |
    | Lignes du Vagrantfile propres à un provider | — | … | … |

=== "English"

    Complete the comparison table: this is the lab **deliverable**. Attach the adapted Vagrantfile.

    | Aspect | Predicted (step 3) | Observed (Hyper-V) | VirtualBox (Lab 6.2) |
    |---|---|---|---|
    | Private network with fixed address | … | … | supported |
    | Port forwarding | … | … | supported |
    | Shared folder | … | … | … |
    | Rights needed | … | … | … |
    | `app` resolved by `web` | … | … | fixed address |
    | Duration of the first `vagrant up` (min) | — | … | … |
    | Vagrant warning messages | — | … | … |
    | Provider-specific lines in the Vagrantfile | — | … | … |

### Étape 7 : Réfléchir et nettoyer / Step 7: Reflect and clean up

```powershell
vagrant destroy -f
vagrant global-status --prune
Get-VM | Where-Object Name -like "tp63-*"
```

=== "Français"

    Les VM `tp63-*` doivent avoir disparu du Gestionnaire Hyper-V. Rédigez ensuite, en quelques lignes, une conclusion sur la **portabilité réelle** de votre environnement : qu'est-ce qui est resté identique, qu'est-ce qu'il a fallu adapter, et pourquoi.

=== "English"

    The `tp63-*` VMs must have disappeared from Hyper-V Manager. Then write, in a few lines, a conclusion on your environment's **real portability**: what stayed the same, what had to be adapted, and why.

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi une box VirtualBox ne fonctionne-t-elle pas avec le provider Hyper-V ?"
        Une box contient un **disque et des métadonnées propres à un format d'hyperviseur** (par exemple `.vmdk` ou `.vdi` pour VirtualBox, `.vhdx` pour Hyper-V), avec des pilotes et une configuration matérielle adaptés. Chaque provider ne sait utiliser que les boxes empaquetées pour lui.

    ??? question "Qu'est-ce qui rend un Vagrantfile « portable » en pratique, et qu'est-ce qui limite cette portabilité ?"
        Les scripts de provisionnement, la structure des machines et leur nom sont portables. Ce qui ne l'est pas : la box, les réglages réseau (adresses fixes, redirections de port), les dossiers partagés et les options propres à chaque provider. On isole ces différences dans des branches ou des blocs dédiés.

    ??? question "Pourquoi `web` utilise-t-il un nom plutôt qu'une adresse avec Hyper-V ?"
        Avec le *Default Switch*, les adresses sont distribuées par DHCP et ne sont pas connues à l'avance, donc impossibles à écrire dans le fichier. Le nom fourni par le commutateur reste valable tant que la machine existe. En contrepartie, la résolution dépend du service de noms de l'hôte et les adresses peuvent changer d'un redémarrage à l'autre.

=== "English"

    ??? question "Why does a VirtualBox box not work with the Hyper-V provider?"
        A box contains a **disk and metadata specific to a hypervisor format** (for example `.vmdk` or `.vdi` for VirtualBox, `.vhdx` for Hyper-V), with adapted drivers and hardware configuration. Each provider can only use boxes packaged for it.

    ??? question "What makes a Vagrantfile “portable” in practice, and what limits this portability?"
        Provisioning scripts, the machines' structure and names are portable. What is not: the box, network settings (fixed addresses, port forwards), shared folders and options specific to each provider. These differences are isolated in dedicated branches or blocks.

    ??? question "Why does `web` use a name rather than an address with Hyper-V?"
        With the *Default Switch*, addresses are handed out by DHCP and not known in advance, so they cannot be written in the file. The name provided by the switch stays valid as long as the machine exists. In return, resolution depends on the host's name service and addresses may change from one restart to the next.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | Vagrant demande des droits d'administrateur ou ne trouve pas le provider | Ouvrir PowerShell **en administrateur** ; vérifier `vmms` et l'activation d'Hyper-V |
    | « No box ... for provider hyperv » | La box n'est pas disponible pour Hyper-V : en choisir une autre (étape 2) ; `vagrant box list` pour voir les providers |
    | Vagrant demande de choisir un commutateur | Nom du commutateur absent ou incorrect dans `bridge:` ; vérifier l'orthographe de `Default Switch` |
    | Demande d'identifiants Windows (SMB) | Dossiers partagés non désactivés : ajouter `synced_folder ... disabled: true` |
    | La VM ne démarre pas (démarrage sécurisé) | La box n'est pas prévue pour le démarrage sécurisé : changer de box, ou régler le démarrage sécurisé de la VM dans Hyper-V (Lab 4.1) |
    | « Timed out waiting for the machine to boot » ou adresse jamais obtenue | Services d'intégration absents dans la box ; laisser plus de temps ; vérifier que le *Default Switch* fonctionne |
    | `app.mshome.net` ne se résout pas | Résolution de noms du *Default Switch* indisponible : récupérer l'adresse de `app` avec `Get-VMNetworkAdapter` et la passer au script (variable d'environnement ou adresse relevée), en acceptant qu'elle puisse changer |
    | Les VM VirtualBox ou VMware ne démarrent plus | Normal avec Hyper-V actif ; voir la bascule du Lab 2.1 |

=== "English"

    | Symptom | Hints |
    |---|---|
    | Vagrant asks for administrator rights or does not find the provider | Open PowerShell **as administrator**; check `vmms` and that Hyper-V is enabled |
    | "No box ... for provider hyperv" | The box is not available for Hyper-V: choose another (step 2); `vagrant box list` to see the providers |
    | Vagrant asks to choose a switch | Switch name missing or wrong in `bridge:`; check the spelling of `Default Switch` |
    | Windows credentials requested (SMB) | Shared folders not disabled: add `synced_folder ... disabled: true` |
    | The VM does not start (Secure Boot) | The box is not designed for Secure Boot: change box, or set the VM's Secure Boot in Hyper-V (Lab 4.1) |
    | "Timed out waiting for the machine to boot" or address never obtained | Integration services missing in the box; allow more time; check the *Default Switch* works |
    | `app.mshome.net` does not resolve | The *Default Switch* name resolution is unavailable: get `app`'s address with `Get-VMNetworkAdapter` and pass it to the script (environment variable or noted address), accepting that it may change |
    | VirtualBox or VMware VMs no longer start | Normal with Hyper-V active; see the switching procedure in Lab 2.1 |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    `vagrant destroy -f` supprime les VM Hyper-V. Gardez les deux dossiers de projet comme référence pour le **mini-projet 1**. Pour revenir aux labs de VirtualBox ou de VMware Workstation, désactivez l'hyperviseur Windows (Lab 2.1). Si vous ne réutilisez pas la box Hyper-V : `vagrant box remove <NOM_BOX_HYPERV> --provider hyperv`.

=== "English"

    `vagrant destroy -f` deletes the Hyper-V VMs. Keep both project folders as a reference for **mini-project 1**. To return to the VirtualBox or VMware Workstation labs, turn the Windows hypervisor off (Lab 2.1). If you do not reuse the Hyper-V box: `vagrant box remove <NOM_BOX_HYPERV> --provider hyperv`.

[Retour au chapitre 6 / Back to chapter 6](cours.md)