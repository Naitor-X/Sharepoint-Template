# SharePoint JavaScript Deployment System

## 🚀 Quick Start - Copy & Paste Ready

### Skript-Editor Template (Funktionierendes Pattern!)
```html
<div id="custom-app-container"></div>
<script>
// ===========================================
// KONFIGURATION (Hier anpassen!)
// ===========================================
var customListPath = ""; // Listen-Name oder vollständige SharePoint-URL
var scriptUrl = "/sites/apps/your-app.html"; // Pfad zur HTML-App-Datei

// Funktion zum Extrahieren des Listen-Namens aus SharePoint-URLs
function extractListNameFromUrl(input) {
    if (!input || input.trim() === "") return "";
    
    const trimmedInput = input.trim();
    
    if (trimmedInput.toLowerCase().includes('http')) {
        try {
            const url = new URL(trimmedInput);
            const urlPattern = /\/[Ll]ists\/([^\/]+)/;
            const match = url.pathname.match(urlPattern);
            
            if (match && match[1]) {
                const listName = match[1];
                // Cross-Site-Unterstützung
                const pathParts = url.pathname.split('/Lists/')[0];
                if (pathParts && pathParts !== '' && !pathParts.startsWith('/_')) {
                    const fullSiteUrl = url.origin + pathParts;
                    window.customAppSiteUrl = fullSiteUrl;
                    console.log('Cross-Site-Zugriff aktiviert. Site-URL:', fullSiteUrl);
                }
                return listName;
            }
        } catch (error) {
            console.error('Fehler beim URL-Parsing:', error);
        }
    } else {
        return trimmedInput; // Direkter Listen-Name
    }
    return "";
}

// App laden und konfigurieren
fetch(scriptUrl)
    .then(response => {
        if (!response.ok) throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        return response.text();
    })
    .then(html => {
        document.getElementById('custom-app-container').innerHTML = html;
        
        // Konfiguration setzen (VOR Script-Ausführung!)
        if (customListPath && customListPath.trim() !== "") {
            const extractedListName = extractListNameFromUrl(customListPath);
            if (extractedListName) {
                window.customAppListPath = extractedListName;
                console.log('Verwende Listen-Name:', extractedListName);
            }
        }
        
        // JavaScript aus geladenem HTML ausführen
        var scripts = document.getElementById('custom-app-container').getElementsByTagName('script');
        for (var i = 0; i < scripts.length; i++) {
            eval(scripts[i].innerHTML);
        }
    })
    .catch(error => {
        console.error('App Load Error:', error);
        document.getElementById('custom-app-container').innerHTML = `
            <div style="padding:15px;background:#fee;border:1px solid #fcc;border-radius:5px;color:#c00;">
                <strong>Fehler beim Laden:</strong> ${error.message}<br>
                <small>App-URL: ${scriptUrl}</small>
            </div>
        `;
    });
</script>
```

---

## ⚡ Sofort-Deployment in 3 Schritten

### Schritt 1: HTML-App hochladen
```
SharePoint → Dokumentenbibliothek → Datei hochladen
Beispiel-Pfad: /sites/apps/meine-app.html
```

### Schritt 2: Konfiguration anpassen
```javascript
var customListPath = "meine_liste"; // Ihre Liste
var scriptUrl = "/sites/apps/meine-app.html"; // Ihre App
```

### Schritt 3: In Skript-Editor einfügen
```
SharePoint-Seite → Bearbeiten → Webpart einfügen → Skript-Editor → Code einfügen
```

---

## 🔧 App-Entwicklung (HTML-Datei)

### Template für neue Apps
```html
<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <title>Meine App</title>
    <style>
        /* CSS hier */
    </style>
</head>
<body>
    <div id="app-content">
        <!-- HTML hier -->
    </div>
    
    <script>
        // Konfiguration aus Skript-Editor übernehmen
        const config = {
            listPath: window.customAppListPath || 'Standard_Liste',
            siteUrl: window.customAppSiteUrl || (typeof _spPageContextInfo !== "undefined" ? _spPageContextInfo.webAbsoluteUrl : ''),
            settings: window.customAppSettings || {}
        };
        
        // SharePoint REST API URL erstellen
        function getItemsUrl() {
            // Cross-Site URL-Parsing
            if (config.listPath.startsWith('http')) {
                const siteMatch = config.listPath.match(/https?:\/\/[^\/]+\/[^\/]+/);
                const listMatch = config.listPath.match(/Lists\/([^\/]+)/);
                if (siteMatch && listMatch) {
                    return `${siteMatch[0]}/_api/web/lists/getbytitle('${listMatch[1]}')/items`;
                }
            }
            
            // Standard: Lokale oder Cross-Site Liste
            const baseUrl = config.siteUrl || (typeof _spPageContextInfo !== "undefined" ? _spPageContextInfo.webAbsoluteUrl : '');
            return `${baseUrl}/_api/web/lists/GetByTitle('${config.listPath}')/items`;
        }
        
        // SharePoint Daten laden
        async function loadData() {
            try {
                const response = await fetch(getItemsUrl(), {
                    method: "GET",
                    credentials: "same-origin",
                    headers: {
                        "Accept": "application/json;odata=verbose"
                    }
                });
                
                if (!response.ok) throw new Error(`HTTP ${response.status}: ${response.statusText}`);
                
                const data = await response.json();
                const items = data.d && data.d.results ? data.d.results : [];
                
                // Daten verarbeiten
                renderData(items);
                
            } catch (err) {
                console.error('Fehler beim Laden:', err);
                document.getElementById('app-content').innerHTML = `
                    <div style="color:#c00;">
                        Fehler: ${err.message}<br>
                        <small>Liste: ${config.listPath} | Site: ${config.siteUrl || 'Aktuelle Site'}</small>
                    </div>
                `;
            }
        }
        
        function renderData(items) {
            // Ihre Daten-Rendering-Logik hier
            document.getElementById('app-content').innerHTML = 
                `<div>Geladene Items: ${items.length}</div>`;
        }
        
        // App initialisieren
        if (typeof _spPageContextInfo !== 'undefined') {
            loadData();
        } else {
            setTimeout(() => {
                if (typeof _spPageContextInfo !== 'undefined') {
                    loadData();
                } else {
                    document.getElementById('app-content').innerHTML = 
                        '<div style="color:#c00;">SharePoint-Kontext nicht verfügbar</div>';
                }
            }, 1000);
        }
    </script>
</body>
</html>
```

