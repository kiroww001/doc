# 💾 Base de données MongoDB - Documentation

> Documentation pour l'utilisation de MongoDB dans Nucleus.

---

## 📋 Table des matières

1. [Vue d'ensemble](#vue-densemble)
2. [Configuration](#configuration)
3. [API MongoDB](#api-mongodb)
4. [Opérations CRUD](#opérations-crud)
5. [Requêtes avancées](#requêtes-avancées)
6. [Collections utilisées](#collections-utilisées)
7. [Bonnes pratiques](#bonnes-pratiques)

---

## Vue d'ensemble

Nucleus utilise **MongoDB** comme base de données principale. L'accès se fait via l'export `mongodb` :

```lua
mdb = exports.mongodb
```

### Avantages de MongoDB pour FiveM

- 📦 **Documents JSON** - Structure flexible, parfaite pour les données de jeu
- 🚀 **Performances** - Requêtes rapides, pas de schéma rigide
- 🔄 **Asynchrone** - Les callbacks ne bloquent pas le thread principal

---

## Configuration

### Dans credentials.cfg

```cfg
set mongodb_url "mongodb://localhost:27017"
set mongodb_database "nucleus"
```

### Connexion vérifiée au démarrage

La ressource `mongodb` doit être démarrée avant `nucleus` dans `resources.cfg` :

```cfg
ensure mongodb
ensure nucleus
```

---

## API MongoDB

### Référence globale

```lua
-- Accès global (défini dans server.lua)
mdb = exports.mongodb
```

### Méthodes disponibles

| Méthode | Description |
|---------|-------------|
| `mdb:find(collection, query, callback)` | Rechercher des documents |
| `mdb:findOne(collection, query, callback)` | Trouver un seul document |
| `mdb:insertOne(collection, document, callback)` | Insérer un document |
| `mdb:insertMany(collection, documents, callback)` | Insérer plusieurs documents |
| `mdb:updateOne(collection, filter, update, options, callback)` | Mettre à jour un document |
| `mdb:updateMany(collection, filter, update, options, callback)` | Mettre à jour plusieurs documents |
| `mdb:deleteOne(collection, filter, callback)` | Supprimer un document |
| `mdb:deleteMany(collection, filter, callback)` | Supprimer plusieurs documents |
| `mdb:count(collection, query, callback)` | Compter les documents |

---

## Opérations CRUD

### Create (Insertion)

#### insertOne - Insérer un document

```lua
mdb:insertOne('players', {
    id = 1,
    identifier = "license:abc123",
    name = "John Doe",
    firstConnexion = os.time(),
    currentCharacter = 0,
    characterLimit = 3,
    rank = 0,
    coins = 0,
    vip = 0,
    metadata = {}
}, function(success, insertedId)
    if success then
        Logger.Info("Joueur créé avec l'ID:", insertedId)
    else
        Logger.Error("Erreur lors de la création du joueur")
    end
end)
```

#### insertMany - Insérer plusieurs documents

```lua
mdb:insertMany('items', {
    { id = "bread", label = "Pain", weightPerUnit = 0.1 },
    { id = "water", label = "Eau", weightPerUnit = 0.5 },
    { id = "medkit", label = "Trousse de soins", weightPerUnit = 1.0 }
}, function(success, insertedCount)
    Logger.Info("Items créés:", insertedCount)
end)
```

### Read (Lecture)

#### find - Rechercher plusieurs documents

```lua
-- Tous les joueurs
mdb:find('players', {}, function(players)
    for _, player in pairs(players) do
        print("Joueur:", player.name)
    end
end)

-- Avec filtre
mdb:find('players', { rank = 5 }, function(admins)
    Logger.Info("Admins trouvés:", #admins)
end)

-- Avec options (tri, limite)
mdb:find('players', {}, {
    sort = { firstConnexion = -1 },  -- -1 = décroissant
    limit = 10
}, function(recentPlayers)
    -- Les 10 derniers joueurs inscrits
end)
```

#### findOne - Trouver un seul document

```lua
mdb:findOne('players', { identifier = "license:abc123" }, function(player)
    if player then
        print("Joueur trouvé:", player.name)
    else
        print("Joueur non trouvé")
    end
end)
```

### Update (Mise à jour)

#### updateOne - Mettre à jour un document

```lua
-- Mise à jour simple avec $set
mdb:updateOne('players', 
    { id = 1 },                          -- Filtre
    { ["$set"] = { coins = 500 } },      -- Mise à jour
    function(success, modifiedCount)
        Logger.Info("Documents modifiés:", modifiedCount)
    end
)

-- Incrémenter une valeur avec $inc
mdb:updateOne('players',
    { id = 1 },
    { ["$inc"] = { coins = 100 } },  -- +100 coins
    function(success)
        -- ...
    end
)

-- Upsert (créer si n'existe pas)
mdb:updateOne('player_settings',
    { playerId = 1 },
    { ["$set"] = { volume = 80, language = "fr" } },
    { upsert = true },
    function(success)
        -- Créé ou mis à jour
    end
)
```

#### updateMany - Mettre à jour plusieurs documents

```lua
-- Donner 100 coins à tous les VIP
mdb:updateMany('players',
    { vip = { ["$gt"] = 0 } },           -- vip > 0
    { ["$inc"] = { coins = 100 } },
    function(success, modifiedCount)
        Logger.Info("VIP récompensés:", modifiedCount)
    end
)
```

### Delete (Suppression)

#### deleteOne - Supprimer un document

```lua
mdb:deleteOne('bans', { id = banId }, function(success, deletedCount)
    if deletedCount > 0 then
        Logger.Info("Ban supprimé")
    end
end)
```

#### deleteMany - Supprimer plusieurs documents

```lua
-- Supprimer les bans expirés
mdb:deleteMany('bans',
    { endAt = { ["$lt"] = os.time() } },  -- endAt < maintenant
    function(success, deletedCount)
        Logger.Info("Bans expirés supprimés:", deletedCount)
    end
)
```

### Count (Comptage)

```lua
mdb:count('players', { vip = { ["$gt"] = 0 } }, function(count)
    Logger.Info("Nombre de VIP:", count)
end)
```

---

## Requêtes avancées

### Opérateurs de comparaison

| Opérateur | Description | Exemple |
|-----------|-------------|---------|
| `$eq` | Égal | `{ age = { ["$eq"] = 25 } }` |
| `$ne` | Différent | `{ status = { ["$ne"] = "banned" } }` |
| `$gt` | Supérieur | `{ level = { ["$gt"] = 10 } }` |
| `$gte` | Supérieur ou égal | `{ level = { ["$gte"] = 10 } }` |
| `$lt` | Inférieur | `{ price = { ["$lt"] = 100 } }` |
| `$lte` | Inférieur ou égal | `{ price = { ["$lte"] = 100 } }` |
| `$in` | Dans la liste | `{ status = { ["$in"] = { "active", "pending" } } }` |
| `$nin` | Pas dans la liste | `{ role = { ["$nin"] = { "admin", "mod" } } }` |

### Opérateurs logiques

```lua
-- AND (implicite)
mdb:find('players', {
    rank = { ["$gte"] = 3 },
    vip = 1
}, callback)

-- OR explicite
mdb:find('players', {
    ["$or"] = {
        { rank = { ["$gte"] = 3 } },
        { vip = 1 }
    }
}, callback)

-- AND explicite
mdb:find('players', {
    ["$and"] = {
        { rank = { ["$gte"] = 3 } },
        { vip = 1 }
    }
}, callback)
```

### Opérateurs de mise à jour

| Opérateur | Description | Exemple |
|-----------|-------------|---------|
| `$set` | Définir une valeur | `{ ["$set"] = { name = "John" } }` |
| `$unset` | Supprimer un champ | `{ ["$unset"] = { tempField = "" } }` |
| `$inc` | Incrémenter | `{ ["$inc"] = { coins = 100 } }` |
| `$push` | Ajouter à un tableau | `{ ["$push"] = { items = newItem } }` |
| `$pull` | Retirer d'un tableau | `{ ["$pull"] = { items = { id = itemId } } }` |
| `$addToSet` | Ajouter si absent | `{ ["$addToSet"] = { tags = "vip" } }` |

### Exemples avancés

```lua
-- Trouver les joueurs avec plus de 1000 coins OU qui sont VIP
mdb:find('players', {
    ["$or"] = {
        { coins = { ["$gt"] = 1000 } },
        { vip = { ["$gt"] = 0 } }
    }
}, function(players)
    -- ...
end)

-- Ajouter un item à l'inventaire
mdb:updateOne('characters',
    { id = charId },
    { ["$push"] = { 
        ["inventory.items"] = {
            id = "item_" .. os.time(),
            name = "bread",
            count = 5
        }
    } },
    function(success)
        -- ...
    end
)

-- Retirer un item de l'inventaire
mdb:updateOne('characters',
    { id = charId },
    { ["$pull"] = { 
        ["inventory.items"] = { id = itemId }
    } },
    function(success)
        -- ...
    end
)
```

---

## Collections utilisées

### players

Stocke les données des comptes joueurs.

```lua
{
    id = 1,                              -- ID unique
    identifier = "license:abc123",       -- License FiveM
    name = "John Doe",                   -- Nom Steam/FiveM
    firstConnexion = 1709312000,         -- Timestamp première connexion
    currentCharacter = 1,                -- ID du personnage actif
    characterLimit = 3,                  -- Nombre max de personnages
    rank = 0,                            -- Rang admin (0 = joueur)
    coins = 0,                           -- Monnaie premium
    vip = 0,                             -- Niveau VIP
    metadata = {}                        -- Données supplémentaires
}
```

### characters

Stocke les personnages des joueurs.

```lua
{
    id = 1,                              -- ID unique
    owner = 1,                           -- ID du joueur propriétaire
    identity = {
        firstname = "John",
        lastname = "Doe",
        dob = "01/01/1990",
        nationality = "Américain"
    },
    skin = { ... },                      -- Apparence du personnage
    sex = 0,                             -- 0 = Homme, 1 = Femme
    status = {
        health = 200,
        armour = 0,
        hunger = 100,
        thirst = 100
    },
    inUse = false,                       -- En cours d'utilisation
    inventory = {
        items = [ ... ],
        weight = 0
    },
    position = { x = 0, y = 0, z = 0, w = 0 },
    isLocked = 0,                        -- Verrouillé par admin
    creationDate = 1709312000,
    metadata = {},
    groups = []                          -- Factions/jobs
}
```

### items

Définition des types d'items.

```lua
{
    id = "bread",                        -- Identifiant unique
    label = "Pain",                      -- Nom affiché
    weightPerUnit = 0.1,                 -- Poids par unité
    imageUrl = "https://...",            -- Image
    metadata = {
        type = "food",                   -- Type d'item
        receive = 20                     -- Valeur de restauration
    }
}
```

### groups

Factions et métiers.

```lua
{
    id = 1,
    name = "police",
    label = "Police de Los Santos",
    grades = [
        { id = 0, name = "Cadet", salary = 500 },
        { id = 1, name = "Officier", salary = 800 },
        { id = 2, name = "Sergent", salary = 1200 }
    ],
    metadata = {}
}
```

### ranks

Rangs administrateurs.

```lua
{
    id = 1,
    name = "Modérateur",
    level = 1,
    permissions = [
        "admin.kick",
        "admin.warn",
        "admin.spectate"
    ],
    metadata = {}
}
```

### bans

Sanctions (bans).

```lua
{
    id = 1,
    playerId = 123,
    reason = "Triche",
    startAt = 1709312000,
    endAt = 1709398400,                  -- -1 = permanent
    active = 1,
    adminId = 1,
    metadata = {}
}
```

### vehicles

Véhicules des joueurs.

```lua
{
    id = 1,
    owner = 1,                           -- ID du personnage
    model = "adder",
    plate = "ABC123",
    garage = "main_garage",
    stored = true,
    mods = { ... },
    metadata = {}
}
```

### inventories

Inventaires externes (coffres, etc.).

```lua
{
    id = "trunk_ABC123",                 -- ID unique
    type = "trunk",                      -- Type (trunk, stash, etc.)
    items = [ ... ],
    maximumWeight = 100,
    metadata = {}
}
```

---

## Bonnes pratiques

### 1. Toujours utiliser des callbacks

```lua
-- ❌ Mauvais - Ne pas ignorer le résultat
mdb:updateOne('players', { id = 1 }, { ["$set"] = { coins = 100 } })

-- ✅ Bon - Gérer le résultat
mdb:updateOne('players', { id = 1 }, { ["$set"] = { coins = 100 } }, function(success)
    if success then
        Logger.Info("Coins mis à jour")
    else
        Logger.Error("Erreur mise à jour coins")
    end
end)
```

### 2. Indexer les champs fréquemment recherchés

Créez des index sur les champs utilisés dans les requêtes fréquentes (via MongoDB Compass ou shell) :

```javascript
// Dans MongoDB Shell
db.players.createIndex({ "identifier": 1 })
db.characters.createIndex({ "owner": 1 })
db.bans.createIndex({ "playerId": 1 })
```

### 3. Éviter les requêtes bloquantes au démarrage

```lua
-- ❌ Mauvais - Bloque le démarrage
local players = {}
mdb:find('players', {}, function(results)
    players = results
end)
-- Le code continue AVANT que le callback soit appelé!

-- ✅ Bon - Utiliser OnStart
m:OnStart(function()
    mdb:find('players', {}, function(results)
        for _, data in pairs(results) do
            local player = _Player:Create(data, false)
            XPlayers[player.id] = player
        end
        Logger.Info("Joueurs chargés:", #results)
    end)
end)
```

### 4. Utiliser upsert quand approprié

```lua
-- Créer ou mettre à jour en une seule opération
mdb:updateOne('player_settings',
    { playerId = playerId },
    { ["$set"] = settings },
    { upsert = true },
    callback
)
```

### 5. Grouper les opérations similaires

```lua
-- ❌ Mauvais - Plusieurs requêtes
for _, item in pairs(items) do
    mdb:insertOne('items', item)
end

-- ✅ Bon - Une seule requête
mdb:insertMany('items', items, function(success, count)
    Logger.Info("Items insérés:", count)
end)
```

### 6. Sauvegarder périodiquement

```lua
-- Sauvegarder tous les joueurs toutes les 5 minutes
CreateThread(function()
    while true do
        Wait(5 * 60 * 1000) -- 5 minutes
        
        for _, player in pairs(XPlayers) do
            if player.isOnline then
                player:Save()
            end
        end
        
        Logger.Debug("Sauvegarde automatique effectuée")
    end
end)
```

---

## Voir aussi

- [Classes principales](../classes/README.md)
- [Système de modules](../modules/README.md)
- [Bonnes pratiques](../best-practices/README.md)

