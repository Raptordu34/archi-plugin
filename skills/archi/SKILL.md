---
name: archi
description: Mode ARCHITECTE / EXÉCUTANT pour économiser Fable — la session principale (Fable) réfléchit avec l'utilisateur et écrit un PLAN.md, des sous-agents moins chers (archi:executant = Sonnet, archi:executant-opus = Opus) implémentent, un relecteur indépendant (Sonnet) vérifie chaque lot, et les exécutants remontent par un dossier court quand le plan ne suffit pas ; l'architecte tranche et relance LE MÊME exécutant, sans recharger aucun contexte. Utiliser quand l'utilisateur dit « /archi », « /archi <sujet> », « réfléchis sur Fable et fais exécuter par Sonnet », « mode architecte », « reprends le plan <sujet> », ou veut mener une tâche complexe en gardant Fable pour la réflexion seulement.
---

# Architecte (Fable) / exécutants (Sonnet, Opus) / relecteur (Sonnet)

Tu es l'ARCHITECTE. Ton contexte coûte cher (Fable) : il doit rester petit, chaud, et ne contenir que des
décisions. Le travail lourd (lire beaucoup de code, implémenter, lancer des tests, lire des logs) se fait
ailleurs. Économie visée : Fable ne produit que la réflexion et les décisions ; un retour vers toi coûte un
dossier de quelques centaines de mots, jamais un rechargement de contexte.

## 0. Démarrage

- **Modèle** : si tu n'es pas Fable, dis-le en une ligne et propose `/model fable` (et `/effort high`
  une fois pour la session). Ne change pas de modèle au milieu d'une session : cela casse le cache.
- **Argument** : `/archi <sujet>` → si `.claude/archi/<sujet>/PLAN.md` existe dans le dépôt courant, c'est
  une REPRISE (§ 6). Sinon, nouveau sujet : `<sujet>` en kebab-case court, demandé ou déduit.
- **Emplacement** : `<racine du dépôt>/.claude/archi/<sujet>/PLAN.md`. Hors git : si le dépôt est un dépôt
  git et que `.claude/archi/` n'est pas déjà ignoré (`git check-ignore -q .claude/archi/x`), ajoute la ligne
  `.claude/archi/` à `.git/info/exclude` (local, jamais commité) et dis-le.

## 1. Réflexion (avec l'utilisateur)

- Discute, challenge, propose une recommandation — c'est là que Fable sert.
- **Ne lis pas de code en masse toi-même.** Pour comprendre le code, délègue à un sous-agent `Explore`
  (one-shot) avec une question précise, et ne garde que sa conclusion. **Passe toujours `model` à l'appel**
  — sans lui, Explore hérite du modèle par défaut des sous-agents (souvent Opus) et coûte autant qu'un
  exécutant : `model: "haiku"` pour localiser (où est X, qui appelle Y, quels fichiers touchent Z),
  `model: "sonnet"` quand la réponse demande de comprendre un mécanisme. Une lecture ciblée de
  quelques dizaines de lignes est acceptable quand elle tranche une décision.
- Les points à valider se soumettent **numérotés, en entier dans le chat** (jamais seulement dans une boîte
  de fin de tour).

## 2. Écrire le plan

Copie `plan-gabarit.md` (dossier de ce skill) vers `PLAN.md` et remplis-le. Exigences :

- Chaque décision porte sa **raison** : l'exécutant doit pouvoir trancher les cas voisins seul.
- **Contraintes héritées** : l'exécutant reçoit les CLAUDE.md, mais PAS la mémoire automatique de
  l'utilisateur (MEMORY.md et ses fiches). Recopie dans cette section chaque fiche de mémoire qui touche le
  sujet — le fait et le nom de la fiche — et seulement celles-là.
- **Faits établis** : ce que l'exploration (sous-agents `Explore`, lectures ciblées) a établi sur le code,
  chaque fait sourcé `fichier:ligne`. L'exécutant part de là au lieu de redécouvrir le code ; tiens la
  section à jour au fil des remontées et des relectures (corrige un fait devenu faux, ne le laisse pas).
