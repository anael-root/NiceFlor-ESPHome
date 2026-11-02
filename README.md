# NiceFlor-ESPHome

Émulation d'une télécommande **Nice Flor-s** (433,92 MHz, rolling code) avec un ESP et [ESPHome](https://esphome.io), pour piloter un portail ou une porte de garage Nice depuis Home Assistant.

> [!NOTE]
> Ce dépôt est un **fork** de [TimelessNL/NiceFlor-ESPHome](https://github.com/TimelessNL/NiceFlor-ESPHome).
> Tout le mérite de l'encodage Nice Flor-s et de la configuration d'origine revient à ses auteurs.
> Ce fork ajoute la compatibilité **ESP32 / ESP-IDF** et met la configuration à jour pour les versions récentes d'ESPHome.

## Nouveautés de ce fork

| | Projet d'origine | Ce fork |
|---|---|---|
| ESP8266 (Arduino) | ✅ | ✅ |
| ESP32 (ESP-IDF) | ❌ | ✅ |
| ESP32 (Arduino) | non testé | non testé |
| ESPHome ≥ 2024.6 (nouvelle syntaxe `ota`) | ❌ | ✅ |

Compilation vérifiée avec **ESPHome 2026.10.1** sur `nodemcuv2` (ESP8266, Arduino) et `esp32dev` (ESP32, ESP-IDF).

### Modifications apportées

- **`niceflors/nice.h`** : le code ne dépend plus d'Arduino.
  - `pgm_read_word_near` est défini en lecture mémoire directe s'il n'existe pas (cas d'ESP-IDF).
  - Le type Arduino `byte` est remplacé par `uint8_t`.
  - Les timings sont renvoyés en `std::vector<int32_t>`, le type attendu par `remote_transmitter.transmit_raw` sur toutes les plateformes (avec la toolchain ESP-IDF, `int32_t` est un `long` et non un `int`).
- **`niceflors.yaml`** :
  - Le bloc `ota:` utilise la syntaxe actuelle (`- platform: esphome`).
  - Les lambdas utilisent `std::vector<int32_t>`.
  - La configuration cible maintenant un ESP32 (`esp32dev`, framework ESP-IDF).
  - Le capteur `adc` sur `VCC` (« Power Supply »), propre à l'ESP8266, a été retiré.

## Matériel

- Un ESP32 ou un ESP8266.
- Un émetteur 433,92 MHz (modulation OOK/ASK).

| Carte | Broche DATA de l'émetteur |
|---|---|
| ESP32 (`esp32dev`) | `GPIO23` |
| ESP8266 (`nodemcuv2`) | `D6` |

## Installation

1. Copiez `niceflors.yaml` et le dossier `niceflors/` dans votre dossier de configuration ESPHome.
2. Créez ou complétez `secrets.yaml` :
   ```yaml
   api_key: "<clé de chiffrement API en base64>"
   ota_password: "..."
   wifi_ssid: "..."
   wifi_password: "..."
   web_password: "..."
   ```
3. Choisissez votre plateforme dans `niceflors.yaml` :

   **ESP32 (ESP-IDF)**, configuration par défaut :
   ```yaml
   esp32:
     board: esp32dev
     framework:
       type: esp-idf

   remote_transmitter:
     pin: GPIO23
   ```

   **ESP8266** :
   ```yaml
   esp8266:
     board: nodemcuv2
     restore_from_flash: true

   remote_transmitter:
     pin: D6
   ```
   Vous pouvez aussi y ajouter la mesure de la tension d'alimentation, qui n'existe que sur ESP8266 :
   ```yaml
   sensor:
     - platform: adc
       pin: VCC
       name: "Power Supply"
   ```
4. Adaptez le numéro de série de la télécommande émulée (`remote_serial`) et, si besoin, les identifiants de boutons (`button_1` à `button_4`) dans la section `globals`.
5. Compilez et flashez :
   ```bash
   esphome run niceflors.yaml
   ```

## Utilisation

Une fois l'ESP ajouté à Home Assistant, les boutons suivants apparaissent :

| Bouton | Action |
|---|---|
| **Pairing** | Émet pendant environ 60 trames pour appairer la télécommande virtuelle avec le récepteur |
| **Step-by-Step** | Commande pas-à-pas (bouton 1) |
| **AUX** | Bouton 2 |
| **OPEN** | Bouton 3 |
| **CLOSE** | Bouton 4 |

Le rolling code est incrémenté à chaque appui et sauvegardé en flash. Le capteur texte **Code** affiche sa valeur courante.

Pour appairer, mettez le récepteur Nice en mode apprentissage (voir sa notice), puis appuyez sur **Pairing**.

## Licence

Apache License 2.0, comme le projet d'origine. Voir [LICENSE](LICENSE).
