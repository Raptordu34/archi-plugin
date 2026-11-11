# archi — architecte / exécutant pour Claude Code

Garder Fable pour la réflexion, faire exécuter par des modèles moins chers.

- **Architecte** (ta session principale, sur Fable) : réfléchit avec toi, écrit un `PLAN.md` (décisions et
  leurs raisons, invariants, périmètre, faits établis, lots, critères d'acceptation), tranche les remontées.
- **`archi:executant`** (Sonnet) / **`archi:executant-opus`** (Opus) : implémentent un lot à partir du plan
  seul ; quand le plan ne suffit pas, ils s'arrêtent et remontent un dossier court. L'architecte tranche et
  relance **le même** exécutant (`SendMessage`) : aucun contexte n'est rechargé.
- **`archi:relecteur`** (Sonnet, lecture seule) : vérifie chaque lot contre les critères, sans lire le compte
  rendu de l'exécutant, et cherche à réfuter.
- **Mémoire** : seul l'architecte écrit dans la mémoire automatique ; les exécutants signalent « À retenir ».

## Installation

```
/plugin marketplace add Raptordu34/archi-plugin
/plugin install archi@archi
```

Puis, dans une nouvelle session : `/model fable`, `/effort high`, et `/archi <sujet>`.
Mise à jour : `/plugin marketplace update archi`.

## Prérequis

Claude Code récent avec accès à Fable (sinon l'architecte peut tourner sur Opus ; l'économie est moindre).
Le plan est rangé dans `.claude/archi/<sujet>/` du dépôt, exclu de git localement (`.git/info/exclude`).
