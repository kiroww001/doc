# 🎨 Interface NUI (Web) - Documentation

> Documentation pour le développement de l'interface web de Nucleus.

---

## 📋 Table des matières

1. [Vue d'ensemble](#vue-densemble)
2. [Structure des fichiers web](#structure-des-fichiers-web)
3. [Communication Lua ↔ JavaScript](#communication-lua--javascript)
4. [Créer une interface de module](#créer-une-interface-de-module)
5. [Exemples pratiques](#exemples-pratiques)
6. [Bonnes pratiques](#bonnes-pratiques)

---

## Vue d'ensemble

L'interface NUI (Natural User Interface) de Nucleus utilise des technologies web standard :

- **HTML** - Structure des pages
- **CSS** - Styles et mise en page
- **JavaScript** - Logique et interactivité

Le framework utilise une **page unique** (`web/index.html`) qui gère toutes les interfaces via des composants.

### Configuration dans fxmanifest

```lua
-- Fichiers accessibles
files({
    "web/index.html",
    "web/**/*.*"
})

-- Page principale
ui_page("web/index.html")
ui_page_preload("yes")  -- Préchargement pour des performances optimales
```

---

## Structure des fichiers web

```
nucleus/web/
├── index.html              # Page principale
├── js/                     # Scripts JavaScript
│   └── main.js            # Script principal
├── public/                 # Assets publics
│   ├── css/
│   └── images/
├── icons/                  # Icônes
├── assets/                 # Assets divers
│
│ # Interfaces par module
├── inventory/              # Interface inventaire
├── hud/                    # Interface HUD
├── bank/                   # Interface banque
├── phone/                  # Interface téléphone
├── heritage/               # Interface création perso
├── premiumShop/            # Boutique premium
└── builder/                # Éditeur
```

---

## Communication Lua ↔ JavaScript

### Lua → JavaScript (Envoyer des données)

**Côté Lua (Client):**
```lua
UI.Send("MonAction", {
    visible = true,
    items = { "item1", "item2" },
    player = {
        name = "John",
        money = 1500
    }
})
```

**Côté JavaScript:**
```javascript
window.addEventListener('message', function(event) {
    const data = event.data;
    
    // Le type correspond à l'action envoyée
    if (data.type === 'MonAction') {
        console.log('Visible:', data.visible);
        console.log('Items:', data.items);
        console.log('Player:', data.player.name);
        
        // Afficher l'interface
        document.getElementById('monInterface').style.display = 'block';
    }
});
```

### JavaScript → Lua (Répondre / Callback)

**Côté JavaScript:**
```javascript
// Utiliser fetch pour appeler un callback Lua
async function sendToLua(action, data) {
    const response = await fetch(`https://${GetParentResourceName()}/${action}`, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(data)
    });
    return response.json();
}

// Exemple d'utilisation
document.getElementById('closeBtn').addEventListener('click', async () => {
    const result = await sendToLua('Inventory:Closed', {
        reason: 'user_closed'
    });
    console.log('Réponse Lua:', result);
});

// Fonction utilitaire pour obtenir le nom de la ressource
function GetParentResourceName() {
    return window.GetParentResourceName ? window.GetParentResourceName() : 'nucleus';
}
```

**Côté Lua (Client):**
```lua
UI.OnCb('Inventory:Closed', function(data, cb)
    print("Interface fermée, raison:", data.reason)
    
    -- Fermer l'inventaire
    m.CloseInventory()
    
    -- Répondre au JavaScript
    cb({ success = true, message = "OK" })
end)
```

### Raccourci pour RegisterNUICallback

```lua
-- UI.OnCb est un alias de RegisterNUICallback
UI.OnCb = RegisterNUICallback
```

---

## Contrôle du focus NUI

### Activer le focus (souris visible, jeu en pause)

```lua
SetNuiFocus(true, true)           -- Focus + curseur
SetNuiFocusKeepInput(true)        -- Garder les inputs (mouvement, etc.)
```

### Désactiver le focus

```lua
SetNuiFocus(false, false)
SetNuiFocusKeepInput(false)
```

### Exemple typique d'ouverture/fermeture

```lua
function OpenMyInterface()
    UI.Send("MyInterface:Open", { data = "..." })
    SetNuiFocus(true, true)
    SetNuiFocusKeepInput(true)
end

function CloseMyInterface()
    UI.Send("MyInterface:Close", {})
    SetNuiFocus(false, false)
    SetNuiFocusKeepInput(false)
end

-- Écouter la fermeture depuis le JS
UI.OnCb('MyInterface:Closed', function(data, cb)
    CloseMyInterface()
    cb({ success = true })
end)
```

---

## Créer une interface de module

### Étape 1 : Créer le dossier web du module

```
nucleus/web/monModule/
├── index.html      # (Optionnel) Page dédiée
├── style.css       # Styles
└── script.js       # Logique
```

### Étape 2 : Intégrer dans l'index.html principal

```html
<!-- Dans web/index.html -->
<div id="monModule-container" style="display: none;">
    <!-- Votre interface -->
    <div class="monModule-panel">
        <h1>Mon Module</h1>
        <div id="monModule-content"></div>
        <button id="monModule-close">Fermer</button>
    </div>
</div>

<!-- Charger les scripts -->
<link rel="stylesheet" href="monModule/style.css">
<script src="monModule/script.js"></script>
```

### Étape 3 : Le JavaScript du module

```javascript
// web/monModule/script.js

const MonModule = {
    container: null,
    
    init() {
        this.container = document.getElementById('monModule-container');
        this.setupEventListeners();
    },
    
    setupEventListeners() {
        // Écouter les messages Lua
        window.addEventListener('message', (event) => {
            const data = event.data;
            
            switch(data.type) {
                case 'MonModule:Open':
                    this.open(data);
                    break;
                case 'MonModule:Close':
                    this.close();
                    break;
                case 'MonModule:Update':
                    this.update(data);
                    break;
            }
        });
        
        // Bouton fermer
        document.getElementById('monModule-close').addEventListener('click', () => {
            this.requestClose();
        });
        
        // Fermer avec Échap
        document.addEventListener('keydown', (e) => {
            if (e.key === 'Escape' && this.isOpen()) {
                this.requestClose();
            }
        });
    },
    
    open(data) {
        this.container.style.display = 'flex';
        this.update(data);
    },
    
    close() {
        this.container.style.display = 'none';
    },
    
    isOpen() {
        return this.container.style.display !== 'none';
    },
    
    update(data) {
        // Mettre à jour l'interface avec les données
        const content = document.getElementById('monModule-content');
        content.innerHTML = `<p>${data.message || 'Aucune donnée'}</p>`;
    },
    
    async requestClose() {
        await fetch(`https://${GetParentResourceName()}/MonModule:Closed`, {
            method: 'POST',
            body: JSON.stringify({ reason: 'user' })
        });
    }
};

// Initialiser au chargement
document.addEventListener('DOMContentLoaded', () => {
    MonModule.init();
});

function GetParentResourceName() {
    return window.GetParentResourceName ? window.GetParentResourceName() : 'nucleus';
}
```

### Étape 4 : Le CSS du module

```css
/* web/monModule/style.css */

#monModule-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: none;
    justify-content: center;
    align-items: center;
    background: rgba(0, 0, 0, 0.5);
    z-index: 1000;
}

.monModule-panel {
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
    border-radius: 10px;
    padding: 30px;
    min-width: 400px;
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.5);
    color: white;
}

.monModule-panel h1 {
    margin: 0 0 20px 0;
    font-size: 24px;
    color: #e94560;
}

#monModule-close {
    background: #e94560;
    border: none;
    color: white;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
    margin-top: 20px;
    transition: background 0.3s;
}

