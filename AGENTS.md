# AGENTS.md — ConnectedDogBowl

## But

Gamelle qui se pèse : cellule de charge + HX711 lus par un NodeMCU (ESP8266) sous ESPHome, mesure envoyée en
litres à Home Assistant par l'API native. Projet de 2020-2021, un seul fichier de configuration, pas de code compilé à la main.

## Fichiers

| Chemin | Rôle |
|---|---|
| `dogbowl.yaml` | configuration ESPHome complète du nœud (seul fichier « code ») |
| `README.md` | tutoriel humain en français + résumé anglais |
| `readmeImgSrc/` | images du README (`image5.webp` photo du montage, `image2.jpg` le chien, `image3.jpg` tableau de bord, `image4.png` câblage, `image1.jpg` / `image6.jpg` / `image7.jpg` photos de composants des vendeurs) |
| `LICENSE` | MIT, Nicolas Holder |

## Versions

- Syntaxe ESPHome de 2020-2021 : `platform: ESP8266` et `board: nodemcuv2` sous `esphome:`, `api.password`,
  `ota.password` sans `platform`. Les versions récentes d'ESPHome attendent un bloc `esp8266:`, une clé
  `api.encryption.key` et `ota: - platform: esphome`. Ne migrer que sur demande, en gardant le même comportement.
- Nom du nœud `dog_bowl` (le tiret bas est déconseillé par les ESPHome récents ; le renommer change l'entité côté Home Assistant).

## Structure de `dogbowl.yaml`

- `esphome:` nom et carte (`nodemcuv2`).
- `wifi:` réseau principal + point d'accès de secours `ap:` ; `captive_portal:` pour reconfigurer le wifi.
- `logger:` journaux (sert à lire la valeur brute pendant l'étalonnage).
- `api:` connexion native à Home Assistant ; `ota:` mises à jour sans fil.
- `sensor:` une seule entrée `platform: hx711` :
  - `dout_pin: D0`, `clk_pin: D1`, `gain: 128`, `update_interval: 10s` ;
  - filtres dans l'ordre : `filter_out: nan` → `calibrate_linear` → `lambda` qui ramène toute valeur négative à 0 ;
  - `unit_of_measurement: litres`, `accuracy_decimals: 2`, `name: "dog bowl"`.

## Champs à remplacer avant de flasher

| Champ | Valeur dans le dépôt | À mettre |
|---|---|---|
| `wifi.ssid` | `your_ssid` | nom du réseau |
| `wifi.password` | `your_ssid_password` | mot de passe wifi |
| `wifi.ap.password` | `change_me_fallback` | mot de passe du point d'accès de secours |
| `api.password` | `your_HA_API_password` | mot de passe API (ou clé de chiffrement en ESPHome récent) |
| `ota.password` | `your_password` | mot de passe OTA |
| `sensor[0].dout_pin`, `clk_pin` | `D0`, `D1` | seulement si le câblage diffère |
| `calibrate_linear` | voir ci-dessous | points mesurés sur sa propre balance |

Préférer `!secret` (fichier `secrets.yaml` d'ESPHome) aux valeurs en clair.

## Étalonnage

```yaml
- calibrate_linear:
  - 164800 -> 0
  - 555000 -> 1
  - 943700 -> 2
```

Forme « valeur brute HX711 -> valeur renvoyée ». Les points du dépôt correspondent à 0, 1 et 2 litres sur la
balance de l'auteur (environ 390 000 points bruts par litre). Ils ne sont pas transposables : relever la valeur
brute dans les journaux avec la gamelle vide, puis avec 1 L et 2 L, et remplacer les trois lignes.

## Câblage (invariant documenté)

HX711 VCC → 3.3V, GND → GND, DAT → D0, CLK → D1. Si une broche change dans le YAML, mettre à jour le tableau
du README et ne pas modifier `readmeImgSrc/image4.png` en silence.

## Côté Home Assistant

Le nœud est découvert automatiquement. Les alertes de l'auteur (seuil 0,25 L : SMS IFTTT, TTS, popup browser_mod)
vivent dans un autre dépôt, `HomeAssistantPublicConfig`, sous `automation/notification/romeo/`, avec l'entité
renommée `sensor.eau_romeo`.

## Invariants

- Un seul nœud, un seul capteur ; garder le filtre `nan` et le plancher à 0.
- Garder le README en français avec son résumé anglais à jour (le dépôt a des lecteurs étrangers).
- Ne rien inventer sur le matériel : les prix et le temps de montage du README sont ceux de l'auteur.

## Ne jamais commiter

- Vrais SSID et mots de passe wifi, mots de passe API / OTA / point d'accès, clés de chiffrement, jetons.
- `secrets.yaml`, dossier `.esphome/`, binaires compilés.
- Adresses IP, coordonnées GPS, adresse, numéros de téléphone.
- Prénoms réels (seul Nicolas est citable ; le chien s'appelle Roméo).
