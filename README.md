# Coach Front

Frontend de l'application **Coach**, développé avec [Angular](https://angular.dev) 21 et [Tailwind CSS](https://tailwindcss.com) 4, conteneurisé avec Docker.

## Prérequis

- [Docker](https://docs.docker.com/get-docker/) et Docker Compose (inclus dans Docker Desktop)
- *Optionnel, pour travailler sans Docker :* Node.js `^20.19`, `^22.12` ou `>=24` et npm

## Démarrage rapide (Docker)

```bash
git clone https://gitlab.com/coach8401241/coach-front.git
cd coach-front
docker compose up -d --build
```

L'application est disponible sur **http://localhost:8080**.

Pour l'arrêter :

```bash
docker compose down
```

## Les deux modes Docker

| Mode | Commande | URL | Description |
|------|----------|-----|-------------|
| Production | `docker compose up -d --build` | http://localhost:8080 | Build optimisé servi par Nginx |
| Développement | `docker compose --profile dev up front-dev --build` | http://localhost:4200 | `ng serve` avec rechargement à chaud |

### Mode production

Le [Dockerfile](Dockerfile) est en deux étapes :

1. une image `node:22-alpine` installe les dépendances et compile l'application (`ng build`) ;
2. une image `nginx:alpine` sert uniquement les fichiers compilés : l'image finale est légère et ne contient pas Node.

La configuration Nginx ([nginx.conf](nginx.conf)) renvoie `index.html` pour toutes les routes inconnues, ce qui permet au routeur Angular de fonctionner quand on recharge une page ou qu'on ouvre directement une URL comme `/profil`.

### Mode développement

Le service `front-dev` ([Dockerfile.dev](Dockerfile.dev)) monte le code source dans le conteneur : toute modification d'un fichier dans `src/` recharge automatiquement la page dans le navigateur.

```bash
docker compose --profile dev up front-dev --build
```

> Après avoir ajouté ou mis à jour une dépendance dans `package.json`, relancez avec `--build` pour réinstaller les `node_modules` du conteneur.

## Commandes Docker utiles

```bash
# Voir les logs
docker compose logs -f front

# Reconstruire l'image après un changement
docker compose up -d --build

# Construire et lancer l'image sans Docker Compose
docker build -t coach-front .
docker run -d -p 8080:80 --name coach-front coach-front

# Ouvrir un shell dans le conteneur de dev (ex : pour lancer une commande ng)
docker compose --profile dev exec front-dev sh
```

## Développement sans Docker

```bash
npm install
npm start          # http://localhost:4200
```

Autres scripts :

| Commande | Description |
|----------|-------------|
| `npm run build` | Build de production dans `dist/coach-front/browser` |
| `npm run watch` | Build de développement en continu |
| `npm test` | Tests unitaires (Vitest) |
| `npx ng generate component nom` | Générer un composant |

## Styles : Tailwind CSS

Le projet utilise **Tailwind CSS v4** via PostCSS (configuré dans [.postcssrc.json](.postcssrc.json)). Utilisez directement les classes utilitaires dans les templates :

```html
<button class="rounded-lg bg-blue-600 px-4 py-2 text-white hover:bg-blue-700">Valider</button>
```

- [src/tailwind.css](src/tailwind.css) importe Tailwind. C'est aussi là qu'on personnalise le thème avec `@theme` (couleurs, polices…).
- [src/styles.scss](src/styles.scss) reste disponible pour les styles globaux en SCSS.

> Tailwind v4 n'est pas compatible avec Sass : n'importez pas `tailwindcss` dans un fichier `.scss`. C'est pour ça qu'il est chargé depuis un fichier `.css` séparé (voir le tableau `styles` dans [angular.json](angular.json)).

Pour l'autocomplétion des classes dans VS Code, installez l'extension recommandée **Tailwind CSS IntelliSense**.

## Structure du projet

```
coach-front/
├── src/
│   ├── app/              # Composants, routes et configuration de l'application
│   ├── index.html
│   ├── main.ts
│   ├── styles.scss       # Styles globaux (SCSS)
│   └── tailwind.css      # Import et thème Tailwind
├── public/               # Fichiers statiques (favicon, images…)
├── Dockerfile            # Image de production (build + Nginx)
├── Dockerfile.dev        # Image de développement (ng serve)
├── docker-compose.yml
├── nginx.conf            # Configuration Nginx (routing SPA, cache, gzip)
└── angular.json
```

## Dépannage

- **Le port 8080 ou 4200 est déjà utilisé** : changez le port de gauche dans [docker-compose.yml](docker-compose.yml), par exemple `"3000:80"`.
- **Le hot reload ne marche pas en mode dev** : le polling est déjà activé (`--poll 2000`) pour les volumes montés depuis Windows/macOS ; vérifiez que vous modifiez bien les fichiers du dossier monté.
- **`npm ERR! Cannot read properties of null (reading 'edgesOut')` en local** : bug de npm 10.3. Mettez npm à jour (`npm install -g npm@latest`) ou utilisez Docker.
