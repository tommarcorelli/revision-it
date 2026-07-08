# Révision IT / Cybersécurité

PWA de révision pour l'informatique, la cybersécurité et le SISR : fiches, flashcards, quiz, examen blanc, répétition espacée (Anki SRS), pièges à connaître, fiches liées et un terminal Linux/Windows simulé avec scénarios guidés.

## Structure du projet

```
index.html         Structure de la page, tous les écrans
style.css          Tous les styles (variables CSS, thème clair/sombre, composants)
data.js            Données des fiches (FICHES, catLabels, catOrder)
terminal-data.js   Données du mode Terminal (TERM_SHELLS, TERM_SCENARIOS)
script.js          Logique applicative (état, rendu, événements, tous les modes)
sw.js              Service worker (cache offline, stratégie network-first)
manifest.json      Métadonnées PWA (nom, icônes, couleurs)
logo.svg           Logo de l'application (utilisé dans la nav et pour les icônes)
icons/             Icônes PWA (192, 512, 512-maskable, apple-touch)
```

Ordre de chargement des scripts : `data.js` → `terminal-data.js` → `script.js`.

## Modes disponibles

| Mode | Description |
|---|---|
| 🏠 Accueil | Dashboard : progression, fiches récentes, catégories faibles, actions rapides |
| 📋 Fiches | Grille de toutes les fiches, filtrable par catégorie et recherche |
| 🃏 Flashcards | Cartes à retourner avec recto/verso |
| 🎯 Quiz | QCM avec timer, filtrable par catégorie |
| 🧠 Anki SRS | Répétition espacée algorithme SM-2 |
| 💻 Terminal | Shell simulé (Linux, Cisco, PowerShell, Docker, Proxmox, pfSense, AD) + scénarios guidés |
| 📊 Stats | Progression par catégorie |
| ⚠️ Pièges | Tous les pièges classiques regroupés, filtrables |
| 📝 Examen Blanc | Examen complet multi-catégories avec bilan et fiches à revoir |

## Favoris & Révision du jour

- **Favoris** : étoile ★ sur chaque carte et dans le panneau détail (`toggleFavorite(id)`), stockés dans `revision_favorites`. Bouton **★ Favoris** dans la barre d'outils des fiches pour ne montrer que les favoris.
- **Export PDF multi-fiches** (`exportFilteredToPDF()`, `exportFichesToPDF()`) : bouton 🖨️ Export PDF dans la vue Fiches, exporte exactement ce qui est affiché (recherche, catégorie, favoris, "à revoir" compris) via la boîte de dialogue d'impression du navigateur (→ "Enregistrer en PDF"). Rendu géré par `#print-export` (index.html) et le bloc `@media print` correspondant dans style.css.
- **Signaler une erreur** (`reportFicheIssue(id)`) : bouton 🚩 Signaler dans le panneau détail de chaque fiche. Copie un rapport pré-rempli (id, titre, catégorie, lien direct `#fiche-ID`) dans le presse-papiers puis ouvre le client mail par défaut avec sujet/corps déjà remplis (adresse à compléter). Notification via `showToast()`, un petit système de toast générique réutilisable ailleurs dans l'app.
- **Révision du jour** : action principale de l'accueil (`startDailyReview()`). Construit un lot priorisé de 20 fiches via `buildDailyList()` — score : dues Anki (+100) > faibles niveau ≤2 (+50) > favoris (+20) > jamais vues (+10). Affiché dans la vue Fiches avec une bannière ; `exitDailyReview()` ou le choix d'une catégorie en sort.

## Stats, recherche & accessibilité

- **Stats enrichies** (`renderStatsSummary()`) : série de révision (🔥 streak via `revision_activity`), fiches vues, maîtrisées, record quiz, répartition des niveaux 1–5, et historique des quiz (`revision_quiz_history`, `recordQuizResult()`).
- **Catégories à réviser en priorité** (`renderStats()`) : toutes les catégories triées par % de maîtrise (moyenne des niveaux 0–5, fiches jamais vues comptées à 0), de la plus faible à la plus forte. Chaque ligne est cliquable et lance `startCategoryReview(cat)` : bascule en Flashcards, filtré sur la catégorie, fiches les moins maîtrisées en premier. Un aperçu (top 3) est aussi affiché sur l'Accueil sous "Catégories à renforcer", avec un lien "Voir toutes les catégories" vers cette vue complète.
- **Recherche** : porte sur titre, sous-titre, définition, points, piège, à-retenir et mots-clés. Taper une recherche depuis un autre mode bascule sur les Fiches ; le compteur de la barre affiche le nombre de résultats ; <kbd>Échap</kbd> vide la recherche.
- **Accessibilité** : focus clavier visible (`:focus-visible`), respect de `prefers-reduced-motion`, `aria-label` sur les boutons icônes, `aria-live` sur le compteur. Onboarding première visite (`maybeShowOnboarding()`, `revision_onboarded`).

