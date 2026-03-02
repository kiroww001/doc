# 🛠️ Fonctions Utilitaires - Documentation

> Documentation des fonctions helper disponibles dans Nucleus.

---

## 📋 Table des matières

1. [Logger - Système de logs](#logger---système-de-logs)
2. [GM.Table - Manipulation de tables](#gmtable---manipulation-de-tables)
3. [Cache - Stockage local (KVP)](#cache---stockage-local-kvp)
4. [Fonctions de génération](#fonctions-de-génération)
5. [Notifications (Client)](#notifications-client)
6. [Keybinds et commandes](#keybinds-et-commandes)
7. [Fonctions diverses](#fonctions-diverses)

---

## Logger - Système de logs

Le système de logging permet d'afficher des messages avec différents niveaux de priorité.

### Niveaux de log

| Niveau | Couleur | Description |
|--------|---------|-------------|
| `FATAL` | Rouge | Erreur fatale, arrêt du script |
| `ERROR` | Rouge | Erreur critique |
| `WARN` | Jaune | Avertissement |
| `INFO` | Bleu | Information |
| `DEBUG` | Cyan | Debug (développement) |
| `TRACE` | Cyan | Trace détaillée |

### Utilisation

```lua
Logger.Trace("Message trace détaillé")
Logger.Debug("Message de debug:", variable)
Logger.Info("Information importante")
Logger.Warn("Attention, quelque chose ne va pas")
Logger.Error("Erreur critique!")  -- Déclenche une erreur Lua
Logger.Fatal("Erreur fatale!")    -- Déclenche une erreur Lua
```

### Configuration

Le niveau de log est contrôlé par la convar `nucleus_logLevel` :

```cfg
# Dans server.cfg
set nucleus_logLevel 3  # TRACE (affiche tout)
set nucleus_logLevel 2  # DEBUG
set nucleus_logLevel 1  # INFO (par défaut)
set nucleus_logLevel 0  # WARN
set nucleus_logLevel -1 # ERROR
set nucleus_logLevel -2 # FATAL
```

### Vérifier si un niveau sera affiché

```lua
if Logger.WillDisplay(Logger.Debug) then
    -- Calculs coûteux seulement si le debug est activé
    local data = expensiveCalculation()
    Logger.Debug("Data:", json.encode(data))
end
```

---

## GM.Table - Manipulation de tables

Fonctions utilitaires pour manipuler les tables Lua.

### SizeOf - Taille d'une table

```lua
local t = { a = 1, b = 2, c = 3 }
local size = GM.Table.SizeOf(t)  -- 3

-- Fonctionne aussi pour les tableaux indexés
local arr = { "a", "b", "c" }
local size = GM.Table.SizeOf(arr)  -- 3
```

### IndexOf / LastIndexOf - Trouver un index

```lua
local arr = { "pomme", "banane", "orange", "banane" }

GM.Table.IndexOf(arr, "banane")      -- 2 (premier)
GM.Table.LastIndexOf(arr, "banane")  -- 4 (dernier)
GM.Table.IndexOf(arr, "kiwi")        -- -1 (non trouvé)
```

### Find / FindIndex - Recherche avec callback

```lua
local players = {
    { id = 1, name = "John" },
    { id = 2, name = "Jane" },
    { id = 3, name = "Bob" }
}

-- Trouver un élément
local jane = GM.Table.Find(players, function(p)
    return p.name == "Jane"
end)
-- { id = 2, name = "Jane" }

-- Trouver l'index
local index = GM.Table.FindIndex(players, function(p)
    return p.name == "Jane"
end)
-- 2
```

### Filter - Filtrer une table

```lua
local numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 }

local evens = GM.Table.Filter(numbers, function(n)
    return n % 2 == 0
end)
-- { 2, 4, 6, 8, 10 }
```

### Map - Transformer une table

```lua
local numbers = { 1, 2, 3, 4, 5 }

local doubled = GM.Table.Map(numbers, function(n, i)
    return n * 2
end)
-- { 2, 4, 6, 8, 10 }
```

### Clone - Copie profonde

```lua
local original = {
    name = "John",
    data = { a = 1, b = 2 }
}

local copy = GM.Table.Clone(original)
copy.data.a = 999  -- Ne modifie pas l'original
```

### Concat - Concaténer des tables

```lua
local t1 = { "a", "b" }
local t2 = { "c", "d" }

local t3 = GM.Table.Concat(t1, t2)
-- { "a", "b", "c", "d" }
```

### Join - Joindre en string

```lua
local arr = { "pomme", "banane", "orange" }

GM.Table.Join(arr, ", ")  -- "pomme, banane, orange"
GM.Table.Join(arr)        -- "pomme,banane,orange" (virgule par défaut)
```

### Reverse - Inverser

```lua
local arr = { 1, 2, 3, 4, 5 }

local reversed = GM.Table.Reverse(arr)
-- { 5, 4, 3, 2, 1 }
```

### Sort - Trier avec ordre personnalisé

```lua
local players = {
    { name = "John", score = 100 },
    { name = "Jane", score = 250 },
    { name = "Bob", score = 150 }
}

-- Trier par score décroissant
for name, player in GM.Table.Sort(players, function(t, a, b)
    return t[a].score > t[b].score
end) do
    print(player.name, player.score)
end
-- Jane 250
-- Bob 150
-- John 100
```

### Set - Convertir en ensemble

```lua
local arr = { "admin", "moderator", "vip" }

local set = GM.Table.Set(arr)
-- { admin = true, moderator = true, vip = true }

-- Vérification rapide
if set["admin"] then
    print("Est admin!")
end
```

---

## Cache - Stockage local (KVP)

Le système de cache utilise les KVP (Key-Value Pairs) de FiveM pour stocker des données persistantes côté client.

### Stocker des données

```lua
-- String
SetCache("playerName", "John")

-- Nombre
SetCache("playerLevel", 42)

-- Booléen
SetCache("notificationsEnabled", true)

-- Table (convertie en JSON)
SetCache("playerSettings", {
    volume = 80,
    language = "fr"
})

-- Avec thread (non-bloquant)
SetCache("key", "value", true)
```

### Récupérer des données

```lua
-- String
local name = GetCache("string", "playerName")

-- Nombre
local level = GetCache("number", "playerLevel")

-- Booléen
local enabled = GetCache("boolean", "notificationsEnabled")

-- Table
local settings = GetCache("table", "playerSettings")
```

### Exemple d'utilisation

```lua
-- Sauvegarder les préférences utilisateur
local function SavePreferences(prefs)
    SetCache("userPrefs", prefs)
end

local function LoadPreferences()
    return GetCache("table", "userPrefs") or {
        volume = 100,
        showHud = true,
        language = "fr"
    }
end

-- Utilisation
local prefs = LoadPreferences()
prefs.volume = 50
SavePreferences(prefs)
```

---

## Fonctions de génération

### GenerateRandomLetters

Génère une chaîne de lettres aléatoires (A-Z).

```lua
GM.GenerateRandomLetters(4)   -- "XKQP"
GM.GenerateRandomLetters(8)   -- "ABCDEFGH"
```

### GenerateRandomNumbers

Génère une chaîne de chiffres aléatoires (0-9).

```lua
GM.GenerateRandomNumbers(4)   -- "7294"
GM.GenerateRandomNumbers(6)   -- "103847"
```

### GenerateTableId

Génère un ID unique pour une table (taille + 1).

```lua
local items = { {}, {}, {} }
local newId = GM.GenerateTableId(items)  -- 4
```

### Exemple : Génération de plaque d'immatriculation

```lua
local function GeneratePlate()
    return GM.GenerateRandomLetters(2) .. "-" .. 
           GM.GenerateRandomNumbers(3) .. "-" .. 
           GM.GenerateRandomLetters(2)
end

print(GeneratePlate())  -- "AB-123-CD"
```

---

## Notifications (Client)

### Notifications simples

```lua
-- Succès (vert)
GM.ShowSuccessNotify("Action réussie!", 5000)

-- Information (bleu)
GM.ShowInfoNotify("Nouveau message reçu", 5000)

-- Avertissement (jaune)
GM.ShowWarningNotify("Attention!", 5000)

-- Erreur (rouge)
GM.ShowErrorNotify("Une erreur est survenue", 5000)
```

### Notification personnalisée

```lua
GM.ShowNotify(
    "Mon message",           -- Texte
    "violet",                -- Couleur (vert, bleu, jaune, rouge, violet, etc.)
    "fas-solid fa-star",     -- Icône FontAwesome
    5000                     -- Durée en ms
)
```

### Notification d'entreprise (grande)

```lua
GM.ShowCompanyNotify(
    "Police de Los Santos",           -- Titre
    "https://example.com/logo.png",   -- Image
    "Nouveau rapport",                -- Label
    "Un nouveau rapport a été créé",  -- Message
    "bleu",                           -- Couleur
    10000                             -- Durée
)
```

### Notification d'aide (en bas à droite)

```lua
GM.ShowHelpNotification("Appuyez sur E pour interagir", "bleu")
```

---

## Keybinds et commandes

### Enregistrer une touche

```lua
GM.RegisterKey(
    "E",                          -- Touche
    "interact",                   -- Nom interne
    "Interagir avec l'objet",     -- Description
    function()                    -- Callback
        print("Touche E pressée!")
    end
)
```

### Constantes des touches

La variable `ButtonsList` contient les codes de toutes les touches :

```lua
ButtonsList = {
    ["ESC"] = 322, ["F1"] = 288, ["F2"] = 289, ["F3"] = 170,
    ["TAB"] = 37, ["Q"] = 44, ["W"] = 32, ["E"] = 38, ["R"] = 45,
    ["SPACE"] = 22, ["LEFTSHIFT"] = 21, ["LEFTCTRL"] = 36,
    -- ... etc.
}

-- Utilisation
if IsControlJustPressed(0, ButtonsList["E"]) then
    print("E pressé!")
end
```

### Enregistrer une commande

```lua
-- Commande avec vérification de permission
GM.RegisterCommand("heal", "admin.heal", function(args)
    -- Seulement si le joueur a la permission "admin.heal"
    local targetId = tonumber(args[1])
    print("Heal player", targetId)
end)

-- Commande avec plusieurs alias
GM.RegisterCommand({ "tp", "teleport" }, "admin.tp", function(args)
    -- ...
end)
```

---

## Fonctions diverses

### Ped local

```lua
-- Obtenir le ped du joueur local
local ped = Ped()  -- Alias de PlayerPedId()

-- Vérifier si le personnage est chargé
if IS_CHAR_LOADED then
    -- Le personnage est prêt
end

-- Vérifier si le personnage est occupé
if IS_CHAR_BUSY then
    return
end
```

### Interface NUI

```lua
-- Envoyer des données au NUI
UI.Send("MonAction", {
    key = "value"
})

-- Écouter une réponse du NUI
UI.OnCb("MonActionResponse", function(data, cb)
    print("Réponse:", json.encode(data))
    cb({ success = true })
end)
```

### Actions système

```lua
-- Exécuter une action enregistrée
GM.ExecuteAction("actionId", {
    param1 = "value"
})

-- Exécuter plusieurs actions
GM.ExecuteActions({
    { id = "action1", params = {}, priority = 1 },
    { id = "action2", params = {}, priority = 2 }
})
```

### HashKey

```lua
-- Obtenir le hash d'une chaîne
local hash = HashKey("mp_m_freemode_01")
-- Alternative native
local hash = GetHashKey("mp_m_freemode_01")
```

---

## Voir aussi

- [Classes principales](../classes/README.md)
- [Système de modules](../modules/README.md)
- [Système d'événements](../events/README.md)

