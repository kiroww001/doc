# 📘 Nucleus Framework - Documentation Développeur

> **Version**: 1.0.0  
> **Auteur**: NucleusStds  
> **Dernière mise à jour**: Mars 2026

---

## 📋 Table des matières

1. [Introduction](#introduction)
2. [Architecture du projet](#architecture-du-projet)
3. [Démarrage rapide](#démarrage-rapide)
4. [Système de modules](./modules/README.md)
5. [Classes principales](./classes/README.md)
6. [Événements](./events/README.md)
7. [Fonctions utilitaires](./utils/README.md)
8. [Interface utilisateur (NUI/Web)](./ui/README.md)
9. [Base de données (MongoDB)](./database/README.md)
10. [Bonnes pratiques](./best-practices/README.md)

---

## Introduction

**Nucleus** est un framework FiveM complet écrit en Lua, conçu pour le développement de serveurs RP (RolePlay). Il utilise une **architecture modulaire** permettant d'activer/désactiver des fonctionnalités facilement et de développer de nouvelles features de manière isolée.

### Caractéristiques principales

- 🧩 **Architecture modulaire** - Chaque fonctionnalité est un module indépendant
- 🔄 **Système d'événements** - Communication fluide entre les modules
- 📊 **MongoDB** - Base de données NoSQL pour la persistance des données
- 🎨 **Interface NUI** - Interface web moderne intégrée
- 🔐 **Gestion des permissions** - Système de rangs et permissions intégré
- 📦 **Auto-génération du manifest** - Le `fxmanifest.lua` est généré automatiquement

---

## Architecture du projet

```
src/
├── configs/                  # Fichiers de configuration serveur
│   ├── config.cfg           # Configuration générale
│   ├── credentials.cfg      # Identifiants (DB, etc.)
│   ├── permissions.cfg      # Permissions ACE
│   ├── resources.cfg        # Ressources à charger
│   └── server.cfg           # Point d'entrée serveur
│
├── resources/
│   ├── [defaults]/          # Dépendances par défaut
│   ├── [utils]/             # Utilitaires externes
│   ├── init/                # Script d'initialisation (génère le manifest)
│   └── nucleus/             # ⭐ CORE DU FRAMEWORK
│       ├── client/          # Code côté client
│       │   ├── classes/     # Classes client (Player, Character, etc.)
│       │   ├── events/      # Gestionnaires d'événements client
│       │   ├── functions/   # Fonctions utilitaires client
│       │   └── libs/        # Bibliothèques (RageUI, etc.)
│       │
│       ├── server/          # Code côté serveur
│       │   ├── classes/     # Classes serveur (XPlayer, XCharacter, etc.)
│       │   ├── commands/    # Commandes admin
│       │   ├── events/      # Gestionnaires d'événements serveur
│       │   ├── functions/   # Fonctions utilitaires serveur
│       │   └── init/        # Initialisation des données
│       │
│       ├── shared/          # Code partagé client/serveur
│       │   ├── cache.lua    # Système de cache KVP
│       │   ├── eventbase.lua# Classe de base pour les événements
│       │   ├── functions.lua# Fonctions utilitaires partagées
│       │   ├── logger.lua   # Système de logging
│       │   ├── main.lua     # Chargement des modules
│       │   ├── modules.lua  # ⭐ SYSTÈME DE MODULES
│       │   ├── shared.lua   # Variables globales partagées
│       │   └── vars.lua     # Variables de configuration
│       │
│       ├── modules/         # ⭐ MODULES DU FRAMEWORK
│       │   ├── [interfaces]/ # Modules d'interface (inventory, phone, etc.)
│       │   ├── [life]/       # Modules gameplay (banking, status, etc.)
│       │   └── [system]/     # Modules système (actions, markers, etc.)
│       │
│       ├── utils/           # Utilitaires du core
│       ├── web/             # Interface NUI (HTML/CSS/JS)
│       └── fxmanifest.src.lua # Template du manifest (généré automatiquement)
```

---

## Démarrage rapide

### Prérequis

- FXServer (FiveM)
- MongoDB
- Node.js (pour yarn)

### Installation

1. Clonez le repository dans votre dossier resources
2. Configurez `configs/credentials.cfg` avec vos identifiants MongoDB
3. Lancez le serveur

### Variables d'environnement importantes

Dans `shared/vars.lua`, vous trouverez les variables globales :

```lua
GM = GM or {}                                    -- Namespace global
GM_ID = GetConvar('sv_id', 'nucleus')            -- ID du serveur
GM_SERVERNAME = GetConvar('sv_name', 'Nucleus')  -- Nom du serveur
IS_DEV = GetConvar('sv_mode', 'dev') == 'dev'    -- Mode développement
IS_SERVER = IsDuplicityVersion()                 -- Côté serveur ?
CURRENT_RESOURCE = GetCurrentResourceName()      -- Nom de la ressource
```

---

## Navigation rapide

| Documentation | Description |
|--------------|-------------|
| [📦 Modules](./modules/README.md) | Comment créer et utiliser les modules |
| [🏗️ Classes](./classes/README.md) | Documentation des classes principales |
| [📡 Événements](./events/README.md) | Système d'événements et communication |
| [🛠️ Utilitaires](./utils/README.md) | Fonctions helper disponibles |
| [🎨 Interface NUI](./ui/README.md) | Développement de l'interface web |
| [💾 Base de données](./database/README.md) | Utilisation de MongoDB |
| [✅ Bonnes pratiques](./best-practices/README.md) | Conventions et recommandations |
| [📝 Types et Annotations](./types/README.md) | Autocomplétion et typage |
