# Lab 2.1 : Préparer le poste pour la virtualisation / Preparing the Computer for Virtualization

| | |
|---|---|
| **Durée / Duration** | 45 à 60 min / 45 to 60 min |
| **Niveau / Level** | Guidé / Guided |
| **Prérequis / Prerequisites** | [Chapitre 2 : cours](cours.md) ; fiche d'inventaire du [Lab 1.2](../ch01-architecture-et-composants-d-un-datacenter/lab-1-2-inventorier-une-machine.md) |
| **Ressources minimales / Minimum resources** | Aucune machine virtuelle. Votre poste Windows 10 ou 11, compte **administrateur local**, un ou deux redémarrages. Espace disque libre : voir l'annexe [Environnement](../../annexes/environnement.md). / No virtual machine. Your Windows 10 or 11 computer, **local administrator** account, one or two restarts. Free disk space: see the [Environment](../../annexes/environnement.md) annex. |
| **Acquis visés / Learning outcomes** | AA1 |

=== "Français"

    !!! abstract "Objectifs"
        - Vérifier que votre poste peut héberger des machines virtuelles
        - Activer la virtualisation matérielle dans le micrologiciel (BIOS/UEFI) si nécessaire
        - Identifier l'état de l'hyperviseur de Windows (Hyper-V et fonctions associées) et savoir l'activer ou le désactiver
        - Préparer vos dossiers de travail et télécharger un ISO en vérifiant son intégrité

=== "English"

    !!! abstract "Objectives"
        - Check that your computer can host virtual machines
        - Enable hardware virtualization in the firmware (BIOS/UEFI) if needed
        - Identify the state of the Windows hypervisor (Hyper-V and related features) and know how to turn it on or off
        - Prepare your working folders and download an ISO while verifying its integrity

## Contexte et schéma / Context and diagram

=== "Français"

    Tous les labs de la matière s'exécutent sur votre poste. Trois conditions doivent être réunies : **la virtualisation matérielle est activée** dans le micrologiciel, **l'état de l'hyperviseur de Windows est maîtrisé** (certains chapitres le demandent activé, d'autres désactivé) et **l'espace disque et les ISO sont prêts**.

    ```mermaid
    flowchart LR
      F["1. Fiche<br/>d'inventaire"] --> U["2. Virtualisation<br/>dans l'UEFI"]
      U --> H["3. État de<br/>l'hyperviseur Windows"]
      H --> I["4. Dossiers<br/>et ISO"]
      I --> S["5. Fiche de<br/>préparation"]
    ```

    !!! warning "Avant de commencer"
        Les étapes 3 et 4 modifient la configuration de démarrage de Windows et nécessitent un **redémarrage**. Fermez vos documents. Si votre disque est chiffré avec BitLocker, assurez-vous de connaître votre **clé de récupération** : une modification de la configuration de démarrage peut la demander. Sur un poste géré par un établissement, demandez l'accord de l'administrateur.

=== "English"

    All the course labs run on your computer. Three conditions must be met: **hardware virtualization is enabled** in the firmware, **the state of the Windows hypervisor is under control** (some chapters require it on, others off) and **disk space and ISOs are ready**.

    ```mermaid
    flowchart LR
      F["1. Inventory<br/>sheet"] --> U["2. Virtualization<br/>in UEFI"]
      U --> H["3. Windows<br/>hypervisor state"]
      H --> I["4. Folders<br/>and ISO"]
      I --> S["5. Preparation<br/>sheet"]
    ```

    !!! warning "Before you start"
        Steps 3 and 4 change the Windows boot configuration and require a **restart**. Close your documents. If your disk is encrypted with BitLocker, make sure you know your **recovery key**: changing the boot configuration may prompt for it. On an institution-managed computer, ask the administrator for approval.

## Étapes / Steps

### Étape 1 : Comparer votre poste aux exigences / Step 1: Compare your computer with the requirements

