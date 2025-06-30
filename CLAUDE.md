# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Projekt-Übersicht

Dies ist ein SharePoint 2019 Template-Repository für die Entwicklung von Webparts und Apps mit:
- **Tech Stack:** Vanilla JavaScript ES6+, HTML, CSS (keine Frameworks)
- **Browser-Ziel:** Microsoft Edge 
- **API:** SharePoint 2019 REST API mit fetch()
- **Deployment:** Zwei-Dateien-System (Haupt-App + Skript-Editor-Wrapper)

## Architektur

### Deployment-Pattern
Das Repository folgt einem zweistufigen Deployment-System:
1. **Haupt-App**: Vollständige HTML-Dateien mit eingebettetem CSS/JS in zentraler Dokumentenbibliothek
2. **Skript-Editor-Wrapper**: Lädt Apps dynamisch via fetch() mit seitenspezifischer Konfiguration

### Konfigurationssystem
Apps werden über globale `window.customApp*` Variablen konfiguriert:
```javascript
window.customAppListPath = "ListenName";
window.customAppSiteUrl = "https://site.com/subsite"; // Optional
window.customAppSettings = { theme: "dark", itemsPerPage: 5 };
```

## SharePoint REST API Standards

### Basis-Konfiguration
```javascript
const baseConfig = {
    credentials: 'same-origin',
    headers: { 'Accept': 'application/json;odata=verbose', 'Content-Type': 'application/json;odata=verbose' }
};
```

### Token-Verwaltung
Für POST/MERGE/DELETE immer X-RequestDigest Token verwenden:
```javascript
const getToken = async () => {
    const res = await fetch('/_api/contextinfo', { method: 'POST', credentials: 'same-origin' });
    return (await res.json()).d.GetContextWebInformation.FormDigestValue;
};
```

### Batch-Requests
Bei 3+ API-Calls Batch-Requests verwenden (siehe ClaudeRules.md für vollständige Implementation)

## Design System

### CSS-Variablen
```css
:root {
    --primary-blue: #104166; 
    --primary-blue-light: #16527c;
    --bg-light: #f4f8fb; 
    --text-dark: #333; 
    --text-white: #fff;
}
```

### Standard Webpart-Struktur
```css
.webpart {
    padding: 15px; 
    border-radius: 10px; 
    box-shadow: 0 4px 8px rgba(0,0,0,0.15);
    background: var(--bg-light);
}
```

## Wichtige Patterns

### URLs
Immer dynamische URLs verwenden:
```javascript
const baseUrl = _spPageContextInfo.webAbsoluteUrl;
```

### Icons
Icon-Helper für einheitliche Asset-Verwaltung:
```javascript
const ICON_BASE = 'https://vorarlberg.polizei.intra.gv.at/PublishingImages/icons/';
const icon = name => `${ICON_BASE}${name.endsWith('.svg') ? name : name + '.svg'}`;
```

### Error Handling
Standardisierte Fehlerbehandlung mit deutschen Meldungen implementieren

## Development Workflow

### Dateierstellung
- HTML-Dateien sind vollständige Apps mit eingebettetem CSS/JS
- Versionierung über Dateinamen (`app-v1.2.html`)
- Responsive Design mit CSS Grid/Flexbox

### Testing
- Manuelles Testing in SharePoint-Umgebung
- Browser-DevTools für Debugging
- Cross-Site-Testing für Multi-Site-Apps

### Quality Checklist
- [ ] fetch() mit Error Handling
- [ ] Token bei POST/MERGE/DELETE  
- [ ] Dynamische URLs mit _spPageContextInfo
- [ ] Batch bei mehreren API-Calls
- [ ] CSS Design System verwendet
- [ ] Icon-Helper verwendet
- [ ] Responsive & Accessible

## Sprache
- Alle Kommentare und Dokumentation auf Deutsch
- UI-Texte und Fehlermeldungen auf Deutsch
- Code-Kommentare sparsam verwenden