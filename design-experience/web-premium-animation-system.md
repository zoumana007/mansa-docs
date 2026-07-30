# Système d’expérience web premium

## Portée

Cette spécification s’applique aux trois interfaces Next.js : `public-web`, `business-web` et `admin-web`. Le niveau de finition recherché est comparable aux meilleurs produits numériques modernes, sans copier l’identité, les textes, les assets ou les animations propriétaires d’une marque existante.

## Stack cible

- Next.js App Router, React, TypeScript et Tailwind CSS.
- Framer Motion pour les transitions d’interface et les micro-interactions déclaratives.
- GSAP et ScrollTrigger pour les séquences complexes contrôlées par le scroll.
- Lenis pour le smooth scroll, désactivable et compatible avec ScrollTrigger.
- Three.js, React Three Fiber et Drei pour les scènes 3D.
- Blender pour produire et optimiser les modèles GLB/GLTF.
- Spline uniquement lorsqu’il réduit réellement le coût de production sans créer une dépendance difficile à maintenir.

## Principes

1. L’animation sert la compréhension et ne bloque jamais une tâche critique.
2. Les pages restent utilisables sans JavaScript d’animation, sans WebGL et avec `prefers-reduced-motion`.
3. Chaque effet coûteux possède un fallback 2D.
4. Les animations de l’Admin sont plus discrètes que celles des sites marketing.
5. Le budget de performance est contrôlé par page et par classe d’appareil.

## Expériences obligatoires

### Hero cinématique

- Entrée progressive du logo, du titre, du texte et des actions.
- Mouvement léger de caméra ou de profondeur, jamais agressif.
- Éclairage, glow et dégradés animés limités au GPU.
- Boutons magnétiques sur desktop, avec fallback standard au clavier, tactile et reduced motion.

### Scroll storytelling

- Séquences reliées sans saut visuel.
- Pinning utilisé avec modération.
- Progression lisible même lorsque le scroll lissé est désactivé.
- Nettoyage systématique des timelines et listeners au démontage.

### Parallax et cartes

- Couches à vitesses différentes avec amplitudes limitées.
- Cartes avec rotation légère, reflets, ombres, glassmorphism et apparition progressive.
- Contraste et lisibilité conservés sur tous les fonds.

### Téléphones et TPE 3D

- Smartphone 3D : rotation, zoom, changement d’écran et présentation guidée des fonctions.
- TPE 3D : approche d’une carte NFC, traitement simulé, confirmation et impression visuelle d’un reçu.
- Aucun vrai paiement ne doit être déclenché depuis une démonstration marketing.
- Chargement différé des scènes, texture compressée, modèle simplifié sur mobile et fallback vidéo/image.

### Textes, icônes et micro-interactions

- Fade, slide, reveal, animation par mot ou lettre et compteurs.
- Hover, focus, pressed, loading, success et error pour boutons, cartes, inputs, menus et icônes.
- Les états fonctionnels restent distincts des effets décoratifs.

### Loader, curseur et navigation

- Loader court avec logo et progression réelle lorsque disponible.
- Curseur personnalisé uniquement sur périphériques précis avec pointeur fin.
- Transitions de page sans clignotement et sans masquer un chargement long.
- Préchargement intelligent limité aux routes et ressources probables.

## Architecture réutilisable

```text
packages/ui/
  animation/
    motion-provider.tsx
    reduced-motion.ts
    performance-tier.ts
    page-transition.tsx
    reveal.tsx
    magnetic-button.tsx
    animated-counter.tsx
  three/
    scene-boundary.tsx
    adaptive-canvas.tsx
    phone-model.tsx
    tpe-model.tsx
    fallback-media.tsx
  theme/
    tokens.ts
    theme-provider.tsx
```

Hooks attendus : `useReducedMotion`, `usePerformanceTier`, `useGsapContext`, `useScrollProgress`, `useMagneticPointer` et `usePageTransition`.

## Gestion des performances

- Viser 60 FPS sur appareils compatibles, sans en faire une promesse absolue.
- Mesurer les longues tâches, CLS, LCP, INP, mémoire et temps GPU lorsque disponible.
- Dynamic imports pour GSAP, Three.js et scènes lourdes.
- Compression Draco/Meshopt et textures KTX2 selon compatibilité.
- Limiter DPR, lumières, ombres, post-processing et nombre de draw calls.
- Suspendre les animations hors écran et lorsque l’onglet est masqué.
- Ne pas déclencher de mise à jour React à chaque frame ; utiliser refs et boucles dédiées.
- Utiliser `requestAnimationFrame`, `will-change` temporairement et transformations GPU.

## Adaptation automatique

Trois niveaux :

- `full` : WebGL et séquences complètes.
- `balanced` : modèles simplifiés, moins de particules et DPR réduit.
- `minimal` : fallback 2D, transitions courtes et aucune animation continue.

Le niveau dépend de reduced motion, type de pointeur, mémoire/appareil lorsque disponible, économie de données, batterie lorsque l’API est accessible et mesure réelle des performances.

## Accessibilité

- Navigation clavier complète et focus visible.
- ARIA uniquement lorsque nécessaire.
- Contrastes conformes et textes lisibles derrière les effets de verre.
- Aucune information uniquement transmise par mouvement ou couleur.
- Bouton de désactivation des animations persisté dans les préférences.
- Les scènes 3D possèdent un texte alternatif et une présentation statique équivalente.

## Documentation de chaque animation

Chaque composant animé doit indiquer : objectif, bibliothèque, déclencheur, durée, easing, dépendances, coût estimé, fallback, comportement reduced motion, points de modification et tests.

## Critères d’acceptation

- Aucun blocage de navigation au clavier.
- Aucun clignotement de route significatif.
- Aucune scène 3D chargée avant d’être utile.
- Fallback fonctionnel sans WebGL.
- Nettoyage des timelines, observers et listeners vérifié.
- Tests visuels desktop, tablette, mobile et reduced motion.
- Les tâches critiques de l’Admin restent rapides avec les animations désactivées.