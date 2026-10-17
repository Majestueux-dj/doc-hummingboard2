# Travail sur la carte (SSH)

À partir du module `sidescan_processing`, le travail ne se fait plus sur le PC mais directement sur la HummingBoard, depuis le PC, par une connexion SSH.

## 1. Pourquoi je suis passer directement sur la carte

| Sur le PC | Sur la carte |
|---|---|
| Ubuntu 26.04 + ROS 2 Lyrical | Conteneur Ubuntu 22.04 + ROS 2 Humble |

## 2. Mise en place du serveur SSH sur la carte

Côté Debian : 

```bash
apt update
apt install -y openssh-server
update-rc.d ssh defaults      # Pour le démarrage automatique à chaque boot
```

## 3. Pour se connecter depuis le PC

La carte étant sur le même réseau Wi-Fi que le PC. J'obtiens son adresse IP avec `ip a` et je regarde l'ip attribué par le réseau sans fil sur la carte.
Ici puisque je travaille directement sur une box personnelle l'ip attribué est le 192.168.1.81
 
```bash
ssh root@192.168.1.81
```

Une fois connecté, on entre dans le conteneur ROS 2 avec l'alias `ros`
(`docker exec -it project_ros bash`, voir la première partie). Chaque fenêtre SSH + `ros`
donne un terminal indépendant : un pour le nœud, un pour le bag, un pour faire les vérifications.

```mermaid
flowchart LR
    PC["PC<br/>terminal 1, 2, 3"] -->|ssh| D["Debian<br/>(carte)"] -->|ros| C["Conteneur<br/>ROS 2 Humble"]
```

## 4. Mettre le projet et l'enregistrement sur la carte (À changer mais j'en parle puisqu'il faisait partie de la liste de ce que j'ai effectué)

Les fichiers sont copiés du PC vers le dossier partagé `/root/project/` (= `/project/`
dans le conteneur) :

```bash
# depuis le PC
scp -r ~/Desktop/Projet_M2/ros2_ws_sss root@192.168.1.81:/root/project/
scp -r ~/Desktop/Projet_M2/cable_test_20261012_203459 root@192.168.1.81:/root/project/
```

| Dossier dans le conteneur | Contenu |
|---|---|
| `/project/ros2_ws_sss/` | Projet à prendre à main |
| `/project/ros2_ws_sss_project/src/` | Mon workspace, un package copié à la fois |
| `/project/cable_test_20261012_203459/` | L'enregistrement (5,3 Go) |

## 5. Problèmes rencontrés et solutions

### Espace disque insuffisant

L'image Debian a été générée avec un rootfs de 8 Go, trop petit pour le bag de 5,3 Go.
La partition a été ensuite agrandie à toute la microSD pour 32 Go :

```bash
lsblk                         # mmcblk1p1 = boot, mmcblk1p2 = rootfs
growpart /dev/mmcblk1 2       # agrandit la partition 2
resize2fs /dev/mmcblk1p2      # agrandit le système de fichiers
df -h /                       # On doit voir que la partition a augmenté
```
