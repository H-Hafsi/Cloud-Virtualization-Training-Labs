# Lab 6.2 : Environnement multi-machines avec provisioning / Multi-Machine Environment with Provisioning

| | |
|---|---|
| **Durée / Duration** | 75 à 90 min / 75 to 90 min |
| **Niveau / Level** | Guidé / Guided |
| **Prérequis / Prerequisites** | [Chapitre 6 : cours](cours.md) ; [Lab 6.1](lab-6-1-premier-vagrantfile.md) réalisé (Vagrant, VirtualBox et box déjà installés) |
| **Ressources minimales / Minimum resources** | **VM** : 2 VM (`app`, `web`), 1 vCPU et 1 Go de RAM chacune. **Hôte** : au moins 4 Go de RAM libres, 15 Go d'espace disque libre, accès Internet. L'hyperviseur Windows doit être **désactivé**. / **VMs**: 2 VMs (`app`, `web`), 1 vCPU and 1 GB RAM each. **Host**: at least 4 GB of free RAM, 15 GB of free disk space, Internet access. The Windows hypervisor must be **off**. |
| **Acquis visés / Learning outcomes** | AA2 |

=== "Français"

    !!! abstract "Objectifs"
        - Décrire deux machines dans un même Vagrantfile avec une boucle et des paramètres par machine
        - Écrire des scripts de provisionnement dans des fichiers et leur passer des paramètres
        - Mettre en place une application à deux niveaux : un service applicatif et un serveur web qui le relaie
        - Opérer sur une seule machine de l'environnement (`up`, `provision`, `destroy`, `ssh`)
        - Diagnostiquer une erreur de provisionnement

=== "English"

    !!! abstract "Objectives"
        - Describe two machines in one Vagrantfile with a loop and per-machine parameters
        - Write provisioning scripts in files and pass them parameters
        - Set up a two-tier application: an application service and a web server relaying it
        - Operate on a single machine of the environment (`up`, `provision`, `destroy`, `ssh`)
        - Troubleshoot a provisioning error

## Contexte et schéma / Context and diagram

=== "Français"

    Vous décrivez un petit environnement de deux machines : `app` exécute un service applicatif très simple, et `web` est un serveur web (nginx) qui relaie les requêtes vers `app`. Tout est construit par `vagrant up`, sans intervention manuelle. Les adresses (`<IP_APP>`, `<IP_WEB>`) suivent le plan de l'annexe [Environnement](../../annexes/environnement.md).

    ```mermaid
    flowchart LR
      H["Poste hôte<br/>127.0.0.1:8080"] --> W["web<br/>nginx (port 80)"]
      W -- "proxy" --> A["app<br/>service (port 8000)"]
    ```

=== "English"

    You describe a small two-machine environment: `app` runs a very simple application service, and `web` is a web server (nginx) that relays requests to `app`. Everything is built by `vagrant up`, with no manual intervention. Addresses (`<IP_APP>`, `<IP_WEB>`) follow the plan in the [Environment](../../annexes/environnement.md) annex.

    ```mermaid
    flowchart LR
      H["Host computer<br/>127.0.0.1:8080"] --> W["web<br/>nginx (port 80)"]
      W -- "proxy" --> A["app<br/>service (port 8000)"]
    ```

## Étapes / Steps

### Étape 1 : Créer le projet / Step 1: Create the project

```powershell
mkdir "<DOSSIER_PROJETS>\vagrant-multi\provision"
cd "<DOSSIER_PROJETS>\vagrant-multi"
Set-Content -Path .gitignore -Value ".vagrant/"
```

=== "Français"

    Notez dans votre plan d'adressage les deux adresses du réseau privé : `<IP_APP>` et `<IP_WEB>`, dans le même sous-réseau.

=== "English"

    Write down in your addressing plan the two private network addresses: `<IP_APP>` and `<IP_WEB>`, in the same subnet.

