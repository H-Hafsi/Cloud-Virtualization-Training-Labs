# Lab 7.2 : Mesurer les performances avec fio / Measuring Performance with fio

| | |
|---|---|
| **Durée / Duration** | 75 à 90 min / 75 to 90 min |
| **Niveau / Level** | Guidé / Guided |
| **Prérequis / Prerequisites** | [Chapitre 7 : cours](cours.md) ; [Lab 7.1](lab-7-1-disques-virtuels-ajouter-convertir-comparer.md) réalisé (3 disques vides A, B, C et snapshot `etat-disques`) |
| **Ressources minimales / Minimum resources** | **VM** : `srv1-vmware` (2 vCPU, 2 Go de RAM) avec les 3 disques du Lab 7.1. **Hôte** : VMware Workstation, au moins 5 Go d'espace disque libre (les disques dynamiques grossissent pendant les tests d'écriture), accès Internet pour installer `fio`. Aucune autre charge lourde sur l'hôte pendant les mesures. L'hyperviseur Windows doit être **désactivé**. / **VM**: `srv1-vmware` (2 vCPUs, 2 GB RAM) with the 3 disks from Lab 7.1. **Host**: VMware Workstation, at least 5 GB of free disk space (dynamic disks grow during write tests), Internet access to install `fio`. No other heavy load on the host during measurements. The Windows hypervisor must be **off**. |
| **Acquis visés / Learning outcomes** | AA3 |

=== "Français"

    !!! abstract "Objectifs"
        - Installer et utiliser `fio` pour mesurer débit, IOPS et latence d'un disque
        - Définir des profils de charge reproductibles (séquentiel, aléatoire, tailles de bloc, profondeur de file)
        - Comparer trois disques virtuels : SCSI dynamique, SCSI réservé, NVMe dynamique
        - Mesurer l'effet de l'allocation dynamique à la première écriture
        - Interpréter les résultats avec prudence : ce que mesure réellement un disque virtuel

=== "English"

    !!! abstract "Objectives"
        - Install and use `fio` to measure a disk's throughput, IOPS and latency
        - Define reproducible load profiles (sequential, random, block sizes, queue depth)
        - Compare three virtual disks: dynamic SCSI, reserved SCSI, dynamic NVMe
        - Measure the effect of dynamic allocation on first write
        - Interpret results carefully: what a virtual disk really measures

## Contexte et schéma / Context and diagram

=== "Français"

    Vous mesurez, avec le même outil et les mêmes profils, les trois disques ajoutés au Lab 7.1. Tous trois reposent **sur le même support physique** (le disque de votre poste) : les écarts que vous observerez viendront donc du **contrôleur virtuel**, de l'**allocation** et des **caches**, pas d'une différence de matériel. C'est précisément ce qu'il faut apprendre à lire.

    ```mermaid
    flowchart LR
      F["fio<br/>(profil de charge)"] --> A["Disque A<br/>SCSI dynamique"]
      F --> B["Disque B<br/>SCSI réservé"]
      F --> C["Disque C<br/>NVMe dynamique"]
      A --> R["Résultats :<br/>IOPS, débit, latence"]
      B --> R
      C --> R
    ```

    !!! danger "Les tests d'écriture détruisent le contenu du disque testé"
        Ce lab écrit sur les **disques entiers** `<DISQUE_A>`, `<DISQUE_B>` et `<DISQUE_C>` (relevés au Lab 7.1). Ils doivent être **vides, non formatés et non montés**. **Ne lancez jamais ces commandes sur le disque système de la VM** : vous détruiriez son système. Vérifiez les noms à l'étape 2.

