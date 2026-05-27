# Dualliste C2: Tactical Security Orchestration Core

<img style="padding-top: 0;" width="130" align="right" src="https://raw.githubusercontent.com/the-abra/Depo1/refs/heads/main/Dualliste.png"  alt="Dualliste C2 Logo" >

Dualliste C2 is a high-performance, unified command-and-control (C2) workspace built for cybersecurity profiling, vulnerability auditing, and network topology discovery. Rather than juggling dozens of terminal windows, Dualliste coordinates security scanners in the background, parses raw outputs, maps assets on an interactive graph, and generates actionable, heuristic-based recommendations.

An integrated AI Consultation panel acts as a real-time operations copilot, providing tactical advice and even executing approved command sequences (HITL YOLO mode).

---

## Architecture & Tech Stack

The platform is designed with a lightweight, decentralized layout to ensure speed and zero heavy dependencies:

*   **Backend**: Written in **Go (Golang)** using the **Gin** HTTP framework. It handles process execution, terminal pseudo-ttys (PTY), and real-time state synchronization via WebSockets.
*   **Database**: File-based session database. All discoveries, timeline events, scan history, and active sessions are serialized as structured JSON archives under `shared/sessions/` for portability and easy backups.
*   **Frontend**: Built with **Next.js** (App Router), **Zustand** for global tactical state management, and **Tailwind CSS** for a flat, cyberpunk-inspired operational HUD.
*   **Active Graph**: Rendered using **React Flow (@xyflow/react)** for dynamic asset and service topology mapping.

---

## Core Features

### 1. Multi-Tool Orchestration Engine
The backend engine loads tool definitions dynamically from YAML configurations in `shared/tools/`. It executes security binaries (like `nmap`, `gobuster`, `feroxbuster`, `httpx`, etc.) locally and streams outputs over WebSockets.
Each tool profile is mapped to raw stdout regex Parsers. The parsed tokens are automatically registered as **Discoveries** (e.g., ports, services, subdomains, vulnerabilities).

### 2. Heuristic Rule System
When new discoveries are logged, the backend evaluates them against the rules defined in `shared/heuristics/core_rules.yaml`. For instance:
*   A newly identified domain triggers a recommendation to map web services via `httpx`.
*   An open port `80` or `443` suggests technology fingerprinting via `whatweb` and directory fuzzing via `feroxbuster`.
These recommendations appear directly in the CTF Playbook / Operation Centre views.

### 3. AI Consultation & HITL YOLO Mode
The left panel houses a live assistant loaded with custom system prompts and API presets (defined in `shared/config/ai_presets.json`). 
*   **Interactive Consultation**: Discuss current scan findings and seek verification commands.
*   **HITL (Human-in-the-loop) Approvals**: When the AI proposes an exploit or verification command, the frontend prompts the user with a confirmation modal. The command is only executed on the host OS once approved.

### 4. Interactive Topology Graph
Discovered nodes are rendered in real-time. Target IPs, hostnames, open ports, and technologies are visually linked to build a clean representation of the network attack surface.
Operators can also manually add notes, credentials, and points of interest to the map, syncing them across the C2 team.

### 5. System Diagnostics & Telemetry
The System landing page and HUD display live CPU, RAM, and Disk utilization of the host machine, along with active scan jobs currently running. A dedicated Platform Health widget performs active environment validation checks (filesystem health, database connectivity, and tool installation audits) to provide live status telemetry.
