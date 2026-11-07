# Orbito

Overlay « morceau en cours » pour **OBS**, simple et entièrement personnalisable. Orbito lit ce que Windows sait déjà de ta musique : **aucun compte, aucune clé, aucune connexion**.

- Titre, artiste, album, pochette et barre de progression en temps réel
- Pochette ronde qui tourne en lecture, effet vinyle, dégradés, ombres, contours
- Toutes les polices installées sur ton PC, plus de soixante réglages et 8 thèmes prêts à l'emploi
- Masquer / afficher l'overlay par raccourci clavier ou par Stream Deck
- Mises à jour automatiques

> Windows 10 / 11 uniquement. Fonctionne avec Spotify, les navigateurs, VLC, foobar2000, Apple Music et la plupart des lecteurs.

## Technologies utilisées

| Couche | Technologie | Rôle |
|---|---|---|
| Lecture du morceau | **Python 3.12** + [`winsdk`](https://github.com/pywinrt/python-winsdk) | Lit le « contrôle global de lecture » de Windows (SMTC) : titre, artiste, album, état, position, durée, pochette |
| Serveur local | `http.server` (bibliothèque standard) | Sert l'interface, l'overlay et une petite API JSON sur `127.0.0.1:8765` |
| Boucle de lecture | `asyncio` dans un thread dédié | Interroge Windows une fois par seconde |
| Interface de réglages | **React 18** + **Vite 5** | Panneau de configuration avec aperçu en direct |
| Overlay OBS | HTML / CSS / JavaScript natifs | Page affichée par la source Navigateur d'OBS |
| Fenêtres | [`pywebview`](https://pywebview.flowrl.com) (WebView2) | Fenêtre de réglages et écran de démarrage |
| Zone de notification | [`pystray`](https://github.com/moses-palmer/pystray) + `Pillow` | Icône, menu clic droit, notifications |
| Liste des polices | `ctypes` + GDI (`EnumFontFamiliesExW`) | Énumère les polices réellement installées |
| Raccourci clavier global | `RegisterHotKey` (user32) | Fonctionne même quand Orbito est réduit ou qu'un jeu a le focus |
| Démarrage avec Windows | `winreg` | Active ou désactive le lancement automatique |
| Mises à jour | API GitHub Releases | Détecte et installe la dernière version |
| Distribution | **PyInstaller** (`--onefile`) | Un unique `Orbito.exe`, interface embarquée |

### Pourquoi pas l'API Spotify ?

Windows expose à toutes les applications multimédia un « contrôle global de lecture » (celui des touches média et du volet de volume). Spotify, mais aussi les navigateurs, VLC, foobar2000 ou Apple Music y publient le morceau en cours. Orbito se branche dessus : ça fonctionne hors ligne, sans authentification, sans quota et avec n'importe quel lecteur compatible. Si Spotify est présent il est prioritaire, sinon Orbito suit la session de lecture active de Windows.

### Comment fonctionne l'overlay

L'interface enregistre la configuration, l'overlay la relit chaque seconde et se met à jour tout seul : aucun WebSocket. Les réglages sont appliqués par JavaScript sur des styles et variables CSS, la progression est interpolée entre deux relevés Windows pour rester fluide, et la rotation de la pochette est une animation CSS que l'on met seulement en pause (elle garde son angle). Le serveur n'écoute que sur `127.0.0.1` : rien n'est exposé sur le réseau.

## Démarrage

1. Lance `Orbito.exe`. Un écran de démarrage s'affiche, puis la fenêtre de réglages s'ouvre.
2. Choisis un thème dans la colonne de gauche et ajuste les réglages : l'aperçu se met à jour en direct.
3. Garde Orbito ouvert pendant que tu streames. Fermer la fenêtre le réduit dans la zone de notification (clic droit sur l'icône pour quitter).

## Ajouter l'overlay dans OBS

1. Ajoute une source **Navigateur**
2. Colle `http://127.0.0.1:8765/overlay` (ne coche pas « Fichier local »)
3. Largeur 900, hauteur 260

La position et la taille se règlent dans Orbito : inutile de redimensionner la source dans OBS. Les polices sont celles du PC où tourne OBS.

## Masquer l'overlay

- **Raccourci clavier** : clique sur le champ, appuie sur ta combinaison, elle est enregistrée aussitôt (par défaut `Ctrl + Alt + H`). Si elle est déjà prise par un autre programme, un message te le dit : choisis-en une autre.
- **Stream Deck** : action *Site web* (catégorie Système), colle une adresse ci-dessous et coche l'accès en arrière-plan (requête GET).
- **Icône de notification** : clic droit puis *Masquer l'overlay*.

| Adresse | Effet |
|---|---|
| `http://127.0.0.1:8765/api/overlay/toggle` | Bascule masqué / affiché |
| `http://127.0.0.1:8765/api/overlay/hide` | Masque |
| `http://127.0.0.1:8765/api/overlay/show` | Affiche |

L'overlay est de nouveau visible au prochain lancement.

## Réglages

| Section | Ce que tu peux changer |
|---|---|
| Cadre | fond, dégradé, contour, arrondi, largeur, marges, ombre |
| Pochette | taille, forme, côté, vinyle, contour, ombre, rotation |
| Texte | police, couleurs, tailles, graisses, majuscules, alignement, défilement |
| Temps et barre | temps écoulé / restant, position et style de la barre |
| Affichage | animation d'entrée, durée de visibilité, échelle, opacité, position |

Thèmes : Orbito, Classique, Verre, Néon, Minimal, Clair, Vinyle, Aurore. Les boutons **Exporter** et **Importer** sauvegardent ou restaurent toute ta configuration. Elle est enregistrée dans `%APPDATA%\Orbito\config.json`.

## Mises à jour

Orbito vérifie au démarrage s'il existe une nouvelle version et l'installe tout seul (désactivable dans le panneau *Mises à jour*). Pendant que tu l'utilises, il te prévient seulement : il ne redémarre jamais en plein stream. Le bouton vert en bas à gauche permet de vérifier à la demande.

## Dépannage

- **L'overlay reste vide dans OBS** : vérifie qu'Orbito est lancé et qu'un morceau est en lecture.
- **« Le port 8765 est déjà utilisé »** : une autre copie d'Orbito tourne encore. Quitte-la depuis la zone de notification ou le Gestionnaire des tâches.
- **Le raccourci ne réagit pas** : un autre programme l'utilise peut-être, choisis une autre combinaison.
- **La pochette est floue** : sa qualité dépend de ce que le lecteur fournit à Windows.