#monModule-close:hover {
    background: #ff6b6b;
}
```

### Étape 5 : Le code Lua du module

```lua
-- modules/[interfaces]/monModule/client/main.lua

local m = iAm("monModule")
local isOpen = false

function m.Open(data)
    if isOpen then return end
    
    isOpen = true
    UI.Send("MonModule:Open", data or {})
    SetNuiFocus(true, true)
end

function m.Close()
    if not isOpen then return end
    
    isOpen = false
    UI.Send("MonModule:Close", {})
    SetNuiFocus(false, false)
end

function m.Update(data)
    UI.Send("MonModule:Update", data)
end

UI.OnCb('MonModule:Closed', function(data, cb)
    m.Close()
    cb({ success = true })
end)

-- Exemple : ouvrir avec une commande
RegisterCommand('monmodule', function()
    m.Open({ message = "Hello World!" })
end, false)
```

---

## Exemples pratiques

### Exemple 1 : Interface d'inventaire simplifiée

```javascript
// JavaScript
const Inventory = {
    open(data) {
        document.getElementById('inventory').style.display = 'grid';
        this.renderItems(data.items);
    },
    
    renderItems(items) {
        const container = document.getElementById('inventory-items');
        container.innerHTML = '';
        
        items.forEach(item => {
            const slot = document.createElement('div');
            slot.className = 'inventory-slot';
            slot.innerHTML = `
                <img src="${item.imageUrl}" alt="${item.label}">
                <span class="item-count">${item.amount}</span>
                <span class="item-label">${item.label}</span>
            `;
            slot.onclick = () => this.useItem(item);
            container.appendChild(slot);
        });
    },
    
    async useItem(item) {
        await fetch(`https://${GetParentResourceName()}/Inventory:UseItem`, {
            method: 'POST',
            body: JSON.stringify({ itemId: item.id })
        });
    }
};
```

```lua
-- Lua
m:OnNet("openInventory", function(items)
    UI.Send("Inventory:Open", { items = items })
    SetNuiFocus(true, true)
end)

