
# Intégration Home Assistant ↔ Victron MultiPlus-II (MP2)

## Présentation

Ce document décrit comment **Home Assistant (HA)** interagit avec l’**onduleur/chargeur Victron MultiPlus-II (MP2)** dans l’architecture actuelle.

L’installation repose sur :

- Victron MultiPlus-II 48/3000
- MPPT Victron
- SmartShunt 500A
- Venus OS sur Raspberry Pi (GX)
- MQTT
- Home Assistant
- ESS Victron (Energy Storage System)

Home Assistant ne pilote directement le MultiPlus-II. et il communique aussi avec **Venus OS**, qui agit comme contrôleur central du système Victron.

Architecture simplifiée :

Panneaux solaires → MPPT → Batterie → MultiPlus-II → Maison / Réseau  
↑  
Venus OS  
↑  
MQTT  
↑  
Home Assistant


---

# Architecture du système

## Couche Victron

L’écosystème Victron gère :

- la charge batterie
- l’onduleur
- les flux réseau / maison
- la logique ESS

| Équipement | Rôle |
|------------|------|
| MultiPlus-II | Onduleur / chargeur |
| MPPT | Régulateur solaire |
| SmartShunt | Surveillance batterie |
| Venus OS | Contrôleur central |


---

# Flux de communication

La communication passe par **MQTT exposé par Venus OS**.

Victron Device → D-Bus Venus OS → dbus-mqtt → MQTT Broker → Home Assistant

Home Assistant peut donc :

- lire les états du système
- envoyer des commandes ESS
- déclencher des automatisations


---

# Contrôles principaux du MP2 depuis Home Assistant

## Consigne de puissance réseau (Grid Setpoint)

Permet de définir la puissance échangée avec le réseau.

Entité typique :

number.gx_device_point_de_consigne_de_puissance_ca

Utilisation :

| Consigne | Comportement |
|----------|-------------|
| 0 W | neutre |
| négatif | injection réseau |
| positif | import réseau |

Exemple :

Grid setpoint = -50W  
Permet d’éviter les imports réseau.


---

# Limitation du courant de charge ESS

Permet de limiter la vitesse de charge de la batterie.

Entité :

number.system_setup_courant_de_charge_maximal_ess

Utilisation :

- limiter la charge nocturne
- protéger les batteries
- adapter la charge au surplus solaire


---

# Limitation de puissance de l'onduleur

Permet de limiter la puissance fournie par la batterie.

Entité :

number.gx_device_limite_de_puissance_maximale_de_l_onduleur_ess

Utilisation :

- bloquer la décharge
- protéger la batterie
- limiter la puissance délivrée


---

# Mode pause du MultiPlus-II

Le MP2 peut être mis en pause virtuellement en définissant :

puissance onduleur = 0 W  
courant de charge = 0 A

Résultat :

- charge batterie bloquée
- décharge batterie bloquée
- système toujours alimenté

Utilisé pour :

- protection batterie
- maintenance
- équilibrage batterie


---

# Contrôle de l'entrée AC

Dans cette installation l’entrée AC peut être coupée via une prise connectée.

Entité :

switch.prise_prod_batteries

Lorsque l’entrée AC est coupée :

- le réseau est déconnecté
- la charge réseau est impossible
- le MP2 passe en mode onduleur


---

# Surveillance de la batterie

Les informations batterie proviennent du SmartShunt :

sensor.victron_unified_interface_smartshunt500a_battery_power  
sensor.gx_device_tension_de_la_batterie_cc  
sensor.gx_device_soc_cible_dynamic_ess

Utilisation :

- déclenchement automatisations
- protection batterie
- optimisation solaire


---

# Protection contre le déséquilibre des batteries

Surveillance via :

sensor.bms1_24v150ah_adc  
sensor.bms2_adc  
sensor.battery_voltage_delta

Logique typique :

| Écart | Action |
|------|------|
| <0.4V | normal |
| 0.4-0.7V | alerte |
| >0.7V | protection |

Actions possibles :

- pause MP2
- arrêt de charge
- notification

Reprise automatique lorsque l’équilibre revient.


---

# Gestion du surplus solaire

Home Assistant pilote plusieurs consommateurs pour absorber le surplus solaire.

Exemples :

- radiateurs
- chauffe‑eau thermodynamique
- routeur solaire
- recharge batterie

Logique simple :

Production solaire > consommation  
→ activation charges


---

# Gestion tarifaire Tempo

Le système intègre les signaux tarifaires RTE Tempo.

Capteurs :

sensor.rte_tempo_couleur_actuelle  
sensor.rte_tempo_prochaine_couleur

Exemple :

Si demain = jour rouge  
et SOC batterie < 80 %

→ blocage décharge batterie  
→ stockage énergie


---

# Automatisations de sécurité

Exemples :

Protection décharge batterie :

battery power < -50W  
→ arrêt radiateurs solaires

Surveillance alarmes MP2 :

- ripple
- batterie faible
- perte réseau

Les alertes utilisent :

script.alert_manager


---

# Avantages de l’intégration Home Assistant

Cette architecture permet :

- automatisations avancées
- optimisation autoconsommation
- protection batterie
- gestion tarifaire
- supervision complète

Home Assistant agit comme un **EMS (Energy Management System)** au-dessus de Victron ESS.


---

# Limites

Points importants :

- ESS Victron reste le contrôleur principal
- éviter les boucles d'automatisation
- respecter les limites du système
- ne pas court-circuiter les sécurités Victron