=== "Français"

    Reprenez votre fiche d'inventaire du Lab 1.2 et comparez-la à l'annexe [Environnement](../../annexes/environnement.md) : mémoire, stockage, nombre de cœurs. **Déterminez votre profil de poste** (tel que défini dans l'annexe) et notez-le : il indique quels labs vous pourrez réaliser seul.

    !!! success "Point de contrôle"
        - [ ] Vous avez noté votre profil de poste
        - [ ] Vous savez quelle quantité d'espace disque est libre sur le disque qui hébergera vos VM

=== "English"

    Take your Lab 1.2 inventory sheet and compare it with the [Environment](../../annexes/environnement.md) annex: memory, storage, number of cores. **Determine your computer profile** (as defined in the annex) and write it down: it shows which labs you will be able to do on your own.

    !!! success "Checkpoint"
        - [ ] You wrote down your computer profile
        - [ ] You know how much free disk space is available on the disk that will host your VMs

### Étape 2 : Vérifier la virtualisation matérielle / Step 2: Check hardware virtualization

=== "Français"

    Ouvrez **PowerShell** et lancez les deux commandes ci-dessous. La première indique si la virtualisation est activée dans le micrologiciel ; la seconde indique si un hyperviseur est **déjà actif** sous Windows.

=== "English"

    Open **PowerShell** and run the two commands below. The first tells whether virtualization is enabled in the firmware; the second tells whether a hypervisor is **already active** under Windows.

```powershell
(Get-CimInstance Win32_Processor).VirtualizationFirmwareEnabled
(Get-CimInstance Win32_ComputerSystem).HypervisorPresent
```

=== "Français"

    Vous pouvez aussi lire cette information dans le **Gestionnaire des tâches** (++ctrl+shift+esc++), onglet **Performance**, **Processeur** : la ligne « Virtualisation » doit indiquer « Activé ».

    Interprétation :

    | `VirtualizationFirmwareEnabled` | `HypervisorPresent` | Situation |
    |---|---|---|
    | `True` | `False` | Situation idéale : virtualisation activée, aucun hyperviseur Windows actif |
    | `True` | `True` | Un hyperviseur Windows est actif (Hyper-V, fonctions associées ou sécurité renforcée) : voir l'étape 4 |
    | `False` | quelconque | La virtualisation doit être activée dans le micrologiciel : étape 3 |

    !!! info "Cas particulier"
        Lorsqu'un hyperviseur Windows est actif, certaines versions de Windows rapportent `VirtualizationFirmwareEnabled` à `False` alors que la virtualisation fonctionne. Si `HypervisorPresent` vaut `True`, la virtualisation matérielle est donc bien disponible : passez directement à l'étape 4.

    !!! success "Point de contrôle"
        - [ ] Vous avez relevé les deux valeurs et identifié votre situation dans le tableau

=== "English"

    You can also read this information in **Task Manager** (++ctrl+shift+esc++), **Performance** tab, **CPU**: the "Virtualization" line should say "Enabled".

    Interpretation:

    | `VirtualizationFirmwareEnabled` | `HypervisorPresent` | Situation |
    |---|---|---|
    | `True` | `False` | Ideal situation: virtualization enabled, no Windows hypervisor active |
    | `True` | `True` | A Windows hypervisor is active (Hyper-V, related features or enhanced security): see step 4 |
    | `False` | any | Virtualization must be enabled in the firmware: step 3 |

    !!! info "Special case"
        When a Windows hypervisor is active, some Windows versions report `VirtualizationFirmwareEnabled` as `False` even though virtualization works. If `HypervisorPresent` is `True`, hardware virtualization is therefore available: go directly to step 4.

    !!! success "Checkpoint"
        - [ ] You noted both values and identified your situation in the table

### Étape 3 : Activer la virtualisation dans l'UEFI (si nécessaire) / Step 3: Enable virtualization in UEFI (if needed)

=== "Français"

    1. Enregistrez votre travail. Ouvrez **Paramètres**, puis **Système**, **Récupération**, et dans « Démarrage avancé » cliquez sur **Redémarrer maintenant**.
    2. Au redémarrage, choisissez **Dépannage**, **Options avancées**, **Paramètres du micrologiciel UEFI**, puis **Redémarrer**.
    3. Dans l'interface du micrologiciel (elle varie selon le constructeur), cherchez une option nommée **Intel Virtualization Technology** (ou **VT-x**) sur un processeur Intel, **SVM Mode** (ou **AMD-V**) sur un processeur AMD. Elle se trouve souvent dans un menu « Advanced », « CPU Configuration » ou « Security ».
    4. Passez l'option sur **Enabled**, enregistrez et quittez (souvent ++f10++).

    Une autre méthode consiste à appuyer, au démarrage, sur la touche d'accès au micrologiciel (souvent ++f2++, ++f10++, ++f12++, ++del++ ou ++esc++ selon le constructeur).

    Après le redémarrage de Windows, relancez la première commande de l'étape 2.

    !!! success "Point de contrôle"
        - [ ] `VirtualizationFirmwareEnabled` renvoie `True` (ou `HypervisorPresent` renvoie `True`)

=== "English"

    1. Save your work. Open **Settings**, then **System**, **Recovery**, and under "Advanced startup" click **Restart now**.
    2. On restart, choose **Troubleshoot**, **Advanced options**, **UEFI Firmware Settings**, then **Restart**.
    3. In the firmware interface (it varies by manufacturer), look for an option named **Intel Virtualization Technology** (or **VT-x**) on an Intel processor, **SVM Mode** (or **AMD-V**) on an AMD processor. It is often under an "Advanced", "CPU Configuration" or "Security" menu.
    4. Set the option to **Enabled**, save and exit (often ++f10++).

    Another method is to press the firmware access key at boot (often ++f2++, ++f10++, ++f12++, ++del++ or ++esc++ depending on the manufacturer).

    After Windows restarts, rerun the first command of step 2.

    !!! success "Checkpoint"
        - [ ] `VirtualizationFirmwareEnabled` returns `True` (or `HypervisorPresent` returns `True`)

### Étape 4 : Examiner et maîtriser l'hyperviseur de Windows / Step 4: Examine and control the Windows hypervisor

=== "Français"

    Windows peut lancer son propre hyperviseur dès le démarrage, parce que certaines fonctionnalités en ont besoin. Cela peut ralentir VirtualBox et VMware Workstation, ou empêcher la virtualisation imbriquée. Ouvrez **PowerShell en administrateur** (clic droit sur PowerShell, « Exécuter en tant qu'administrateur »).

    **a) Voir quelles fonctionnalités sont activées :**

=== "English"

    Windows can start its own hypervisor at boot, because some features need it. This can slow down VirtualBox and VMware Workstation, or prevent nested virtualization. Open **PowerShell as administrator** (right-click PowerShell, "Run as administrator").

    **a) See which features are enabled:**

