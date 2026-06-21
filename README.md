# Mathidog 🦊

Application ludo-éducative de mathématiques pour les élèves de **CP (6-7 ans)**, conforme au programme de l'Éducation nationale française. Conçue pour être utilisée seul par un enfant qui commence à lire.

---

## Démarrage rapide

```bash
npm install
npm run dev        # Serveur de développement → http://localhost:5173
npm run build      # Build de production → dossier dist/
npm run preview    # Prévisualiser le build
```

> **Note** : les polices (Baloo 2 + Atkinson Hyperlegible) sont chargées depuis Google Fonts au premier lancement. Pour un fonctionnement hors-ligne dès le premier chargement, téléchargez-les et servez-les localement (voir `/public/manifest.json` et `src/styles/tokens.css`).

---

## Arborescence du projet

```
petitmath/
├── public/
│   ├── icons/               # Icônes PWA (192, 512, maskable)
│   ├── favicon.svg
│   ├── manifest.json        # Manifeste PWA
│   └── sw.js                # Service Worker (cache-first, hors-ligne)
│
├── src/
│   ├── types/
│   │   ├── curriculum.ts    # LevelId, Skill, ContentBlock, ExerciseQuestion…
│   │   ├── progress.ts      # AppProgress, SkillProgress, badges…
│   │   └── navigation.ts    # Union type Screen (machine à états)
│   │
│   ├── data/
│   │   ├── badges.ts        # Catalogue des 9 badges
│   │   └── levels/
│   │       ├── cp/          # Programme CP complet (18 compétences)
│   │       │   ├── domains.ts
│   │       │   ├── index.ts
│   │       │   └── skills/  # numeration, calcul, problemes, geometrie, mesures
│   │       ├── ce1/index.ts # Placeholder (available: false)
│   │       ├── ce2/index.ts
│   │       ├── cm1/index.ts
│   │       ├── cm2/index.ts
│   │       └── index.ts     # Registre central LEVELS + helpers
│   │
│   ├── engine/
│   │   ├── rng.ts           # Source d'aléa (Math.random ou graine)
│   │   ├── scoring.ts       # scoreToPercent, starsFromPercent, seuils
│   │   ├── frenchNumbers.ts # nombreEnLettres(n) 0-100
│   │   └── generators/      # 18 générateurs de questions (un par compétence)
│   │       ├── numeration.ts
│   │       ├── calcul.ts
│   │       ├── problemes.ts
│   │       ├── geometrie.ts
│   │       ├── mesures.ts
│   │       └── index.ts     # Registre GENERATORS + generateQuestion()
│   │
│   ├── store/
│   │   ├── storage.ts       # loadProgress / saveProgress (localStorage)
│   │   ├── progressStore.ts # Logique pure (applyAttempt, badges, déverrouillage…)
│   │   └── ProgressContext.tsx  # Context React + hook useProgress()
│   │
│   ├── hooks/
│   │   └── useSpeech.ts     # Web Speech API (TTS français, manuel)
│   │
│   ├── components/
│   │   ├── ui/
│   │   │   ├── BigButton.tsx   # Bouton "autocollant" (tactile, gros)
│   │   │   ├── index.tsx       # Card, ProgressBar, StarRating, TopBar…
│   │   │   └── Mascot.tsx      # Renard Mathéo (SVG, 4 expressions)
│   │   ├── visuals/
│   │   │   └── ContentBlockView.tsx  # Rendu de tous les types ContentBlock
│   │   └── exercise/
│   │       ├── ExerciseRunner.tsx    # Orchestrateur des 4 phases pédagogiques
│   │       ├── QuestionView.tsx      # Affichage d'une question + indice
│   │       ├── AnswerChoices.tsx     # Réponses à choix multiples
│   │       ├── NumberPad.tsx         # Clavier numérique tactile
│   │       └── ResultSummary.tsx     # Récapitulatif de fin de série
│   │
│   ├── screens/
│   │   ├── OnboardingScreen.tsx    # Saisie du prénom (1er lancement)
│   │   ├── HomeScreen.tsx          # Accueil (Continuer / Exercices / …)
│   │   ├── ExercisesMenuScreen.tsx # Choix du domaine
│   │   ├── SkillListScreen.tsx     # Parcours "marelle" des compétences
│   │   ├── ExerciseSessionScreen.tsx
│   │   ├── RevisionScreen.tsx      # Re-pratiquer les compétences débloquées
│   │   ├── ProgressScreen.tsx      # Compétences, badges, historique
│   │   └── CertificateScreen.tsx   # Certificat de fin de niveau
│   │
│   ├── styles/
│   │   ├── tokens.css     # Variables CSS (couleurs, typo, espacements…)
│   │   ├── animations.css # Animations sobres (prefers-reduced-motion OK)
│   │   └── global.css     # Reset + styles de base
│   │
│   ├── utils/
│   │   ├── array.ts   # shuffle, pick, randInt, range, uniq
│   │   ├── date.ts    # nowIso, formatDateFr, formatDateTimeFr
│   │   └── id.ts      # uid() — identifiants légers pour les questions
│   │
│   ├── App.tsx        # Machine à états Screen (navigation sans routeur)
│   ├── main.tsx       # Point d'entrée React + enregistrement SW
│   └── vite-env.d.ts
│
├── index.html
├── vite.config.ts
├── tsconfig.json
└── package.json
```

