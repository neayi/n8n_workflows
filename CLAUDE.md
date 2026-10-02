# CLAUDE.md

Dépôt de travail pour les workflows n8n du Centre National d'Agroécologie (CNA) / Triple Performance.
Les workflows vivent sur l'instance n8n, pas dans ce dépôt : on les lit et on les modifie via le serveur MCP n8n.

## Environnement

- Instance n8n : https://n8n.tripleperformance.fr (MCP configuré dans `.mcp.json`, serveur `n8n`).
- Plugin `n8n-skills` activé (`.claude/settings.json`) : charger le skill adapté avant toute action n8n
  (debugging, node-configuration, agents, expressions, workflow-lifecycle…).
- Langue : l'utilisateur échange en français ; prompts, noms de colonnes et messages de version en français.

## Arborescence du dépôt

- `workflows/` : exports JSON de workflows (vide pour l'instant).
- `code-nodes/` : code des nodes Code, versionné à part (vide pour l'instant).
- `fixtures/` : données de test / pin data (vide pour l'instant).



## Pratiques de travail

- Exécutions volumineuses : `get_workflow_execution` avec `includeData` dépasse vite la limite ; filtrer par `nodeNames`,
  utiliser `truncateData`, et analyser le fichier sauvegardé avec `jq`.
- Les exécutions manuelles ne sauvegardent pas la progression : le résultat n'est lisible qu'une fois l'exécution terminée.
- Chaque appel au node OpenAI coûte : demander avant de lancer une exécution sur toute la liste des candidatures.
- Après chaque `update_workflow`, vérifier les `connections` avec `get_workflow_details`.