=== "English"

    You measure, with the same tool and the same profiles, the three disks added in Lab 7.1. All three rely on **the same physical medium** (your computer's disk): the differences you observe will therefore come from the **virtual controller**, the **allocation** and **caches**, not from a hardware difference. This is precisely what you must learn to read.

    ```mermaid
    flowchart LR
      F["fio<br/>(load profile)"] --> A["Disk A<br/>dynamic SCSI"]
      F --> B["Disk B<br/>reserved SCSI"]
      F --> C["Disk C<br/>dynamic NVMe"]
      A --> R["Results:<br/>IOPS, throughput, latency"]
      B --> R
      C --> R
    ```

    !!! danger "Write tests destroy the tested disk's content"
        This lab writes on the **whole disks** `<DISQUE_A>`, `<DISQUE_B>` and `<DISQUE_C>` (noted in Lab 7.1). They must be **empty, unformatted and unmounted**. **Never run these commands on the VM's system disk**: you would destroy its system. Check the names in step 2.

## Étapes / Steps

### Étape 1 : Préparer la VM / Step 1: Prepare the VM

=== "Français"

    1. Éteignez la VM puis restaurez le snapshot `etat-disques` (**VM**, **Snapshot**, **Snapshot Manager**) pour partir de disques vierges. Démarrez-la.
    2. Notez le **type du disque de votre poste** (SSD ou HDD, fiche du Lab 1.2) : il conditionne tous les résultats.
    3. Installez les outils :

=== "English"

    1. Shut the VM down then restore the `etat-disques` snapshot (**VM**, **Snapshot**, **Snapshot Manager**) to start from blank disks. Start it.
    2. Note your computer's **disk type** (SSD or HDD, Lab 1.2 sheet): it conditions all the results.
    3. Install the tools:

```bash
sudo apt update && sudo apt install -y fio sysstat
fio --version
```

=== "Français"

    !!! success "Point de contrôle"
        - [ ] `fio` est installé et affiche sa version
        - [ ] Le type du disque de l'hôte est noté

=== "English"

    !!! success "Checkpoint"
        - [ ] `fio` is installed and shows its version
        - [ ] The host disk type is noted

### Étape 2 : Identifier et vérifier les disques de test / Step 2: Identify and verify the test disks

```bash
lsblk -o NAME,SIZE,TYPE,TRAN,FSTYPE,MOUNTPOINT
```

=== "Français"

    Repérez les trois disques de **2 Go** : leurs colonnes `FSTYPE` et `MOUNTPOINT` doivent être **vides**. Le disque système (le plus grand, avec des partitions montées) ne doit **jamais** être testé. Notez les noms : `<DISQUE_A>` (SCSI dynamique), `<DISQUE_B>` (SCSI réservé), `<DISQUE_C>` (NVMe, de la forme `nvme0n1`). Le disque NVMe se reconnaît à son nom ; pour distinguer A de B, souvenez-vous de l'ordre d'ajout et vérifiez ensuite sur l'hôte lequel des deux fichiers `.vmdk` grossit pendant les tests.

    !!! success "Point de contrôle"
        - [ ] Trois disques de 2 Go, sans système de fichiers ni point de montage
        - [ ] Les noms `<DISQUE_A>`, `<DISQUE_B>`, `<DISQUE_C>` sont notés et vérifiés

=== "English"

    Find the three **2 GB** disks: their `FSTYPE` and `MOUNTPOINT` columns must be **empty**. The system disk (the largest, with mounted partitions) must **never** be tested. Note the names: `<DISQUE_A>` (dynamic SCSI), `<DISQUE_B>` (reserved SCSI), `<DISQUE_C>` (NVMe, of the form `nvme0n1`). The NVMe disk is recognized by its name; to tell A from B, remember the order of addition and check afterwards on the host which of the two `.vmdk` files grows during the tests.

    !!! success "Checkpoint"
        - [ ] Three 2 GB disks, with no file system or mount point
        - [ ] The names `<DISQUE_A>`, `<DISQUE_B>`, `<DISQUE_C>` are noted and verified

### Étape 3 : Une première mesure commentée / Step 3: A first commented measurement

=== "Français"

    Lancez une lecture aléatoire en blocs de 4 Kio, une requête à la fois, sur le disque A :

=== "English"

    Run a random read with 4 KiB blocks, one request at a time, on disk A:

```bash
sudo fio --name=essai --filename=/dev/<DISQUE_A> --rw=randread --bs=4k --iodepth=1 \
  --direct=1 --ioengine=libaio --runtime=20 --time_based --group_reporting
```

=== "Français"

    Lisez la sortie, elle contient l'essentiel :

    | Élément de la sortie | Signification |
    |---|---|
    | `IOPS=` et `BW=` | Opérations par seconde et débit (reliés par : débit = IOPS × taille du bloc) |
    | `clat` (*completion latency*) | Durée entre l'envoi et la fin de la requête : moyenne (`avg`) et **centiles** |
    | `99.00th=[...]` | 99 % des requêtes ont été plus rapides que cette valeur : mesure de la **régularité** |
    | `util=` (ligne `Disk stats`) | Taux d'occupation du disque pendant le test |

    Les paramètres : `--rw` choisit le motif, `--bs` la taille de bloc, `--iodepth` la profondeur de file, `--direct=1` contourne le cache de l'invité, `--runtime` avec `--time_based` impose la durée.

    **Vérifiez la relation** : multipliez `IOPS` par 4 Kio et comparez à `BW`.

    !!! success "Point de contrôle"
        - [ ] Vous avez relevé IOPS, débit, latence moyenne et centile 99 %
        - [ ] Le produit IOPS × 4 Kio correspond au débit affiché

=== "English"

    Read the output, it holds the essentials:

    | Output element | Meaning |
    |---|---|
    | `IOPS=` and `BW=` | Operations per second and throughput (related by: throughput = IOPS × block size) |
    | `clat` (*completion latency*) | Time between sending and completion of the request: average (`avg`) and **percentiles** |
    | `99.00th=[...]` | 99% of requests were faster than this value: a measure of **regularity** |
    | `util=` (`Disk stats` line) | Disk busy rate during the test |

    The parameters: `--rw` selects the pattern, `--bs` the block size, `--iodepth` the queue depth, `--direct=1` bypasses the guest cache, `--runtime` with `--time_based` sets the duration.

    **Check the relation**: multiply `IOPS` by 4 KiB and compare with `BW`.

    !!! success "Checkpoint"
        - [ ] You noted IOPS, throughput, average latency and 99% percentile
        - [ ] The IOPS × 4 KiB product matches the displayed throughput

### Étape 4 : La matrice complète / Step 4: The full matrix

=== "Français"

    On mesure maintenant cinq profils sur chacun des trois disques. Les profils couvrent les cas décrits dans le cours :

    | Profil | Motif | Bloc | Profondeur de file |
    |---|---|---|---|
    | P1 | Lecture séquentielle | 1 Mio | 8 |
    | P2 | Écriture séquentielle | 1 Mio | 8 |
    | P3 | Lecture aléatoire | 4 Kio | 1 |
    | P4 | Lecture aléatoire | 4 Kio | 32 |
    | P5 | Écriture aléatoire | 4 Kio | 32 |

    Cette boucle exécute les 15 mesures (15 s chacune, environ 4 minutes en tout) et conserve les lignes utiles dans un fichier. Ne touchez pas à la VM ni à l'hôte pendant son exécution :

=== "English"

    We now measure five profiles on each of the three disks. The profiles cover the cases described in the lesson:

    | Profile | Pattern | Block | Queue depth |
    |---|---|---|---|
    | P1 | Sequential read | 1 MiB | 8 |
    | P2 | Sequential write | 1 MiB | 8 |
    | P3 | Random read | 4 KiB | 1 |
    | P4 | Random read | 4 KiB | 32 |
    | P5 | Random write | 4 KiB | 32 |

    This loop runs the 15 measurements (15 s each, about 4 minutes in total) and keeps the useful lines in a file. Do not touch the VM or the host while it runs:

```bash
for d in <DISQUE_A> <DISQUE_B> <DISQUE_C>; do
  for p in "read 1M 8" "write 1M 8" "randread 4k 1" "randread 4k 32" "randwrite 4k 32"; do
    set -- $p
    echo "== /dev/$d  rw=$1  bs=$2  iodepth=$3"
    sudo fio --name=t --filename=/dev/$d --rw=$1 --bs=$2 --iodepth=$3 \
      --direct=1 --ioengine=libaio --runtime=15 --time_based --group_reporting \
      | grep -E "IOPS=|lat \(|99.00th"
  done
done | tee ~/resultats-fio.txt
```

=== "Français"

    Reportez dans la matrice de l'étape 7 : **IOPS**, **débit**, **latence moyenne** et **centile 99 %** pour chaque couple (disque, profil). Le format exact de la sortie dépend de la version de `fio` ; si les lignes filtrées ne correspondent pas, relancez une mesure sans le `grep`.

    !!! success "Point de contrôle"
        - [ ] Le fichier `~/resultats-fio.txt` contient les 15 mesures
        - [ ] Les valeurs sont reportées dans la matrice

=== "English"

    Enter in the step 7 matrix: **IOPS**, **throughput**, **average latency** and **99% percentile** for each (disk, profile) pair. The exact output format depends on the `fio` version; if the filtered lines do not match, rerun a measurement without the `grep`.

    !!! success "Checkpoint"
        - [ ] The `~/resultats-fio.txt` file contains the 15 measurements
        - [ ] The values are entered in the matrix

### Étape 5 : L'effet de l'allocation dynamique / Step 5: The effect of dynamic allocation

=== "Français"

    Pour mesurer la **première écriture** sur un disque dynamique neuf, restaurez d'abord le snapshot `etat-disques` (disques vierges), redémarrez, puis écrivez **deux fois de suite** la totalité du disque A, avec le même profil, et comparez :

=== "English"

    To measure the **first write** on a brand new dynamic disk, first restore the `etat-disques` snapshot (blank disks), restart, then write the whole of disk A **twice in a row**, with the same profile, and compare:

```bash
for i in 1 2; do
  echo "== passe $i"
  sudo fio --name=ecriture --filename=/dev/<DISQUE_A> --rw=write --bs=1M --iodepth=8 \
    --direct=1 --ioengine=libaio --size=2G | grep -E "IOPS=|BW=|lat \("
done
```

=== "Français"

    La première passe force l'hyperviseur à **allouer** l'espace dans le fichier du disque ; la seconde écrit sur de l'espace déjà alloué. Observez aussi, **sur l'hôte**, la taille du fichier du disque dynamique avant et après (même commande qu'au Lab 7.1, étape 3) : elle approche maintenant la taille déclarée.

    !!! success "Point de contrôle"
        - [ ] Vous avez comparé la première et la seconde passe (débit et latence)
        - [ ] La taille du fichier du disque dynamique a augmenté sur l'hôte

=== "English"

    The first pass forces the hypervisor to **allocate** space in the disk file; the second writes onto already allocated space. Also observe, **on the host**, the dynamic disk file's size before and after (same command as Lab 7.1, step 3): it now approaches the declared size.

    !!! success "Checkpoint"
        - [ ] You compared the first and second pass (throughput and latency)
        - [ ] The dynamic disk file's size increased on the host

### Étape 6 : Observer en direct avec iostat / Step 6: Observe live with iostat

=== "Français"

    Ouvrez deux sessions SSH vers la VM. Dans la première, lancez une charge d'écriture aléatoire plus longue sur le disque C ; dans la seconde, observez le disque :

=== "English"

    Open two SSH sessions to the VM. In the first, run a longer random write load on disk C; in the second, watch the disk:

```bash
# Session 1
sudo fio --name=charge --filename=/dev/<DISQUE_C> --rw=randwrite --bs=4k --iodepth=32 \
  --direct=1 --ioengine=libaio --runtime=30 --time_based --group_reporting > /dev/null

# Session 2
iostat -x <DISQUE_C> 2 6
```

=== "Français"

    Repérez les colonnes du nombre d'opérations par seconde (écritures), du débit, de la **file d'attente moyenne** et du **taux d'utilisation** (`%util`). Comparez avec les chiffres rendus par `fio` : ils doivent être cohérents.

    !!! success "Point de contrôle"
        - [ ] Les valeurs d'`iostat` pendant la charge sont cohérentes avec celles de `fio`

=== "English"

    Find the columns for the number of operations per second (writes), throughput, **average queue** and **utilization** (`%util`). Compare with the figures returned by `fio`: they should be consistent.

    !!! success "Checkpoint"
        - [ ] `iostat` values during the load are consistent with `fio`'s

### Étape 7 : Matrice de résultats et analyse / Step 7: Results matrix and analysis

=== "Français"

    Remplissez la matrice (une cellule : IOPS / débit / latence moyenne / centile 99 %) : c'est le **livrable** du lab. Joignez `resultats-fio.txt`.

    | Profil | Disque A (SCSI dyn.) | Disque B (SCSI réservé) | Disque C (NVMe dyn.) |
    |---|---|---|---|
    | P1 lecture séquentielle 1 Mio | … | … | … |
    | P2 écriture séquentielle 1 Mio | … | … | … |
    | P3 lecture aléatoire 4 Kio, file 1 | … | … | … |
    | P4 lecture aléatoire 4 Kio, file 32 | … | … | … |
    | P5 écriture aléatoire 4 Kio, file 32 | … | … | … |

    Puis répondez en quelques lignes :

    1. Quel profil donne le plus grand **débit** ? le plus grand nombre d'**IOPS** ? Est-ce cohérent avec « débit = IOPS × bloc » ?
    2. Quel effet a la **profondeur de file** (P3 contre P4) sur les IOPS et sur la latence ?
    3. Les trois disques sont-ils très différents ? Qu'est-ce qui l'explique, sachant qu'ils reposent sur le même disque physique ?
    4. Que vous apprend la comparaison de la première et de la seconde passe d'écriture ?
    5. Quelles **limites** de cette mesure devez-vous signaler (caches de l'hôte, autres charges, durée courte, support de l'hôte) ?

=== "English"

    Fill in the matrix (one cell: IOPS / throughput / average latency / 99% percentile): this is the lab **deliverable**. Attach `resultats-fio.txt`.

    | Profile | Disk A (dyn. SCSI) | Disk B (reserved SCSI) | Disk C (dyn. NVMe) |
    |---|---|---|---|
    | P1 sequential read 1 MiB | … | … | … |
    | P2 sequential write 1 MiB | … | … | … |
    | P3 random read 4 KiB, queue 1 | … | … | … |
    | P4 random read 4 KiB, queue 32 | … | … | … |
    | P5 random write 4 KiB, queue 32 | … | … | … |

    Then answer in a few lines:

    1. Which profile gives the highest **throughput**? the highest number of **IOPS**? Is that consistent with "throughput = IOPS × block"?
    2. What effect does **queue depth** have (P3 versus P4) on IOPS and on latency?
    3. Are the three disks very different? What explains it, given that they rely on the same physical disk?
    4. What does the comparison of the first and second write pass teach you?
    5. Which **limits** of this measurement must you report (host caches, other loads, short duration, host medium)?

## Questions de réflexion / Reflection questions

=== "Français"

    ??? question "Pourquoi une profondeur de file plus grande augmente-t-elle généralement les IOPS, mais aussi la latence par requête ?"
        Avec plusieurs requêtes en parallèle, le disque et le contrôleur peuvent les traiter simultanément et réordonner les accès : le nombre d'opérations par seconde monte. Mais chaque requête attend plus longtemps son tour dans la file : la latence individuelle augmente. C'est le compromis entre débit global et réactivité.

    ??? question "Pourquoi ne faut-il surtout pas lancer ces commandes sur le disque système de la VM ?"
        Un test d'écriture sur un périphérique brut écrit sur **tout le disque**, sans tenir compte du système de fichiers : il détruit les partitions, le système et les données. Il faut le réserver à des disques vides et dédiés au test.

    ??? question "Pourquoi un disque NVMe virtuel n'est-il pas forcément beaucoup plus rapide qu'un disque SCSI virtuel ?"
        Le disque est un **fichier** sur le disque physique de l'hôte : la vitesse maximale est celle de ce support, quel que soit le contrôleur virtuel présenté à l'invité. Le contrôleur NVMe virtuel peut réduire le surcoût logiciel, mais il ne rend pas le support plus rapide.

=== "English"

    ??? question "Why does a higher queue depth generally increase IOPS, but also per-request latency?"
        With several requests in parallel, the disk and controller can process them simultaneously and reorder accesses: the number of operations per second rises. But each request waits longer for its turn in the queue: individual latency increases. This is the trade-off between overall throughput and responsiveness.

    ??? question "Why must you never run these commands on the VM's system disk?"
        A write test on a raw device writes on the **whole disk**, ignoring the file system: it destroys partitions, system and data. It must be reserved for empty disks dedicated to the test.

    ??? question "Why is a virtual NVMe disk not necessarily much faster than a virtual SCSI disk?"
        The disk is a **file** on the host's physical disk: the maximum speed is that of this medium, whatever virtual controller is presented to the guest. The virtual NVMe controller can reduce software overhead, but it does not make the medium faster.

## Dépannage / Troubleshooting

=== "Français"

    | Symptôme | Pistes |
    |---|---|
    | `fio: command not found` | `sudo apt install -y fio` (accès Internet par la carte NAT) |
    | `Permission denied` sur `/dev/...` | Lancer `fio` avec `sudo` |
    | `No such file or directory` sur le disque | Mauvais nom de périphérique : revérifier avec `lsblk` |
    | `fio` refuse le moteur `libaio` | Remplacer par `--ioengine=io_uring` ou `--ioengine=psync` selon la version |
    | Résultats très variables d'une mesure à l'autre | Autres charges sur l'hôte ou dans les VM ; répéter la mesure ; fermer les applications gourmandes |
    | Valeurs étonnamment élevées en lecture | Cache de l'hôte (non contourné par `--direct=1` dans l'invité) : noter la limite dans l'analyse |
    | Le `grep` ne renvoie rien | Le format de sortie a changé : relancer sans `grep` et adapter le filtre |
    | Le disque dynamique remplit le disque de l'hôte | Prévoir 5 Go libres ; restaurer le snapshot et supprimer les disques inutiles |

=== "English"

    | Symptom | Hints |
    |---|---|
    | `fio: command not found` | `sudo apt install -y fio` (Internet access through the NAT card) |
    | `Permission denied` on `/dev/...` | Run `fio` with `sudo` |
    | `No such file or directory` on the disk | Wrong device name: recheck with `lsblk` |
    | `fio` rejects the `libaio` engine | Replace with `--ioengine=io_uring` or `--ioengine=psync` depending on the version |
    | Results vary a lot from one measurement to the next | Other loads on the host or in VMs; repeat the measurement; close demanding applications |
    | Surprisingly high read values | Host cache (not bypassed by `--direct=1` in the guest): note the limit in the analysis |
    | The `grep` returns nothing | The output format changed: rerun without `grep` and adapt the filter |
    | The dynamic disk fills the host's disk | Plan for 5 GB free; restore the snapshot and delete unneeded disks |

## Nettoyage et reprise / Cleanup and restart

=== "Français"

    Les disques de test ne contiennent plus rien d'utile : conservez-les pour le chapitre suivant si vous en avez besoin (RAID, chapitre 8), ou restaurez le snapshot `etat-disques` pour retrouver des disques vierges. Supprimez `~/resultats-fio.txt` après l'avoir joint à votre compte rendu. Pour revenir à la VM de référence sans disques supplémentaires, restaurez `etat-propre` et supprimez les fichiers `disque-*.vmdk`.

=== "English"

    The test disks no longer hold anything useful: keep them for the next chapter if you need them (RAID, chapter 8), or restore the `etat-disques` snapshot to get blank disks back. Delete `~/resultats-fio.txt` after attaching it to your report. To return to the reference VM without extra disks, restore `etat-propre` and delete the `disque-*.vmdk` files.

[Retour au chapitre 7 / Back to chapter 7](cours.md)