### Étape 2 : Les scripts de provisionnement / Step 2: The provisioning scripts

=== "Français"

    Créez le fichier `provision/app.sh`. Il crée une page, puis déclare un **service système** qui exécute un petit serveur HTTP sur le port 8000 et le redémarre en cas d'arrêt. Chaque commande peut être rejouée sans effet indésirable :

=== "English"

    Create the file `provision/app.sh`. It creates a page, then declares a **system service** that runs a small HTTP server on port 8000 and restarts it if it stops. Each command can be replayed without side effects:

```bash
#!/bin/bash
set -e
mkdir -p /srv/app
echo "<h1>Reponse du serveur app ($(hostname))</h1>" > /srv/app/index.html
cat > /etc/systemd/system/app.service <<'UNIT'
[Unit]
Description=Service applicatif de demonstration
After=network.target

[Service]
WorkingDirectory=/srv/app
ExecStart=/usr/bin/python3 -m http.server 8000
Restart=always

[Install]
WantedBy=multi-user.target
UNIT
systemctl daemon-reload
systemctl enable --now app.service
```

=== "Français"

    Créez le fichier `provision/web.sh`. Il reçoit l'adresse de `app` en **premier argument** (`$1`), installe nginx et le configure comme relais :

=== "English"

    Create the file `provision/web.sh`. It receives `app`'s address as its **first argument** (`$1`), installs nginx and configures it as a relay:

```bash
#!/bin/bash
set -e
export DEBIAN_FRONTEND=noninteractive
APP_IP="$1"
apt-get update
apt-get install -y nginx
cat > /etc/nginx/sites-available/default <<CONF
server {
    listen 80 default_server;
    location / {
        proxy_pass http://${APP_IP}:8000;
    }
}
CONF
nginx -t
systemctl reload nginx
```

=== "Français"

    Créez enfin un petit fichier à téléverser, `provision/a-propos.txt`, par exemple une ligne : `Environnement construit par Vagrant`.

    !!! success "Point de contrôle"
        - [ ] Trois fichiers dans `provision/` : `app.sh`, `web.sh` et `a-propos.txt`
        - [ ] Vous pouvez expliquer ce que fait chaque script en une phrase

=== "English"

    Finally, create a small file to upload, `provision/a-propos.txt`, for example one line: `Environment built by Vagrant`.

    !!! success "Checkpoint"
        - [ ] Three files in `provision/`: `app.sh`, `web.sh` and `a-propos.txt`
        - [ ] You can explain what each script does in one sentence

### Étape 3 : Le Vagrantfile / Step 3: The Vagrantfile

=== "Français"

    Écrivez `Vagrantfile`. Un tableau Ruby décrit les machines ; une boucle les déclare. Remplacez `<NOM_DE_LA_BOX>`, `<IP_APP>` et `<IP_WEB>` :

=== "English"

    Write `Vagrantfile`. A Ruby hash describes the machines; a loop declares them. Replace `<NOM_DE_LA_BOX>`, `<IP_APP>` and `<IP_WEB>`:

```ruby
MACHINES = {
  "app" => { ip: "<IP_APP>", script: "provision/app.sh" },
  "web" => { ip: "<IP_WEB>", script: "provision/web.sh" }
}

Vagrant.configure("2") do |config|
  config.vm.box = "<NOM_DE_LA_BOX>"

  MACHINES.each do |name, cfg|
    config.vm.define name do |m|
      m.vm.hostname = name
      m.vm.network "private_network", ip: cfg[:ip]
      if name == "web"
        m.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
      end
      m.vm.provider "virtualbox" do |vb|
        vb.name = "tp62-#{name}"
        vb.memory = 1024
        vb.cpus = 1
      end
      m.vm.provision "file", source: "provision/a-propos.txt", destination: "/home/vagrant/a-propos.txt"
      m.vm.provision "shell", path: cfg[:script], args: [MACHINES["app"][:ip]]
    end
  end
end
```

