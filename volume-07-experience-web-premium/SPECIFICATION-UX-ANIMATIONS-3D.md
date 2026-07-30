# Spécification UX premium, animations et 3D

## Périmètre

Cette spécification s’applique au site public Mansa, au site professionnel Mansa Business et, avec sobriété, au portail Admin Web. L’objectif est une expérience originale, immersive, rapide et accessible, inspirée du niveau de finition des meilleurs produits numériques modernes sans copier leur identité.

## Stack cible

Next.js App Router, React, TypeScript, Tailwind CSS, GSAP, ScrollTrigger, Lenis, Framer Motion, Three.js, React Three Fiber, Drei, modèles Blender optimisés et Spline seulement lorsqu’il réduit réellement le coût de production sans enfermer l’architecture.

## Système d’animation

- Registre central des animations et durées.
- Hooks `useReducedMotion`, `useDeviceCapability`, `useGsapContext`, `useScrollProgress` et `usePageTransition`.
- Composants réutilisables : `Reveal`, `StaggerText`, `MagneticButton`, `ParallaxLayer`, `AnimatedCounter`, `PageTransition`, `LazyScene3D`, `PhoneShowcase`, `TpeShowcase` et `PremiumLoader`.
- Nettoyage obligatoire de chaque timeline, listener et `ScrollTrigger` au démontage.
- Interdiction d’animer les propriétés qui déclenchent inutilement layout/paint lorsqu’un transform ou une opacité suffit.

## Expériences obligatoires

### Hero cinématique

Entrée progressive du logo, du titre et des appels à l’action. Mouvement léger de caméra, éclairage adaptatif, profondeur et contenu immédiatement compréhensible. Le LCP ne doit pas dépendre du chargement de la scène 3D.

### Scroll storytelling

Le défilement pilote les chapitres, transformations, changements d’écran et transitions. Les sections se chevauchent visuellement sans saut. Les états restent déterministes au retour arrière.

### Parallax multicouche

Premier plan, contenu, arrière-plan et lumière évoluent à des vitesses différentes. L’amplitude diminue sur tablette et mobile et disparaît en mode mouvement réduit.

### Cartes premium

Rotation légère, ombre, reflet, verre, profondeur et halo au survol. L’interaction reste possible au clavier ; l’effet suit le focus sans exiger un pointeur.

### Smartphone 3D

Rotation, zoom, changement d’écran et présentation des parcours Client, Commerçant, Admin Lite et Annuaire. Les captures sont des textures optimisées et remplaçables depuis le CMS ou un manifeste versionné.

### TPE 3D

Approche d’une carte NFC, lecture, traitement, confirmation et impression simulée du reçu. La démonstration ne doit jamais ressembler à une transaction réelle ni contenir de donnée sensible.

### Lumière et micro-interactions

Glow, gradients animés, reflets, ombres réalistes, boutons magnétiques, focus, click feedback, menus, inputs, icônes et cartes. Chaque effet a un état statique de secours.

### Texte et chiffres

Fade, slide, reveal, animation par mots ou lettres et compteurs. Le texte sémantique existe dans le DOM avant l’animation et reste lisible par les technologies d’assistance.

### Loader et transitions de pages

Loader logo + progression seulement lorsque nécessaire. Préchargement ciblé, transition sans flash, conservation du focus et annonce de navigation aux lecteurs d’écran.

### Curseur personnalisé

Desktop uniquement, avec détection de pointeur fin. Désactivation automatique sur mobile, mouvement réduit, économie d’énergie ou appareil peu puissant.

## Performance

- Objectif : animation proche de 60 FPS lorsque réaliste.
- Budget initial recommandé : JavaScript marketing critique limité, scènes 3D chargées dynamiquement, modèles compressés DRACO/Meshopt, textures WebP/AVIF/KTX2 selon compatibilité.
- Utiliser `next/image`, dynamic imports, lazy loading, code splitting, cache, préchargement sélectif et `requestAnimationFrame`.
- Aucun rendu React par frame pour une animation continue ; préférer refs et moteurs dédiés.
- Mesurer LCP, INP, CLS, poids JS, mémoire GPU, nombre de draw calls et temps de frame.
- Prévoir trois niveaux : `full`, `reduced`, `static` selon préférences et capacité.
- Fallback 2D obligatoire pour chaque scène 3D importante.

## Responsive et accessibilité

Clavier complet, focus visible, ARIA, contrastes, ordre de lecture logique, zoom, textes redimensionnables et commandes désactivant les animations. `prefers-reduced-motion` est respecté sans exception. Les contenus essentiels ne reposent jamais uniquement sur mouvement, couleur ou profondeur.

## Documentation de chaque animation

Chaque composant doit indiquer : objectif, bibliothèque, déclencheur, durée, easing, propriétés animées, stratégie de cleanup, coût estimé, comportement responsive, mode réduit, fallback statique, tests et procédure de modification.

## Critères d’acceptation

1. Aucun flash ou saut visible entre pages et sections.
2. Aucun contenu essentiel n’est bloqué par WebGL.
3. Le site reste pleinement utilisable sans animation.
4. Les scènes lourdes ne chargent pas avant nécessité.
5. Les animations ne créent pas de re-rendu React par frame.
6. Les tests contrôlent le mode mouvement réduit.
7. Les performances sont vérifiées sur mobile moyen de gamme, pas seulement sur ordinateur haut de gamme.
8. Les modèles 3D, textures et captures ne contiennent aucune marque tierce non autorisée.