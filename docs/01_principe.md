# Principe du projet

## 1. Le sonar à balayage latéral (SSS)

- Le sonar émet une impulsion sonore vers la gauche (bâbord) et vers la droite (tribord).
- Il écoute les échos : plus un écho revient tard, plus l'objet est loin ;
  et plus il est fort, plus l'objet réfléchit le son (le câble lui réfléchit fort).
- Une émission = un ping = une ligne d'intensités (0 à 255) côté gauche et côté droit.
- Juste sous l'AUV, il n'y a que de l'eau : c'est la colonne de l'eau,
  une bande sombre au centre de l'image.

En empilant les pings au fur et à mesure que l'AUV avance, on obtient une image :
ici le waterfall. Le câble y apparaît comme une ligne.

Si on sait où est la ligne, on sait où est le câble par rapport à l'AUV, et on peut corriger le cap pour le suivre.

## 2. La chaîne de traitement

| Étape | Ce qui entre | Ce qui sort |
|---|---|---|
| Unity | — | un ping sonar |
| `waterfall_node` | pings | image waterfall |
| `cable_detector_cv` | image | position et angle du câble |
| `cable_follower` | position du câble + le cap actuel | cap et vitesse à suivre |
| Unity | consigne | l'AUV tourne |

En gros c'est une boucle fermée : l'AUV tourne, le sonar voit le câble autrement. Il y a donc une nouvelle détection et nouvelle consigne.

## 3. Le pont Unity vers ROS

Comme Unity ne parle pas le protocole de ROS 2. Un pont TCP relie les deux :

| | ROS-TCP-Connector | ROS-TCP-Endpoint |
|---|---|---|
| Où | Dans Unity (plugin C#) | Côté ROS (nœud Python) |
| Rôle | Envoie les données de la simulation, reçoit les commandes | Republie les données sur des topics ROS et renvoie les commandes à Unity |

Ce qui passe sur le pont :

- Unity vers ROS : `SideScanSonar` sur `/unity_simulation/side_scan_data` ;
- ROS vers Unity : `AUVCommand` sur `/auv/offboard_cmd`, et le mode sur `/auv/set_mode`.

!!! "Nous n'avons pas besoin de Unity pour l'instant"
    L'enregistrement `cable_test_20261012_203459` (`ros2 bag`) rejoue les messages tels
    qu'ils avaient été publiés : il remplace à la fois Unity et le pont.
    Il contient `/unity_simulation/side_scan_data` et `/sidescan/waterfall`.
