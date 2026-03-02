# 📦 Système de Modules - Documentation

> Le système de modules est le cœur de Nucleus. Chaque fonctionnalité est encapsulée dans un module indépendant.

---

## 📋 Table des matières

1. [Concept des modules](#concept-des-modules)
2. [Structure d'un module](#structure-dun-module)
3. [Créer un nouveau module](#créer-un-nouveau-module)
4. [API des modules](#api-des-modules)
5. [Communication entre modules](#communication-entre-modules)
6. [Cycle de vie d'un module](#cycle-de-vie-dun-module)
7. [Exemples pratiques](#exemples-pratiques)

---

## Concept des modules

Un **module** dans Nucleus est une unité fonctionnelle autonome qui :
- Possède son propre état et ses propres variables
- Peut émettre et écouter des événements
- Peut communiquer avec d'autres modules
- Est automatiquement enregistré au démarrage

### Catégories de modules

Les modules sont organisés en trois catégories :

| Catégorie | Chemin | Description |
|-----------|--------|-------------|
| `[interfaces]` | `modules/[interfaces]/` | Interfaces utilisateur (inventaire, téléphone, HUD, etc.) |
| `[life]` | `modules/[life]/` | Gameplay et vie du joueur (banking, status, météo, etc.) |
| `[system]` | `modules/[system]/` | Systèmes internes (actions, markers, errors, etc.) |

---

## Structure d'un module

Chaque module suit cette structure :

```
modules/[category]/monModule/
├── manifest.lua          # ⭐ OBLIGATOIRE - Configuration du module
├── shared/               # Code partagé client/serveur
│   └── config.lua        # Configuration et variables partagées
├── client/               # Code côté client uniquement
│   └── main.lua          # Logique client
└── server/               # Code côté serveur uniquement
    └── main.lua          # Logique serveur
```

### Le fichier `manifest.lua`

C'est le fichier de configuration du module, similaire à un `fxmanifest.lua` mais simplifié :

```lua
-- manifest.lua
game('gta5')              -- Jeu cible ('gta5', 'rdr3', 'common')
author('VotrePseudo')     -- Auteur du module

-- Scripts partagés (exécutés côté client ET serveur)
shared_scripts({
    "shared/*.lua"
})

-- Scripts client uniquement
client_scripts({
    "client/*.lua"
})

-- Scripts serveur uniquement
server_scripts({
    "server/*.lua"
})

-- Fichiers accessibles (pour NUI, etc.)
files({
    "config/config.json"
})
```

### Options avancées du manifest

```lua
-- Cibler plusieurs jeux
games({ 'gta5', 'rdr3' })

-- Cibler un projet spécifique
project('rp:life')        -- Seulement pour le projet RP Life
projects({ 'rp:life', 'rp:serious' })

-- Cibler une map spécifique  
map('gta5')
maps({ 'gta5', 'custom_map' })

-- Dépendances (autres modules requis)
dependency('inventory')
dependencies({ 'inventory', 'banking' })

-- Module serveur uniquement
server_only('yes')

-- Description et version
description('Mon super module')
version('1.0.0')
```

---

## Créer un nouveau module

### Étape 1 : Créer la structure des dossiers

```
modules/[interfaces]/monNouveauModule/
├── manifest.lua
├── shared/
│   └── config.lua
├── client/
│   └── main.lua
└── server/
    └── main.lua
```

### Étape 2 : Configurer le manifest.lua

```lua
-- modules/[interfaces]/monNouveauModule/manifest.lua

game('gta5')
author('MonPseudo')

shared_scripts({
    "shared/*.lua"
})

client_scripts({
    "client/*.lua"
})

server_scripts({
    "server/*.lua"
})
```

### Étape 3 : Créer le fichier de configuration partagé

```lua
-- modules/[interfaces]/monNouveauModule/shared/config.lua

---@class CFGMonModule
local CFG = {}

---@module SHMonModule : Module
local m = iAm("monNouveauModule")  -- ⚠️ Le nom doit correspondre au dossier

-- Configuration du module
CFG.MaVariable = "valeur"
CFG.MonNombre = 42
CFG.MonTableau = { "a", "b", "c" }

-- Enregistrer la config dans le module
m:CFG(CFG)
```

### Étape 4 : Code côté client

```lua
-- modules/[interfaces]/monNouveauModule/client/main.lua

---@module CMonModule : SHMonModule
local m = iAm("monNouveauModule")
---@type CFGMonModule
local CFG = m:CFG()

-- Événement au démarrage du module
m:OnStart(function()
    print("Module monNouveauModule démarré côté client!")
end)

-- Exemple de fonction
function m.MaFonction()
    print("Ma variable:", CFG.MaVariable)
end

-- Écouter un événement réseau
m:OnNet("monEvent", function(data)
    print("Reçu:", json.encode(data))
end)

-- Thread (boucle)
m:Thread(function()
    while true do
        Wait(1000)
        -- Code exécuté chaque seconde
    end
end)
```

### Étape 5 : Code côté serveur

```lua
-- modules/[interfaces]/monNouveauModule/server/main.lua

---@module SMonModule : SHMonModule
local m = iAm("monNouveauModule")
---@type CFGMonModule
local CFG = m:CFG()

-- Événement au démarrage du module
m:OnStart(function()
    print("Module monNouveauModule démarré côté serveur!")
end)

-- Callback serveur (le client peut l'appeler et recevoir une réponse)
m:RegisterServerCallback("getData", function(source, cb, param1)
    local player = GM.GetPlayerFromSource(source)
    if not player then
        return cb(false, "Joueur non trouvé")
    end
    
    -- Traitement...
    cb(true, { data = "valeur" })
end)

-- Écouter un événement réseau
m:OnNet("saveData", function(source, data)
    local player = GM.GetPlayerFromSource(source)
    -- Sauvegarder les données...
end)
```

### Étape 6 : Redémarrer le serveur

Le système `init` détectera automatiquement le nouveau module et mettra à jour le `fxmanifest.lua`.

---

## API des modules

### Récupérer un module

```lua
-- Méthode recommandée
local m = iAm("nomDuModule")

-- Alternative
local m = Modules.Get("nomDuModule")

-- Avec assertion (erreur si non trouvé)
local m = Modules.Get("nomDuModule", true)
```

### Propriétés du module

| Propriété | Type | Description |
|-----------|------|-------------|
| `m.name` | string | Nom du module |
| `m.path` | string | Chemin du module |
| `m.author` | string | Auteur |
| `m.version` | string | Version |
| `m.description` | string | Description |
| `m.state` | string | État (`uninitialized`, `starting`, `started`, `stopping`, `stopped`) |

### Méthodes du module

#### Configuration

```lua
-- Définir la configuration
m:CFG({ key = "value" })

-- Récupérer la configuration
local cfg = m:CFG()
```

#### Variables personnalisées

```lua
-- Définir une variable
m.maVariable = "valeur"      -- Via métatable
m:Set("maVariable", "valeur") -- Méthode explicite

-- Récupérer une variable
local val = m.maVariable      -- Via métatable
local val = m:Get("maVariable") -- Méthode explicite
```

#### Cycle de vie

```lua
-- Démarrer le module
m:Start()

-- Arrêter le module
m:Stop()

-- Écouter les événements du cycle de vie
m:OnStart(function()
    print("Module en cours de démarrage")
end)

m:OnStarted(function()
    print("Module démarré")
end)

m:OnStop(function()
    print("Module en cours d'arrêt")
end)

m:OnStopped(function()
    print("Module arrêté")
end)
```

#### Threads

```lua
-- Créer un thread (asynchrone)
m:Thread(function()
    while true do
        Wait(1000)
        -- ...
    end
end)

-- Créer un thread immédiat (synchrone)
m:ThreadNow(function()
    -- Exécuté immédiatement
end)
```

#### Fichiers

```lua
-- Charger un fichier JSON
local data = m:LoadFile("config/data.json")

-- Sauvegarder un fichier JSON
m:SaveFile("config/data.json", { key = "value" })

-- Obtenir le chemin du module
local path = m:GetPath() -- "modules/[interfaces]/monModule"

-- Obtenir l'URL NUI
local nuiPath = m:GetCfxNuiPath() -- "https://cfx-nui-nucleus/modules/[interfaces]/monModule"
```

---

## Communication entre modules

### Événements locaux (même côté)

```lua
local m = iAm("monModule")

-- Émettre un événement
m:Emit("monEvent", data1, data2)

-- Émettre de manière synchrone
m:EmitSync("monEvent", data1, data2)

-- Écouter un événement
m:On("monEvent", function(data1, data2)
    print("Reçu:", data1, data2)
end)

-- Écouter une seule fois
m:Once("monEvent", function(data)
    print("Reçu une fois:", data)
end)

-- Supprimer un listener
m:RemoveListener("monEvent", maFonction)

-- Supprimer tous les listeners d'un événement
m:RemoveListeners("monEvent")
```

### Événements réseau

#### Côté Client

```lua
local m = iAm("monModule")

-- Écouter un événement du serveur
m:OnNet("serverEvent", function(data)
    print("Message du serveur:", data)
end)

-- Envoyer un événement au serveur
m:EmitServer("clientEvent", data)

-- Envoyer avec latence (gros volumes)
m:EmitLatentServer("clientEvent", 128000, data) -- 128KB/s

-- Appeler un callback serveur
m:TriggerServerCallback("getData", function(success, data)
    if success then
        print("Données:", json.encode(data))
    end
end, param1, param2)

-- OU nouvelle syntaxe
m:EmitServerCb("getData", function(success, data)
    -- ...
end, param1, param2)
```

#### Côté Serveur

```lua
local m = iAm("monModule")

-- Écouter un événement du client
m:OnNet("clientEvent", function(source, data)
    local player = GM.GetPlayerFromSource(source)
    print("Message du joueur", player:GetName(), ":", data)
end)

-- Envoyer à un client
m:EmitClient("serverEvent", playerId, data)

-- Envoyer à plusieurs clients
m:EmitClients("serverEvent", { playerId1, playerId2 }, data)

-- Envoyer avec latence
m:EmitLatentClient("serverEvent", playerId, 128000, data)

-- Enregistrer un callback
m:RegisterServerCallback("getData", function(source, cb, param1, param2)
    -- Traitement...
    cb(true, { result = "data" })
end)

-- Callback avec vérification de permission admin
m:OnNetAdmin("adminAction", "admin.manage", function(source, data)
    -- Seulement si le joueur a la permission "admin.manage"
end)
```

### Accéder à un autre module

```lua
local m = iAm("monModule")
local inventory = Modules.Get("inventory")

-- Appeler une fonction d'un autre module
if inventory then
    inventory.OpenInventory(targetInventory)
end

-- Écouter un événement d'un autre module
inventory:On("itemAdded", function(item)
    print("Item ajouté:", item.name)
end)
```

---

## Cycle de vie d'un module

```
┌─────────────────────────────────────────────────────────────┐
│                     CHARGEMENT DU SERVEUR                   │
├─────────────────────────────────────────────────────────────┤
│  1. init/init.lua lit tous les manifest.lua                │
│  2. Génère le fxmanifest.lua avec tous les modules         │
│  3. Les scripts shared/*.lua sont chargés                  │
│  4. shared/main.lua enregistre les modules via Modules()   │
├─────────────────────────────────────────────────────────────┤
│                     DÉMARRAGE DES MODULES                   │
├─────────────────────────────────────────────────────────────┤
│  5. client/client.lua ou server/server.lua appelle         │
│     Modules.GetList() puis module:Start() pour chacun      │
│  6. L'événement 'start' est émis                           │
│  7. L'état passe à 'started'                               │
│  8. L'événement 'started' est émis                         │
└─────────────────────────────────────────────────────────────┘
```

### États possibles

| État | Description |
|------|-------------|
| `uninitialized` | Module enregistré mais pas démarré |
| `starting` | Module en cours de démarrage |
| `started` | Module actif et fonctionnel |
| `stopping` | Module en cours d'arrêt |
| `stopped` | Module arrêté |

---

## Exemples pratiques

### Exemple 1 : Module simple de notifications

```lua
-- modules/[interfaces]/notifs/shared/config.lua
local m = iAm("notifs")
local CFG = {}
CFG.DefaultDuration = 5000
m:CFG(CFG)
```

```lua
-- modules/[interfaces]/notifs/client/main.lua
local m = iAm("notifs")
local CFG = m:CFG()

function m.Show(message, type, duration)
    UI.Send("Notify:Create", {
        msg = message,
        color = type or "bleu",
        timer = duration or CFG.DefaultDuration
    })
end

m:OnNet("showNotif", function(message, type, duration)
    m.Show(message, type, duration)
end)
```

```lua
-- modules/[interfaces]/notifs/server/main.lua
local m = iAm("notifs")

function m.SendToPlayer(playerId, message, type, duration)
    m:EmitClient("showNotif", playerId, message, type, duration)
end

function m.SendToAll(message, type, duration)
    for _, player in pairs(XPlayers) do
        if player.isOnline then
            m:EmitClient("showNotif", player.source, message, type, duration)
        end
    end
end
```

### Exemple 2 : Module avec base de données

```lua
-- modules/[life]/rewards/server/main.lua
local m = iAm("rewards")

m:OnStart(function()
    -- Charger les récompenses depuis MongoDB
    mdb:find("rewards", {}, function(rewards)
        m.rewards = rewards
        Logger.Info("Rewards loaded: " .. #rewards)
    end)
end)

m:RegisterServerCallback("claimReward", function(source, cb, rewardId)
    local player = GM.GetPlayerFromSource(source)
    if not player then return cb(false, "Joueur non trouvé") end
    
    local char = GM.GetCharacterFromId(player:GetCurrentCharacterId())
    if not char then return cb(false, "Personnage non trouvé") end
    
    local reward = m.rewards[rewardId]
    if not reward then return cb(false, "Récompense non trouvée") end
    
    -- Donner la récompense
    char:AddItem(reward.itemName, reward.amount)
    
    -- Sauvegarder que le joueur a claim
    mdb:updateOne("player_rewards", 
        { playerId = player:GetId(), rewardId = rewardId },
        { ["$set"] = { claimed = true, claimedAt = os.time() } },
        { upsert = true }
    )
    
    cb(true, "Récompense réclamée!")
end)
```

---

## Voir aussi

- [Classes principales](../classes/README.md)
- [Système d'événements](../events/README.md)
- [Base de données MongoDB](../database/README.md)

