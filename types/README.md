# 📝 Types et Annotations - Documentation

> Guide pour utiliser les annotations de type dans Nucleus (compatible avec EmmyLua/Lua Language Server).

---

## 📋 Table des matières

1. [Introduction](#introduction)
2. [Annotations de base](#annotations-de-base)
3. [Types du framework](#types-du-framework)
4. [Documenter son code](#documenter-son-code)
5. [Exemples complets](#exemples-complets)

---

## Introduction

Nucleus utilise les annotations **EmmyLua** pour fournir l'autocomplétion et la documentation inline dans les IDE comme VS Code (avec l'extension Lua Language Server) ou IntelliJ.

### Configuration VS Code

1. Installez l'extension **Lua** par sumneko
2. Ajoutez dans `.vscode/settings.json` :

```json
{
    "Lua.diagnostics.globals": [
        "GM", "Logger", "Modules", "iAm", "UI",
        "GetPlayerPed", "CreateThread", "Wait", "Citizen",
        "RegisterCommand", "TriggerEvent", "TriggerServerEvent",
        "AddEventHandler", "RegisterNetEvent", "RegisterNUICallback",
        "SetNuiFocus", "SendNUIMessage", "GetGameTimer",
        "PlayerPedId", "GetEntityCoords", "GetEntityHeading",
        "exports", "json", "IS_SERVER", "IS_DEV",
        "XPlayers", "XCharacters", "XItems", "mdb"
    ],
    "Lua.workspace.library": [
        "path/to/nucleus/shared"
    ]
}
```

---

## Annotations de base

### Types primitifs

```lua
---@param name string          -- Chaîne de caractères
---@param count number         -- Nombre
---@param enabled boolean      -- Booléen
---@param data table           -- Table générique
---@param callback function    -- Fonction
---@param optional? string     -- Paramètre optionnel
---@param mixed string|number  -- Type union
```

### Retour de fonction

```lua
---@return string              -- Retourne une string
---@return boolean, string     -- Retourne deux valeurs
---@return string?             -- Peut retourner nil
```

### Variables

```lua
---@type string
local name = "John"

---@type number[]
local numbers = { 1, 2, 3 }

---@type table<string, number>
local scores = { john = 100, jane = 95 }
```

---

## Types du framework

### Module

```lua
---@class Module
---@field name string
---@field path string
---@field author string
---@field version string
---@field description string
---@field state "uninitialized"|"starting"|"started"|"stopping"|"stopped"
```

### Player (Serveur)

```lua
---@class XPlayer
---@field id number
---@field identifier string
---@field name string
---@field source number
---@field isOnline boolean
---@field currentCharacter number
---@field characterLimit number
---@field rank number
---@field coins number
---@field vip number
---@field metadata table
```

### Character (Serveur)

```lua
---@class XCharacter
---@field id number
---@field owner number
---@field identity CharacterIdentity
---@field skin table
---@field sex number
---@field status CharacterStatus
---@field inUse boolean
---@field inventory CharacterInventory
---@field position Vector4
---@field isLocked number
---@field groups table
---@field metadata table
```

### Sous-types

```lua
---@class CharacterIdentity
---@field firstname string
---@field lastname string
---@field dob string
---@field nationality string
---@field phone? string

---@class CharacterStatus
---@field health number
---@field armour number
---@field hunger number
---@field thirst number
---@field stress? number

---@class CharacterInventory
---@field items InventoryItem[]
---@field weight number

---@class InventoryItem
---@field id string
---@field name string
---@field count number
---@field metadata? table
```

### Vector4

```lua
---@class Vector4
---@field x number
---@field y number
---@field z number
---@field w number
```

---

## Documenter son code

### Documenter un module

```lua
-- En haut du fichier shared/config.lua
---@class CFGMonModule
---@field MaxItems number Nombre maximum d'items
---@field Cooldown number Temps de recharge en ms
---@field EnableDebug boolean Activer le mode debug
local CFG = {}

---@module SHMonModule : Module
local m = iAm("monModule")
```

### Documenter côté client

```lua
-- En haut du fichier client/main.lua
---@module CMonModule : SHMonModule
local m = iAm("monModule")
---@type CFGMonModule
local CFG = m:CFG()
```

### Documenter côté serveur

```lua
-- En haut du fichier server/main.lua
---@module SMonModule : SHMonModule
local m = iAm("monModule")
---@type CFGMonModule
local CFG = m:CFG()
```

### Documenter une fonction

```lua
--- Ajoute un item à l'inventaire du personnage
---@param itemName string Le nom de l'item
---@param quantity number La quantité à ajouter
---@param metadata? table Métadonnées optionnelles
---@return boolean success Si l'ajout a réussi
---@return string message Message descriptif
function Character:AddItem(itemName, quantity, metadata)
    -- ...
end
```

### Documenter un callback

```lua
---@alias PlayerCallback fun(source: number, cb: fun(success: boolean, data: any), ...: any)

--- Enregistre un callback serveur
---@param eventName string Nom de l'événement
---@param callback PlayerCallback La fonction callback
function Module:RegisterServerCallback(eventName, callback)
    -- ...
end
```

---

## Exemples complets

### Module complet annoté

```lua
-- modules/[interfaces]/shop/shared/config.lua

---@class ShopItem
---@field id string
---@field name string
---@field price number
---@field category string
---@field imageUrl? string

---@class CFGShop
---@field Items ShopItem[]
---@field MaxPurchaseQuantity number
---@field DiscountVIP number Pourcentage de réduction pour les VIP
local CFG = {}

---@module SHShop : Module
local m = iAm("shop")

CFG.Items = {
    { id = "bread", name = "Pain", price = 5, category = "food" },
    { id = "water", name = "Eau", price = 3, category = "drink" }
}
CFG.MaxPurchaseQuantity = 99
CFG.DiscountVIP = 10

m:CFG(CFG)
```

```lua
-- modules/[interfaces]/shop/server/main.lua

---@module SShop : SHShop
local m = iAm("shop")
---@type CFGShop
local CFG = m:CFG()

--- Calcule le prix d'un item pour un joueur
---@param itemId string L'ID de l'item
---@param playerId number L'ID du joueur
---@return number|nil price Le prix calculé ou nil si item non trouvé
local function calculatePrice(itemId, playerId)
    local item = GM.Table.Find(CFG.Items, function(i) 
        return i.id == itemId 
    end)
    
    if not item then return nil end
    
    local player = GM.GetPlayerFromId(playerId)
    if player and player:IsVIP() then
        return item.price * (1 - CFG.DiscountVIP / 100)
    end
    
    return item.price
end

--- Traite l'achat d'un item
---@param source number ID source du joueur
---@param itemId string ID de l'item à acheter
---@param quantity number Quantité souhaitée
---@return boolean success Si l'achat a réussi
---@return string message Message de retour
function m.ProcessPurchase(source, itemId, quantity)
    local player = GM.GetPlayerFromSource(source)
    if not player then
        return false, "Joueur non trouvé"
    end
    
    if quantity > CFG.MaxPurchaseQuantity then
        return false, "Quantité maximale dépassée"
    end
    
    local price = calculatePrice(itemId, player:GetId())
    if not price then
        return false, "Item non trouvé"
    end
    
    local totalPrice = price * quantity
    local char = GM.GetCharacterFromId(player:GetCurrentCharacterId())
    
    if not char then
        return false, "Personnage non trouvé"
    end
    
    -- Vérifier les fonds et effectuer l'achat...
    
    return true, "Achat effectué!"
end

m:RegisterServerCallback("buyItem", function(source, cb, itemId, quantity)
    local success, message = m.ProcessPurchase(source, itemId, quantity or 1)
    cb(success, message)
end)
```

### Récupérer un module avec le bon type

```lua
---@type CInventory
local inventory = Modules.Get("inventory")

---@type CHud
local hud = Modules.Get("hud")

---@type SAdminMenu
local adminMenu = Modules.Get("adminMenu")
```

---

## Conventions de nommage des types

| Préfixe | Signification | Exemple |
|---------|---------------|---------|
| `C` | Client | `CInventory`, `CHud` |
| `S` | Server | `SInventory`, `SAdminMenu` |
| `SH` | Shared | `SHInventory` |
| `CFG` | Configuration | `CFGInventory` |
| `X` | Instance stockée | `XPlayer`, `XCharacter` |
| `_` | Classe/constructeur | `_Player`, `_Character` |

---

## Voir aussi

- [Système de modules](./modules/README.md)
- [Classes principales](./classes/README.md)
- [Bonnes pratiques](./best-practices/README.md)

