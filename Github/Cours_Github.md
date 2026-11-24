# Github

L’objectif est de savoir prendre une tâche, travailler sur ta branche, faire relire tes changements et les intégrer dans `main`. Le cours suppose que Git est installé et que tu as accès à un dépôt partagé, par exemple sur GitHub ou GitLab.

## 1. Comprendre ce qui est local et ce qui est partagé

Git conserve l’historique du projet sous forme de **commits**. Un commit enregistre un ensemble de changements avec un message. Une **branche** te permet de faire évoluer le projet pour une tâche donnée pendant que tes collègues avancent sur les leurs.

| Terme | Ce qu’il désigne |
|---|---|
| `main` | La branche principale dans ta copie locale. |
| `origin` | Le nom que Git attribue généralement au dépôt distant lors du clonage. |
| `origin/main` | Le dernier état connu localement de la branche `main` distante. `git fetch origin` actualise cette référence. |
| Pull request (PR) ou merge request (MR) | Une demande de relecture et d’intégration d’une branche dans une autre. |

`fetch` récupère les nouveautés sans modifier ta branche courante. `pull` récupère les nouveautés et les intègre dans ta branche courante. `push` publie tes commits sur le dépôt distant.

## 2. Partir de `main` et créer une branche

Clone le dépôt une seule fois. Remplace l’URL d’exemple par celle de ton équipe.

```bash
git clone https://exemple.com/equipe/projet.git
cd projet
```

Pour chaque nouvelle tâche, pars de la branche principale à jour. Ici, on va ajouter une section sur le lancement des tests au `README.md`.

```bash
git switch main
git pull --ff-only
git switch -c docs/lancer-les-tests
```

`--ff-only` refuse l’intégration si les deux historiques ont divergé. Si la commande échoue pour cette raison, lance `git status` et `git log --oneline --left-right main...origin/main` pour identifier les commits de chaque côté avant de continuer.

## 3. Vérifier, enregistrer et publier tes changements

Modifie `README.md`, puis vérifie ce que tu as changé et ce que tu t’apprêtes à enregistrer.

```bash
git status
git diff
git add README.md
git diff --staged
git commit -m "Documente le lancement des tests"
git push -u origin docs/lancer-les-tests
```

`git add` sélectionne les changements du prochain commit. `git diff --staged` te montre exactement ce qui sera enregistré, ce qui aide à repérer une modification hors sujet avant de la publier. L’option `-u` associe ta branche locale à sa branche distante. Pour les commits suivants sur cette branche, `git push` suffit.

Sur GitHub ou GitLab, ouvre une PR ou MR de `docs/lancer-les-tests` vers `main`. Explique ce que tu as changé, pourquoi et comment tu l’as vérifié. Si un collègue demande une correction, fais un nouveau commit sur la même branche puis lance `git push`. La demande de relecture se met à jour automatiquement.

## 4. Intégrer les changements de l’équipe et résoudre un conflit

Pendant la relecture, `main` peut avancer. Si votre équipe utilise la fusion pour synchroniser les branches de travail, assure-toi d’abord que `git status` indique un répertoire de travail propre, puis lance ces commandes depuis ta branche.

```bash
git fetch origin
git merge origin/main
```

Un **conflit** survient quand Git a besoin que tu décides comment combiner certains changements. Par exemple, un collègue a modifié la commande de test dans `main` pendant que tu réécrivais le même passage dans le `README.md`.

1. Lance `git status` pour repérer les fichiers en conflit.
2. Ouvre-les et repère les marqueurs `<<<<<<<`, `=======` et `>>>>>>>`. Réécris le passage tel qu’il doit apparaître dans la version finale et retire les marqueurs. Si l’intention d’un collègue est ambiguë, vérifie avec lui.
3. Vérifie le résultat et lance les tests concernés.
4. Enregistre la résolution et publie-la.

```bash
git add README.md
git merge --continue
git push
```

Si plusieurs fichiers sont en conflit, ajoute chacun des fichiers résolus avant `git merge --continue`. Pour abandonner une fusion encore en cours, utilise `git merge --abort`. Certaines équipes utilisent `git rebase` à la place de `git merge` pour synchroniser les branches. Réserve la réécriture d’une branche partagée aux situations où les personnes qui y travaillent se sont coordonnées.

## 5. Intégrer la PR et repartir sur une nouvelle tâche

Une fois la demande approuvée et les vérifications passées, fusionne-la sur la plateforme si les règles de l’équipe te le permettent. Mets ensuite ta copie locale à jour.

```bash
git switch main
git pull --ff-only
```

Tu peux supprimer ta branche locale devenue inutile avec `git branch -d docs/lancer-les-tests`. Si la plateforme a regroupé les commits de la PR en un seul commit (*squash merge*), `-d` peut refuser. Vérifie alors que la PR a bien été fusionnée et que ta branche ne contient aucun travail supplémentaire avant de la supprimer.

## 6. Corriger les erreurs courantes

| Situation | Commande ou action |
|---|---|
| Tu as placé un fichier par erreur dans le prochain commit. | `git restore --staged fichier` le retire de la sélection et conserve tes modifications. |
| Tu veux corriger le message du dernier commit local, avant de le publier. | `git commit --amend` ouvre l’éditeur de message. |
| Tu veux annuler un commit ordinaire déjà partagé. | `git revert <identifiant>` crée un nouveau commit qui inverse ses changements. S’il est sur `main`, fais le revert depuis une branche et ouvre une PR selon les règles de l’équipe. |
| Un `push` est refusé parce que la branche distante a avancé. | Fais `git fetch origin`, examine les nouveaux commits, puis intègre-les selon la méthode de l’équipe avant de réessayer. |

## Exercice à faire à deux

Sur un dépôt d’entraînement, choisissez une ligne du `README.md`. Créez chacun une branche depuis le même `main`, puis modifiez cette ligne de deux façons différentes. La première personne ouvre une PR que la seconde relit, puis vous la fusionnez. La seconde personne récupère ensuite `origin/main` et le fusionne dans sa propre branche pour déclencher le conflit. Décidez ensemble de la formulation finale, résolvez le conflit et ouvrez la deuxième PR.

Vous avez réussi si la deuxième PR montre uniquement les changements encore nécessaires par rapport à `main` et si vous pouvez expliquer pourquoi vous avez gardé la formulation finale.
