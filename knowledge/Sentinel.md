# SENTINEL
## Smart Evidence-based Naval Tracking for Inspection and Notified Logging

---

# Project Overview

SENTINEL is a defence-grade naval safety inspection and monitoring platform developed by Arcaisys for the DISC Innovation Challenge.

The system replaces traditional paper-based naval safety rounds with a cryptographically secure, evidence-backed, digitally verifiable inspection ecosystem.

SENTINEL is designed specifically for:
- Naval ships
- Air-gapped military environments
- High-security compartments
- Real-time inspection accountability
- Tamper-proof audit logging

---

# Full Form

SENTINEL:
Smart Evidence-based Naval Tracking for Inspection and Notified Logging

---

# Problem Statement

Traditional naval safety rounds rely on:
- Manual compartment inspections
- Paper logbooks
- Human trust-based verification

This creates major operational vulnerabilities:

## Existing Problems
- Inspection logs can be falsified
- No real-time visibility of rounds
- No proof inspections actually occurred
- Paper records are fragile
- Records are difficult to audit
- No cryptographic accountability
- No structured inspection enforcement

---

# Core Objective

To create a tamper-evident, cryptographically verifiable, fully offline-capable naval inspection ecosystem capable of:

- Authenticating officers
- Tracking physical inspections
- Enforcing inspection tasks
- Capturing evidence
- Maintaining immutable audit trails
- Operating entirely on-premise

---

# Core System Philosophy

SENTINEL is built around five principles:

1. Physical Presence Verification
2. Cryptographic Accountability
3. Offline-First Operation
4. Evidence-Based Inspection
5. Air-Gapped Security

---

# Five-Layer System Architecture

SENTINEL uses a modular five-layer architecture.

---

# L1 — Input Layer

## Components
- Tier 1 Raspberry Pi Zero 2W nodes
- Tier 2 CM4 handheld devices
- Tier 3 Jetson Nano devices
- RFID/NFC readers
- IR camera modules
- Officer RFID badge reader

## Responsibilities
- RFID checkpoint scanning
- Officer identity acquisition
- Evidence image capture
- Inspection event creation
- Timestamp generation
- Record assembly

## Key Rule
Physical RFID/NFC scan is mandatory at every checkpoint.

---

# L2 — Verification Layer

## Components
- ArcFace
- MTCNN
- Inspection Validation Engine
- Image Quality Validator

## Responsibilities
- Passive facial recognition
- Officer verification
- Task enforcement
- Evidence validation
- Fallback authentication handling

## Security Logic
- Tier 2 uses CPU-based recognition
- Tier 3 uses GPU acceleration
- No liveness prompts required

---

# L3 — Storage Layer

## Components
- AES-256 encrypted storage
- Wi-Fi sync client
- Ethernet sync client
- Conflict resolver
- Offline record buffer

## Responsibilities
- Encrypted local persistence
- Offline operation
- 72-hour buffering
- Synchronization management
- SHA-256 integrity verification

---

# L4 — Server Layer

## Components
- FastAPI backend
- PostgreSQL database
- Celery task queue
- Alert engine
- Audit system
- Local NTP server

## Responsibilities
- Master inspection database
- Alert generation
- SLA enforcement
- Hash verification
- Centralized monitoring

## Constraints
- Entirely on-premise
- No internet dependency
- Ship LAN only

---

# L5 — Output Layer

## Components
- React.js dashboard
- Leaflet.js spatial map
- Evidence gallery
- Compliance analytics
- PDF reporting system
- WebSocket engine

## Responsibilities
- Live round tracking
- Analytics visualization
- Task management
- Evidence review
- Tamper monitoring

---

# Three-Tier Hardware Architecture

SENTINEL uses purpose-specific hardware tiers.

---

# Tier 1 — Fixed Logging Node

## Hardware
- Raspberry Pi Zero 2W

## Deployment Zones
- Low-security passageways
- Utility spaces
- Storage areas

## Functions
- RFID/NFC scanning
- AES-256 encryption
- Environment photo capture

## Important Constraint
NO facial recognition occurs on Tier 1.

Officer identity is inherited from Tier 2 round-start authentication.

## Physical Design
- IP67 enclosure
- Salt-resistant
- Vibration-damped mounting

---

# Tier 2 — Handheld Officer Device

## Hardware
- Raspberry Pi Compute Module 4

## Role
Primary handheld inspection device carried by officers.

## Functions
- Facial recognition
- RFID scanning
- Task enforcement
- Wi-Fi/Ethernet synchronization
- Evidence capture

## Features
- Passive biometric authentication
- 12-hour battery
- Battery Critical mode
- Belt holster deployment

## Physical Design
- IP67 rugged enclosure
- Drop resistant

---

# Tier 3 — High-Security Verification Node

## Hardware
- NVIDIA Jetson Nano 4GB

## Deployment Zones
- Engine Room
- Ammunition Stores
- CIC
- Critical compartments

## Functions
- GPU-accelerated facial recognition
- Instant access verification
- RFID logging
- Task enforcement

## Key Advantage
Sub-second biometric verification using NVIDIA Maxwell GPU acceleration.

---

# Multi-Factor Authentication Flow

## Step 1 — RFID Badge Scan
Officer scans RFID badge.

## Step 2 — Facial Recognition
MTCNN detects face.
ArcFace generates 512-dimensional embedding.

## Step 3 — Similarity Verification
Cosine similarity compared against threshold.

## Step 4 — Session Creation
Inspection session opens.

## Step 5 — Fallback Logic
If recognition fails:
- PIN + RFID fallback activated
- Failure logged

## Step 6 — Lockout
Three consecutive failures trigger:
- 30-minute lockout
- Supervisor escalation

