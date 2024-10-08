# Linux Professional Institute Certification 1 

<br>

---

## Sommaire

- 101 [Architecture système](https://github.com/NG3IT/LPIC-1/blob/main/LPIC-1.md#101-architecture-syst%C3%A8me-)
- 101.1 [Détermination et configuration des paramètres du matériel](https://github.com/NG3IT/LPIC-1/blob/main/LPIC-1.md#1011-d%C3%A9termination-et-configuration-des-param%C3%A8tres-du-mat%C3%A9riel-)
- 101.2 [Démarrage du système]()

<br>

---

## 101 **Architecture système** 💻

<br>

### 101.1 **Détermination et configuration des paramètres du matériel** 💾

<br>

**Inspection du matériel**

<br>

La commande pour identifier les périphériques connectés en PCI (Peripheral Component Interconnect) est **lspci**

```bash
# Informations globales sur les devices connectés aux interfaces PCI
lspci

# Informations détaillées spécifiques à un device PCI
lspci -s <id_du_device> -v
lspci -s 01:00 -v

# information concernant le module du noyau utilisé par le device PCI
lspci -s <id_du_device> -k
lspci -s 01:00 -k

```

La commande pour identifier les périphériques connectés en USB (Universal Serial Bus) est **lsusb**

```bash
# Informations globales sur les devices connectés aux ports USB
lsusb

# Arborescence des devices USB
lsusb -t

# Informations détaillées spécifiques à un device USB
lsusb -v -d <id_du_vendor>:<id_du_product>
lsusb -v -d 1781:0c9f

# Informations détaillées spécifiques à un device USB
lsusb -s <id_du_bus>:<id_du_port>
lsusb -s 01:20
```

Les commandes pour interragir avec les **modules du noyau**

```bash
# Affichage de tous les modules actuellement chargés
lsmod

# Décharger un module du noyau (fonctionne seulement s'il n'est pas en cours d'utilisation)
modprobe -r <nom_du_module>
modprode -r snd-hda-intel

# Fichier de persistence des paramètres concernant les modules
/etc/modprobe.conf
/etc/modprobe.d/

# Affiche tous les paramètres disponible d'un module
modinfo -p <nom_du_module>
modinfo -p nouveau

# Modification des paramètres persistent d'un module
vim /etc/modprobe.d/<nom_du_module>
vim /etc/modprobe.d/nouveau
options <nom_du_module> <option>=<valeur>
options nouveau modset=0

# Bloquer le chargement d'un module
vim /etc/modprobe.d/blacklist.conf
blacklist <nom_du_module>
blacklist nouveau
```

![image](https://github.com/user-attachments/assets/c7bc0797-f0a1-4e92-8580-8676a01a192e)


<br>

**Fichiers d'informations et fichiers de périphériques**

<br>

Les commandes **lspci**, **lsusb** et **lsmod** récupèrent les informations dans des fichiers spéciaux stockés dans les pseudo-systèmes de fichiers **/proc** et **/sys**.
Ces répertoires sont des points de montage dans l'espace RAM pour stocker la configuration d'exécution et les informations sur les processus en cours.
Ces pseudo-fs n'existe que lorsque le système est en cours d'exécution.

![image](https://github.com/user-attachments/assets/46802ea1-052b-41fa-a673-213bde1735b5)

```bash
# Exemples de fichiers dans /proc
# Informations détaillées sur les processeurs du système
/proc/cpuinfo

# Liste des numéros des interruptions par I/O pour chaque CPU
/cpu/interrupts

# Liste les plages de ports d'I/O actuellement enregistrées et utilisées
/proc/ioports

# Liste des cannaux DMA (Direct Memory Access)
/proc/dma
```

Les répertoires **/dev** est directement lié aux devices. Chaque fichiers à l'intérieur de **/dev** est associé à un device (/dev/hda1, /dev/hda2, ...)

Les périphériques amovibles sont gérés par le sous-système **udev**, qui créé les devices correspondants dans **/dev**. Le noyau Linux capture l'évènement de détection hardware et le transmet au processus udev, qui identifie le périphérique et crée dynamiquement les fichiers correspondants dans /dev, en utilisant un jeu de règles prédéfinies. **udev** se charge des devices déjà présents lors du démarrage du système et ceux ajoutés lorsque le système est en cours d'exécution. **udev** s'appui sur le **sysFS (/sys)** pour les informations relatives au hardware. Le répertoire contenant les règles prédéfinies est **/etc/udev/rules.d/**

Voici les dénominations en fonction des types de storage devices :
- /dev/hda, /dev/hdb, /dev/hdc, ... -> SATA, SCSI, ...
- /dev/fd0, /dev/fd1, /dev/fd3, ... -> FLoppy
- /dev/sda1, /dev/sda2, /dev/sdb1, ... -> IDE, SSD, USB, ...
- /dev/mmcblk0p1, /dev/mmcblk0p2, /dev/mmcblk1p1, ... -> Carte SD
- /dev/nvme0n1p1, /dev/nvme0n1p2, ... -> NVMe

<br>

---

### 101.2 **Détermination et configuration des paramètres du matériel** 🚀

**Sujet abordés :**
- Passage de commandes au chargeur de démarrage et passage de paramètres d'amorçage au noyau
- Démontrer sa connaissance des séquences d’amorçage depuis le BIOS / UEFI jusqu’à l’achèvement des séquences de démarrage
- Compréhension de l’init SysV et de systemd
- Sensibilisation à Upstart
- Consulter les événements de la phase de démarrage dans les journaux (logs)

**Commandes associées :**
- dmesg
- journalctl
- BIOS
- UEFI
- bootloader
- kernel
- initramfs
- init
- SysVinit
- systemd

<br>

**BIOS (Basic Input/Ouput System)**

![image](https://github.com/user-attachments/assets/2a6a1541-2889-4cdc-b476-ea12fa756153)

Les **étapes préopératoires pour amorcer un système équipé d'un BIOS** sont les suivantes :
- L'auto-test **POST** (Power-On Self-Test)
- Activation des composants basiques comme la **sortie vidéo**, le **clavier** et les **médias de stockage**
- Exécution de la **première phase du chargeur de démarrage** à partir du MBR (les 440 premiers octets)
- Exécution de la **seconde phase du chargeur de démarrage**

**UEFI (Unified Extensible Firmware Interface)**

L'**UEFI** ne s'appuie pas sur le MBR mais **NVRAM** (Non Volatile RAM) attachée à la carte mère dans laquelle se trouve dans **applications EFI** (chargeur d'amorçage, sélecteurs d'OS, outils de diagnostic, ...) contenues dans la **EFI System Partition** (**ESP**). Les systèmes de fichiers compatibles sont le FAT12, le FAT16 et le FAT32. 

Les **étapes préopératoires pour amorcer un système équipé d'un UEFI** sont les suivantes :
- L'auto-test **POST** (Power-On Self-Test)
- Activation des composants basiques comme la **sortie vidéo**, le **clavier** et les **médias de stockage**
- 
