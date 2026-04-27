# CLAUDE.md — Performance Manager
## Contexte pour Claude Code

---

## PROJET
Dashboard athlétique premium pour Julie Bastière, Performance Manager indépendante (lancement juin 2026).
Stack : HTML/CSS/JS vanilla + Chart.js · Notion API · Cloudflare Workers · GitHub Pages

---

## INFRASTRUCTURE

### GitHub
- Repo : `bastierejulie-maker/performance-manager` (public)
- Branch principale : `main`
- Déploiement : GitHub Pages automatique → `bastierejulie-maker.github.io/performance-manager`

### Notion
- Workspace : 🏟️ Performance Manager
- Intégration : `Performance Manager` (ID: `34e69796-db3f-8185-ae71-0027cc4a12d3`)
- Proxy : `notion-proxy.bastierejulie.workers.dev` (Cloudflare Worker)
  - Miroir direct de l'API Notion v1
  - Clé stockée en secret `NOTION_KEY` dans Cloudflare
  - CORS ouvert pour GitHub Pages

### Bases de données Notion
| Base | ID |
|---|---|
| Joueurs | `34b69796-db3f-8002-9344-eba7a2c4dbe4` |
| Stats Match | `34d69796-db3f-801c-aed5-d8cfa048dbba` |
| Tests Athlétiques | `34d69796-db3f-8047-b6f9-e361371604a9` |

---

## FICHIERS DU REPO

```
performance-manager/
├── index.html          # Hub de navigation
├── athletique.html     # Dashboard performance physique
├── stats-match.html    # Analyse match avec filtres
├── nutrition.html      # Suivi nutrition
└── CLAUDE.md          # Ce fichier
```

---

## CONVENTIONS DE CODE

### Thème
```css
:root {
  --bg:     #F4F5F7;   /* fond page */
  --bg2:    #ECEEF2;   /* fond secondaire */
  --card:   #FFFFFF;   /* cartes */
  --border: rgba(0,0,0,.08);
  --text:   #1A1D23;
  --muted:  #6B7280;
  --high:   #0F6E56;   /* vert performance */
  --mid:    #BA7517;   /* orange moyen */
  --low:    #A32D2D;   /* rouge faible */
}
```

### Couleurs clubs (dynamiques selon joueur)
```js
const CLUBS = {
  'PSG':       { bg:'#003D83', accent:'#DA291C', avatar:'#001F5B' },
  'Barcelone': { bg:'#A50044', accent:'#004D98', avatar:'#8B0038' },
  'Real Madrid':{ bg:'#FEBE10', accent:'#00529F', avatar:'#D4A800' },
  'OM':        { bg:'#009FC3', accent:'#26356B', avatar:'#007FA0' },
  'Lyon':      { bg:'#1A1A2E', accent:'#E4002B', avatar:'#111' },
};
```

### Appel Notion via proxy
```js
const PROXY = 'https://notion-proxy.bastierejulie.workers.dev';

async function notionQuery(dbId, filter) {
  const r = await fetch(`${PROXY}/v1/databases/${dbId}/query`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(
      filter && Object.keys(filter).length
        ? { filter, page_size: 100 }
        : { page_size: 100 }
    )
  });
  if (!r.ok) throw new Error('Proxy ' + r.status);
  return (await r.json()).results || [];
}
```

### Propriétés Notion — extraction
```js
function getProp(page, name) {
  const p = page.properties?.[name];
  if (!p) return null;
  if (p.title)    return p.title.map(t => t.plain_text).join('');
  if (p.rich_text)return p.rich_text.map(t => t.plain_text).join('');
  if (p.select)   return p.select?.name || null;
  if (p.number !== undefined) return p.number;
  if (p.url)      return p.url;
  if (p.date)     return p.date?.start || null;
  if (p.files)    return p.files?.[0]?.file?.url || p.files?.[0]?.external?.url || null;
  return null;
}
```

---

## NOMS DE COLONNES NOTION (exacts)

### Base Stats Match
- `Adversaire`, `Date`, `Saison`, `Résultats`, `Compétition`
- `Titulaire/remplaçant` (select: Titulaire / Remplaçant)
- `⏱️ Temps de jeu` (number, minutes)
- `But(s) marqué(s)`, `Passe décissive`
- `Tirs`, `Tirs cadrés`, `Duels`, `Duels gagnés`
- `Distance parcourue (m)`, `Distance en sprint (m)`, `Nombre de sprint`
- `Vmax` (km/h), `Accmax` (m/s²)
- `Note joueur`, `Rapport GPS`, `Rapport match`

### Base Tests Athlétiques
- `CMJ (cm)`, `SJ (cm)`, `VMA (km/h)`
- `Date`, `Saison`

### Base Joueurs
- `Nom complet` (title), `Photo` (url), `Club actuel` (rich_text)
- `Poste` (select), `Statut` (select), `Actif/Inactif` (select)
- `Nationalité sportive`, `Agence`, `Pied fort`, `date de naissance`

---

## NORMES PAR POSTE (référence élite)
| Poste | Vmax | AccMax | CMJ | SJ | VMA |
|---|---|---|---|---|---|
| Excentré | 35.0 | 7.5 | 48 | 44 | 18 |
| Attaquant | 34.0 | 7.0 | 46 | 42 | 18 |
| Milieu offensif | 32.5 | 6.5 | 44 | 40 | 18 |
| Milieu défensif | 31.5 | 6.5 | 43 | 39 | 18 |
| Latéral | 33.0 | 7.0 | 45 | 41 | 18 |
| Défenseur central | 31.0 | 6.0 | 46 | 42 | 18 |
| Gardien | 28.0 | 6.5 | 55 | 50 | 14 |

VMA : 18 = bon · 20+ = excellent (sauf gardien)

---

## RÈGLES IMPORTANTES
1. **Jamais de fond sombre** — thème clair uniquement
2. **Pas de localStorage** — données en mémoire JS uniquement
3. **Toujours un fallback démo** si le proxy échoue
4. **Filtre vide interdit** dans les requêtes Notion — utiliser `{}` conditionnel
5. **IDs Notion** : format avec tirets `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
6. **Chart.js 4.4.1** depuis cdnjs.cloudflare.com
7. **Fonts** : Barlow + Barlow Condensed (Google Fonts)
8. **Couleurs** dynamiques selon le club du joueur

---

## DÉPLOIEMENT
Push sur `main` → GitHub Pages se met à jour automatiquement (~30 secondes)

## MAKE (automatisation)
Scénario Notion → GitHub : webhook Notion déclenche un rebuild quand une donnée change
Config : voir fichier `make_scenarios.md`
