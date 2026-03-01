# Victron Unified ESP32 → Venus OS (MQTT)

Ce dépôt décrit une config **ESPHome/ESP32** qui agrège **BLE + VE.Direct** (redondant) pour remonter des équipements Victron dans **Venus OS** via **MQTT**, en mode “GX-like” (télémétrie + historique minimal).

> ⚠️ Important : le fichier `.common.yaml` ne gère **que le Wi‑Fi / API / OTA** (rien lié à Venus OS).

---

## Matériel couvert (conforme au YAML v6.2.0)

- **1 × SmartShunt 500A**
- **2 × SmartSolar MPPT 100/20** (nommés :
  - `mppt_100_20`
  - `mppt_100_20_2`)

Sources de données (redondance) :
- **BLE** (prioritaire quand disponible)
- **VE.Direct** (fallback, et utilisé en priorité pour certaines valeurs “stables”)

---

## Dépendances côté Venus OS (obligatoire)

### 1) Virtual device “battery” via dbus-mqtt-devices (freakent)
Ce service lit les topics `device/<clientId>/Proxy` + `device/<clientId>/Status` et crée le device virtuel (battery) dans dbus/Venus.

Repo : https://github.com/freakent/dbus-mqtt-devices (branche/versions selon ton choix)

### 2) Virtual “solarcharger” via venus-os_dbus-mqtt-solar-charger (mr-manuel)
Ce service consomme les topics `venus/solarcharger/...` et crée les services `solarcharger` dans dbus/Venus.

Repo : https://github.com/mr-manuel/venus-os_dbus-mqtt-solar-charger

> Avec 2 MPPT, il faut **2 instances** (2 fichiers de conf) avec des `device_instance` différents.

---

## Topics MQTT utilisés

### A) Battery (SmartShunt) → Venus (via dbus-mqtt-devices)

**Status (présence/keep services)**
- Topic : `device/victron_unified/Status` (retain=true)
- Envoyé :
  - au `on_connect`
  - toutes les ~30s (refresh)

**Proxy (données)**
- Topic : `device/victron_unified/Proxy` (retain=false)
- Fréquence : ~2s
- Contenu : JSON avec `topicPath` (fourni par DBus) + `values` mappées GX-like.

**Keepalive**
- Topic : `R/<portalId>/keepalive`
- Fréquence : ~40s

> `portalId` + `topicPath` sont appris via un message entrant :
> - Topic : `device/victron_unified/DBus`
> - Champs attendus : `portalId`, `topicPath.battery1.W`, `topicPath.solarcharger1.W`, `topicPath.solarcharger2.W`

### B) Solar chargers (MPPT) → Venus (via mr-manuel)

- Topic MPPT 1 : `venus/solarcharger/mppt_100_20` (retain=true, ~5s)
- Topic MPPT 2 : `venus/solarcharger/mppt_100_20_2` (retain=true, ~5s)

---

## Mapping des valeurs (ce que publie l’ESP)

### 1) Batterie (SmartShunt) – valeurs principales

Le JSON publié dans `device/victron_unified/Proxy` alimente notamment :

- `Dc/0/Voltage`
- `Dc/0/Current`
- `Dc/0/Power`
- `Soc`
- `ConsumedAh`
- `InstalledCapacity`
- `History/Daily/0/MinVoltage`
- `History/Daily/0/MaxVoltage`
- `TimeToGo` (minutes, si dispo)

Choix de sources (logique) :
- BLE prioritaire quand non-NaN
- Fallback VE.Direct
- Pour `Dc/0/Power`, priorité à `smartshunt500a_vd_power` (VE.Direct) si dispo, sinon calcul/template.

### 2) MPPT – payload “clean” (GX-like minimal)

Chaque MPPT publie un JSON de forme (simplifié) :

- `Pv.V`
- `Yield.Power`
- `Yield.System` (kWh lifetime, côté ESP)
- `Dc.0.Voltage` (batterie)
- `Dc.0.Current` (courant charge)
- `History.Daily.0.Yield`
- `History.Daily.0.MaxPower`
- `History.Daily.0.MaxPvVoltage`
- `ErrorCode`
- `State`
- `Mode`
- `MppOperationMode`

Notes :
- `State` est dérivé de `charging_mode_id` VE.Direct quand disponible.
- La tension batterie MPPT peut fallback sur la tension SmartShunt si besoin.

---

## Ce que le projet NE fait PAS (important)

- ❌ **Pas de pilotage ESS/DVCC** (aucun contrôle, aucune consigne écrite vers Venus).
- ❌ Pas de modification de paramètres MPPT / Multi via MQTT.
- ✅ Uniquement **télémétrie** + un peu d’“historique daily” (min/max voltage, yield, max power…).

---

## Configuration des 2 instances mr-manuel (exemple)

Exemple de squelette INI (à adapter) :

### MPPT 1
- `device_name = SmartSolar MPPT 100/20`
- `device_instance = 20`
- `topic = venus/solarcharger/mppt_100_20`

### MPPT 2
- `device_name = SmartSolar MPPT 100/20_2`
- `device_instance = 21` (ou autre)
- `topic = venus/solarcharger/mppt_100_20_2`

Astuce : `history_days = 14` (si tu as constaté le bug <14).

---

## Dépannage rapide

- Si rien n’apparaît dans Venus :
  - Vérifier que MQTT est activé sur Venus OS et que les identifiants sont corrects.
  - Vérifier que `device/victron_unified/DBus` est bien reçu par l’ESP (log ESPHome).
  - Vérifier que `dbus-mqtt-devices` tourne bien côté Venus.
  - Vérifier que **les 2 instances** mr-manuel tournent (une par MPPT).

- Si le battery virtuel “disparaît” :
  - vérifier `device/victron_unified/Status` (retain) + le keepalive `R/<portalId>/keepalive`.

---

## Version

- YAML : **v6.2.0** (GX ESS improvements / MQTT Venus OS improvements)
