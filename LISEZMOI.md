# Prototype — Module de capture biométrique (vote à distance)

Page web unique (`index.html`) qui démontre le parcours de capture décrit dans le dossier de conception :
contrôle du terminal → visage avec défi actif → empreintes sans contact, main posée sur une feuille blanche A4 (4 doigts de chaque main, puis chaque pouce : ordre 4-4-1-1) → chiffrement → effacement.

**Ce que le prototype ne fait pas** : il ne compare rien au RNPP, son attestation d'intégrité est simulée,
et ses modèles de détection du vivant sont des modèles publics non certifiés. Il sert à éprouver l'ergonomie,
les seuils de qualité et le format du paquet, pas la sécurité.

## Tester sur un téléphone

La caméra n'est accessible qu'en HTTPS. Trois façons simples :

1. **GitHub Pages** : déposer `index.html` dans un dépôt, activer Pages, ouvrir l'URL `https://…github.io/…` sur le téléphone.
2. **Tunnel depuis votre poste** :
   ```
   npx serve .            # sert le dossier sur http://localhost:3000
   npx localtunnel --port 3000   # ou : cloudflared tunnel --url http://localhost:3000
   ```
   puis ouvrir l'URL HTTPS affichée sur le téléphone.
3. **Poste de travail** : `npx serve .` puis `http://localhost:3000` dans Chrome (la webcam sert de caméra ;
   cocher « Tenter quand même la capture d'empreintes » pour essayer la main devant la webcam).

Navigateur recommandé : Chrome Android (torche et mise au point pilotables). Sur iOS, Safari fonctionne
mais n'expose ni la torche ni la mise au point.

Le premier chargement télécharge environ 13 Mo de modèles (jsDelivr), mis en cache ensuite.

## Capture des empreintes

Poser une feuille blanche A4 sur une table sombre, puis la main dessus, dos contre le papier.
Téléphone en portrait : feuille en largeur devant soi, ses bords haut et bas visibles à l'écran.
La distance entre ces deux bords (210 mm) donne l'échelle. Pour le papier US Letter, régler
`CFG.SHEET.WIDTH_MM` à 215.9 ; pour une feuille A4 pliée en deux, à 148.5.

## Paramètres

Tous les seuils sont regroupés en tête de script dans l'objet `CFG` (distance inter-pupillaire, pose,
luminance, netteté, résolution des empreintes, nombre de tentatives, durée de session). Ils sont
volontairement exposés pour être calibrés lors des essais.

Ajouter `#debug` à l'URL expose l'état interne dans `window.__proto` pour les tests.

## Dépendance

`@vladmandic/human` 3.3.6 (licence MIT) — TensorFlow.js et modèles MediaPipe : détection du visage,
maillage 468 points, iris, anti-usurpation, vivacité, détection et squelette de la main.