## Personnalisation de l'affichage

- **Thème clair / sombre** : bouton 🌙/☀️ dans la barre (attribut `data-theme="dark"` sur `<html>`, stocké dans `revision_dark`). Sans choix manuel, suit la préférence système (`prefers-color-scheme`) et réagit à ses changements.
- **Impression / PDF** : bouton 🖨️ dans le panneau détail (`window.print()`) ; les styles `@media print` isolent la fiche ouverte.
- **Recherche** : les termes trouvés sont surlignés (`highlightTerm()` → `<mark class="hl">`).
- **Couleur d'accent** : 6 palettes (bleu, émeraude, violet, ambre, rose, cyan) via le bouton 🎨 de la barre du haut (popover) **ou** ⚙️ Paramètres → « Couleur d'accent ». Attribut `data-accent` sur `<html>`, stocké dans `revision_accent`. Les palettes surchargent les variables `--accent*` en clair comme en sombre (voir haut de `style.css`). `toggleAccentMenu()` ouvre le popover ; `setAccent(name)` applique et synchronise toutes les pastilles.
- **Navigation mobile** : sous 860px, la sidebar devient un tiroir coulissant ouvert par le bouton ☰ (fonctions `toggleSidebar()` / `closeSidebar()`).

## Ajouter une fiche

Les fiches sont dans `data.js`, tableau `FICHES` :

```js
{
  id: 105,                    // unique, entier
  cat: "reseau",              // clé de catLabels
  titre: "Le protocole DHCP",
  sub: "DORA, bail, ports UDP 67/68",
  schema: `<svg>...</svg>`,   // optionnel
  def: "DHCP attribue automatiquement...",
  points: ["Point clé 1", "Point clé 2"],
  piege: "Erreur fréquente à éviter.",
  retenir: "Phrase de synthèse.",
  keywords: ["DORA", "lease", "UDP 67/68"]
}
```

Les `keywords` sont utilisés pour la recherche, les fiches liées automatiques et les questions à trous du Quiz.

## Ajouter ou modifier une catégorie

Dans `data.js`, mettre à jour **les deux ensemble** :
- `catLabels` — code → libellé affiché (`secu: "🔒 Sécurité"`)
- `catOrder` — ordre d'affichage dans la sidebar

## Ajouter une commande dans le terminal

Dans `terminal-data.js`, objet `TERM_SHELLS` :

```js
linux: {
  label: "🐧 Linux/Bash",
  prompt: "user@linux:~$",
  color: "#86efac",
  intro: [{t:"ok", s:"Message d'accueil"}],
  commands: {
    "pwd": () => [{t:"ok", s:"/home/user"}],
    // t: "ok" | "warn" | "err" | "info" | "head"
  }
}
```

## Ajouter un scénario guidé

Dans `terminal-data.js`, tableau `TERM_SCENARIOS` :

```js
{
  id: "mon-scenario",
  label: "🔍 Mon scénario",
  shell: "linux",
  desc: "Description courte.",
  steps: [
    {
      titre: "Étape 1 — ...",
      contexte: "Explication avant l'étape.",
      hint: "Indice si bloqué.",
      expected: ["commande-exacte", "variante-acceptée"],
      output: [{t:"ok", s:"résultat"}],
      explication: "Ce qu'il faut comprendre."
    }
  ]
}
```

## Scénarios disponibles (14)

- 🔍 Forensique Linux
- 🛡️ Durcissement SSH
- 🔵 Config VLAN Cisco
- 🟦 Investigation Windows
- 🐳 Docker & Kubernetes
- 🔴 Investigation Active Directory
- 🔵 Config OSPF Cisco
- 🟧 Création VM Proxmox
- 🔶 Tunnel IPSec pfSense
- 🟦 Durcissement Windows Server
- 🔐 Config SSL/TLS
- 🚑 Dépannage service Linux
- 🚧 Règle pare-feu bloquante (pfSense)
- 💾 Sauvegarde & restauration Proxmox

## Service Worker / cache

Stratégie network-first sur `.html`, `.js`, `.css` (toujours la dernière version), cache-first sur les icônes et `manifest.json`.

**À chaque déploiement majeur** : monter le numéro de version dans `CACHE_NAME` de `sw.js` pour invalider le cache des utilisateurs existants.

Version actuelle : `revision-it-v24`
