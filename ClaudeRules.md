# SharePoint 2019 Development Ruleset

## Projekt-Setup
- **Stack:** Nur JavaScript ES6+, HTML, CSS - keine Frameworks/Libraries
- **Browser:** Microsoft Edge (IE-Support nicht erforderlich)
- **API:** SharePoint 2019 REST API mit fetch()
- **Deployment:** Siehe @deployment.md

## REST API Essentials

### Standard Headers & Token
```javascript
const baseConfig = {
    credentials: 'same-origin',
    headers: { 'Accept': 'application/json;odata=verbose', 'Content-Type': 'application/json;odata=verbose' }
};

const getToken = async () => {
    const res = await fetch('/_api/contextinfo', { method: 'POST', credentials: 'same-origin' });
    return (await res.json()).d.GetContextWebInformation.FormDigestValue;
};
```

### CRUD Headers
- **POST:** `X-RequestDigest: token`
- **MERGE:** `X-RequestDigest: token, IF-MATCH: *, X-HTTP-Method: MERGE`
- **DELETE:** `X-RequestDigest: token, IF-MATCH: *, X-HTTP-Method: DELETE`

### Batch-Requests (bei 3+ API-Calls)
```javascript
const batchRequest = async (requests) => {
    const batchId = `batch_${Date.now()}`;
    const token = await getToken();
    
    let body = `--${batchId}\r\nContent-Type: multipart/mixed; boundary="changeset_${Date.now()}"\r\n\r\n`;
    requests.forEach(req => {
        body += `--changeset_${Date.now()}\r\nContent-Type: application/http\r\n\r\n`;
        body += `${req.method} ${req.url} HTTP/1.1\r\nAccept: application/json;odata=verbose\r\n`;
        if (req.method !== 'GET') body += `X-RequestDigest: ${token}\r\n`;
        body += `\r\n${req.body ? JSON.stringify(req.body) : ''}\r\n\r\n`;
    });
    body += `--changeset_${Date.now()}--\r\n--${batchId}--`;
    
    return fetch('/_api/$batch', {
        method: 'POST', credentials: 'same-origin',
        headers: { 'Content-Type': `multipart/mixed; boundary="${batchId}"`, 'X-RequestDigest': token },
        body
    });
};
```

## Design System

### CSS Variables
```css
:root {
    --primary-blue: #104166; --primary-blue-light: #16527c;
    --bg-light: #f4f8fb; --text-dark: #333; --text-white: #fff;
}
```

### Webpart Structure
```css
.webpart {
    padding: 15px; border-radius: 10px; box-shadow: 0 4px 8px rgba(0,0,0,0.15);
    margin-bottom: 15px; background: var(--bg-light);
}
.webpart-title {
    font: bold 16px/1.2 sans-serif; color: var(--text-white); padding: 5px 15px;
    background: linear-gradient(135deg, var(--primary-blue), var(--primary-blue-light));
    border-radius: 8px;
}
```

## Icons & Assets
```javascript
const ICON_BASE = 'https://vorarlberg.polizei.intra.gv.at/PublishingImages/icons/';
const icon = name => `${ICON_BASE}${name.endsWith('.svg') ? name : name + '.svg'}`;
```

## Mandatory Practices

### URLs - Immer dynamisch
```javascript
const baseUrl = _spPageContextInfo.webAbsoluteUrl;
const listUrl = `${baseUrl}/_api/web/lists/getbytitle('${listName}')/items`;
```

### Error Handling
```javascript
const handleError = (error, context) => {
    console.error(`SP Error ${context}:`, error);
    const messages = { 403: 'Keine Berechtigung', 404: 'Nicht gefunden', 500: 'Server Fehler' };
    return messages[error.status] || 'Unbekannter Fehler';
};
```

### Performance
- **Batch:** 3+ API-Calls → Batch-Request
- **DOM:** DocumentFragment für multiple Updates
- **Lists:** $select, $filter, $top für Queries

## Quality Checklist
- [ ] fetch() mit Error Handling
- [ ] Token bei POST/MERGE/DELETE
- [ ] Dynamische URLs mit _spPageContextInfo
- [ ] Batch bei mehreren API-Calls
- [ ] CSS Design System verwendet
- [ ] Icon-Helper verwendet
- [ ] Responsive & Accessible

## Common Patterns

### List Operations
```javascript
// Get Items
const getItems = async (list, odata = '') => {
    const res = await fetch(`${baseUrl}/_api/web/lists/getbytitle('${list}')/items${odata}`, baseConfig);
    return (await res.json()).d.results;
};

// Create Item
const createItem = async (list, data) => {
    const token = await getToken();
    const res = await fetch(`${baseUrl}/_api/web/lists/getbytitle('${list}')/items`, {
        ...baseConfig, method: 'POST',
        headers: { ...baseConfig.headers, 'X-RequestDigest': token },
        body: JSON.stringify({ __metadata: { type: `SP.Data.${list}ListItem` }, ...data })
    });
    return await res.json();
};
```

### Multi-List Dashboard
```javascript
const loadDashboard = async () => {
    const requests = [
        { method: 'GET', url: '/_api/web/lists/getbytitle(\'News\')/items?$top=5' },
        { method: 'GET', url: '/_api/web/lists/getbytitle(\'Tasks\')/items?$filter=Active eq true' }
    ];
    return await batchRequest(requests);
};
```

---
**Deployment:** Kompatibel mit SharePoint Deployment-System via `window.customAppConfig`