# Plan — <sujet>

Dépôt : <chemin> · Branche : <branche> · Créé le : <date>

## Objectif
<ce qu'on veut obtenir et pourquoi, en 3-5 lignes>

## Décisions
<!-- Chaque décision avec sa raison : l'exécutant doit pouvoir trancher les cas voisins seul. -->
- **D-1** — <décision>. Raison : <pourquoi>.

## Invariants
<!-- Ce qui ne doit pas changer : contrats, formats, comportements existants. -->
- <invariant>

## Contraintes héritées
<!-- Fiches de la mémoire de l'utilisateur utiles à ce sujet (l'exécutant ne voit pas cette mémoire).
     Une ligne par fiche : le fait, puis sa source. -->
- <fait> — source : mémoire « <nom-de-la-fiche> »

## Faits établis
<!-- Ce qu'on sait du code, vérifié (exploration, remontées, relectures). L'exécutant part de là au lieu de
     redécouvrir le code. Chaque fait sourcé ; on corrige ou on barre un fait devenu faux. -->
- <fait> — `fichier:ligne`

## À lire avant de travailler
<!-- Les seuls documents utiles à ces lots. Remplace la liste de lecture générale du CLAUDE.md du dépôt. -->
- <chemin> — pourquoi

## Périmètre
- Dans le périmètre : <fichiers, modules>
- Hors périmètre : <ce qu'on ne touche pas ; dépendances interdites>

## Lots
<!-- Un lot = une session d'exécutant. État : à faire / en cours / remonté / en relecture / fait.
     Séquentiels par défaut ; « parallèle avec Lx » seulement si aucun fichier ni contrat commun. -->
### L1 — <titre> · état : à faire
- Quoi : <description>
- Critères d'acceptation :
  - [ ] <commande ou observation précise> → <résultat attendu>

## Exécutants
<!-- Rempli par l'architecte : lot → agent (type, ID), pour reprendre après redémarrage. -->
| Lot | Type | ID | État |
|-----|------|----|------|

## Journal des décisions
<!-- On ajoute à la fin, on ne réécrit pas. Une entrée par remontée tranchée. -->
