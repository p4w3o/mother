# Mother Project

![Architecture Diagram](4e1ef18e-9039-4dc5-8701-06955ff218ce.png)

---

## English Version

**Mother Project** is a macOS suite enabling the execution and management of local scripts in Roblox through a client-server ecosystem. It combines a user interface, isolated agents, and Lua scripts to interact with game instances.

The project consists of four main components: `motherLIB`, `motherCLI`, `motherSS`, and `motherSERVER`.

---

## Architecture and Object Flow

Mother Project uses a **centralized client-server architecture** with local agents capable of executing scripts in isolation. Each component has a specific role and communicates with others to orchestrate commands.

### 1. macOS Client

#### **MotherCLI**
- Swift-based user interface.
- Enables:
  - Sending commands to **motherSERVER**.
  - Managing agents (create, monitor, delete).
  - Injecting **motherLIB** (dylib) into **RobloxPlayer** via `taskForPID`.
- Does not require Roblox to run, but can interact with Roblox via injection.

#### **MotherLIB** (macOS dylib)
- Optional but essential for:
  - **Bytecode manipulation** in Roblox Player.
  - Local script execution.
  - Server ID verification and additional security checks.
- Injected by **motherCLI**, enabling advanced local script execution.

**Client-Local Flow:**
1. User requests **motherCLI** to inject **motherLIB** into RobloxPlayer.
2. **MotherLIB** loads into the process and prepares the player for scripts.
3. Scripts execute locally and report back to the server via agents.

---

### 2. Central Server: **MotherSERVER** (Python)

**MotherSERVER** orchestrates all agents from creation to deletion and handles script execution.

#### Main Functions
- Full agent management:
  - **Creation**: Launches isolated environments with Wine, installs Roblox, and configures the agent.
  - **Monitoring**: Ensures agents are active and ready for commands.
  - **Deletion**: Cleans environments to avoid accumulation.
- Distributes scripts from **motherCLI**.
- Tracks agent status (connected, running, failed).

#### Server-Agent Flow
1. **MotherSERVER** receives a command from **motherCLI**.
2. Selects available agents and sends the command.
3. Each agent executes locally via **motherLIB** (if injection needed) or via **motherSS** in-game.
4. Agent returns execution status and logs to the server.
5. Server updates agent state (active, idle, finished).

#### **Agents**
- Operate in isolated environments (Wine + Roblox).
- Execute scripts without affecting the host system.
- Perform an initial `SS check` for script validity and server connectivity.
- Can execute scripts:
  - **Locally**: via **motherLIB** with bytecode manipulation.
  - **In-game**: via **motherSS** long-polling.

---

### 3. MotherSS (Lua Script)
- Deployed inside a Roblox game.
- Provides **long-polling** to fetch and execute server-sent scripts.
- Contact point for Roblox agents.

---

## Global Flow

1. **User command** via **motherCLI**.
2. **Server distribution**: **motherSERVER** selects agents and sends scripts.
3. **Agent execution**:
   - Locally via **motherLIB** (bytecode manipulation).
   - In-game via **motherSS**.
4. **Status reporting**: Agent sends logs back to server.
5. **Server tracking**: Updates agent state, execution logs.

**Local vs Server distinction:**
- **Local**: Actions in Roblox via **motherLIB**, direct execution, bytecode verification.
- **Server**: Orchestration, agent lifecycle, command distribution, and status tracking.

---

## Example Usage

1. **Local Lua Script Execution**
```bash
$ motherCLI inject --target RobloxPlayerPID
$ motherCLI run --script myScript.lua --agent 1
```

3. **Script Distribution via Server**
```bash
$ motherCLI send --script globalScript.lua --agents all
```

4. **Create/Delete Agents**
```bash
$ motherCLI agent create --count 3
$ motherCLI agent delete --id 2
```

---

## Technologies

- **Swift (macOS)**: `motherCLI`
- **Python**: `motherSERVER`
- **Lua**: `motherSS`
- **C/C++ (macOS dylib)**: `motherLIB`
- **Wine**: Agent isolation

---

## Notes

- `motherLIB` is optional but required for advanced local execution.
- Agents are disposable and isolated.
- Client-server architecture centralizes script distribution.
- Injection and script execution can be abused; this project documents technical functionality only.

---

## Legal Warning

Manipulating scripts in Roblox is subject to Roblox's terms of service and local laws. Use responsibly.

---

# Version Française

![Diagramme Architecture](4e1ef18e-9039-4dc5-8701-06955ff218ce.png)

---

## Mother Project

**Mother Project** est une suite macOS permettant l’exécution et la gestion de scripts locaux dans Roblox via un écosystème client-serveur. Le projet combine une interface utilisateur, des agents isolés et des scripts Lua pour interagir avec des instances de jeu.

Le projet se compose de quatre éléments principaux : `motherLIB`, `motherCLI`, `motherSS` et `motherSERVER`.