---

## 📋 Konfigurationsoptionen

### Listen-Konfiguration
```javascript
// Lokale Liste (gleiche Site)
var customListPath = "Meine_Aufgaben";

// Cross-Site mit vollständiger URL (automatische Erkennung)
var customListPath = "https://vorarlberg.polizei.intra.gv.at/fbinfo/lka/OSEneu/Lists/zz_config_startseite_top/AllItems.aspx";

// Cross-Site mit separater Site-URL
window.customAppSiteUrl = "https://site.com/andere-abteilung";
var customListPath = "Projekte";
```

### Erweiterte Einstellungen
```javascript
window.customAppSettings = {
    itemsPerPage: 10,
    theme: 'blue',
    showImages: true,
    filterOptions: ['Active', 'Completed']
};
```

---

## 🛠️ Entwickler-Hinweise

### Warum dieses Pattern funktioniert:
1. **Container zuerst**: `<div id="container"></div>` wird erstellt
2. **HTML laden**: Komplette App wird per `fetch()` geladen
3. **Konfiguration setzen**: Window-Variablen **vor** Script-Ausführung
4. **Script ausführen**: Manuell mit `eval()` nach dem Laden

### Häufige Fehler:
❌ **Falsch**: Script ausführen bevor Konfiguration gesetzt ist
❌ **Falsch**: Direkte DOM-Injection ohne Script-Ausführung
❌ **Falsch**: Timing-Probleme bei der Konfiguration

✅ **Richtig**: Reihenfolge → Laden → Konfigurieren → Ausführen

### SharePoint REST API Standards
```javascript
// Basis-Konfiguration
const baseConfig = {
    credentials: 'same-origin',
    headers: { 
        'Accept': 'application/json;odata=verbose', 
        'Content-Type': 'application/json;odata=verbose' 
    }
};

// Token für POST/MERGE/DELETE
const getToken = async () => {
    const res = await fetch('/_api/contextinfo', { method: 'POST', credentials: 'same-origin' });
    return (await res.json()).d.GetContextWebInformation.FormDigestValue;
};
```

### Design System Variablen
```css
:root {
    --primary-blue: #104166; 
    --primary-blue-light: #16527c;
    --bg-light: #f4f8fb; 
    --text-dark: #333; 
    --text-white: #fff;
}

.webpart {
    padding: 15px; 
    border-radius: 10px; 
    box-shadow: 0 4px 8px rgba(0,0,0,0.15);
    background: var(--bg-light);
}
```

---

## 📁 Projekt-Struktur

```
SharePoint-Template/
├── deployment.md           ← Dieses Dokument
├── CLAUDE.md              ← Projekt-Konfiguration
├── Ansprechpersonen/      ← Funktionierendes Beispiel
│   ├── index.html         ← HTML-App
│   └── Skript-Editor      ← Skript-Editor-Code
├── header-card-menu_v1.html ← Standard Design
├── header_card-menu_v2.html ← Glasmorphism
├── header-card-menu_v3.html ← Minimal Grid
└── Skript-Editor-Fixed.txt  ← Korrekte Skript-Editor-Codes
```

---

## 🔍 Troubleshooting

### Häufige Probleme:
```javascript
// Problem: App lädt nicht
// Lösung: scriptUrl in Browser-Konsole testen
console.log('Testing URL:', scriptUrl);

// Problem: Liste nicht gefunden  
// Lösung: API-URL in Konsole prüfen
console.log('API URL:', getItemsUrl());

// Problem: Berechtigungen
// Lösung: SharePoint-Berechtigung zur Dokumentenbibliothek prüfen
```

### Debug-Modus aktivieren:
```javascript
// In Skript-Editor hinzufügen für Debugging
window.debugMode = true;
```

---

**🎯 Dieses Dokument für schnelle Copy & Paste Deployments optimiert!**