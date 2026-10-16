# Quiz du chapitre 6 : Vagrant, décrire ses environnements / Chapter 6 Quiz: Vagrant, Describing Your Environments

=== "Français"

    !!! abstract "Consignes"
        Autoévaluation : 10 questions à choix multiples, environ 10 minutes. Une seule réponse est correcte par question. Répondez d'abord de mémoire, puis cliquez sur « Réponse » pour vérifier. Si une réponse vous surprend, relisez la section correspondante du [cours](cours.md).

    **Question 1.** Quel est le principal bénéfice de décrire un environnement dans un fichier plutôt que de le construire à la main ?

    - A. Les machines s'exécutent plus vite
    - B. Les machines consomment moins de mémoire
    - C. L'environnement est reproductible, partageable et versionnable
    - D. On n'a plus besoin d'hyperviseur

    ??? success "Réponse"
        **C.** Le fichier documente la configuration et produit le même résultat sur un autre poste. Il ne change ni les performances des VM ni le besoin d'un hyperviseur.

    **Question 2.** Qu'est-ce qu'une box Vagrant ?

    - A. Une image de base empaquetée à partir de laquelle Vagrant crée une machine
    - B. Le fichier qui décrit les machines
    - C. L'hyperviseur qui exécute la VM
    - D. Un script de provisionnement

    ??? success "Réponse"
        **A.** La box est téléchargée une fois et réutilisée. Le fichier de description est le Vagrantfile ; l'hyperviseur est le provider.

    **Question 3.** Dans le vocabulaire de Vagrant, qu'est-ce qu'un provider ?

    - A. Un script de configuration
    - B. Une image de base
    - C. Un réseau virtuel
    - D. L'hyperviseur qui exécute réellement la VM

    ??? success "Réponse"
        **D.** VirtualBox, VMware, Hyper-V ou libvirt sont des providers. Il ne faut pas le confondre avec le *provider* d'OpenTofu, étudié au chapitre 20.

    **Question 4.** Quelle commande supprime les machines et leurs disques ?

    - A. `vagrant halt`
    - B. `vagrant destroy`
    - C. `vagrant reload`
    - D. `vagrant provision`

    ??? success "Réponse"
        **B.** `vagrant halt` éteint seulement les machines, qui existent toujours. `vagrant reload` les redémarre en relisant le Vagrantfile, et `vagrant provision` rejoue le provisionnement.

    **Question 5.** Quel élément d'un projet Vagrant ne doit pas être versionné ?

    - A. Le dossier `.vagrant/`
    - B. Le `Vagrantfile`
    - C. Les scripts de provisionnement
    - D. Le fichier `.gitignore`

    ??? success "Réponse"
        **A.** Le dossier `.vagrant/` ne contient que l'état local du poste (identifiants des machines) : le partager n'a aucun sens et provoquerait des conflits.

    **Question 6.** Quand le provisionnement s'exécute-t-il ?

    - A. À chaque `vagrant up`
    - B. Jamais, il faut le lancer à la main dans la VM
    - C. À la première création de la machine, ou à la demande (`vagrant provision`, `--provision`)
    - D. Uniquement lors d'un `vagrant halt`

    ??? success "Réponse"
        **C.** Sur une machine déjà créée, `vagrant up` ne rejoue pas le provisionnement ; après un `destroy`, la création suivante est une première création.

    **Question 7.** Quelle instruction déclare plusieurs machines dans un même Vagrantfile ?

    - A. `config.vm.box`
    - B. `config.vm.network`
    - C. `config.vm.provision`
    - D. `config.vm.define`

    ??? success "Réponse"
        **D.** Chaque machine est déclarée par un bloc `config.vm.define`, avec ses propres paramètres (nom d'hôte, adresse, ressources).

    **Question 8.** Dans quel ordre les provisioners d'une machine s'exécutent-ils ?

    - A. Dans un ordre aléatoire
    - B. Dans l'ordre où ils sont déclarés dans le Vagrantfile
    - C. Dans l'ordre alphabétique de leur type
    - D. En parallèle

    ??? success "Réponse"
        **B.** Par exemple, un provisioner `file` déclaré avant un `shell` s'exécute avant lui.

    **Question 9.** Quelle limite rencontre-t-on avec le provider Hyper-V par rapport à VirtualBox ?

    - A. Le provisionnement est impossible
    - B. Il n'existe aucune box pour Hyper-V
    - C. Pas d'adresse fixe sur un réseau privé ni de redirection de port
    - D. Vagrant n'a pas besoin de droits administrateur

    ??? success "Réponse"
        **C.** Avec Hyper-V, les adresses viennent du commutateur (par exemple le Default Switch) et la redirection de port n'existe pas. Il faut aussi une session administrateur, et une box prévue pour Hyper-V.

    **Question 10.** À quoi sert `set -e` en tête d'un script de provisionnement ?

    - A. À arrêter le script à la première commande en échec
    - B. À afficher chaque ligne avant de l'exécuter
    - C. À rendre le script rejouable sans condition
    - D. À exécuter le script en tant que root

    ??? success "Réponse"
        **A.** Sans `set -e`, le script continuerait après une erreur et pourrait masquer le problème. Afficher les lignes relève de `set -x`, et l'exécution en root dépend de l'option de provisionnement.

=== "English"

    !!! abstract "Instructions"
        Self-assessment: 10 multiple-choice questions, about 10 minutes. Only one answer is correct per question. Answer from memory first, then click "Answer" to check. If an answer surprises you, reread the matching section of the [lesson](cours.md).

    **Question 1.** What is the main benefit of describing an environment in a file rather than building it by hand?

    - A. Machines run faster
    - B. Machines use less memory
    - C. The environment is reproducible, shareable and versionable
    - D. A hypervisor is no longer needed

    ??? success "Answer"
        **C.** The file documents the configuration and produces the same result on another computer. It changes neither VM performance nor the need for a hypervisor.

    **Question 2.** What is a Vagrant box?

    - A. A packaged base image from which Vagrant creates a machine
    - B. The file that describes the machines
    - C. The hypervisor that runs the VM
    - D. A provisioning script

    ??? success "Answer"
        **A.** The box is downloaded once and reused. The description file is the Vagrantfile; the hypervisor is the provider.

    **Question 3.** In Vagrant's vocabulary, what is a provider?

    - A. A configuration script
    - B. A base image
    - C. A virtual network
    - D. The hypervisor that actually runs the VM

    ??? success "Answer"
        **D.** VirtualBox, VMware, Hyper-V or libvirt are providers. Not to be confused with OpenTofu's *provider*, studied in chapter 20.

    **Question 4.** Which command deletes the machines and their disks?

    - A. `vagrant halt`
    - B. `vagrant destroy`
    - C. `vagrant reload`
    - D. `vagrant provision`

    ??? success "Answer"
        **B.** `vagrant halt` only shuts the machines down, which still exist. `vagrant reload` restarts them while rereading the Vagrantfile, and `vagrant provision` replays provisioning.

    **Question 5.** Which element of a Vagrant project must not be versioned?

    - A. The `.vagrant/` folder
    - B. The `Vagrantfile`
    - C. The provisioning scripts
    - D. The `.gitignore` file

    ??? success "Answer"
        **A.** The `.vagrant/` folder only holds the computer's local state (machine identifiers): sharing it makes no sense and would cause conflicts.

    **Question 6.** When does provisioning run?

    - A. At every `vagrant up`
    - B. Never, it must be run by hand in the VM
    - C. At the machine's first creation, or on request (`vagrant provision`, `--provision`)
    - D. Only during a `vagrant halt`

    ??? success "Answer"
        **C.** On an already created machine, `vagrant up` does not replay provisioning; after a `destroy`, the next creation is a first creation.

    **Question 7.** Which statement declares several machines in a single Vagrantfile?

    - A. `config.vm.box`
    - B. `config.vm.network`
    - C. `config.vm.provision`
    - D. `config.vm.define`

    ??? success "Answer"
        **D.** Each machine is declared by a `config.vm.define` block, with its own settings (hostname, address, resources).

    **Question 8.** In which order do a machine's provisioners run?

    - A. In random order
    - B. In the order they are declared in the Vagrantfile
    - C. In alphabetical order of their type
    - D. In parallel

    ??? success "Answer"
        **B.** For example, a `file` provisioner declared before a `shell` one runs before it.

    **Question 9.** Which limit is met with the Hyper-V provider compared with VirtualBox?

    - A. Provisioning is impossible
    - B. There is no box for Hyper-V
    - C. No fixed address on a private network and no port forwarding
    - D. Vagrant does not need administrator rights

    ??? success "Answer"
        **C.** With Hyper-V, addresses come from the switch (for example the Default Switch) and port forwarding does not exist. An administrator session is also required, along with a box designed for Hyper-V.

    **Question 10.** What is `set -e` used for at the top of a provisioning script?

    - A. To stop the script at the first failing command
    - B. To display each line before running it
    - C. To make the script replayable unconditionally
    - D. To run the script as root

    ??? success "Answer"
        **A.** Without `set -e`, the script would continue after an error and could hide the problem. Displaying lines is `set -x`, and running as root depends on the provisioning option.