# vitrine-proto-deploy

Publie le **fork du designer** sur **https://proto.vitrinedemocratique.com**, derrière
Cloudflare Access. Les données sont les mêmes que sur dev et prod : l'API sur Neon.

## Pour le designer

Votre dépôt : **[`clessn/proto_vitrine`](https://github.com/clessn/proto_vitrine)**, un fork
de [`ellipse-science/vitrine-showcase.github.io`](https://github.com/ellipse-science/vitrine-showcase.github.io)
appartenant à l'équipe. Vous y avez le droit d'écriture (acceptez l'invitation GitHub).

1. `gh repo clone clessn/proto_vitrine` — la branche par défaut est **`proto`**.
2. Travaillez sur `proto` (ou des branches que vous y fusionnez). Vous et votre agent
   pouvez pousser, réécrire, tout casser : rien n'atteint dev ni prod.
3. Poussez sur `proto`. Dans les **~10 à 20 minutes**, proto.vitrinedemocratique.com
   affiche votre dernier commit (relève toutes les 10 min, puis ~10 min de build).
   Connexion par courriel via Cloudflare Access.
4. Pour récupérer le travail de l'équipe : `gh repo sync clessn/proto_vitrine -b develop`,
   puis fusionnez `develop` dans `proto`.
5. Quand un morceau est prêt : **PR de `clessn/proto_vitrine` vers
   `ellipse-science/vitrine-showcase.github.io:develop`**, revue par l'équipe.

Les GitHub Actions sont **désactivées** sur `proto_vitrine` : le build se fait ici.

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
| Variable | `PROTO_REPO` | `clessn/proto_vitrine` |
| Variable | `PROTO_BRANCH` | `proto` |
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