---

# Facial Recognition Thresholds

## Low Security
Threshold: 0.50+

## Mid Security
Threshold: 0.55+

## High Security
Threshold: 0.60+

---

# Emergency Mode

During:
- Fire
- Flood
- Damage control
- Ship-wide alarms

The system automatically enters Emergency Mode.

## In Emergency Mode
- Biometrics disabled
- RFID/PIN-only verification
- Inspection tasks suspended
- Records flagged in audit trail

---

# Inspection Validation System

## Concept
Supervisors define mandatory inspection tasks for each compartment.

Examples:
- Inspect extinguisher pressure
- Capture bilge sensor image
- Record fuel level

## Enforcement Logic
A round cannot be marked complete unless:
- All required tasks are completed
- Required photos are captured
- Evidence passes validation

---

# Evidence Integrity Chain

All captured evidence:
- Timestamped on-device
- SHA-256 signed
- Linked to inspection record

## Security Constraint
Gallery uploads are impossible.

Only live captures are accepted.

---

# Connectivity Architecture

SENTINEL uses an offline-first architecture.

## Supported Sync Methods

### Wi-Fi Sync
- Automatic
- Background upload
- Ship access point based

### Ethernet Sync
- Docked high-speed transfer
- Simultaneous charging
- Gigabit synchronization

## Unsupported
- Bluetooth
- BLE
- Peer-to-peer syncing

---

# Air-Gapped Security Model

## Characteristics
- Entirely on-premise
- No cloud dependency
- No internet requirement
- Ship-local infrastructure only

---

# Time Synchronization

## Single Source of Truth
Local ship NTP server.

## Functions
- Synchronizes all SID devices
- Maintains audit consistency
- Tracks clock drift
- Enables forensic verification

---

# Inspection Record Structure

Each inspection generates:

- Inspection ID
- Officer ID
- Compartment ID
- Timestamp
- MFA results
- Validation task results
- Image references
- Sync status
- SHA-256 signed hash

## Security Mechanism
Any modification breaks the hash and triggers tamper alerts.

---

# Supervisor Dashboard

The dashboard is a React.js SPA hosted on-premise.

---

# Dashboard View 1 — Live Round Tracking

## Features
- Real-time officer location
- Spatial ship map
- Status tiles
- Zone filtering
- WebSocket refresh

## Status Colors
- Green → Complete
- Amber → Overdue
- Red → Missed

---

# Dashboard View 2 — Task Management

## Functions
- Configure inspection tasks
- OTA task updates
- Per-compartment rules
- Version control

---

# Dashboard View 3 — Compliance Analytics

## Metrics
- Completion percentage
- Missed rounds
- Officer statistics
- Zone heatmaps
- Trend analytics

---

# Dashboard View 4 — Evidence Gallery

## Features
- Searchable inspection photos
- Compartment filtering
- Officer filtering
- Date filtering
- Annotation support

---

# Dashboard View 5 — Tamper Alerts

## Trigger
SHA-256 mismatch detection.

## Alert Includes
- Officer ID
- Record ID
- Timestamp
- Compartment
- Tamper reason

---

# Core Technologies

## Processing Hardware
- Raspberry Pi Zero 2W
- Raspberry Pi CM4
- NVIDIA Jetson Nano

## Backend
- FastAPI
- PostgreSQL
- Celery

## Frontend
- React.js
- Leaflet.js
- WebSockets

## AI Stack
- ArcFace
- MTCNN

## Security
- AES-256
- SHA-256
- TLS 1.3

## Networking
- Wi-Fi
- Gigabit Ethernet
- Local NTP

---

# Key Innovations

## Innovation 1
Three-tier security-specific hardware architecture.

## Innovation 2
RFID-triggered automatic logging without manual input.

## Innovation 3
Supervisor-enforced evidence-backed inspections.

## Innovation 4
Passive facial recognition without liveness prompts.

## Innovation 5
Dual-path synchronization without BLE.

## Innovation 6
Air-gapped NTP time integrity.

## Innovation 7
Cryptographic chain-of-custody for every inspection event.

---

# Deliverables

## Hardware Deliverables
- Tier 1 SID nodes
- Tier 2 handheld units
- Tier 3 high-security nodes
- RFID tag kits

## Software Deliverables
- Facial recognition engine
- Dashboard
- Sync engine
- Reporting system
- Inspection validation engine
- Audit system

## Infrastructure Deliverables
- Local NTP system
- On-premise deployment stack
- API integration layer

---

# Timeline

## Milestone 1 — Prototype
Months 1–3

## Milestone 2 — Pilot
Months 4–9

## Milestone 3 — Productionisation
Months 10–18

---

# Validation Targets

## RFID Detection Accuracy
≥ 99.5%

## Facial Recognition Accuracy
≥ 92%

## Inspection Validation Completion
100%

---

# Operational Design Principles

- Offline-first
- Air-gapped
- Evidence-driven
- Cryptographically secure
- Ruggedized
- Maritime-ready
- Supervisor-controlled
- Tamper-evident

---

# Maritime Hardening

All hardware is:
- IP67 rated
- Salt resistant
- Vibration resistant
- Shock resistant

---

# Security Model Summary

SENTINEL combines:
- Physical RFID verification
- Facial biometric authentication
- Cryptographic record signing
- AES-256 encryption
- SHA-256 tamper detection
- Offline operation
- Local NTP audit integrity

to create a defence-grade naval inspection accountability platform.

---

# Final Summary

SENTINEL is a fully air-gapped, cryptographically verifiable, evidence-based naval safety inspection ecosystem designed to eliminate falsified rounds, improve operational accountability, and create tamper-proof audit trails for naval ship safety operations.