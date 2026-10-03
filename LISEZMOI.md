# Prototype — Module de capture biométrique (vote à distance)

Page web unique (`index.html`) qui démontre le parcours de capture décrit dans le dossier de conception :
contrôle du terminal → visage avec défi actif → empreintes sans contact, main posée sur un fond blanc (4 doigts de chaque main, puis chaque pouce : ordre 4-4-1-1) → chiffrement → effacement.

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

Navigateurs : Chrome sous Android ; sur iPhone, Safari ou Chrome, qui reposent tous deux sur le moteur WebKit
(photo pleine définition disponible depuis iOS 18.4).

Le premier chargement télécharge environ 13 Mo de modèles (jsDelivr), mis en cache ensuite.

## Capture des empreintes

Depuis la v21, la main se présente de côté dans une **zone fixe**, comme le visage dans son ovale : paume vers
l'objectif, doigts légèrement écartés et pointés vers le bord de l'écran (vers la gauche pour la main droite,
vers la droite pour la main gauche), les quatre bouts de doigts dans la zone claire. Les pouces suivent, un par un.

- La zone ne change jamais de taille ni de place pendant la session. Elle ne présuppose ni la taille des doigts,
  ni leur écartement, ni leur longueur relative : un seul cadre pour les quatre bouts de doigts.
- La feuille blanche n'est plus obligatoire : la couleur du fond est prise devant la zone, celle de la peau au
  pied de la zone. Un fond clair et uni reste préférable.
- La capture se déclenche quand les **crêtes sont visibles** sur chaque doigt (mesure de netteté prise à la
  définition native), à partir de 250 ppi : chacun trouve ainsi la distance où son téléphone fait la mise au point.
- L'**échelle** est mesurée de deux façons indépendantes de la taille de la main : largeur des doigts et
  écartement des crêtes (0,46 mm en moyenne chez l'adulte), combinées (60 % crêtes, 40 % largeur).

L'ancienne disposition (main à plat sur une feuille blanche, doigts vers le haut) reste accessible en ajoutant
`?disposition=fond` à l'adresse. Sur iPhone, `?objectif=triple` essaie la caméra virtuelle qui bascule
automatiquement en mode macro.

Quand la photo du navigateur n'est pas nettement plus définie que la vidéo (moins de 1,3 fois), le prototype ne
la prend pas et utilise l'image la plus nette de la rafale : l'aperçu n'est pas interrompu.

## Résolution : ce que permet le navigateur

La norme vise 500 ppi (400 au minimum). Dans un navigateur, elle n'est pas atteignable sur tous les téléphones :
les navigateurs anciens ne donnent accès qu'à une vidéo (le prototype demande la résolution maximale annoncée,
4032 × 3024 sur iPhone ; depuis iOS 18.4, la photo est aussi disponible),
et la caméra principale des iPhone Pro ne fait pas la mise au point en deçà d'environ 20 cm. On obtient alors
de l'ordre de 150 ppi en 1080p, 300 ppi en 4K. Le prototype déclenche donc automatiquement à partir de
250 ppi (130 ppi si la vidéo est limitée à 1080p) et marque la prise « sous la norme ». Une application native
(photo 48 Mpx) atteint environ 700 ppi à la même distance.

La photo est demandée à la définition maximale annoncée par le capteur (sans cette précision, WebKit rend une
photo à la taille de la vidéo). Le rapport réel photo / vidéo est mesuré à la première prise et remplace
l'estimation ; si la résolution mesurée sur la photo reste sous le seuil, la prise est refaite automatiquement.

La capture est entièrement automatique : aucun bouton. Si les doigts ne sont pas retrouvés sur l'image,
une vue de diagnostic s'affiche quatre secondes (en rouge ce qui a été pris pour de la peau, en vert les
doigts retenus), puis la capture reprend.

Objectif utilisé pour les doigts : la caméra principale par défaut. Pour essayer un autre objectif, ajouter
`?objectif=tele` ou `?objectif=ultra` à l'adresse. Le journal indique la résolution réellement obtenue.

## Acquisition complète et extraction

L'acquisition est complète, comme à l'enrôlement : visage puis dix doigts (4-4-1-1). Seul le bouton
« Doigt absent ou blessé » dispense d'une prise ; l'exception est consignée dans le paquet.

Quelle que soit l'issue (tentatives épuisées, bouton « Arrêter », session expirée, erreur), le parcours
se termine sur le récapitulatif : il montre ce qui a été capturé, y compris la meilleure image du visage
non validée, et liste les éléments manquants avec leur motif. Un paquet incomplet peut être chiffré
pour examen ; le service d'authentification simulé le reçoit intact mais refuse l'authentification.
Au moment de la capture, le prototype extrait les minuties de chaque doigt après normalisation locale du
contraste et lissage le long des crêtes, ce qui rend l'extraction indépendante du teint de la peau ; un doigt
est exploitable à partir de 12 minuties sur au moins 60 mm² de crêtes lisibles et un gabarit facial qui sert à vérifier que la personne du défi est celle
de la photo de référence. Le serveur refait sa propre extraction sur les images.

## Paramètres

Tous les seuils sont regroupés en tête de script dans l'objet `CFG` (distance inter-pupillaire, pose,
luminance, netteté, résolution des empreintes, nombre de tentatives, durée de session). Ils sont
volontairement exposés pour être calibrés lors des essais.

Ajouter `#debug` à l'URL expose l'état interne dans `window.__proto` pour les tests.

## Dépendance

`@vladmandic/human` 3.3.6 (licence MIT) — TensorFlow.js et modèles MediaPipe : détection du visage,
maillage 468 points, iris, anti-usurpation, vivacité, détection et squelette de la main.
