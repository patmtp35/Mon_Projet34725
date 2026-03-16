# ⚡ Configuration Victron ESS – Installation Patrick

Documentation de la configuration stable après optimisation du système Victron.

## Architecture

Installation composée de :

- MultiPlus-II 48/3000/35
- 2 batteries LiFePO4 CloudEnergy 24V 150Ah en série (48V / 150Ah)
- SmartShunt 500A
- 2 MPPT Victron SmartSolar
- 4 micro-onduleurs APsystems
- 8 panneaux 500 W dédiés à l'autoconsommation / AC-coupling (2 panneaux par APsystems)
- VenusOS sur Raspberry Pi
- ESS actif
- Home Assistant pour pilotage du surplus
- Routeur solaire MSUNPV
- Radiateurs pilotés

Objectifs atteints :

- Charge complète **100 %**
- Déséquilibre batterie très faible (**≈ 0.09 V**)
- ESS stable
- SOC cohérent

---

# Batteries

Configuration :

```text
Type : LiFePO4
Configuration : 2 batteries 24V en série
Capacité totale : 150Ah
Tension nominale : 48V
```

---

# MultiPlus-II (VE Configure)

## Général

```text
Enable battery monitor : ON
Battery capacity : 150 Ah
Charge efficiency : 0.95
SOC end of bulk : 95 %
```

## Chargeur

```text
Charger enabled : ON
Lithium batteries : ON
Charge curve : Fixed
```

### Tensions

```text
Absorption voltage : 54.8 V
Float voltage : 53.6 V
```

### Temps

```text
Absorption time : 1 h
Repeated absorption interval : 7 days
```

### Courant

```text
Charge current : 35 A
```

---

# MPPT SmartSolar

Configuration identique pour tous les MPPT.

```text
Battery voltage : 48V
Charger enabled : ON
Expert mode : ON
```

### Tensions

```text
Absorption voltage : 54.8 V
Float voltage : 53.6 V
Equalization : OFF
```

### Bulk

```text
Re-bulk voltage offset : 0.20 V
```

### Absorption

```text
Absorption duration : 25 min
Tail current : 4.5 A
```

---

# APsystems / AC-coupling

Configuration :

```text
4 micro-onduleurs APsystems
2 panneaux de 500 W par APsystems
Soit 8 panneaux / 4000 Wc environ
Usage principal : autoconsommation + AC-coupling
```

Rôle dans le système :

- production AC directe pour les charges maison
- participation à l'autoconsommation
- soutien du système ESS côté AC-coupling
- réduction de la sollicitation batterie en journée

---

# SmartShunt 500A

## Paramètres batterie

```text
Battery capacity : 150 Ah
Charged voltage : 54.8 V
Tail current : 4 %
Charged detection time : 3 min
Peukert exponent : 1.05
Charge efficiency : 99 %
```

## Divers

```text
Discharge floor : 25 %
Current threshold : 0.10 A
Time-to-go averaging : 3 min
```

---

# ESS (VenusOS)

## Limites

```text
Minimum SOC : 15 %
Maximum inverter power : 1900 W
ESS charge current : 30 A
```

## Grid

```text
Grid setpoint : -10 W
Peak shaving : Disabled
Limit AC import : OFF
Limit AC export : OFF
```

---

# DVCC

```text
Limit charge current : ON
Maximum charge current : 30 A
Limit managed battery voltage : ON
Maximum charge voltage : 56.2 V

SVS : ON
STS : ON
SCS : ON
```

---

# Grid Feed-in

```text
AC coupled PV feed-in : ON
DC coupled PV feed-in : ON
Limit system feed-in : OFF
Feed-in limiting active : NO
```

---

# Home Assistant – Logique énergétique

Priorité de consommation du surplus :

1. Maison
2. Radiateurs pilotés
3. Routeur solaire MSUNPV
4. Injection réseau

Prises pilotées :

```text
switch.prise_rad_solaire_salon
switch.prise_rad_solaire_garage
switch.prise_rad_solaire_tv
```

Le script :

```text
script.msunpv_routage_on_off
```

n'est plus utilisé.

---

# Résultat obtenu

Cycle de charge observé :

```text
100 % atteint vers 17:59
BMS1 : 27.10 V
BMS2 : 27.19 V
Delta : ~0.09 V
```

Comportement ESS :

```text
Batterie pleine → passage en Float
Maison alimentée par batterie
Grid proche de 0W
```

---

# Réglage final validé

```text
Absorption : 54.8 V
Float : 53.6 V
Tail current : 4 %

SOC min ESS : 15 %
Grid setpoint : -10 W
```

Ce réglage permet :

- charge complète
- faible déséquilibre
- fonctionnement ESS stable

---

