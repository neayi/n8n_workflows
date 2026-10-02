# CLAUDE.md

Dépôt de travail pour les workflows n8n du Centre National d'Agroécologie (CNA) / Triple Performance.
Les workflows vivent sur l'instance n8n, pas dans ce dépôt : on les lit et on les modifie via le serveur MCP n8n.

## Environnement

- Instance n8n : https://n8n.tripleperformance.fr (MCP configuré dans `.mcp.json`, serveur `n8n`).
- Plugin `n8n-skills` activé (`.claude/settings.json`) : charger le skill adapté avant toute action n8n
  (debugging, node-configuration, agents, expressions, workflow-lifecycle…).
- Le dossier n8n de travail a l'id `zHqnphI1NcT3PqPZ`, dans le projet personnel de Bertrand (`Ca0u5tf3nFUuKeBL`).
- Langue : l'utilisateur échange en français ; prompts, noms de colonnes et messages de version en français.

## Arborescence du dépôt

- `workflows/` : exports JSON de workflows (vide pour l'instant).
- `code-nodes/` : code des nodes Code, versionné à part (vide pour l'instant).
- `fixtures/` : données de test / pin data (vide pour l'instant).

## Credentials à utiliser (projet de Bertrand)

Plusieurs credentials du même type existent sur l'instance, certains appartenant à d'autres projets.
Toujours choisir explicitement :

| Usage | Nom | Id |
|---|---|---|
| Google Sheets | Google Sheets account | `w6ogNlXTL4DEEdwH` |
| Google Drive | Google Drive account - bertrand.gorge@neayi.com | `fN7KOz8GZ6kK0h6L` |
| OpenAI | OpenAi account | `oh6sBvJPjjSXb6GO` |

## Workflow « My workflow 5 » (`GcOw0SFSX41fVxdG`) : analyse des candidatures CNA

Analyse par IA des CV reçus pour le poste de Responsable du Pôle Agronomie.

```
Manual trigger → Get row(s) in sheet → Filter → Download CV → Extract CV text → Analyze CV → Format CV analysis for sheet
```

- Source : Google Sheet « Candidatures job CNA » (`1mqg2b2-83ZBuzOJBPieujoWvmv1VKbO9IB7MXPg0_ng`), onglet `candidatures` (gid=0).
  La colonne `Date` est au format `dd/MM/yyyy HH:mm:ss` ; `CV` contient une URL Google Drive.
- Filter : CV non vide ET Date non vide ET Date > 2026-09-01. La condition « Date non vide » est indispensable :
  certaines lignes n'ont pas de date et `DateTime.fromFormat('')` fait planter le Filter (« Invalid DateTime »).
- Download CV : Google Drive, `fileId` en mode `url` = `{{ $json.CV }}`. Ne gère que les PDF partagés avec le compte Drive ;
  un CV Word ou inaccessible fait échouer l'exécution. « Execute Once » peut être activé pour les tests (1 seul CV).
- Analyze CV : node OpenAI (`@n8n/n8n-nodes-langchain.openAi` v2.3, Responses API), modèle `gpt-5.4-mini`.
  Le prompt système (grille de 15 critères notés 0-5) est rédigé par l'utilisateur : ne pas le réécrire sans demande.
  Sortie en `json_schema` strict (`options.textFormat`) ; le JSON parsé est dans `$json.output[0].content[0].text`.
- Format CV analysis for sheet : Edit Fields qui aplatit le JSON en 48 colonnes (`row_number`, coordonnées « (CV) »,
  `<Critère> - note` / `<Critère> - raison`, listes jointes par retour à la ligne, `Synthèse IA`, `Recommandation IA`, `Alertes IA`).
  Les en-têtes correspondants doivent exister dans le Sheet avant d'y écrire.

Points ouverts :
- Le modèle applique mal les règles de recommandation (ex. ACS/bio = 1 → devrait donner « Profil à discuter »).
  Proposition : recalculer la recommandation dans n8n à partir des notes plutôt que faire confiance au LLM.
- Reste à ajouter l'écriture dans le Sheet (Google Sheets « Update row », clé `row_number`).

## Pratiques de travail

- Exécutions volumineuses : `get_workflow_execution` avec `includeData` dépasse vite la limite ; filtrer par `nodeNames`,
  utiliser `truncateData`, et analyser le fichier sauvegardé avec `jq`.
- Les exécutions manuelles ne sauvegardent pas la progression : le résultat n'est lisible qu'une fois l'exécution terminée.
- Chaque appel au node OpenAI coûte : demander avant de lancer une exécution sur toute la liste des candidatures.
- Après chaque `update_workflow`, vérifier les `connections` avec `get_workflow_details`.
