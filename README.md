# vitrine-proto-deploy

Publie le **fork du designer** sur **https://proto.vitrinedemocratique.com**, derrière
Cloudflare Access. Les données sont les mêmes que sur dev et prod : l'API sur Neon.

## Pour le designer

1. Forkez [`ellipse-science/vitrine-showcase.github.io`](https://github.com/ellipse-science/vitrine-showcase.github.io).
2. Travaillez sur la branche convenue (par défaut `proto`) de votre fork. Vous et votre
   agent y avez tous les droits.
3. Poussez. Dans les **~10 à 20 minutes**, proto affiche votre dernier commit
   (relève toutes les 10 min, puis ~10 min de build).
4. Pour récupérer le travail de l'équipe : bouton **Sync fork** sur GitHub, ou
   `gh repo sync <vous>/vitrine-showcase.github.io -b develop`, puis fusionnez `develop`
   dans votre branche.
5. Quand un morceau est prêt : **PR de votre fork vers `develop`**, revue par l'équipe.

Pour itérer vite, rien ne remplace `npm run dev` en local ; proto sert à montrer et à
faire valider.

Vous n'avez besoin d'**aucun secret** : ni jeton Cloudflare, ni clé d'API. Sans clé, un
build local lit les JSON commités dans le dépôt (même structure, un peu moins frais).

## Comment ça marche

`.github/workflows/deployer.yml`, toutes les 10 minutes :

| Tâche | Exécute le code du fork ? | Secrets |
|---|---|---|
| `detect` : la tête de la branche a-t-elle changé depuis le dernier déploiement ? | non | jeton Pages (lecture de la liste des déploiements) |
| `build` : `npm ci && npm run build` en mode `api` | **oui** | `PROTO_API_KEY` seulement (lecture seule) |
| `deploy` : `wrangler pages deploy` de l'artefact | **non** | jeton Pages |

Le designer ne détient jamais de jeton Cloudflare : un jeton Pages couvre **tous** les
projets Pages du compte `capp`, prod comprise. D'où ce dépôt séparé, plutôt qu'une
branche ou un workflow dans `vitrine-showcase` (où un droit d'écriture donnerait accès à
ses 24 secrets, dont la clé de déploiement qui contourne les rulesets).

`PROTO_API_KEY` porte une **liste explicite de jeux** et un quota quotidien. Jamais `*` :
dans `workers/api/src/auth.ts`, `*` satisfait aussi le contrôle `sync`, qui autorise la
synchro et l'écriture de l'illustration dans R2.

## Réglages

| Où | Nom | Valeur |
|---|---|---|
| Variable | `PROTO_REPO` | `<compte>/vitrine-showcase.github.io` |
| Variable | `PROTO_BRANCH` | branche publiée |
| Variable | `CLOUDFLARE_ACCOUNT_ID` | compte `capp` |
| Environnement `deploy` (branche `main` seulement) | `CLOUDFLARE_API_TOKEN` | jeton **Pages Write** dédié |
| Environnement `build` (branche `main` seulement) | `PROTO_API_KEY` | clé `proto-designer` |

Forcer un déploiement : **Actions → Déployer le proto du designer → Run workflow**
(case « force »).

Côté Cloudflare : projet Pages `vitrine-democratique-proto`, apps Access « Vitrine —
proto designer » (domaine perso) et « … (pages.dev) » (`*.pages.dev` compris). Ajouter
ou retirer une personne se fait dans les **deux** apps.

## Fin du projet

Révoquer la clé `proto-designer` (api.vitrinedemocratique.com/admin), supprimer le
jeton Pages, désactiver le workflow. GitHub suspend aussi de lui-même les crons d'un
dépôt public sans commit depuis 60 jours.
