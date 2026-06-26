# TACTICAL SHOOTER – OpenGL C++17
### Projet de cours – 50 points | Visual Studio 2022

---

## CRITÈRES SATISFAITS (50 pts)

| Critère                                    | Statut |
|--------------------------------------------|--------|
| Fichier OBJ avec UV-map                    | ✅ `models/gun.obj` – généré proc. avec coordonnées UV par face |
| Shaders séparés `.vert` / `.frag`          | ✅ 4 paires : `world`, `enemy`, `gun`, `particle` |
| Textures appliquées sur le modèle OBJ      | ✅ `gun_texture.png` (procédurale) via `gun.frag` + `uTexture` |
| Multi armes à feu (3 armes)                | ✅ Fusil d'assaut · Shotgun · Sniper |
| Logique de jeu (shoot / score / vies)      | ✅ Ennemis IA, HP, respawn, dégâts joueur |
| **HUD visible** : Temps · Score · Direction| ✅ Timer MM:SS · Score 6 chiffres · Compas N/S/E/W · Mini-map |

---

## ARBORESCENCE DU PROJET

```
TacticalShooter/
├── TacticalShooter.sln          ← ouvrir avec VS 2022
├── TacticalShooter.vcxproj
├── src/
│   ├── main.cpp                 ← code source unique (~750 lignes)
│   └── stb_image.h              ← chargement de textures PNG/JPG
├── shaders/
│   ├── world.vert / world.frag  ← sol, murs, caisses + brouillard
│   ├── enemy.vert / enemy.frag  ← ennemis + rim-light
│   ├── gun.vert  / gun.frag     ← modèle OBJ vue FPS + métal
│   └── particle.vert / particle.frag ← sang / flash bouche
├── models/
│   └── gun.obj + gun.mtl        ← généré automatiquement au lancement
└── textures/
    └── (optionnel : ground.png, wall.png, enemy.png, gun.png)
        Si absentes → textures procédurales générées en mémoire
```

---

## INSTALLATION – DÉPENDANCES

### Méthode recommandée : vcpkg (la plus simple)

```batch
git clone https://github.com/microsoft/vcpkg
cd vcpkg
bootstrap-vcpkg.bat
vcpkg install glfw3:x64-windows glew:x64-windows glm:x64-windows
vcpkg integrate install
```

Après `vcpkg integrate install`, Visual Studio détecte automatiquement  
les includes et les libs. **Aucun chemin à modifier.**

### Méthode manuelle (si pas de vcpkg)

1. Télécharger et installer :
   - **GLFW** : https://www.glfw.org/download.html (binaires 64-bit)
   - **GLEW** : https://glew.sourceforge.net/ (binaires 64-bit)
   - **GLM**  : https://github.com/g-truc/glm (header-only, copier le dossier `glm/`)

2. Dans `TacticalShooter.vcxproj`, modifier `<VCPKG_ROOT>` avec le bon chemin,  
   ou ouvrir les **Propriétés du projet** → C/C++ → Répertoires Include & Bibliothèque.

---

## COMPILATION

1. Ouvrir `TacticalShooter.sln` avec **Visual Studio 2022**
2. Sélectionner **Release | x64** (ou Debug)
3. **Ctrl+Shift+B** pour compiler
4. Le post-build copie automatiquement :
   - `glfw3.dll` et `glew32.dll` dans le dossier de sortie
   - les dossiers `shaders/`, `textures/`, `models/` dans le dossier de sortie

---

## CONTRÔLES EN JEU

| Touche               | Action                         |
|----------------------|--------------------------------|
| **W / A / S / D**    | Déplacement (ou flèches)       |
| **Souris**           | Visée                          |
| **Clic gauche**      | Tirer                          |
| **1 / 2 / 3**        | Changer d'arme                 |
| **R**                | Recharger                      |
| **P**                | Pause                          |
| **ESC**              | Quitter                        |

---

## ARMES

| # | Nom           | Cadence | Munitions | Dégâts | Particularité         |
|---|---------------|---------|-----------|--------|-----------------------|
| 1 | Assault Rifle | 8 c/s   | 30        | 25     | Automatique           |
| 2 | Shotgun       | 1.5 c/s | 6         | 15×8   | 8 projectiles / tir  |
| 3 | Sniper Rifle  | 1 c/s   | 10        | 90     | Précision max         |

---

## FONCTIONNALITÉS VISIBLES À L'ÉCRAN

```
┌───────────────────────────────────────────────────────┐
│ [SCORE 000150]    [ 01:23 ]    [KILLS 003]            │ ← TOP BAR
│                  DIR: NE  ← COMPAS AVEC FLÈCHES        │
├───────────────────────────────────────────────────────┤
│                                                       │
│  [MINI-MAP]  [HP ██████░░ 72]                         │ ← OVERLAY
│                                                       │
│                    +  ← VISEUR                        │ ← 3D VIEW
│                                                       │
│  [1 ASSAULT RIFLE  28/30]                             │ ← WEAPON HUD
│  [2 SHOTGUN]                                          │
│  [3 SNIPER RIFLE]                                     │
│                                        [DIR: NE 045°] │
└───────────────────────────────────────────────────────┘
```

---

## SHADERS – DESCRIPTION TECHNIQUE

### `world.vert / world.frag`
- **Éclairage Phong** (ambient + diffuse + specular Blinn-Phong)
- **Brouillard de distance** (linéaire, 60–120 m)
- Texture répétée via `uUseTexture`

### `enemy.vert / enemy.frag`
- Même base Phong
- **Rim-light** rougeâtre pour faire ressortir la silhouette ennemie

### `gun.vert / gun.frag`
- **Métal brillant** : SPEC_POW = 96, SPEC_STR = 0.70
- **Effet Fresnel** sur les bords (halo bleuté)
- Lit la texture UV du fichier OBJ

### `particle.vert / particle.frag`
- Particules sphériques avec masque de distance
- Glow additif en cœur (muzzle flash orange, sang rouge)
- Alpha fade basé sur la durée de vie

---

## MODÈLE OBJ – gun.obj

Le fichier `gun.obj` est **généré automatiquement** au premier lancement  
dans le dossier `models/`. Il représente un fusil d'assaut composé de  
7 pièces géométriques (corps, canon, crosse, chargeur, poignée, lunette,  
guidon avant) avec :

- **Coordonnées UV** pour chaque face (réseau 4×2 de tuiles)
- **Normales par face** (6 directions d'axe)
- **Matériau** lié à `gun_texture.png`

Pour remplacer par un vrai modèle 3D (Blender, etc.) :  
simplement écraser `models/gun.obj` – le chargeur OBJ est générique.

---

## LOGIQUE DE JEU

- **12 ennemis** avec machine à états : PATROL → CHASE (< 20m) → ATTACK (< 4m)
- **IA de patrouille** : cible aléatoire dans l'arène, rebond
- **Respawn** des ennemis tués après 4 secondes
- **Dégâts joueur** : -8 HP par attaque ennemie
- **Game Over** : temps écoulé (90s) ou HP = 0
- **Score** : +100 par kill, +10 par respawn ennemi
- **Précision** : shots / hits affichés en temps réel

---

## CRÉDITS / BIBLIOTHÈQUES

- **GLFW 3** – fenêtre et entrées – https://www.glfw.org
- **GLEW**  – chargement OpenGL – https://glew.sourceforge.net
- **GLM**   – mathématiques 3D – https://github.com/g-truc/glm
- **stb_image.h** – chargement PNG/JPG (domaine public) – https://github.com/nothings/stb
