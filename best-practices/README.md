# ✅ Bonnes Pratiques - Documentation

> Conventions, recommandations et standards de développement pour Nucleus.

---

## 📋 Table des matières

1. [Structure du code](#structure-du-code)
2. [Conventions de nommage](#conventions-de-nommage)
3. [Gestion des modules](#gestion-des-modules)
4. [Sécurité](#sécurité)
5. [Performances](#performances)
6. [Debugging](#debugging)
7. [Git et versioning](#git-et-versioning)

---

## Structure du code

### Organisation des fichiers

```lua
-- ✅ Bon - Un fichier par fonctionnalité
modules/[interfaces]/monModule/
├── client/
│   ├── main.lua        -- Point d'entrée, initialisation
│   ├── ui.lua          -- Gestion de l'interface
│   └── events.lua      -- Gestionnaires d'événements
├── server/
│   ├── main.lua        -- Point d'entrée serveur
│   ├── database.lua    -- Opérations BDD
│   └── callbacks.lua   -- Callbacks serveur
└── shared/
    └── config.lua      -- Configuration partagée
```

### Structure d'un fichier Lua

```lua
-- 1. Annotations de type (documentation)
---@module CMonModule : SHMonModule

-- 2. Récupération du module et dépendances
local m = iAm("monModule")
---@type CFGMonModule
local CFG = m:CFG()

-- 3. Autres modules nécessaires
---@type CHud
local hud = Modules.Get("hud")

-- 4. Variables locales du module
local isActive = false
local cache = {}

-- 5. Fonctions privées (locales)
local function validateInput(data)
    return data and type(data) == "table"
end

-- 6. Fonctions publiques du module
function m.DoSomething(param)
    if not validateInput(param) then
        return false
    end
    -- ...
    return true
end

-- 7. Événements du cycle de vie
m:OnStart(function()
    Logger.Info("Module démarré")
end)

-- 8. Événements réseau
m:OnNet("eventName", function(data)
    -- ...
end)

-- 9. Threads (boucles)
m:Thread(function()
    while true do
        Wait(1000)
        -- ...
    end
end)
```

---

## Conventions de nommage

### Variables et fonctions

```lua
-- Variables locales : camelCase
local playerData = {}
local isInventoryOpen = false
local maxItemCount = 100

-- Constantes : SCREAMING_SNAKE_CASE
local MAX_PLAYERS = 32
local DEFAULT_TIMEOUT = 5000

-- Fonctions : camelCase ou PascalCase pour les publiques
local function calculateDistance(pos1, pos2) end
function m.OpenInventory() end
function m.GetPlayerData() end

-- ❌ Éviter
local player_data = {}
local ISOPEN = false
```

### Modules

```lua
-- Nom du module : camelCase, descriptif
"inventory"      -- ✅
"playerStatus"   -- ✅
"adminMenu"      -- ✅
"inv"            -- ❌ Trop court
"PlayerInv"      -- ❌ Pas cohérent
```

### Événements

```lua
-- Format : moduleName:actionName
m:Emit("inventoryUpdated", data)
m:OnNet("itemTransfer", callback)

-- Pour les événements globaux
"nucleus:playerLoaded"
"nucleus:characterSelected"

-- ❌ Éviter
"inv_update"
"ITEM_TRANSFER"
```

### Callbacks serveur

```lua
-- Format : verbe + objet
m:RegisterServerCallback("getData", ...)
m:RegisterServerCallback("buyItem", ...)
m:RegisterServerCallback("transferMoney", ...)

-- ❌ Éviter
m:RegisterServerCallback("data", ...)
m:RegisterServerCallback("item", ...)
```

---

## Gestion des modules

### Dépendances entre modules

```lua
-- ✅ Bon - Vérifier si le module existe
local inventory = Modules.Get("inventory")
if inventory then
    inventory.AddItem(...)
end

-- ✅ Bon - Dépendance obligatoire (erreur si absent)
local inventory = Modules.Get("inventory", true)

-- ❌ Mauvais - Accès direct sans vérification
Modules.Get("inventory").AddItem(...)  -- Peut crash si nil
```

### Configuration des modules

```lua
-- ✅ Bon - Config centralisée dans shared/
-- shared/config.lua
local CFG = {}
CFG.MaxItems = 50
CFG.CooldownMs = 5000
CFG.EnableFeatureX = true
m:CFG(CFG)

-- Utilisation
local CFG = m:CFG()
if CFG.EnableFeatureX then
    -- ...
end

-- ❌ Mauvais - Valeurs hardcodées dans le code
if maxItems > 50 then  -- D'où vient 50 ?
```

### Communication inter-modules

```lua
-- ✅ Bon - Utiliser les événements
local inventory = Modules.Get("inventory")
inventory:On("itemAdded", function(item)
    m.UpdateUI()
end)

-- ✅ Bon - API publique claire
local success = inventory.AddItem("bread", 5)

-- ❌ Mauvais - Accès direct aux variables internes
inventory.someInternalVar = 123
```

---

## Sécurité

### Validation côté serveur (CRUCIAL)

```lua
-- ❌ DANGEREUX - Faire confiance au client
m:OnNet("buyItem", function(source, itemId, price)
    -- Le client pourrait envoyer un prix de 0!
    removePlayerMoney(source, price)
    giveItem(source, itemId)
end)

-- ✅ SÉCURISÉ - Toujours valider côté serveur
m:OnNet("buyItem", function(source, itemId)
    local player = GM.GetPlayerFromSource(source)
    if not player then return end
    
    -- Valider l'item
    local item = GM.GetItemFromId(itemId)
    if not item then return end
    
    -- Prix déterminé côté serveur
    local price = item:GetPrice()
    
    -- Vérifier les fonds
    local char = GM.GetCharacterFromId(player:GetCurrentCharacterId())
    if not char then return end
    
    local account = char:GetAccount()
    if account.money < price then
        player:ShowErrorNotify("Fonds insuffisants")
        return
    end
    
    -- Transaction sécurisée
    account:RemoveMoney(price)
    char:AddItem(itemId, 1)
    player:ShowSuccessNotify("Achat effectué!")
end)
```

### Ne jamais exposer de données sensibles

```lua
-- ❌ DANGEREUX - Envoyer toutes les données
m:EmitClient("playerData", source, player)

-- ✅ SÉCURISÉ - Filtrer les données
m:EmitClient("playerData", source, {
    name = player:GetName(),
    vip = player:GetVIP()
    -- Pas de license, pas de données admin, etc.
})
```

### Vérifier les permissions

```lua
-- ✅ Bon - Vérifier les permissions admin
m:OnNetAdmin("banPlayer", "admin.ban", function(source, targetId, reason)
    -- Seulement exécuté si le joueur a "admin.ban"
end)

-- ✅ Bon - Vérification manuelle
m:OnNet("adminAction", function(source, data)
    local player = GM.GetPlayerFromSource(source)
    if not player then return end
    
    local rank = GM.GetRankFromId(player.rank)
    if not rank or not rank:HasPermission("admin.manage") then
        player:ShowErrorNotify("Permission refusée")
        return
    end
    
    -- Action admin...
end)
```

### Limiter les actions (rate limiting)

```lua
local cooldowns = {}

m:OnNet("spamableAction", function(source, data)
    local now = GetGameTimer()
    local lastAction = cooldowns[source] or 0
    
    if now - lastAction < 1000 then  -- 1 seconde cooldown
        return
    end
    
    cooldowns[source] = now
    -- Traitement...
end)
```

---

## Performances

### Threads optimisés

```lua
-- ❌ Mauvais - Thread trop rapide sans raison
CreateThread(function()
    while true do
        -- Code simple
        Wait(0)  -- 60+ fois par seconde!
    end
end)

-- ✅ Bon - Adapter la fréquence au besoin
CreateThread(function()
    while true do
        if needFastUpdate then
            -- Code critique
            Wait(0)
        else
            -- Code normal
            Wait(500)
        end
    end
end)

-- ✅ Excellent - Utiliser des événements quand possible
m:On("stateChanged", function()
    updateUI()
end)
```

### Mise en cache

```lua
-- ❌ Mauvais - Requête BDD à chaque frame
CreateThread(function()
    while true do
        mdb:findOne('settings', { key = "value" }, function(result)
            -- ...
        end)
        Wait(100)
    end
end)

-- ✅ Bon - Cache avec invalidation
local cachedSettings = nil
local cacheTime = 0

function m.GetSettings()
    if cachedSettings and GetGameTimer() - cacheTime < 60000 then
        return cachedSettings
    end
    
    -- Rafraîchir le cache
    mdb:findOne('settings', { key = "value" }, function(result)
        cachedSettings = result
        cacheTime = GetGameTimer()
    end)
    
    return cachedSettings
end
```

### Éviter les opérations coûteuses

```lua
-- ❌ Mauvais - Parcours complet à chaque vérification
function IsPlayerNearby(playerId)
    for _, player in pairs(XPlayers) do
        if player.id == playerId then
            return #(GetEntityCoords(PlayerPedId()) - player:GetCoords()) < 10
        end
    end
end

-- ✅ Bon - Accès direct
function IsPlayerNearby(playerId)
    local player = XPlayers[playerId]
    if not player or not player.isOnline then return false end
    return #(GetEntityCoords(PlayerPedId()) - player:GetCoords()) < 10
end
```

### Pooling et réutilisation

```lua
-- ❌ Mauvais - Création d'objets en boucle
CreateThread(function()
    while true do
        local data = { x = 0, y = 0, z = 0 }  -- Nouvelle table à chaque fois
        -- ...
        Wait(0)
    end
end)

-- ✅ Bon - Réutiliser l'objet
local tempData = { x = 0, y = 0, z = 0 }
CreateThread(function()
    while true do
        tempData.x, tempData.y, tempData.z = 0, 0, 0
        -- ...
        Wait(0)
    end
end)
```

---

## Debugging

### Utiliser le Logger

```lua
-- Niveaux de log appropriés
Logger.Trace("Détail très fin:", variable)    -- Développement
Logger.Debug("Info de debug:", data)           -- Debug
Logger.Info("Événement important")             -- Production
Logger.Warn("Situation anormale")              -- Attention
Logger.Error("Erreur critique!")               -- Erreur
```

### Messages utiles

```lua
-- ❌ Mauvais
Logger.Debug("here")
Logger.Debug("data:", data)

-- ✅ Bon
Logger.Debug(("Module %s: Joueur %s a acheté %s x%d"):format(
    m.name, 
    player:GetName(), 
    item.name, 
    quantity
))
```

### Debug conditionnel

```lua
if IS_DEV then
    -- Code de debug seulement en développement
    Logger.Debug("État du cache:", json.encode(cache, { indent = true }))
end
```

### Tracer les performances

```lua
local function measureTime(name, fn)
    local start = GetGameTimer()
    local result = fn()
    local elapsed = GetGameTimer() - start
    
    if elapsed > 10 then  -- Plus de 10ms
        Logger.Warn(("Performance: %s a pris %dms"):format(name, elapsed))
    end
    
    return result
end

-- Utilisation
measureTime("loadInventory", function()
    return loadInventoryFromDB(playerId)
end)
```

---

## Git et versioning

### Structure des commits

```
type(scope): description

- feat: Nouvelle fonctionnalité
- fix: Correction de bug
- docs: Documentation
- style: Formatage (pas de changement de code)
- refactor: Refactoring
- perf: Amélioration des performances
- test: Ajout de tests
- chore: Tâches diverses
```

**Exemples:**
```
feat(inventory): ajout du drag & drop
fix(phone): correction crash appel entrant
docs(modules): documentation système de modules
refactor(banking): optimisation des requêtes DB
```

### Branches

```
main           # Production stable
develop        # Développement
feature/xxx    # Nouvelles fonctionnalités
fix/xxx        # Corrections
hotfix/xxx     # Corrections urgentes production
```

### Fichiers à ignorer (.gitignore)

```gitignore
# Cache
cache/

# Logs
*.log

# Base de données locale
db/

# Credentials
configs/credentials.cfg

# IDE
.idea/
.vscode/
*.sublime-*

# OS
.DS_Store
Thumbs.db

# Node
node_modules/
```

---

## Checklist nouveau module

- [ ] Créer la structure de dossiers
- [ ] Créer le `manifest.lua`
- [ ] Créer le fichier de config partagé
- [ ] Implémenter la logique serveur
- [ ] Implémenter la logique client
- [ ] Ajouter les validations de sécurité
- [ ] Tester en local
- [ ] Documenter l'API publique
- [ ] Révision de code
- [ ] Commit avec message approprié

---

## Voir aussi

- [Système de modules](../modules/README.md)
- [Classes principales](../classes/README.md)
- [Base de données MongoDB](../database/README.md)