```powershell
vagrant validate
```

=== "Français"

    Les provisioners d'une machine s'exécutent **dans l'ordre où ils sont déclarés** : d'abord la copie du fichier, puis le script. Le script reçoit `<IP_APP>` en argument ; `app.sh` l'ignore, `web.sh` s'en sert.

    !!! success "Point de contrôle"
        - [ ] `vagrant validate` ne signale aucune erreur
        - [ ] Vous savez dire où l'on ajouterait une troisième machine

=== "English"

    A machine's provisioners run **in the order they are declared**: first the file copy, then the script. The script receives `<IP_APP>` as an argument; `app.sh` ignores it, `web.sh` uses it.

    !!! success "Checkpoint"
        - [ ] `vagrant validate` reports no error
        - [ ] You can say where a third machine would be added

### Étape 4 : Construire l'environnement / Step 4: Build the environment

```powershell
vagrant up
vagrant status
```

=== "Français"

    Les machines sont créées **dans l'ordre de déclaration** (`app` puis `web`). Surveillez l'exécution des provisioners dans la sortie. Testez ensuite depuis l'hôte :

=== "English"

    Machines are created **in declaration order** (`app` then `web`). Watch the provisioners run in the output. Then test from the host:

```powershell
curl.exe http://<IP_APP>:8000
curl.exe http://127.0.0.1:8080
```

