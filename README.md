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
- Chaque page est chargée dans **une seule iframe, une seule fois**. La
  boucle infinie est obtenue en réordonnant ces 3 iframes via CSS (`order`)
  plutôt qu'en dupliquant des iframes "clones" — donc pas de rechargement,
  pas d'état perdu, et pas de flash/clignotement au passage d'une page à
  l'autre en boucle.
- Le geste lui-même utilise un **défilement natif du navigateur avec
  `scroll-snap`** (`#swipeZone` est un conteneur `overflow-x:scroll`), et
  non plus un suivi manuel de `pointerdown`/`pointermove`/`pointerup`. Le
  scroll natif est géré par le thread de rendu/compositeur du navigateur,
  ce qui le rend beaucoup plus fiable si un des sites embarqués (ex: rendu
  3D en continu) sature le thread JS principal — le geste ne "se perd"
  plus même quand la page est sous forte charge. Le JS ne fait que
  *lire* la position de scroll pour synchroniser visuellement le grand
  carrousel au-dessus, et rétablir la boucle infinie une fois le geste
  terminé.
- Techniquement, il est impossible de superposer une zone de swipe
  *par-dessus* une iframe cross-origin tout en laissant les clics la
  traverser (limitation de sécurité des navigateurs) — c'est pourquoi la
  zone est placée à côté des iframes plutôt que dessus.

## Personnalisation

- Les URLs des 3 pages sont dans `index.html`, tableau `PAGES`.
- Les icônes (`icon-192.png`, `icon-512.png`) peuvent être remplacées par
  tes propres visuels PWA.

## Géolocalisation et autres permissions sensibles

Par défaut, un navigateur **bloque géolocalisation, caméra et micro à l'intérieur
d'une iframe**, même si le site les demande. Chaque `<iframe>` a maintenant
`allow="geolocation; camera; microphone; clipboard-write"` pour laisser passer
ces demandes de permission jusqu'au site embarqué (qui devra quand même
demander l'autorisation à l'utilisateur comme d'habitude, pour son propre
domaine).

## Limite connue : un site très lourd peut quand même ralentir la mise à jour visuelle

Si un des 3 sites fait tourner du rendu 3D/WebGL en continu (typiquement
yourmine, qui charge un "mesh"), il peut monopoliser le thread principal du
navigateur, surtout si le navigateur n'isole pas complètement l'iframe dans
son propre processus. Depuis le passage au scroll natif, **le geste de
swipe lui-même reste capté et suivi par le doigt** même dans ce cas (c'est
géré par le compositeur, pas par le JS de la page). Ce qui peut encore
ralentir légèrement, c'est la synchronisation visuelle du grand carrousel
au-dessus (qui, elle, dépend du JS) — elle peut sembler suivre avec un
petit temps de retard le temps que le site embarqué souffle, mais elle
rattrape toujours son retard, elle ne se bloque plus. La cause racine
(charge du site embarqué) reste hors de portée de cette PWA.
