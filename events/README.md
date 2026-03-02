# 📡 Système d'Événements - Documentation

> Documentation complète du système d'événements dans Nucleus.

---

## 📋 Table des matières

1. [Vue d'ensemble](#vue-densemble)
2. [EventBase - Classe de base](#eventbase---classe-de-base)
3. [Événements de modules](#événements-de-modules)
4. [Événements réseau](#événements-réseau)
5. [Callbacks serveur](#callbacks-serveur)
6. [Bonnes pratiques](#bonnes-pratiques)

---

## Vue d'ensemble

Nucleus utilise un système d'événements à plusieurs niveaux :

| Type | Description | Utilisation |
|------|-------------|-------------|
| **Événements locaux** | Communication au sein d'un même côté (client ou serveur) | Entre modules, cycle de vie |
| **Événements réseau** | Communication entre client et serveur | Synchronisation, actions |
| **Callbacks** | Requête-réponse entre client et serveur | Récupérer des données |

### Format des noms d'événements

```lua
-- Événements de module (automatiquement préfixés)
"nucleus:moduleName:eventName"

-- Événements globaux
"nucleus:eventName"
```

---

## EventBase - Classe de base

`EventBase` est la classe fondamentale pour la gestion des événements. Chaque module en possède une instance.

### Méthodes disponibles

```lua
-- Écouter un événement
eventBase:on(eventName, callback)      -- Écoute permanente
eventBase:once(eventName, callback)    -- Écoute unique

-- Émettre un événement
eventBase:emit(eventName, ...)         -- Émission asynchrone (dans des threads)
eventBase:emitSync(eventName, ...)     -- Émission synchrone (même thread)

-- Supprimer des listeners
eventBase:removeListener(eventName, fn) -- Supprimer un listener spécifique
eventBase:removeListeners(eventName)    -- Supprimer tous les listeners d'un event
eventBase:removeAllListeners()          -- Supprimer TOUS les listeners
```

### Exemple d'utilisation directe

```lua
local events = EventBase()

-- Écouter
events:on("playerJoined", function(playerData)
    print("Joueur connecté:", playerData.name)
end)

-- Émettre
events:emit("playerJoined", { name = "John", id = 1 })
```

---

## Événements de modules

Chaque module possède son propre gestionnaire d'événements.

### Écouter des événements

```lua
local m = iAm("monModule")

-- Écoute permanente
m:On("monEvent", function(data)
    print("Événement reçu:", data)
end)

-- Écoute unique (se désabonne après le premier appel)
m:Once("monEvent", function(data)
    print("Reçu une seule fois:", data)
end)
```

### Émettre des événements

```lua
local m = iAm("monModule")

-- Émission asynchrone (recommandé)
-- Chaque listener est exécuté dans son propre thread
m:Emit("monEvent", data1, data2)

-- Émission synchrone
-- Les listeners sont exécutés séquentiellement dans le même thread
m:EmitSync("monEvent", data1, data2)
```

### Supprimer des listeners

```lua
local m = iAm("monModule")

-- Fonction nommée pour pouvoir la supprimer
local function monHandler(data)
    print(data)
end

m:On("monEvent", monHandler)

-- Plus tard...
m:RemoveListener("monEvent", monHandler)

-- Ou supprimer tous les listeners de l'événement
m:RemoveListeners("monEvent")
```

### Événements du cycle de vie

```lua
local m = iAm("monModule")

-- Avant le démarrage complet
m:OnStart(function()
    print("Module en cours de démarrage...")
    -- Initialisation
end)

-- Après le démarrage complet
m:OnStarted(function()
    print("Module démarré!")
    -- Le module est prêt
end)

-- Avant l'arrêt
m:OnStop(function()
    print("Module en cours d'arrêt...")
    -- Nettoyage
end)

-- Après l'arrêt complet
m:OnStopped(function()
    print("Module arrêté!")
end)
```

---

## Événements réseau

### Communication Client → Serveur

#### Côté Client (émetteur)

```lua
local m = iAm("monModule")

-- Envoyer un événement simple
m:EmitServer("saveData", {
    key = "value",
    items = { 1, 2, 3 }
})

-- Envoyer avec latence (gros volumes de données)
-- Le 2ème paramètre est le débit en bytes/seconde
m:EmitLatentServer("uploadData", 128000, largeData)
```

#### Côté Serveur (récepteur)

```lua
local m = iAm("monModule")

-- Écouter l'événement
-- NOTE: Le premier paramètre est toujours `source` (ID du joueur)
m:OnNet("saveData", function(source, data)
    local player = GM.GetPlayerFromSource(source)
    if not player then return end
    
    print("Données reçues de", player:GetName())
    print("Key:", data.key)
    
    -- Traitement...
end)
```

### Communication Serveur → Client

#### Côté Serveur (émetteur)

```lua
local m = iAm("monModule")

-- Envoyer à UN client
m:EmitClient("updateUI", playerId, {
    balance = 1500,
    items = { "item1", "item2" }
})

-- Envoyer à PLUSIEURS clients
m:EmitClients("announcement", { player1Id, player2Id }, {
    message = "Maintenance dans 5 minutes"
})

-- Envoyer avec latence (gros volumes)
m:EmitLatentClient("syncData", playerId, 128000, largeData)
m:EmitLatentClients("syncData", playerIds, 128000, largeData)
```

#### Côté Client (récepteur)

```lua
local m = iAm("monModule")

-- Écouter l'événement
m:OnNet("updateUI", function(data)
    print("Nouvelle balance:", data.balance)
    -- Mettre à jour l'interface...
end)
```

### Événements avec permissions admin

```lua
-- Côté Serveur
local m = iAm("monModule")

-- Seulement si le joueur a la permission spécifiée
m:OnNetAdmin("adminAction", "admin.ban", function(source, data)
    -- Cet événement n'est traité que si le joueur a "admin.ban"
    local target = GM.GetPlayerFromId(data.targetId)
    -- Bannir le joueur...
end)
```

```lua
-- Côté Client
local m = iAm("monModule")

-- Envoyer une action admin
m:EmitServerAdmin("adminAction", "admin.ban", {
    targetId = 123,
    reason = "Triche"
})
```

---

## Callbacks serveur

Les callbacks permettent au client de demander des données au serveur et d'attendre une réponse.

### Enregistrer un callback (Serveur)

```lua
local m = iAm("monModule")

-- Méthode moderne
m:OnNetCb("getData", function(source, cb, param1, param2)
    local player = GM.GetPlayerFromSource(source)
    if not player then
        return cb(false, "Joueur non trouvé")
    end
    
    -- Traitement...
    local result = {
        name = player:GetName(),
        data = param1 .. param2
    }
    
    -- Retourner le résultat
    cb(true, result)
end)

-- Ancienne syntaxe (dépréciée mais fonctionnelle)
m:RegisterServerCallback("getData", function(source, cb, param1, param2)
    -- ...
end)
```

### Appeler un callback (Client)

```lua
local m = iAm("monModule")

-- Méthode moderne
m:EmitServerCb("getData", function(success, data)
    if success then
        print("Nom:", data.name)
        print("Data:", data.data)
    else
        print("Erreur:", data)
    end
end, "param1", "param2")

-- Ancienne syntaxe (dépréciée mais fonctionnelle)
m:TriggerServerCallback("getData", function(success, data)
    -- ...
end, "param1", "param2")

-- Avec latence (gros volumes)
m:EmitLatentServerCb("getData", function(success, data)
    -- ...
end, 128000, "param1", "param2")
```

### Exemple complet : Récupérer l'inventaire

```lua
-- Serveur
local m = iAm("inventory")

m:RegisterServerCallback("getInventory", function(source, cb)
    local player = GM.GetPlayerFromSource(source)
    if not player then
        return cb(false, "Joueur non trouvé")
    end
    
    local char = GM.GetCharacterFromId(player:GetCurrentCharacterId())
    if not char then
        return cb(false, "Personnage non trouvé")
    end
    
    local inventory = char:GetInventory()
    cb(true, inventory)
end)
```

```lua
-- Client
local m = iAm("inventory")

function m.RefreshInventory()
    m:TriggerServerCallback("getInventory", function(success, inventory)
        if not success then
            GM.ShowErrorNotify(inventory)
            return
        end
        
        m.currentInventory = inventory
        m.UpdateUI()
    end)
end
```

---

## Événements globaux (hors modules)

### Côté Client

```lua
-- Écouter un événement réseau global
GM.OnNet("nucleus:updatePlayer", function(playerData)
    Player = playerData
end)

-- Envoyer au serveur
GM.EmitServer("nucleus:requestData", params)
```

### Côté Serveur

```lua
-- Envoyer à un client spécifique
GM.EmitClient("nucleus:updatePlayer", source, playerData)

-- Envoyer à tous les clients
GM.EmitAllClients("nucleus:announcement", {
    message = "Bienvenue!"
})
```

---

## Interface NUI (UI)

### Envoyer des données à l'interface web

```lua
-- Côté Client uniquement
UI.Send("MonAction", {
    key = "value",
    items = { 1, 2, 3 }
})
```

Le JavaScript reçoit dans `window.addEventListener('message', ...)` :
```javascript
{
    type: "MonAction",
    key: "value",
    items: [1, 2, 3]
}
```

### Écouter les callbacks NUI

```lua
-- Côté Client
UI.OnCb('MonActionResponse', function(data, cb)
    print("Réponse UI:", json.encode(data))
    
    -- Traitement...
    
    -- Répondre au JavaScript
    cb({ success = true })
end)
```

---

## Bonnes pratiques

### 1. Nommez vos événements clairement

```lua
-- ❌ Mauvais
m:Emit("e1", data)
m:OnNet("x", callback)

-- ✅ Bon
m:Emit("inventoryUpdated", inventory)
m:OnNet("requestItemTransfer", callback)
```

### 2. Validez toujours les données côté serveur

```lua
m:OnNet("buyItem", function(source, data)
    -- ✅ Toujours valider
    if not data or not data.itemId then
        return
    end
    
    local player = GM.GetPlayerFromSource(source)
    if not player then return end
    
    local itemId = tostring(data.itemId)
    local quantity = tonumber(data.quantity) or 1
    
    if quantity <= 0 or quantity > 99 then
        return
    end
    
    -- Traitement sécurisé...
end)
```

### 3. Utilisez les callbacks pour les requêtes

```lua
-- ❌ Mauvais - Deux événements séparés
-- Client
m:EmitServer("getData")
m:OnNet("dataReceived", function(data) ... end)

-- ✅ Bon - Callback
m:EmitServerCb("getData", function(success, data)
    if success then
        -- Utiliser data
    end
end)
```

### 4. Évitez les boucles d'événements

```lua
-- ❌ Danger - Boucle infinie
m:On("eventA", function()
    m:Emit("eventB")
end)
m:On("eventB", function()
    m:Emit("eventA") -- Boucle!
end)
```

### 5. Nettoyez les listeners inutilisés

```lua
local handler = function(data)
    -- ...
end

m:On("tempEvent", handler)

-- Quand l'événement n'est plus nécessaire
m:RemoveListener("tempEvent", handler)
```

### 6. Utilisez `Once` pour les événements ponctuels

```lua
-- ✅ Se désabonne automatiquement après le premier appel
m:Once("playerReady", function()
    -- Initialisation unique
end)
```

---

## Voir aussi

- [Système de modules](../modules/README.md)
- [Classes principales](../classes/README.md)
- [Interface NUI](../ui/README.md)

