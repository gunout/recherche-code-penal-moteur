# ⚖️ Recherche avancée — Code pénal français

Moteur de recherche avancé, entièrement côté client, pour explorer le Code pénal français à partir des données ouvertes de la DILA. Interface sobre, accessible et rapide, avec syntaxe booléenne, recherche floue, facettes dynamiques, favoris, historique et export.

> ⚠️ **Outil non officiel** — Ce projet est pédagogique et n'est affilié ni à l'État français, ni à Légifrance, ni à la DILA.

---

## 📑 Sommaire

- [Aperçu](#-aperçu)
- [Fonctionnalités](#-fonctionnalités)
- [Démo](#-démo)
- [Installation](#-installation)
- [Utilisation](#-utilisation)
- [Syntaxe de recherche](#-syntaxe-de-recherche)
- [Raccourcis clavier](#️-raccourcis-clavier)
- [Architecture technique](#️-architecture-technique)
- [Données](#-données)
- [Structure du projet](#-structure-du-projet)
- [Accessibilité](#-accessibilité)
- [Compatibilité](#-compatibilité)
- [Limitations connues](#️-limitations-connues)
- [Feuille de route](#️-feuille-de-route)
- [Contribuer](#-contribuer)
- [Licence](#-licence)
- [Remerciements](#-remerciements)

---

## 🔎 Aperçu

Un fichier HTML unique, sans build ni dépendance, qui :

- Récupère l'intégralité des articles du Code pénal depuis le dataset Hugging Face [`louisbrulenaudet/code-penal`](https://huggingface.co/datasets/louisbrulenaudet/code-penal) (source DILA / Légifrance).
- Les indexe en mémoire dans le navigateur.
- Offre une recherche BM25-like avec opérateurs booléens, recherche de phrases, préfixes, exclusion de termes et recherche par numéro d'article.
- Affiche les résultats avec snippets contextuels, mise en évidence des termes, et filtres à facettes.

**Aucun serveur, aucune base de données, aucune inscription.**

---

## ✨ Fonctionnalités

### 🔍 Recherche

- Syntaxe booléenne : `AND`, `OR`, `NOT`, `-exclusion`, `"phrase exacte"`, `préfixe*`
- Recherche par numéro : `art:311-1`, `num:311-1`, `article:311-1`
- Recherche floue (fallback automatique) via distance de Levenshtein ≤ 2
- Scoring BM25 (`k1 = 1.2`, `b = 0.75`) avec pondération titre / hiérarchie / numéro
- Snippets intelligents : fenêtre glissante centrée sur les termes les plus pertinents
- Surlignage insensible aux accents et à la casse
- Autocomplétion : numéros d'articles + mots-clés fréquents

### 🗂️ Facettes dynamiques

- Filtrer par **État** (Vigueur / Abrogé)
- Filtrer par **Livre**
- Filtrer par **Titre**
- Compteurs recalculés en temps réel

### 🛠️ Outils

- Tri : pertinence, numéro d'article, date (asc/desc), longueur
- Pagination intelligente (20 résultats par page)
- Favoris persistants (`localStorage`)
- Historique des 15 dernières recherches (`localStorage`)
- Export CSV (UTF-8 BOM, séparateur `;`, compatible Excel FR)
- Copie d'un article ou de la page courante
- Partage par URL (état encodé dans le query string)
- Impression avec feuille de style dédiée

### ♿ Accessibilité

- Lien d'évitement (skip link)
- Attributs ARIA (`role`, `aria-live`, `aria-expanded`, `aria-controls`…)
- Navigation clavier complète (flèches, Entrée, Échap)
- Contrastes conformes WCAG AA
- Focus visibles personnalisés

---

## 🚀 Démo

Aucune démo en ligne officielle n'est maintenue. Pour tester localement, ouvrez simplement `moteur.html` dans un navigateur moderne.

> 💡 **Astuce GitHub Pages** : poussez le fichier `index.html` (renommage de `moteur.html`) à la racine du dépôt, activez Pages dans **Settings → Pages**, et votre moteur sera accessible à l'URL `https://<utilisateur>.github.io/<repo>/`.

---

## 📦 Installation

### Option 1 — Utilisation directe (recommandée)

```bash
git clone https://github.com/<votre-compte>/<votre-repo>.git
cd <votre-repo>
# Ouvrez moteur.html dans votre navigateur
```

Aucune dépendance, aucun `npm install`, aucun build.

### Option 2 — Serveur local (pour éviter les restrictions CORS)

Certains navigateurs restreignent `fetch` depuis `file://`. Un petit serveur local règle le problème :

```bash
# Python 3
python -m http.server 8000

# ou Node.js (via npx)
npx serve .
```

Puis ouvrez `http://localhost:8000/moteur.html`.

### Option 3 — GitHub Pages

1. Renommez `moteur.html` en `index.html`
2. Committez et poussez
3. Activez **Settings → Pages → Source : main / root**
4. C'est en ligne 🎉

---

## 🧭 Utilisation

1. **Lancement** : la page charge automatiquement tous les articles (~2 000+) — un indicateur de progression s'affiche.
2. **Recherche** : tapez votre requête dans la barre principale, puis `Entrée`.
3. **Affinage** : utilisez les cases à cocher de la barre latérale (facettes).
4. **Tri & pagination** : sélectionnez le tri dans la barre d'outils, naviguez page par page.
5. **Favoris** : cliquez sur ☆ pour sauvegarder un article, retrouvez-le dans la barre latérale.
6. **Export** : bouton 📊 CSV pour exporter tous les résultats filtrés.

---

## 📖 Syntaxe de recherche

| Syntaxe | Signification | Exemple |
|---|---|---|
| `mot1 mot2` | Les deux mots (AND implicite) | `vol arme` |
| `"phrase exacte"` | Expression littérale | `"vol avec violence"` |
| `mot1 AND mot2` | Les deux obligatoires | `vol AND arme` |
| `mot1 OR mot2` | L'un ou l'autre | `vol OR extorsion` |
| `-mot` ou `NOT mot` | Exclusion | `vol -mineur` |
| `préfixe*` | Commence par | `cambrio*` |
| `art:numéro` | Article précis | `art:311-1` |

### Exemples combinés

```text
"vol avec violence" AND -mineur
corruption OR détournement OR favoritisme
art:222-1
cambrio* escroquerie
```

### Recherche floue

Si aucun résultat exact n'est trouvé pour une requête composée de mots obligatoires, le moteur tente une correction orthographique automatique (Levenshtein ≤ 2 selon la longueur). Un toast vous prévient de la correction appliquée.

---

## ⌨️ Raccourcis clavier

| Raccourci | Action |
|---|---|
| <kbd>/</kbd> | Focus sur la barre de recherche |
| <kbd>?</kbd> | Ouvrir l'aide sur la syntaxe |
| <kbd>Entrée</kbd> | Lancer la recherche |
| <kbd>Échap</kbd> | Fermer les suggestions ou la modale |
| <kbd>Ctrl</kbd> + <kbd>K</kbd> | Effacer la recherche |
| <kbd>↑</kbd> / <kbd>↓</kbd> | Naviguer dans l'autocomplétion |

---

## 🏗️ Architecture technique

### Stack

- **HTML/CSS/JS vanilla** — aucune dépendance externe
- **CSS moderne** : variables, grid, flexbox, `backdrop-filter`
- **JS ES2019+** : `Set`, `Map`, `async/await`, `fetch`, `IntersectionObserver` (non utilisé ici mais supporté)

### Pipeline de recherche

```text
Requête utilisateur
      │
      ▼
┌─────────────────┐
│  Tokenizer      │  → opérateurs (AND/OR/NOT), phrases, termes
└────────┬────────┘
         ▼
┌─────────────────┐
│  Parser         │  → { must, mustNot, should, phrases, prefix, numero }
└────────┬────────┘
         ▼
┌─────────────────┐
│  Scoring BM25   │  → filtré sur facettes, pondéré titre/numéro
└────────┬────────┘
         ▼
┌─────────────────┐
│ Fallback flou   │  → si 0 résultat et must non vide
└────────┬────────┘
         ▼
┌─────────────────┐
│ Tri + Pagination│
└────────┬────────┘
         ▼
      Rendu
```

### Stockage local

| Clé | Contenu |
|---|---|
| `cp_history_v1` | 15 dernières requêtes |
| `cp_favoris_v1` | Numéros d'articles en favoris |
| `cp_facets_v1` | (réservé, non utilisé actuellement) |

### Synchronisation URL

Toute la session est encodée dans le query string :

```text
?q=vol+arme&etat=VIGUEUR&livre=Livre+III&tri=numero&p=2
```

Rechargez ou partagez le lien : l'état est restauré automatiquement.

---

## 📊 Données

- **Source** : Hugging Face — [`louisbrulenaudet/code-penal`](https://huggingface.co/datasets/louisbrulenaudet/code-penal)
- **Origine** : DILA (Direction de l'information légale et administrative)
- **Licence** : Licence Ouverte 2.0 (Etalab)
- **API** : `https://datasets-server.huggingface.co/rows`
- **Split** : `train`
- **Pagination** : par lots de 100 lignes

### Champs extraits

| Champ | Description |
|---|---|
| `num` | Numéro d'article (ex. `311-1`) |
| `texte` | Texte intégral de l'article |
| `title_main` | Intitulé principal |
| `etat` | `VIGUEUR` ou `ABROGE` |
| `sectionParentTitre` | Hiérarchie complète |
| `dateDebut` | Timestamp Unix de mise en vigueur |
| `ref` | Référence Légifrance |

---

## 📁 Structure du projet

```text
.
├── moteur.html      # Application complète (HTML + CSS + JS inline)
├── README.md        # Ce fichier
└── LICENSE          # MIT
```

L'application est volontairement monolithique pour faciliter le déploiement et l'audit.

---

## ♿ Accessibilité

Ce projet vise la conformité **WCAG 2.1 niveau AA** :

- Structure sémantique HTML5 (`<header>`, `<main>`, `<nav>`, `<article>`)
- Régions `aria-live` pour les mises à jour dynamiques
- Lien d'évitement « Aller au contenu principal »
- Tous les contrôles accessibles au clavier
- Contraste texte/fond ≥ 4.5:1
- Focus visibles et personnalisés (`:focus-visible`)
- Formulaires étiquetés (`aria-label`, `<label>`)

Les retours et signalements de problèmes d'accessibilité sont bienvenus via les issues.

---

## 🌐 Compatibilité

| Navigateur | Version minimale |
|---|---|
| Chrome / Edge | 90+ |
| Firefox | 88+ |
| Safari | 14+ |
| Opera | 76+ |

**Non supporté** : Internet Explorer (toutes versions).

Fonctionne sur mobile et tablette (design responsive, breakpoint à 768 px et 1024 px).

---

## ⚠️ Limitations connues

- **Recherche floue** : uniquement en fallback, jamais sur les requêtes composées complexes.
- **IDF non calculé** : tous les termes rares comptent de la même manière.
- **Longueur moyenne fixée à 80 mots** (approximation).
- **Pas d'index inversé** : la recherche est linéaire (O(n)), acceptable pour ~2 000 articles, insuffisant au-delà.
- **Dépendance à l'API Hugging Face** : si elle est indisponible, l'app ne charge rien.
- **Données en mémoire** : ~2 000 articles × ~1 Ko = ~2 Mo (OK), mais pas adapté à des corpus massifs.
- **Pas de cache** : les données sont retéléchargées à chaque rechargement (pas de service worker).

---

## 🗺️ Feuille de route

- [ ] Service worker + cache offline (PWA)
- [ ] Index inversé + web worker pour la recherche
- [ ] Comparaison de versions d'articles (diffs)
- [ ] Export PDF
- [ ] Thème sombre
- [ ] Recherche dans la jurisprudence associée
- [ ] Internationalisation (EN, ES)
- [ ] Tests unitaires (Vitest / Playwright)

---

## 🤝 Contribuer

Les contributions sont les bienvenues !

1. Forkez le dépôt
2. Créez une branche : `git checkout -b feature/ma-fonctionnalite`
3. Committez : `git commit -m "feat: ajout de X"`
4. Poussez : `git push origin feature/ma-fonctionnalite`
5. Ouvrez une Pull Request

### Conventions de commit

Ce projet suit [Conventional Commits](https://www.conventionalcommits.org/) :

- `feat:` nouvelle fonctionnalité
- `fix:` correction de bug
- `docs:` documentation
- `style:` formatage
- `refactor:` refactoring
- `perf:` performance
- `test:` tests

### Signaler un bug

Ouvrez une issue en précisant :

- Navigateur et version
- Étapes de reproduction
- Comportement attendu vs observé
- Captures d'écran si pertinent

---

## 📄 Licence

Ce projet est distribué sous licence **MIT** — voir le fichier [LICENSE](LICENSE) pour plus de détails.

```text
MIT License

Copyright (c) 2025 <votre nom>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 🙏 Remerciements

- **DILA** — production et diffusion des données juridiques
- **Légifrance** — source officielle du Code pénal
- **data.gouv.fr** — plateforme d'open data
- **Hugging Face** — hébergement du dataset et API `rows`
- **@louisbrulenaudet** — mise à disposition du dataset `code-penal`
- **Etalab** — Licence Ouverte 2.0

---


<div align="center">

⚖️ **Outil pédagogique non officiel** — Non affilié à l'État français

Données sous Licence Ouverte 2.0 (DILA)

Fait pour la communauté juridique open source.

</div>

---

<div align="center">

### 🇫🇷 Gunout · 2026

![Made in France](https://img.shields.io/badge/Made_in-France-002395?style=flat-square&labelColor=FFFFFF&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCA5MDAgNjAwIj48cmVjdCB3aWR0aD0iOTAwIiBoZWlnaHQ9IjYwMCIgZmlsbD0iIzAwMjM5NSIvPjxyZWN0IHdpZHRoPSI5MDAiIGhlaWdodD0iNDAwIiB5PSIxMDAiIGZpbGw9IiNmZmYiLz48cmVjdCB3aWR0aD0iOTAwIiBoZWlnaHQ9IjIwMCIgeT0iNDAwIiBmaWxsPSIjZWQyOTM5Ii8+PC9zdmc+)
![GitHub](https://img.shields.io/badge/GitHub-gunout-181717?style=flat-square&logo=github&logoColor=white)
![Year](https://img.shields.io/badge/2026-ED2939?style=flat-square&labelColor=FFFFFF)

<sub>© 2026 <strong>gunout</strong> — Tous droits réservés.</sub>

</div>
