# Agent IA - Automatisation de création de notes orchestrée

Agent IA construit avec n8n qui transforme un brouillon de notes (mots-clés, fragments, phrases incomplètes) en une page Notion structurée et présentable. Déclenchable depuis macOS via un raccourci Apple Shortcuts.

## Architecture

```
┌──────────────────┐    POST     ┌──────────────────────┐
│ Raccourci macOS  │  ────────▶  │  Webhook n8n         │
│ (Apple Shortcuts)│   JSON      │  /note-summarizer    │
└──────────────────┘             └──────────┬───────────┘
                                            │
                                            ▼
                                 ┌──────────────────────┐
                                 │ Synthèse Claude      │
                                 │ (Sonnet 4.5)         │
                                 │ JSON structuré       │
                                 └──────────┬───────────┘
                                            │
                                            ▼
                                 ┌──────────────────────┐
                                 │ Parse JSON           │
                                 │ (Code node)          │
                                 └──────────┬───────────┘
                                            │
                                            ▼
                                 ┌──────────────────────┐
                                 │ Construire blocs     │
                                 │ Notion (Code node)   │
                                 └──────────┬───────────┘
                                            │
                                            ▼
                                 ┌──────────────────────┐
                                 │ Notion API           │
                                 │ Création de page     │
                                 └──────────┬───────────┘
                                            │
                                            ▼
                                 ┌──────────────────────┐
                                 │ Réponse JSON         │
                                 │ (notion_url, etc.)   │
                                 └──────────────────────┘
```

## Composants

### 1. Workflow n8n (`workflow.json`)

Six nodes en chaîne :

- **Reception du brouillon** : webhook POST `/note-summarizer`, accepte `{draft, source?, context?}`
- **Synthese par Claude** : appelle Claude Sonnet 4.5 avec un prompt système qui force une réponse JSON structurée (titre, résumé, points clés, actions, tags, catégorie, contenu markdown)
- **Parse JSON Claude** : extrait et valide le JSON retourné par Claude (gère les fences markdown, les structures de réponse variables)
- **Construire blocs Notion** : convertit le contenu markdown en blocs Notion natifs (callout, headings, bullets, to-do, paragraphs)
- **Creer page Notion** : POST sur l'API Notion `/v1/pages` avec les propriétés et le contenu
- **Reponse au client** : renvoie un JSON `{ok, notion_url, notion_id, title, summary, tags, category}`

### 2. Raccourci macOS (`macos-shortcut.md`)

Apple Shortcut "Note → Notion" qui demande un brouillon, l'envoie au webhook, ouvre la page Notion créée. Voir le fichier dédié pour les instructions de construction.

### 3. Base Notion

Database avec le schéma suivant (propriétés à créer dans Notion) :

| Propriété | Type | Description |
|-----------|------|-------------|
| Titre | Title | Titre généré par Claude |
| Catégorie | Select | Réunion, Idée, Tâche, Recherche, Personnel, Autre |
| Tags | Multi-select | 2-5 tags en minuscules |
| Source | Rich text | Origine de la note (ex: "Raccourci macOS") |

L'ID de la base est hardcodé dans le node "Creer page Notion" du workflow. À modifier si la base change.

## Prérequis

- Un compte n8n (cloud ou self-hosted)
- Une clé API Anthropic (Claude)
- Une intégration Notion avec accès à la base cible
- macOS pour le raccourci Apple Shortcuts (optionnel - le webhook peut être appelé depuis n'importe quoi qui fait du HTTP)

## Installation

### 1. Importer le workflow n8n

1. Ouvre ton instance n8n
2. Crée un nouveau workflow → menu trois points → "Import from file"
3. Sélectionne `workflow.json`
4. Configure les credentials :
   - **Anthropic API** : ton API key
   - **Notion API** : ton Internal Integration Token (https://www.notion.so/profile/integrations)

### 2. Créer la base Notion

1. Crée une nouvelle database dans Notion avec le schéma ci-dessus
2. Partage la database avec ton intégration Notion (clic ... → Connexions → ajoute ton intégration)
3. Récupère l'ID de la database (32 caractères dans l'URL)
4. Dans le workflow n8n, ouvre le node "Creer page Notion" et remplace `9054fb74-7505-4ced-a696-b644d76a1b51` par ton ID

### 3. Activer le workflow

Active le workflow dans n8n. Note l'URL du webhook de production : `https://<ton-instance>/webhook/note-summarizer`.

### 4. Construire le raccourci macOS

Suis les instructions dans `macos-shortcut.md`.

## Usage

Trois façons de déclencher :

```bash
# Via curl (test rapide)
curl -X POST https://<ton-instance>/webhook/note-summarizer \
  -H "Content-Type: application/json" \
  -d '{"draft": "tes notes en vrac ici", "source": "curl-test"}'
```

```text
# Via raccourci macOS
⌃⌥⌘N → tape ton brouillon → la page Notion s'ouvre dans le navigateur
```

```text
# Via API depuis n'importe quoi (Slack, Telegram, autre workflow n8n, app custom...)
POST https://<ton-instance>/webhook/note-summarizer
Body: {"draft": "...", "source": "...", "context": "..."}
```

## Schéma de la requête

```json
{
  "draft": "string (requis) - le brouillon brut",
  "source": "string (optionnel) - origine de la note",
  "context": "string (optionnel) - contexte additionnel pour Claude"
}
```

## Schéma de la réponse

```json
{
  "ok": true,
  "notion_url": "https://www.notion.so/...",
  "notion_id": "uuid",
  "title": "Titre généré",
  "summary": "Résumé en 2-3 phrases",
  "tags": ["tag1", "tag2"],
  "category": "Réunion"
}
```

## Personnalisation

- **Modèle Claude** : modifie `modelId` dans le node "Synthese par Claude"
- **Catégories Notion** : édite le tableau `validCategories` dans le node "Construire blocs Notion" et le prompt système du node Claude
- **Format de la page Notion** : modifie la fonction de construction des blocs (callout, headings, etc.) dans "Construire blocs Notion"
- **Prompt système** : édite le champ `system` du node Claude pour changer le ton, le format de sortie, les contraintes

## Stack technique

- **n8n** : orchestration du workflow
- **Claude Sonnet 4.5** : synthèse et structuration
- **Notion API v1** (version 2022-06-28) : stockage et présentation
- **Apple Shortcuts** : déclenchement depuis macOS

## Licence

MIT
