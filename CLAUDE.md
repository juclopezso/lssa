# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

University lab assignment for a Software Architecture course (LSSA 2026i). The project analyzes the **ERTMS (European Rail Traffic Management System)** from a structural architecture perspective — classifying components, connectors, and tiers.

## Repository Structure

- `actividad-4.1-punto1.md` — Main working document with architectural analysis (component classification by tier, connectors, cross-tier analysis, missing components)
- `entregable.tex` — LaTeX deliverable template (Spanish, compiled with standard LaTeX toolchain)
- `ertms-intro-transcript.txt` — Reference transcript explaining ERTMS/ETCS levels
- PDF files — Lab instructions and ERTMS course material

## Build

Compile the LaTeX deliverable:
```
pdflatex entregable.tex
```

## Domain Context

The system under analysis is a 5-tier ERTMS architecture:
- **T1 Presentation**: Portal, Dashboard, Driver Interface (DMI)
- **T2 Communication**: API Gateway
- **T3 Logic**: Microservices (passengers, routes, trains, position-time, tickets, MAS)
- **T4 Data**: Per-service databases + Data Lake
- **T5 Physical**: Onboard Unit (OBU/EVC), Train Sensors, Brake Actuators, Balises

ETCS levels (0, NTC, 1, 2, 3) differ in how track-to-train communication works — from passive balises (L1) to continuous radio via GSM-R/RBC (L2) to satellite-based moving block (L3).

## Language

All content is written in **Spanish**.