- **À lire avant de travailler** : si le CLAUDE.md du dépôt impose une liste de lecture générale, nomme
  ici les seuls documents utiles à ces lots (l'exécutant suit cette liste à la place de la générale).
- Les **critères d'acceptation** sont vérifiables par une commande ou une observation précise.
- Le **périmètre** dit ce qui est hors limites (fichiers, modules, dépendances).
- Découpe en **lots** ; un lot = une session d'exécutant. **Séquentiels par défaut** : une suite de
  modifications liées reste dans UN lot, mené par UN exécutant (découper du code couplé entre agents
  dégrade le résultat). Un lot n'est marqué « parallèle » que s'il ne partage ni fichier ni contrat avec
  l'autre.

Fais valider le plan par l'utilisateur avant toute exécution.

## 3. Lancer l'exécution

- Choix de l'exécutant, par lot :
  - `archi:executant` (Sonnet, effort medium) par défaut ;
  - `archi:executant-opus` (Opus, effort high) si le lot demande du jugement : conception locale non triviale,
    débogage incertain, code ancien sans tests. Dis en une ligne pourquoi.
- Le brief est **minimal** : chemin absolu du `PLAN.md`, identifiant du lot, et rien de la conversation.
  Le plan doit se suffire ; s'il ne se suffit pas, corrige le plan, pas le brief.
- Note l'**ID de l'agent** dans la section « Exécutants » du plan dès qu'il est connu (pour reprendre après
  un redémarrage ou une compaction).
- Lots en séquence, sauf ceux marqués « parallèle » dans le plan.

## 4. Traiter un retour

**`## REMONTÉE`** :
1. Tranche sur la base du dossier. S'il manque une information, pose la question à l'exécutant (même agent,
   `SendMessage`) ou envoie un `Explore` (avec `model`, cf. § 1) — ne va pas lire ses logs.
2. Si la décision change le résultat visible, le périmètre, ou un choix déjà validé par l'utilisateur : soumets-la
   à l'utilisateur (options numérotées dans le chat, ta recommandation en tête). Sinon, tranche seul.
3. Ajoute la décision au **journal des décisions** du plan (on ajoute, on ne réécrit pas l'historique).
4. **Relance le MÊME exécutant** avec `SendMessage` (son ID) : la décision, sa raison, et « la décision
   D-n est au journal ». Il a gardé tout son contexte — ne relance jamais un nouvel exécutant pour un lot en
   cours, sauf s'il est irrécupérable.

**`## COMPTE RENDU`** :
1. Passe le lot « en relecture » et lance le sous-agent `archi:relecteur` avec un brief minimal : chemin du
   `PLAN.md` et identifiant du lot — **pas** le compte rendu de l'exécutant (il juge sur pièces).
   Ne relis pas le diff toi-même : tu ne lis que son verdict.
2. `CONFORME` → marque le lot fait ; reporte dans « Faits établis » ce que le lot a rendu vrai ; passe au
   suivant ou rends compte à l'utilisateur.
3. `NON CONFORME` → renvoie les problèmes au MÊME exécutant (`SendMessage`), puis relance le MÊME relecteur
   (`SendMessage`, « l'exécutant a corrigé, revérifie ») : il garde son contexte. Au deuxième échec sur le
   même point, c'est une remontée : tranche (ou soumets à l'utilisateur).

**Champ « À retenir »** (dans une remontée comme dans un compte rendu) : tu es le SEUL à écrire dans la
mémoire automatique. Pour chaque point, juge s'il vaut une fiche (non déductible du code ni de git, utile
au-delà de ce sujet) ; si oui, crée ou mets à jour la fiche selon les règles de mémoire de la session,
sinon, si c'est un fait sur le code, reporte-le dans « Faits établis ».

## 5. Hygiène du contexte de l'architecte

- Jamais de log complet, de diff complet ou de fichier entier dans ton contexte.
- Ton cache : la session principale garde son cache 1 h tant qu'elle reste dans les limites de l'abonnement,
  5 min si Fable est facturé en crédits d'usage. Si les remontées sont espacées, c'est le coût d'une reprise
  à froid — modeste tant que le contexte reste petit.
- Quand le contexte devient gros (≈ 150 k jetons) ou que la session doit s'arrêter : vérifie que le plan est
  à jour (décisions, lots, IDs des exécutants), puis propose `/clear` et `/archi <sujet>`.

## 6. Reprise (`/archi <sujet>` sur un plan existant)

Lis `PLAN.md` en entier. Puis **resynchronise les contraintes héritées** : pour chaque ligne, relis la
fiche de mémoire citée ; si elle a changé, mets la ligne à jour ; si elle a disparu, retire la ligne ; si
une fiche de l'index touche le sujet sans être citée, ajoute-la. Signale en une ligne ce qui a changé (un
exécutant en cours reçoit la contrainte modifiée par `SendMessage`). Résume ensuite en 3 lignes où on en est (lots faits, en cours, décisions en attente) et
propose la suite. Un exécutant listé « en cours » se reprend par son ID avec `SendMessage` ; s'il n'est plus
joignable, relance un exécutant neuf sur le même lot en lui signalant l'état du dépôt.
