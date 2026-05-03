# Raccourci macOS - Note → Notion

Apple Shortcut qui déclenche le workflow n8n depuis ton Mac. Demande un brouillon dans une popup, l'envoie au webhook, ouvre automatiquement la page Notion créée.

## Construction du raccourci

Ouvre l'app **Raccourcis** sur macOS, clique sur **+** pour créer un nouveau raccourci, renomme-le **"Note → Notion"**.

Ajoute les actions suivantes dans cet ordre.

### Action 1 - Demander une entrée

Recherche `Demander une entrée` et ajoute l'action.

- **Question** : `Ton brouillon de note ?`
- **Type** : Texte
- Coche **Autoriser plusieurs lignes** (Afficher davantage)

### Action 2 - Obtenir le contenu d'une URL

Recherche `Obtenir le contenu d'une URL` et ajoute l'action.

- **URL** : `https://<ton-instance-n8n>/webhook/note-summarizer`
- **Méthode** : POST
- **En-têtes** :
  - `Content-Type` = `application/json`
- **Corps de la requête** : JSON
  - Clé `draft` (Texte) - valeur : variable **Saisie fournie** (l'output de l'action 1)
  - Clé `source` (Texte) - valeur : `Raccourci macOS`

### Action 3 - Obtenir une valeur de dictionnaire

Recherche `valeur de dictionnaire` et ajoute l'action.

- **Obtenir** : Valeur
- **pour** : `notion_url`
- **dans** : variable **Contenu de l'URL** (l'output de l'action 2)

### Action 4 - Ouvrir des URL

Recherche `Ouvrir des URL` et ajoute l'action.

- **URL** : variable **Valeur du dictionnaire** (l'output de l'action 3)

### Action 5 (optionnelle) - Afficher la notification

Recherche `notification` et ajoute l'action.

- **Titre** : `Note créée dans Notion`
- **Corps** : variable **Valeur du dictionnaire**

## Configuration des déclencheurs

Dans le panneau Info du raccourci (icône **i** en haut à droite) :

- **Épingler dans la barre de menus** : ajoute une icône Raccourcis dans la barre de menus macOS pour un accès rapide
- **Ajouter un raccourci clavier** : suggéré `⌃⌥⌘N` pour déclencher depuis n'importe où
- **Recevoir le texte depuis Quick Actions** (optionnel) : permet de déclencher depuis le clic droit sur du texte sélectionné

## Variables clés

| Variable | Source | Utilisation |
|----------|--------|-------------|
| Saisie fournie | Action 1 (Demander une entrée) | Texte du brouillon utilisateur |
| Contenu de l'URL | Action 2 (Obtenir le contenu) | Réponse JSON du webhook |
| Valeur du dictionnaire | Action 3 (Obtenir valeur) | URL de la page Notion créée |

## Dépannage

**La notification s'affiche mais aucune page Notion n'apparaît**
Le webhook est joignable mais l'API Notion a échoué. Vérifie le token Notion dans les credentials n8n et que ton intégration a bien accès à la base cible.

**Le brouillon n'est pas pris en compte (note "vide" générée)**
La variable `draft` dans le JSON n'est pas correctement liée à `Saisie fournie`. Vérifie que la valeur affichée dans la cellule est un bloc coloré (variable) et non du texte normal.

**Erreur 404 ou timeout sur le webhook**
Le workflow n8n n'est pas activé, ou l'URL est mauvaise. Vérifie l'URL exacte dans n8n (production path, pas test path) et que le workflow est bien activé.

**Erreur 401 unauthorized depuis Notion**
Token Notion invalide ou expiré. Régénère un token depuis https://www.notion.so/profile/integrations et mets-le à jour dans les credentials n8n.