=== "Français"

    Résultat attendu : la première commande contacte `app` directement ; la seconde passe par `web` (port 8080 de l'hôte, puis nginx, puis `app`) et renvoie **la même page**, produite par `app`.

    !!! success "Point de contrôle"
        - [ ] `vagrant status` indique deux machines `running`
        - [ ] Les deux requêtes renvoient la page de `app`

=== "English"

    Expected result: the first command contacts `app` directly; the second goes through `web` (host port 8080, then nginx, then `app`) and returns **the same page**, produced by `app`.

    !!! success "Checkpoint"
        - [ ] `vagrant status` shows two `running` machines
        - [ ] Both requests return `app`'s page

### Étape 5 : Opérer sur une seule machine / Step 5: Operate on a single machine

```powershell
vagrant ssh web -c "cat /home/vagrant/a-propos.txt; systemctl is-active nginx"
vagrant ssh app -c "systemctl is-active app"
vagrant ssh app -c "sudo systemctl stop app"
curl.exe -i http://127.0.0.1:8080
vagrant ssh app -c "sudo systemctl start app"
curl.exe http://127.0.0.1:8080
```

=== "Français"

    Quand `app` est arrêté, `web` répond une erreur de **passerelle** (code 502) : nginx fonctionne mais ne peut plus joindre son service. Après le redémarrage de `app`, la page revient. Vous venez de constater la **dépendance** entre les deux machines.

    !!! success "Point de contrôle"
        - [ ] Le fichier téléversé est lisible sur `web`
        - [ ] Vous avez observé le code d'erreur quand `app` est arrêté, puis le retour à la normale

=== "English"

    When `app` is stopped, `web` answers a **gateway** error (code 502): nginx works but can no longer reach its service. After `app` restarts, the page returns. You have just observed the **dependency** between the two machines.

    !!! success "Checkpoint"
        - [ ] The uploaded file is readable on `web`
        - [ ] You observed the error code while `app` was stopped, then the return to normal

### Étape 6 : Rejouer et reconstruire une machine / Step 6: Replay and rebuild one machine

```powershell
vagrant provision app
curl.exe http://127.0.0.1:8080
vagrant destroy -f web
vagrant status
vagrant up web
curl.exe http://127.0.0.1:8080
```

=== "Français"

    `vagrant provision app` rejoue le script sur une machine déjà en marche : grâce à la façon dont il est écrit, le résultat reste le même (le script est **rejouable**). Détruire `web` seul laisse `app` intacte ; `vagrant up web` la reconstruit entièrement à partir de la box et du script.

    !!! success "Point de contrôle"
        - [ ] Le service répond après le rejeu du provisionnement
        - [ ] `web` est reconstruite seule et le site redevient accessible

=== "English"

    `vagrant provision app` replays the script on an already running machine: thanks to the way it is written, the result stays the same (the script is **replayable**). Destroying `web` alone leaves `app` untouched; `vagrant up web` rebuilds it entirely from the box and the script.

    !!! success "Checkpoint"
        - [ ] The service answers after provisioning is replayed
        - [ ] `web` is rebuilt alone and the site is reachable again

### Étape 7 : Provoquer et corriger une erreur / Step 7: Cause and fix an error

=== "Français"

    Dans `provision/web.sh`, introduisez volontairement une faute (par exemple `apt-get instal -y nginx`, sans le deuxième `l`), puis rejouez :

=== "English"

    In `provision/web.sh`, deliberately introduce a mistake (for example `apt-get instal -y nginx`, without the second `l`), then replay:

```powershell
vagrant provision web
```

=== "Français"

    Vagrant s'arrête avec un message d'erreur qui indique **la commande**, son **code de retour** et sa **sortie**. Corrigez le fichier, puis rejouez :

=== "English"

    Vagrant stops with an error message that shows **the command**, its **exit code** and its **output**. Fix the file, then replay:

```powershell
vagrant provision web
curl.exe http://127.0.0.1:8080
```

=== "Français"

    L'option `set -e` en tête de script arrête l'exécution à la première commande en échec : sans elle, le script continuerait et masquerait l'erreur.

    !!! success "Point de contrôle"
        - [ ] Vous avez lu le message d'erreur et identifié la ligne fautive
        - [ ] Après correction, le provisionnement réussit

=== "English"

    The `set -e` option at the top of the script stops execution at the first failing command: without it, the script would continue and hide the error.

    !!! success "Checkpoint"
        - [ ] You read the error message and identified the faulty line
        - [ ] After the fix, provisioning succeeds

### Étape 8 : Fiche de synthèse / Step 8: Summary sheet

=== "Français"

    Remplissez la fiche ci-dessous : c'est le **livrable** du lab. Joignez le dossier du projet (Vagrantfile et dossier `provision/`, sans `.vagrant/`).

    | Rubrique | Valeur |
    |---|---|
    | Adresses des deux machines | … |
    | Ordre de création des machines et d'exécution des provisioners | … |
    | Réponse de `curl.exe` direct sur `app` / par `web` | … / … |
    | Code de réponse de `web` quand `app` est arrêté | … |
    | Machine détruite et reconstruite seule : durée approximative | … |
    | Erreur provoquée : commande fautive, code de retour | … |
    | Fichiers à versionner / à exclure | … / … |

=== "English"

    Fill in the sheet below: this is the lab **deliverable**. Attach the project folder (Vagrantfile and `provision/` folder, without `.vagrant/`).

    | Item | Value |
    |---|---|
    | Addresses of the two machines | … |
    | Machine creation order and provisioner execution order | … |
    | `curl.exe` response directly on `app` / through `web` | … / … |
    | `web` response code when `app` is stopped | … |
    | Machine destroyed and rebuilt alone: approximate duration | … |
    | Error caused: faulty command, exit code | … |
    | Files to version / to exclude | … / … |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi passer l'adresse de `app` en argument plutôt que de l'écrire dans `web.sh` ?"
        L'adresse n'est alors définie qu'à **un seul endroit** (le Vagrantfile). La modifier suffit à reconfigurer l'ensemble, et le script reste réutilisable avec une autre adresse.

    ??? question "Pourquoi écrire des scripts de provisionnement rejouables ?"
        On peut les relancer après une modification ou une erreur sans craindre d'effets indésirables (doublons, échecs parce que « ça existe déjà »). C'est la propriété d'idempotence, que nous retrouverons avec Ansible et OpenTofu au chapitre 20.

    ??? question "Quel est l'intérêt de décrire deux machines dans un seul Vagrantfile plutôt que dans deux projets séparés ?"
        On construit, détruit et versionne l'ensemble cohérent d'un coup, avec des paramètres partagés (box, plage d'adresses) et des machines qui se retrouvent sur le même réseau privé, tout en gardant la possibilité d'agir sur une machine à la fois.

=== "English"

    ??? question "Why pass `app`'s address as an argument rather than writing it in `web.sh`?"
        The address is then defined in **a single place** (the Vagrantfile). Changing it is enough to reconfigure everything, and the script stays reusable with another address.

    ??? question "Why write replayable provisioning scripts?"
        They can be rerun after a change or an error without fearing side effects (duplicates, failures because "it already exists"). This is the idempotence property, which we will meet again with Ansible and OpenTofu in chapter 20.

    ??? question "What is the benefit of describing two machines in a single Vagrantfile rather than in two separate projects?"
        The coherent whole is built, destroyed and versioned at once, with shared parameters (box, address range) and machines that end up on the same private network, while still being able to act on one machine at a time.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | `vagrant validate` signale une erreur Ruby | Virgule, guillemet ou `end` manquant ; compter les `do` et les `end` |
    | Conflit d'adresse sur le réseau privé | Les deux adresses doivent être différentes et dans la plage autorisée par VirtualBox pour le réseau hôte seul |
    | `apt-get` échoue pendant le provisionnement | Accès Internet de la VM ; relancer `vagrant provision web` |
    | `nginx -t` signale une erreur | Syntaxe du fichier de configuration (accolades, point-virgule) ; `APP_IP` vide si l'argument n'est pas passé |
    | `curl.exe` sur le port 8080 ne répond pas | `web` non provisionnée ; port 8080 déjà utilisé sur l'hôte ; pare-feu |
    | Le service `app` n'est pas actif | `vagrant ssh app -c "systemctl status app"` ; relancer `vagrant provision app` |
    | Fin de ligne Windows dans un script (`bad interpreter`, `\r`) | Enregistrer les scripts avec des fins de ligne **Unix (LF)** dans l'éditeur |
    | Un script ne se rejoue pas | Utiliser `vagrant provision <machine>` ou `vagrant up --provision` |

=== "English"

    | Symptom | Hints |
    |---|---|
    | `vagrant validate` reports a Ruby error | Missing comma, quote or `end`; count the `do` and `end` |
    | Address conflict on the private network | The two addresses must differ and be within the range VirtualBox allows for the host-only network |
    | `apt-get` fails during provisioning | The VM's Internet access; rerun `vagrant provision web` |
    | `nginx -t` reports an error | Configuration file syntax (braces, semicolon); empty `APP_IP` if the argument is not passed |
    | `curl.exe` on port 8080 gets no answer | `web` not provisioned; port 8080 already in use on the host; firewall |
    | The `app` service is not active | `vagrant ssh app -c "systemctl status app"`; rerun `vagrant provision app` |
    | Windows line endings in a script (`bad interpreter`, `\r`) | Save the scripts with **Unix (LF)** line endings in the editor |
    | A script is not replayed | Use `vagrant provision <machine>` or `vagrant up --provision` |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    Gardez le projet : le [Lab 6.3](lab-6-3-meme-environnement-sur-un-autre-provider.md) réutilise ce même environnement sur un autre provider. Pour libérer de l'espace, `vagrant destroy -f` dans le dossier du projet détruit les deux machines. Pour repartir de zéro, supprimez aussi le dossier `.vagrant/`.

=== "English"

    Keep the project: [Lab 6.3](lab-6-3-meme-environnement-sur-un-autre-provider.md) reuses this same environment on another provider. To free up space, `vagrant destroy -f` in the project folder destroys both machines. To start from scratch, also delete the `.vagrant/` folder.

[Retour au chapitre 6 / Back to chapter 6](cours.md)