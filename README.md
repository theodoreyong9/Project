# Trio Viewer (PWA)

PWA minimaliste qui affiche 3 sites dans des iframes et permet de naviguer
en boucle entre eux via une **zone de swipe invisible** en bas de l'écran.

## Déploiement automatique sur GitHub Pages

1. Crée un nouveau dépôt GitHub (ex: `trio-viewer`).
2. Mets-y tout le contenu de ce dossier (garde bien le dossier `.github/workflows/`).
3. Pousse sur la branche `main` :
   ```bash
   git init
   git add .
   git commit -m "Trio viewer PWA"
   git branch -M main
   git remote add origin https://github.com/TON_USER/trio-viewer.git
   git push -u origin main
   ```
4. Dans le dépôt GitHub : **Settings → Pages → Build and deployment → Source**,
   choisis **GitHub Actions** (pas "Deploy from a branch").
5. Le workflow `.github/workflows/deploy.yml` se déclenche automatiquement à
   chaque push sur `main` et publie le site. L'URL sera du type :
   `https://TON_USER.github.io/trio-viewer/`

Ensuite, tout nouveau `git push` sur `main` redéploie automatiquement.

## ⚠️ Point important : les 3 sites doivent accepter d'être affichés en iframe

Les navigateurs bloquent l'affichage en iframe si le site cible envoie les
en-têtes `X-Frame-Options: DENY/SAMEORIGIN` ou une `Content-Security-Policy`
avec `frame-ancestors` restrictif. C'est notamment le cas par défaut pour
beaucoup de sites hébergés sur Firebase Hosting ou GitHub Pages selon leur
configuration.

Si une des 3 pages reste blanche dans l'iframe :
- Vérifie dans la console développeur du navigateur (message du type
  *"Refused to display ... in a frame"*).
- Il faudra alors soit modifier la configuration du site cible pour
  autoriser l'inclusion en iframe, soit envisager une autre approche
  (ouverture en plein écran / nouvel onglet plutôt qu'iframe).

## Fonctionnement du swipe

- Les 3 iframes n'occupent PAS toute la hauteur de l'écran : une fine bande
  de 26px (+ zone de sécurité iOS) est réservée en bas, en dehors des
  iframes (`#swipeZone`). Comme cette bande ne recouvre jamais le contenu
  des sites, **tout reste cliquable normalement dans les 3 apps**, y compris
  tout en bas de leur interface.
- Cette bande capte les gestes tactiles (touch) et souris (pointer).
- Techniquement, il est impossible de superposer une zone de swipe
  *par-dessus* une iframe cross-origin tout en laissant les clics la
  traverser (limitation de sécurité des navigateurs) — c'est pourquoi la
  zone est placée à côté des iframes plutôt que dessus.
- Un glissement horizontal de plus de 40px déclenche le passage à la page
  suivante/précédente, avec une boucle infinie sans à-coup (technique des
  slides clonés en début/fin de piste).
- De petits points quasi invisibles en bas indiquent la page active
  (purement indicatif, non interactifs).

## Personnalisation

- Les URLs des 3 pages sont dans `index.html`, tableau `PAGES`.
- Les icônes (`icon-192.png`, `icon-512.png`) peuvent être remplacées par
  tes propres visuels PWA.
