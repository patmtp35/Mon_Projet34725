# Victron ESS Integration via Shelly UNI & Shelly Pro 3EM

This project integrates Shelly devices into a Victron ESS system using Node-RED.

It provides:

- Virtual Victron Grid Meter
- Virtual Victron PV Inverter (in my cas 4 DS3-H ApsSystems)
- Battery monitoring (voltage + temperature)
- Smart DVCC charge control
- ESS stabilization

No CAN battery communication is required.

---

# Architecture

Shelly UNI → Node-RED → DVCC → ESS → MultiPlus

Shelly Pro 3EM → Node-RED → Virtual Grid Meter → ESS

SmartShunt → VenusOS → SOC + Battery Current

---

# Shelly UNI Flow (Battery Monitoring)

## Purpose

This flow turns two Shelly UNI devices into a software Battery Management System.

It monitors:

- Battery 1 voltage
- Battery 2 voltage
- Voltage delta
- Battery temperatures
- Temperature delta
- Battery SOC (SmartShunt)

---

## Measurements

Shelly UNI provides:

- ADC voltage measurement
- DS18B20 temperature sensors

Voltage and temperature are stored and compared.

---

## Voltage Delta Protection

Voltage delta:

delta = | batt1 - batt2 |

Used to detect imbalance between series batteries.

Thresholds:

- <0.35V → OK
- >0.35V → Limit charge
- >0.7V → Strong limit
- >0.9V → Block charge

---

## Temperature Protection

Minimum battery temperature controls DVCC current.

Limits:

- <0°C → Charge blocked
- 0–3°C → 5A
- 3–6°C → 10A
- 6–10°C → 20A
- >10°C → Normal

---

## SOC Protection

SmartShunt SOC is used to reduce charge current near full.

Limits:

- <80% → Normal
- 80–90% → 25A
- 90–95% → 15A
- >95% → 5A

Purpose:

- Prevent aggressive bulk charging
- Improve ESS stability
- Protect cells

---

## ESS Commands

DVCC Limit:

{ dvcc_charge_limit: XX }

Charge Block:

{ mode: "keep_charged" }

---

# Shelly Pro 3EM Flow (Grid + Solar)

## Purpose

This flow converts a Shelly Pro 3EM into:

- Victron Grid Meter
- Victron PV Inverter

Compatible with ESS and VRM.

---

## Grid Meter

Provides:

- Grid power
- Voltage
- Current
- Imported energy
- Exported energy

Used by ESS for:

- Zero injection
- Power compensation
- Consumption calculation

---

## PV Inverter

Provides:

- Solar production power
- Solar energy
- Voltage
- Current

Used for:

- VRM solar graphs
- ESS calculations

---

## ESS Stabilization Filters

### Deadband

±10W

Removes jitter around zero.

---

### EMA Filter

Alpha = 0.35

Smooths power measurement.

---

### Rate Limit

Max step = 80W

Prevents ESS oscillations.

---

# Features

- Stable ESS operation
- Accurate grid measurement
- Solar production tracking
- Battery protection
- Smart DVCC control
- No CAN battery required

---

# Hardware

Typical setup:

- Victron MultiPlus II
- Victron SmartShunt
- Shelly UNI (x2)
- Shelly Pro 3EM
- VenusOS device


