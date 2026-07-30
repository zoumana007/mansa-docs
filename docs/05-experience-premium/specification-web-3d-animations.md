# Spécification UX/UI premium des interfaces web

Cette spécification s’applique au site public, au site professionnel et au portail Admin Web. L’objectif est un rendu fintech haut de gamme, immersif, responsive et performant, inspiré du niveau de finition des meilleures expériences numériques modernes, sans copier une identité existante.

## Stack cible

- Next.js App Router, React, TypeScript et Tailwind CSS.
- GSAP et ScrollTrigger pour les timelines et le storytelling au scroll.
- Lenis pour un défilement fluide, désactivable et compatible accessibilité.
- Framer Motion pour les transitions d’interface et micro-interactions React.
- Three.js, React Three Fiber et Drei pour la 3D.
- Blender pour produire et optimiser les modèles GLB/GLTF.
- Spline uniquement lorsqu’il réduit réellement le coût d’implémentation sans dégrader les performances.

## Expériences obligatoires

### Hero cinématique

- Entrée progressive du logo, du titre et des appels à l’action.
- Mouvement léger de caméra et éclairage moderne.
- Modèle 3D ou fallback 2D selon la puissance de l’appareil.
- Boutons avec attraction magnétique légère, feedback de pression et états clavier visibles.

### Scroll storytelling

- Timelines synchronisées au scroll avec `ScrollTrigger`.
- Sections raccordées sans saut, clignotement ni changement brutal de hauteur.
- Pinning limité et testé sur mobile.
- Préservation du contrôle utilisateur : aucune animation ne doit bloquer la navigation.

### Parallax et profondeur

- Couches de fond, contenu et premier plan à vitesses différentes.
- Déplacements limités afin d’éviter nausée, surconsommation et perte de lisibilité.
- Désactivation via `prefers-reduced-motion`.

### Cartes premium

- Rotation 3D légère, reflet dynamique, ombre réaliste et verre contrôlé.
- Apparition progressive et interaction au survol/focus.
- Aucune information essentielle ne dépend uniquement du hover.

### Smartphones 3D

- Rotation, zoom et changement d’écran pilotés par timeline.
- Écrans fournis sous forme de textures optimisées.
- Chargement différé et poster 2D avant disponibilité du modèle.

### TPE 3D

- Présentation du terminal, approche d’une carte NFC, paiement simulé, confirmation et impression de reçu.
- Démonstration purement visuelle, clairement séparée d’un vrai flux transactionnel.
- Sons optionnels désactivés par défaut.

### Lumière et matières

- Glows, gradients animés, reflets, ombres et éclairage adaptatif.
- Intensité limitée pour conserver contraste et lisibilité.
- Pas de shaders coûteux sans mesure de performance.

### Micro-interactions

- Boutons, icônes, cartes, champs, menus, hover, focus, click, succès, erreur et chargement.
- Durées cohérentes et tokens centralisés.
- Feedback immédiat inférieur à 100 ms pour les interactions locales.

### Texte, icônes et compteurs

- Fade, slide, reveal, animation par mot ou lettre, compteur animé.
- Contenu final présent dans le DOM pour SEO et accessibilité.
- Les animations de lettres ne doivent pas casser la lecture des technologies d’assistance.

### Loader et transitions de pages

- Loader logo et progression seulement lorsque le chargement réel le justifie.
- Transitions sans écran blanc ni clignotement.
- Préchargement intelligent des routes probables, sans gaspillage réseau.

### Curseur personnalisé

- Desktop uniquement, pointeur système conservé comme fallback.
- Effets magnétiques limités aux éléments explicitement marqués.
- Désactivation automatique pour tactile, reduced-motion et appareils faibles.

## Architecture réutilisable

```text
packages/
  animation-core/
    gsap/
    motion/
    scroll/
    performance/
    accessibility/
  three-kit/
    canvas/
    lights/
    loaders/
    models/
    fallbacks/
  design-system/
    tokens/
    components/
    themes/
```

Composants et hooks attendus :

- `MotionProvider`
- `ReducedMotionProvider`
- `useDeviceCapability`
- `useGsapContext`
- `useScrollScene`
- `useMagneticPointer`
- `PageTransition`
- `RevealText`
- `AnimatedCounter`
- `ParallaxLayer`
- `ThreeSceneBoundary`
- `ModelWithFallback`
- `PhoneShowcase`
- `TerminalShowcase`

## Budget de performance

- Cible : fluidité proche de 60 FPS sur matériel compatible.
- Mesurer, ne pas supposer : FPS, long tasks, mémoire GPU, taille JS et temps d’interactivité.
- Chargement dynamique des scènes 3D et bibliothèques coûteuses.
- Compression Draco ou Meshopt, textures WebP/AVIF/KTX2 lorsque pertinent.
- Réduction du DPR sur mobile et appareils faibles.
- Pause automatique des animations hors écran ou onglet inactif.
- Pas de création d’objet ou de fonction lourde à chaque frame React.
- Utiliser `requestAnimationFrame` et nettoyer tous les listeners/timelines.

## Modes adaptatifs

- **Full** : 3D, parallax et effets complets.
- **Balanced** : 3D simplifiée, DPR réduit et moins de particules.
- **Lite** : fallbacks 2D, transitions courtes et aucune scène persistante.
- **Reduced motion** : animations décoratives supprimées, transitions instantanées ou discrètes.

Le mode est déterminé par préférence utilisateur, capacité appareil, économie d’énergie et mesure réelle.

## Accessibilité

- Navigation clavier complète.
- Focus visibles et ordre logique.
- ARIA seulement lorsqu’un élément natif ne suffit pas.
- Contrastes conformes WCAG AA au minimum.
- Commande utilisateur pour réduire ou désactiver les animations.
- Aucun scroll-jacking.
- Les canvas 3D décoratifs sont ignorés par les lecteurs d’écran ; une alternative textuelle décrit les démonstrations utiles.

## Documentation de chaque animation

Chaque composant animé doit préciser : objectif, bibliothèque, déclencheur, durée, easing, propriétés animées, coût estimé, stratégie responsive, reduced-motion, fallback, méthode de modification et tests.

## Critères d’acceptation

- Aucun flash blanc entre les routes principales.
- Aucun blocage du scroll ou de la navigation clavier.
- Les pages restent utilisables sans JavaScript 3D.
- Les composants coûteux ne sont pas inclus dans le bundle initial sans nécessité.
- Les animations s’arrêtent correctement au démontage.
- Les trois interfaces web partagent les primitives mais gardent leur identité fonctionnelle.
- Les audits Lighthouse, Web Vitals et profils de performance sont archivés avant mise en production.