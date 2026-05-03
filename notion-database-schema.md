# Schéma de la base Notion

La base Notion qui reçoit les notes générées par l'agent doit avoir exactement le schéma suivant pour que le workflow fonctionne sans modification.

## Propriétés requises

| Nom | Type Notion | Valeurs / Format | Obligatoire |
|-----|-------------|------------------|-------------|
| **Titre** | Title | Texte libre généré par Claude (max 80 caractères) | Oui |
| **Catégorie** | Select | `Réunion`, `Idée`, `Tâche`, `Recherche`, `Personnel`, `Autre` | Oui |
| **Tags** | Multi-select | 2-5 tags libres en minuscules sans `#` | Oui |
| **Source** | Rich text | Origine de la note (ex: `Raccourci macOS`, `curl-test`, `Slack`) | Oui |

## Structure du contenu de la page

Chaque page créée par l'agent suit cette structure de blocs :

```
📝 Callout (résumé en 2-3 phrases)
─── divider ───
## Points clés
- bullet 1
- bullet 2
...

## Actions à mener
☐ to-do 1
☐ to-do 2
...

─── divider ───
## Détails

## Contexte
paragraphe libre

## Décisions
- bullet 1
- bullet 2

## Actions
- bullet 1

## Suivi
paragraphe libre
```

Les sections sous "Détails" sont générées uniquement si pertinentes (basées sur le brouillon utilisateur).

## Récupérer l'ID de la base

L'URL de la database ressemble à :

```
https://www.notion.so/<workspace>/<DATABASE_ID>?v=<view_id>
```

Le `DATABASE_ID` est une chaîne de 32 caractères hexadécimaux. Dans le workflow n8n, cet ID est hardcodé dans le node "Creer page Notion" :

```js
parent: { database_id: "9054fb74-7505-4ced-a696-b644d76a1b51" }
```

À remplacer par ton propre ID si tu utilises une autre base.

## Partage avec l'intégration

L'intégration Notion utilisée par n8n doit avoir accès à la base. Dans Notion :

1. Ouvre la page de la base (pas une page enfant)
2. Clic sur les **...** en haut à droite → **Connexions**
3. **Ajouter des connexions** → cherche ton intégration → ajoute-la

Sans cette étape, l'API Notion retourne une erreur 404 ("object not found") même avec un token valide.