---

## Architecture pédagogique

### Les 18 compétences CP

| Domaine | Compétences |
|---------|------------|
| 🔢 Nombres | Reconnaître · Lire · Écrire · Comparer/Ranger · Dizaines & Unités · Suite jusqu'à 100 |
| ➕ Calcul | Additions · Soustractions · Doubles · Moitiés · Compléments à 10 |
| 🧩 Problèmes | Problèmes additifs · Problèmes soustractifs |
| 🔺 Formes | Reconnaître les formes · Côtés et coins |
| ⏰ Mesures | L'heure · La monnaie · Les longueurs |

### Les 4 phases pédagogiques (par compétence)

1. **Explication** — présentation très courte avec la mascotte Mathéo
2. **Exemple illustré** — une situation concrète résolue pas à pas
3. **Entraînement guidé** — 3-4 questions avec indice toujours visible
4. **Exercice autonome** — 6-8 questions chronométrées, score comptabilisé

### Système de progression

- **Étoiles** : 0→1 (≥40 %), 1→2 (≥60 %), 2→3 (≥90 %)
- **Déverrouillage** : la compétence suivante s'ouvre à ≥60 %
- **Maîtrise** : compétence "validée" à ≥90 %
- **Badges** : 9 badges basés sur l'apprentissage (jamais sur la régularité quotidienne)

---

## Ajouter le niveau CE1 (et suivants)

1. **Créer les compétences** dans `src/data/levels/ce1/skills/`  
   (copier la structure de `src/data/levels/cp/skills/` et adapter le contenu)

2. **Écrire les générateurs** dans `src/engine/generators/`  
   et les enregistrer dans `src/engine/generators/index.ts`

3. **Mettre à jour `src/data/levels/ce1/index.ts`** :
   ```ts
   export const CE1_CURRICULUM: LevelCurriculum = {
     levelId: 'CE1',
     available: true,      // ← passer à true
     domains: DOMAINS,
     skills: [...CE1_NUMERATION, ...CE1_CALCUL, ...],
   };
   ```

Aucun autre fichier n'a besoin d'être modifié : les écrans, le moteur d'exercices et le système de progression sont entièrement génériques.

---

## Choix techniques

| Aspect | Décision |
|--------|----------|
| Navigation | Machine à états React (`useState<Screen>`) — sans dépendance à un routeur |
| Persistance | `localStorage` (`petitmath:progress:v1`) avec gestion d'erreur silencieuse |
| Hors-ligne | Service Worker cache-first (`public/sw.js`) — pas de workbox |
| TTS | Web Speech API, fr-FR, **jamais automatique** (bouton écouter) |
| Génération de questions | Fonctions pures + `Math.random` — pratique infinie sans base de données |
| Gamification | Badges uniquement liés à l'apprentissage — **aucun mécanisme addictif** |
| Polices | Baloo 2 (titres/chiffres) + Atkinson Hyperlegible (texte, facilite la lecture) |
| Icônes | Émojis système — aucune dépendance externe |

---

## Déploiement

Le build de production (`npm run build`) produit un dossier `dist/` **portable** (base `./`) : il peut être servi depuis n'importe quel sous-dossier ou ouvert directement depuis une clé USB (mode fichier local).

```bash
# Exemple : déploiement GitHub Pages
npm run build
# puis pousser le contenu de dist/ sur la branche gh-pages
```
