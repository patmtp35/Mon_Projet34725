# README_MP2.md
# Configuration primaire MultiPlus-II (MP2)

## Présentation

Ce document décrit la configuration primaire du **Victron MultiPlus-II (MP2)**
à l'aide de l'outil Victron (VEConfigure ou équivalent).

Cette configuration doit être adaptée à **chaque installation** et en
particulier aux batteries utilisées.

⚠️ Les valeurs indiquées doivent être adaptées à votre batterie.

---

## Important

Si vous utilisez :

- SmartShunt
- MPPT SmartSolar

Il est **IMPORTANT** que les paramètres batteries soient **identiques** sur :

- MultiPlus-II
- SmartShunt
- MPPT SmartSolar
- VenusOS (DVCC si activé)

Sinon vous risquez :

- Mauvaise estimation du SOC
- Mauvaise charge batterie
- Passage prématuré en Float
- ESS instable

Les paramètres doivent être configurés via :

- VEConfigure (MultiPlus-II)
- Application VictronConnect (SmartShunt et MPPT)

---

## Outil utilisé

Configuration réalisée avec :

- Interface MK3 USB
- VEConfigure

Connexion :

MultiPlus-II → MK3 → PC

---

## Paramètres batteries

A adapter selon votre batterie.

Exemple :

- Type batterie : LiFePO4
- Tension système : 48V
- Capacité : à adapter

Exemples de paramètres :

Bulk voltage :
à adapter

Absorption voltage :
à adapter

Float voltage :
à adapter

Absorption time :
à adapter

Tail current :
à adapter

Low voltage shutdown :
à adapter

Restart voltage :
à adapter

---

## Synchronisation des paramètres

Les paramètres suivants doivent être identiques :

- Bulk voltage
- Absorption voltage
- Float voltage
- Absorption time
- Charge current limit

Sur :

- MultiPlus-II
- MPPT SmartSolar
- SmartShunt
- VenusOS DVCC

---

## DVCC (VenusOS)

Si DVCC est activé :

VenusOS pilote les chargeurs.

Dans ce cas :

- Vérifier tension max DVCC
- Vérifier courant max DVCC

Menu :

Settings → DVCC

---

## Vérifications

Après configuration :

- Vérifier que la charge démarre correctement
- Vérifier passage Bulk → Absorption → Float
- Vérifier SOC cohérent
- Vérifier tension identique sur tous les appareils

---

## Notes importantes

Cette configuration :

- est adaptée à chaque installation
- doit être validée électriquement
- doit respecter les spécifications batterie

Une mauvaise configuration peut :

- endommager les batteries
- réduire la durée de vie
- provoquer un fonctionnement ESS incorrect

---

## Recommandation

Toujours :

1 Configurer MP2

2 Configurer MPPT

3 Configurer SmartShunt

4 Vérifier DVCC

5 Vérifier VenusOS