```powershell
Get-WindowsOptionalFeature -Online | Where-Object { $_.FeatureName -in 'Microsoft-Hyper-V-All','VirtualMachinePlatform','HypervisorPlatform','Microsoft-Windows-Subsystem-Linux','Containers-DisposableClientVM' } | Select-Object FeatureName, State
```

=== "Français"

    Cette commande liste, avec leur état (`Enabled` ou `Disabled`), Hyper-V, la Plateforme de machine virtuelle, la Plateforme de l'hyperviseur Windows, le Sous-système Windows pour Linux et Windows Sandbox. Si une fonctionnalité est absente de la liste, elle n'existe pas dans votre édition de Windows (Hyper-V, par exemple, n'est pas disponible en édition Famille).

    **b) Voir l'état du lancement de l'hyperviseur au démarrage :**

=== "English"

    This command lists, with their state (`Enabled` or `Disabled`), Hyper-V, Virtual Machine Platform, Windows Hypervisor Platform, Windows Subsystem for Linux and Windows Sandbox. If a feature is missing from the list, it does not exist in your Windows edition (Hyper-V, for example, is not available in Home edition).

    **b) See the hypervisor launch state at boot:**

```powershell
bcdedit /enum "{current}" | findstr /i hypervisorlaunchtype
```

=== "Français"

    Si la ligne `hypervisorlaunchtype` indique `Auto`, l'hyperviseur Windows démarre avec le système ; avec `Off`, il est désactivé. Si la commande ne renvoie aucune ligne, la valeur n'a jamais été définie sur ce poste.

    **c) Sauvegarder la configuration de démarrage** avant toute modification :

=== "English"

    If the `hypervisorlaunchtype` line shows `Auto`, the Windows hypervisor starts with the system; with `Off`, it is disabled. If the command returns no line, the value has never been set on this computer.

    **c) Back up the boot configuration** before any change:

```powershell
bcdedit /export "$HOME\bcd-sauvegarde"
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] Vous avez la liste des fonctionnalités liées à la virtualisation et leur état
        - [ ] Vous connaissez la valeur de `hypervisorlaunchtype`
        - [ ] Le fichier `bcd-sauvegarde` existe dans votre dossier personnel

=== "English"

    !!! success "Checkpoint"
        - [ ] You have the list of virtualization-related features and their state
        - [ ] You know the value of `hypervisorlaunchtype`
        - [ ] The `bcd-sauvegarde` file exists in your home folder

### Étape 5 : Désactiver ou réactiver l'hyperviseur de Windows / Step 5: Turn the Windows hypervisor off or on

=== "Français"

    Selon le chapitre, vous aurez besoin de l'un ou l'autre état. Ces états sont indicatifs ; chaque lab précise celui qu'il exige.

    | État souhaité | Quand | Remarque |
    |---|---|---|
    | **Hyperviseur Windows activé** | Labs Hyper-V (chapitre 4, labs sur les commutateurs virtuels Hyper-V) | Hyper-V doit être installé (édition Pro, Enterprise ou Éducation) |
    | **Hyperviseur Windows désactivé** | Labs de VirtualBox, VMware Workstation, et labs des Parties 3 et 4 (KVM, Proxmox VE, ESXi, DevStack) | Meilleures performances et virtualisation imbriquée disponible |

    **Pour désactiver** l'hyperviseur au prochain démarrage :

=== "English"

    Depending on the chapter, you will need one state or the other. These states are indicative; each lab states the one it requires.

    | Desired state | When | Note |
    |---|---|---|
    | **Windows hypervisor on** | Hyper-V labs (chapter 4, labs on Hyper-V virtual switches) | Hyper-V must be installed (Pro, Enterprise or Education edition) |
    | **Windows hypervisor off** | VirtualBox and VMware Workstation labs, and Part 3 and Part 4 labs (KVM, Proxmox VE, ESXi, DevStack) | Better performance and nested virtualization available |

    **To turn off** the hypervisor at next boot:

```powershell
bcdedit /set hypervisorlaunchtype off
```

=== "Français"

    **Pour le réactiver** :

=== "English"

    **To turn it back on**:

```powershell
bcdedit /set hypervisorlaunchtype auto
```

=== "Français"

    Dans les deux cas, **redémarrez** puis relancez la deuxième commande de l'étape 2 : `HypervisorPresent` doit valoir `False` après une désactivation, `True` après une réactivation.

    !!! warning "Si `HypervisorPresent` reste à `True` après la désactivation"
        D'autres éléments peuvent maintenir l'hyperviseur actif : les fonctionnalités **Plateforme de machine virtuelle**, **Plateforme de l'hyperviseur Windows**, **Sous-système Windows pour Linux** ou **Windows Sandbox** listées à l'étape 4, ainsi que l'**Intégrité de la mémoire** (Sécurité Windows, Sécurité de l'appareil, Isolation du noyau). Ne les désactivez que si vous n'en avez pas besoin pour vos autres usages, et notez ce que vous modifiez pour pouvoir le rétablir.

    !!! success "Point de contrôle"
        - [ ] Vous avez appliqué l'état voulu, redémarré et vérifié `HypervisorPresent`
        - [ ] Vous savez revenir à l'état précédent (commande inverse, fichier `bcd-sauvegarde`)

=== "English"

    In both cases, **restart** then rerun the second command of step 2: `HypervisorPresent` should be `False` after turning off, `True` after turning on.

    !!! warning "If `HypervisorPresent` stays `True` after turning it off"
        Other elements can keep the hypervisor active: the **Virtual Machine Platform**, **Windows Hypervisor Platform**, **Windows Subsystem for Linux** or **Windows Sandbox** features listed in step 4, as well as **Memory integrity** (Windows Security, Device security, Core isolation). Only turn them off if you do not need them for your other uses, and write down what you change so that you can restore it.

    !!! success "Checkpoint"
        - [ ] You applied the desired state, restarted and checked `HypervisorPresent`
        - [ ] You know how to return to the previous state (reverse command, `bcd-sauvegarde` file)

### Étape 6 : Préparer les dossiers et un ISO / Step 6: Prepare the folders and an ISO

=== "Français"

    1. Choisissez un disque ayant assez d'espace libre (étape 1) et créez deux dossiers : un pour les **ISO**, un pour les **machines virtuelles**. Notez leurs chemins : vous les réutiliserez dans tous les labs.
    2. Téléchargez depuis le **site officiel d'Ubuntu** l'ISO d'**Ubuntu Server** correspondant à la version de référence indiquée dans l'annexe Environnement, ainsi que le fichier de sommes de contrôle SHA-256 publié à côté.
    3. Calculez l'empreinte SHA-256 de l'ISO téléchargé et comparez-la à celle du site (remplacez le chemin par le vôtre) :

=== "English"

    1. Choose a disk with enough free space (step 1) and create two folders: one for **ISOs**, one for **virtual machines**. Write down their paths: you will reuse them in all labs.
    2. From the **official Ubuntu website**, download the **Ubuntu Server** ISO matching the reference version given in the Environment annex, together with the SHA-256 checksum file published next to it.
    3. Compute the SHA-256 hash of the downloaded ISO and compare it with the one on the site (replace the path with yours):

```powershell
Get-FileHash -Algorithm SHA256 "C:\chemin\vers\ubuntu-server.iso"
```

=== "Français"

    L'empreinte affichée doit être **identique** à celle publiée par le site. Si elle diffère, le téléchargement est corrompu ou incomplet : supprimez le fichier et recommencez.

    !!! success "Point de contrôle"
        - [ ] Deux dossiers créés (ISO, machines virtuelles) et leurs chemins notés
        - [ ] L'empreinte SHA-256 de l'ISO correspond à celle du site officiel

=== "English"

    The displayed hash must be **identical** to the one published by the site. If it differs, the download is corrupted or incomplete: delete the file and start again.

    !!! success "Checkpoint"
        - [ ] Two folders created (ISOs, virtual machines) and their paths noted
        - [ ] The ISO's SHA-256 hash matches the one on the official site

### Étape 7 : Fiche de préparation / Step 7: Preparation sheet

=== "Français"

    Remplissez la fiche ci-dessous : c'est le **livrable** du lab.

    | Rubrique | Valeur |
    |---|---|
    | Profil de poste (selon l'annexe Environnement) | … |
    | Espace disque libre sur le disque des VM (Go) | … |
    | `VirtualizationFirmwareEnabled` (avant / après) | … / … |
    | `HypervisorPresent` (avant / après) | … / … |
    | `hypervisorlaunchtype` actuel | … |
    | Fonctionnalités de virtualisation activées sur Windows | … |
    | Dossier des ISO | … |
    | Dossier des machines virtuelles | … |
    | ISO téléchargé et empreinte SHA-256 vérifiée | oui / non |
    | Modifications effectuées (pour pouvoir les annuler) | … |

=== "English"

    Fill in the sheet below: this is the lab **deliverable**.

    | Item | Value |
    |---|---|
    | Computer profile (according to the Environment annex) | … |
    | Free disk space on the VM disk (GB) | … |
    | `VirtualizationFirmwareEnabled` (before / after) | … / … |
    | `HypervisorPresent` (before / after) | … / … |
    | Current `hypervisorlaunchtype` | … |
    | Virtualization features enabled on Windows | … |
    | ISO folder | … |
    | Virtual machines folder | … |
    | ISO downloaded and SHA-256 hash verified | yes / no |
    | Changes made (so that they can be undone) | … |

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi la virtualisation matérielle est-elle souvent désactivée par défaut dans le micrologiciel ?"
        Les constructeurs la laissent parfois désactivée par prudence (sécurité, consommation) ou par héritage. Elle n'est utile que si l'on exécute un hyperviseur : il est donc normal de devoir l'activer soi-même.

    ??? question "Pourquoi deux hyperviseurs de niveaux différents peuvent-ils entrer en conflit sur un même poste Windows ?"
        Les extensions matérielles (VT-x, AMD-V) ne peuvent être pilotées en direct que par un seul hyperviseur à la fois. Lorsque Windows lance le sien, les autres hyperviseurs doivent passer par lui, ce qui réduit leurs performances et peut empêcher la virtualisation imbriquée.

    ??? question "Pourquoi vérifier l'empreinte d'un ISO téléchargé ?"
        Pour s'assurer que le fichier est complet et n'a pas été altéré (erreur de téléchargement ou fichier modifié). Installer un système à partir d'un ISO corrompu ou falsifié compromettrait toute l'infrastructure qui en dépend.

=== "English"

    ??? question "Why is hardware virtualization often disabled by default in the firmware?"
        Manufacturers sometimes leave it off out of caution (security, power use) or by legacy. It is only useful when a hypervisor is run: it is therefore normal to have to enable it yourself.

    ??? question "Why can two hypervisors at different levels conflict on the same Windows computer?"
        The hardware extensions (VT-x, AMD-V) can only be driven directly by one hypervisor at a time. When Windows starts its own, the other hypervisors must go through it, which lowers their performance and can prevent nested virtualization.

    ??? question "Why verify the hash of a downloaded ISO?"
        To make sure the file is complete and has not been altered (download error or modified file). Installing a system from a corrupted or tampered ISO would compromise the whole infrastructure that depends on it.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | Aucune option de virtualisation dans l'UEFI | Le processeur ou le constructeur ne la propose pas, ou elle est verrouillée par un mot de passe ou une politique d'établissement ; contacter l'enseignant |
    | « Accès refusé » avec `bcdedit` ou `Get-WindowsOptionalFeature` | Relancer PowerShell **en administrateur** |
    | `bcdedit` ne renvoie aucune ligne `hypervisorlaunchtype` | La valeur n'est pas définie : cela signifie en général que l'hyperviseur n'est pas configuré pour démarrer ; vérifier `HypervisorPresent` |
    | `HypervisorPresent` reste à `True` après désactivation | Voir l'avertissement de l'étape 5 (fonctionnalités Windows, Intégrité de la mémoire, autre logiciel utilisant WSL 2 ou la plateforme de machine virtuelle) |
    | Fonctionnalité Hyper-V absente de la liste | Édition Famille de Windows : Hyper-V n'y est pas disponible (voir l'annexe Environnement) |
    | Windows demande une clé de récupération BitLocker | Saisir la clé de récupération associée à votre compte ; la conserver à portée de main avant de recommencer |
    | Empreinte SHA-256 différente de celle du site | Supprimer l'ISO et le télécharger à nouveau, depuis le site officiel uniquement |

=== "English"

    | Symptom | Hints |
    |---|---|
    | No virtualization option in UEFI | The processor or manufacturer does not offer it, or it is locked by a password or institutional policy; contact the instructor |
    | "Access denied" with `bcdedit` or `Get-WindowsOptionalFeature` | Reopen PowerShell **as administrator** |
    | `bcdedit` returns no `hypervisorlaunchtype` line | The value is not set: this usually means the hypervisor is not configured to start; check `HypervisorPresent` |
    | `HypervisorPresent` stays `True` after turning off | See the warning in step 5 (Windows features, Memory integrity, other software using WSL 2 or the virtual machine platform) |
    | Hyper-V feature missing from the list | Windows Home edition: Hyper-V is not available there (see the Environment annex) |
    | Windows asks for a BitLocker recovery key | Enter the recovery key linked to your account; keep it at hand before trying again |
    | SHA-256 hash differs from the site's | Delete the ISO and download it again, from the official site only |

## Retour à l'état initial / Reverting

=== "Français"

    Pour annuler vos modifications de l'hyperviseur : remettez `hypervisorlaunchtype` à sa valeur d'origine (`auto` ou `off`, selon la fiche), puis redémarrez. En dernier recours, la configuration de démarrage sauvegardée à l'étape 4 peut être restaurée avec `bcdedit /import "$HOME\bcd-sauvegarde"` (PowerShell administrateur), suivie d'un redémarrage. Conservez votre fiche de préparation : elle sera utile à chaque fois que vous changerez d'état entre les chapitres.

=== "English"

    To undo your hypervisor changes: set `hypervisorlaunchtype` back to its original value (`auto` or `off`, according to your sheet), then restart. As a last resort, the boot configuration backed up in step 4 can be restored with `bcdedit /import "$HOME\bcd-sauvegarde"` (administrator PowerShell), followed by a restart. Keep your preparation sheet: it will be useful every time you switch state between chapters.

[Retour au chapitre 2 / Back to chapter 2](cours.md)