---

## Architecture et flux des objets

Mother Project repose sur une **architecture client-serveur centralisée**, avec des agents locaux capables d’exécuter des scripts de manière isolée. Chaque composant a un rôle précis et communique avec les autres pour orchestrer les commandes.

### 1. Client macOS

#### **MotherCLI**
- Interface utilisateur écrite en Swift.
- Permet :
  - L’envoi de commandes à **motherSERVER**.
  - La gestion des agents (création, surveillance, suppression).
  - L’injection de **motherLIB** (dylib) dans **RobloxPlayer** via `taskForPID`.
- Ne nécessite pas Roblox pour fonctionner, mais peut interagir avec Roblox via injection.

#### **MotherLIB** (dylib macOS)
- Optionnelle mais essentielle pour :
  - La **manipulation du bytecode** dans Roblox Player.
  - L’exécution locale de scripts.
  - La vérification de l’ID du serveur Roblox et autres contrôles de sécurité.
- Injectée par **motherCLI**, permettant une exécution locale avancée.

**Flux client-local :**
1. L’utilisateur demande via **motherCLI** l’injection de **motherLIB** dans RobloxPlayer.
2. **MotherLIB** se charge dans le processus et prépare le joueur pour recevoir des scripts.
3. Les scripts sont exécutés localement et peuvent signaler le résultat au serveur via les agents.

---

### 2. Serveur central : **MotherSERVER** (Python)

**MotherSERVER** orchestre tous les agents, de la création à la suppression, et gère l’exécution des scripts.

#### Fonctions principales
- Gestion complète des agents :
  - **Création** : lance des environnements isolés avec Wine, installe Roblox et configure l’agent.
  - **Supervision** : vérifie que chaque agent est actif et prêt à recevoir des scripts.
  - **Suppression** : nettoie les environnements pour éviter toute accumulation.
- Distribution des scripts envoyés par **motherCLI**.
- Suivi de l’état de chaque agent (connecté, en exécution, échec).

#### Flux serveur-agent
1. **MotherSERVER** reçoit une commande de **motherCLI**.
2. Sélectionne les agents disponibles et envoie la commande.
3. Chaque agent exécute localement via **motherLIB** ou dans le jeu via **motherSS**.
4. L’agent renvoie les logs et l’état d’exécution au serveur.
5. Le serveur met à jour l’état de l’agent (actif, en attente, terminé).

#### **Agents**
- Fonctionnent dans un environnement isolé (Wine + Roblox).
- Exécutent les scripts sans impacter le système hôte.
- Effectuent une vérification initiale (`SS check`) pour valider le script et la connexion serveur.
- Peuvent exécuter des scripts :
  - **Localement** : via **motherLIB** avec manipulation du bytecode.
  - **Dans le jeu** : via **motherSS** long-polling.

---

### 3. MotherSS (Script Lua)
- Déployé dans un jeu Roblox.
- Permet le **long-polling** pour récupérer et exécuter les scripts envoyés par le serveur.
- Point de contact pour les agents Roblox.

---

## Flux global

1. **Commande utilisateur** via **motherCLI**.
2. **Distribution serveur** : **motherSERVER** sélectionne les agents et envoie les scripts.
3. **Exécution agent** :
   - Localement via **motherLIB** (manipulation du bytecode).
   - Dans le jeu via **motherSS**.
4. **Retour d’état** : l’agent renvoie les logs au serveur.
5. **Suivi serveur** : mise à jour de l’état et des logs d’exécution.

**Distinction Local / Serveur :**
- **Local** : actions dans Roblox via **motherLIB**, exécution directe, vérification bytecode.
- **Serveur** : orchestration, cycle de vie des agents, distribution des commandes, suivi d’état.

---

## Exemples d’usage

1. **Exécution locale d’un script Lua**
```bash
$ motherCLI inject --target RobloxPlayerPID
$ motherCLI run --script myScript.lua --agent 1
```

2. **Distribution de script via serveur**
```bash
$ motherCLI send --script globalScript.lua --agents all
```

3. **Création / suppression d’agents**
```bash
$ motherCLI agent create --count 3
$ motherCLI agent delete --id 2
```
---

## Technologies utilisées

- **Swift (macOS)** : `motherCLI`
- **Python** : `motherSERVER`
- **Lua** : `motherSS`
- **C/C++ (dylib macOS)** : `motherLIB`
- **Wine** : isolation des agents

---

## Remarques

- `motherLIB` est optionnelle mais indispensable pour l’exécution locale avancée.
- Les agents sont jetables et isolés pour limiter les risques.
- L’architecture client-serveur centralise la distribution des scripts.
- L’injection et l’exécution de scripts peuvent être abusives, ce projet documente seulement le fonctionnement technique.

---

## Avertissement légal

La manipulation de scripts dans Roblox est soumise aux conditions d’utilisation de Roblox et aux lois locales. À utiliser de manière responsable.