UI.OnCb('Inventory:UseItem', function(data, cb)
    m:EmitServer("useItem", data.itemId)
    cb({ success = true })
end)
```

### Exemple 2 : Notification toast

```javascript
// JavaScript - Système de notifications
const Notify = {
    container: document.getElementById('notifications'),
    
    show(message, type = 'info', duration = 5000) {
        const toast = document.createElement('div');
        toast.className = `toast toast-${type}`;
        toast.innerHTML = `
            <i class="${this.getIcon(type)}"></i>
            <span>${message}</span>
        `;
        
        this.container.appendChild(toast);
        
        // Animation d'entrée
        setTimeout(() => toast.classList.add('show'), 10);
        
        // Suppression après durée
        setTimeout(() => {
            toast.classList.remove('show');
            setTimeout(() => toast.remove(), 300);
        }, duration);
    },
    
    getIcon(type) {
        const icons = {
            success: 'fas fa-check-circle',
            error: 'fas fa-times-circle',
            warning: 'fas fa-exclamation-triangle',
            info: 'fas fa-info-circle'
        };
        return icons[type] || icons.info;
    }
};

// Écouter les messages
window.addEventListener('message', (event) => {
    if (event.data.type === 'Notify:Create') {
        const { msg, color, timer } = event.data;
        Notify.show(msg, color, timer);
    }
});
```

---

## Bonnes pratiques

### 1. Toujours gérer la touche Échap

```javascript
document.addEventListener('keydown', (e) => {
    if (e.key === 'Escape') {
        // Fermer l'interface active
        closeCurrentInterface();
    }
});
```

### 2. Utiliser des CSS Variables pour les thèmes

```css
:root {
    --primary-color: #e94560;
    --background-dark: #1a1a2e;
    --background-light: #16213e;
    --text-color: #ffffff;
    --border-radius: 8px;
}

.panel {
    background: var(--background-dark);
    color: var(--text-color);
    border-radius: var(--border-radius);
}
```

### 3. Optimiser les performances

```javascript
// Utiliser requestAnimationFrame pour les animations
function animate() {
    // Mise à jour de l'interface
    requestAnimationFrame(animate);
}

// Débouncer les événements fréquents
function debounce(func, wait) {
    let timeout;
    return function(...args) {
        clearTimeout(timeout);
        timeout = setTimeout(() => func.apply(this, args), wait);
    };
}
```

### 4. Gérer les erreurs réseau

```javascript
async function sendToLua(action, data) {
    try {
        const response = await fetch(`https://${GetParentResourceName()}/${action}`, {
            method: 'POST',
            body: JSON.stringify(data)
        });
        return await response.json();
    } catch (error) {
        console.error('Erreur communication Lua:', error);
        return { success: false, error: error.message };
    }
}
```

### 5. Structure modulaire JavaScript

```javascript
// Module pattern
const MyModule = (function() {
    // Variables privées
    let _isOpen = false;
    let _data = null;
    
    // Méthodes privées
    function _render() { /* ... */ }
    
    // API publique
    return {
        init() { /* ... */ },
        open(data) { /* ... */ },
        close() { /* ... */ }
    };
})();
```

---

## Voir aussi

- [Système de modules](../modules/README.md)
- [Système d'événements](../events/README.md)
- [Classes principales](../classes/README.md)

