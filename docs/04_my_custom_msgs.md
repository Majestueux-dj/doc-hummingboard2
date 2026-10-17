# my_custom_msgs

## les messages du projet

## 1. Principe

Pour que deux nœuds se parlent, ils doivent être d'accord sur le format des données :
quels champs, de quel type. Ce package définit ces formats.

- À la compilation, ROS génère automatiquement les classes C++ et Python correspondantes.
- Tous les autres packages en dépendent : il est compilé en premier.

## 2. Les 4 messages

| Message | Topic | Publié par | Lu par |
|---|---|---|---|
| `SideScanSonar` | `/unity_simulation/side_scan_data` | Unity | `waterfall_node`, `cable_follower` |
| `CableDetection` | `/cable_detection` | `cable_detector_cv` (ou `_ai`) | `cable_follower` |
| `AUVCommand` | `/auv/offboard_cmd` | `cable_follower` | Unity |
| `CableFollowerDebug` | `/cable_follower/guidance_debug` | `cable_follower` | affichage (`rqt_plot`) |

Entre `waterfall_node` et le détecteur, l'image est un message standard de ROS
(`sensor_msgs/Image`).

### `SideScanSonar` : entrée de la chaîne

| Champ | Signification |
|---|---|
| `range`, `range_resolution` | Portée (m) et mètres par case pour la conversion pixels en mètres |
| `port_data`, `starboard_data` | Intensités 0–255 à gauche / à droite, du proche au loin |
| `pos_x`, `pos_y`, `pos_z` | Position de l'AUV dans Unity |
| `altitude`, `depth` | Hauteur au-dessus du fond, profondeur |
| `pitch`, `roll`, `heading` | Attitude et cap en degrés |
| `speed` | La Vitesse en m/s |

### `CableDetection` : le câble est ici

| Champ | Signification |
|---|---|
| `header` | Horodatage recopié de l'image source (mesure du retard de la chaîne) |
| `detected`, `confidence` | Câble trouvé ou non, et confiance |
| `source` | `SOURCE_CV` (0) ou `SOURCE_AI` (1) |
| `p1`, `p2` | Extrémités du segment dans l'image (pixels) |
| `angle_rad` | Angle du câble par rapport à la verticale de l'image |
| `lateral_offset_px` | Distance AUV → câble en pixels (+ = tribord) |

### `AUVCommand` : la consigne (sortie de la chaîne)

| Champ | Signification |
|---|---|
| `heading_deg` | Cap à prendre (degrés) |
| `speed_mps` | Vitesse voulue (m/s) |

### `CableFollowerDebug` : l'intérieur du guidage

Distance au câble en mètres, écart à la distance visée, cap du câble, cap envoyé, côté suivi,
mode actif. Aucun nœud ne le lit : il sert à tracer des courbes pour régler le guidage.

## 3. Les fichiers de compilation

| Fichier | Élément clé | Rôle |
|---|---|---|
| `package.xml` | `rosidl_default_generators` | L'outil qui transforme les `.msg` en code |
| | `<member_of_group>rosidl_interface_packages</member_of_group>` | Déclare « ce package fournit des messages » |
| `CMakeLists.txt` | `rosidl_generate_interfaces(...)` | Génère le code des 4 messages |

Un `SideScanSonar.msg` devient la classe C++ `my_custom_msgs::msg::SideScanSonar`,
incluse avec `#include <my_custom_msgs/msg/side_scan_sonar.hpp>`.

## 4. Vérification

```bash
ros2 interface package my_custom_msgs                    # liste les 4 messages
ros2 interface show my_custom_msgs/msg/CableDetection    # affiche un message
ros2 topic hz /unity_simulation/side_scan_data           # avec le bag lancé
```
