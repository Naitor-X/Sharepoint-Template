# SharePoint JavaScript Deployment System

## Zwei-Dateien-Architektur

### 1. Haupt-App (`app.html`)
- Vollständige HTML-Datei mit CSS/JS
- Upload in zentrale Dokumentenbibliothek
- Wiederverwendbar für multiple Seiten

### 2. Skript-Editor-Wrapper
- Lädt Haupt-App via `fetch()`
- Seitenspezifische Konfiguration
- Platziert in SharePoint Skript-Editor-Webpart

## Standard-Implementation

### App-Loader Template
```javascript
// Skript-Editor-Code für jede Seite
window.customAppListPath = "MeineListe";  // Liste oder SharePoint-URL
window.customAppSiteUrl = "https://site.com/subsite";  // Optional für Cross-Site
window.customAppSettings = { theme: "dark", itemsPerPage: 5 };  // App-spezifisch

fetch('/sites/apps/app.html')
  .then(res => res.text())
  .then(html => document.getElementById('app-container').innerHTML = html)
  .catch(err => console.error('App Load Error:', err));
```

### App-Konfiguration (in Haupt-App)
```javascript
// Standard-Konfiguration mit Fallbacks
const config = {
    listPath: window.customAppListPath || 'DefaultList',
    siteUrl: window.customAppSiteUrl || _spPageContextInfo.webAbsoluteUrl,
    settings: window.customAppSettings || {}
};

// Cross-Site URL-Parsing
const getApiUrl = (listName) => {
    if (listName.startsWith('http')) {
        const siteMatch = listName.match(/https?:\/\/[^\/]+\/[^\/]+/);
        const listMatch = listName.match(/Lists\/([^\/]+)/);
        return siteMatch && listMatch ? 
            `${siteMatch[0]}/_api/web/lists/getbytitle('${listMatch[1]}')/items` :
            `${config.siteUrl}/_api/web/lists/getbytitle('${listName}')/items`;
    }
    return `${config.siteUrl}/_api/web/lists/getbytitle('${listName}')/items`;
};
```

## Deployment-Workflow

### Setup
1. **Haupt-App** in `/sites/apps/` oder zentrale Dokumentenbibliothek
2. **Skript-Editor** auf Zielseite mit Konfiguration
3. **Container-Element** für App-Injection

### Verwendung
```html
<!-- In SharePoint-Seite -->
<div id="app-container">Lade App...</div>
<script>
window.customAppListPath = "Aktuelle Liste";
// ... App-Loader Code
</script>
```

## Konfigurationsoptionen

### Listen-Konfiguration
```javascript
// Lokale Liste
window.customAppListPath = "MeineAufgaben";

// Cross-Site Liste via URL
window.customAppListPath = "https://site.com/dept/_layouts/15/start.aspx#/Lists/Aufgaben";

// Cross-Site mit expliziter Site-URL
window.customAppSiteUrl = "https://site.com/andere-abteilung";
window.customAppListPath = "Projekte";
```

### App-Settings
```javascript
window.customAppSettings = {
    itemsPerPage: 10,
    theme: 'blue',
    showImages: true,
    filterOptions: ['Active', 'Completed']
};
```

## Error Handling

### Basis-Validation
```javascript
const validateConfig = () => {
    if (!config.listPath) {
        throw new Error('Konfiguration: customAppListPath erforderlich');
    }
    
    if (!document.getElementById('app-container')) {
        throw new Error('Container-Element #app-container nicht gefunden');
    }
};
```

### User-Friendly Messages
```javascript
const showConfigError = (message) => {
    document.getElementById('app-container').innerHTML = `
        <div style="padding:15px;background:#fee;border:1px solid #fcc;border-radius:5px;">
            <strong>Konfigurationsfehler:</strong> ${message}<br>
            <small>Prüfen Sie die Skript-Editor-Konfiguration</small>
        </div>
    `;
};
```

## Best Practices

### Performance
- App-Caching via Browser-Cache
- Lazy Loading für große Apps
- Minified HTML für Production

### Security
- Validiere alle Konfigurationswerte
- Sanitize User-Inputs
- Verwende SharePoint-Berechtigungen

### Maintenance
- Versionierung in App-Namen (`app-v1.2.html`)
- Fallback für alte Konfigurationen
- Logging für Deployment-Issues

---

**Quick Start:** Kopiere App-Loader Template → Anpassen der Konfigurationsvariablen → Skript-Editor einfügen