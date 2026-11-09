# HA Blueprints

Blueprints Home Assistant réutilisables, maintenus par [Glawnn](https://github.com/Glawnn).

## Blueprints disponibles

| Blueprint | Description |
|---|---|
| [Hue Dimmer v2 — On/Off + luminosité](blueprints/automation/hue_dimmer_v2_onoff.yaml) | Marche/arrêt, variation de luminosité et comportements d'appui court/long configurables (deux cibles) avec une télécommande Philips Hue Dimmer Switch v2 (Zigbee2MQTT). |

## Installation

[![Ouvrir votre instance Home Assistant et afficher le dialogue d'import du blueprint.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FGlawnn%2Fha.blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fhue_dimmer_v2_onoff.yaml)

Ou manuellement :

1. **Paramètres → Automatisations et scènes → Blueprints**
2. **Importer un blueprint**
3. Coller l'URL du fichier sur GitHub (forme `github.com/.../blob/...`)
4. **Preview**, puis **Importer**

## Hue Dimmer v2 — On/Off + luminosité

Contrôle une lumière, un groupe, une zone ou un appareil avec une télécommande
Philips Hue Dimmer Switch v2 connectée via **Zigbee2MQTT**.

Le formulaire est organisé en sections : **Télécommande**, **Cibles**,
**Appui court**, **Appui long** et **Réglages avancés** (repliée).

### Inputs

| Input | Type | Défaut | Description |
|---|---|---|---|
| `remote` | appareil | — | Télécommande Hue Dimmer Switch v2 (filtrée sur l'intégration MQTT). |
| `target` | cible | — | Cible 1 : lumière(s), zone, appareil ou étiquette à contrôler. |
| `target_2` | cible | vide | Cible 2 optionnelle, utilisée par les comportements qui la mentionnent. |
| `short_press` | menu | `toggle_target` | Comportement de l'appui court sur le bouton haut. |
| `short_press_action` | action | vide | Action libre, affichée seulement si `short_press` = `custom`. |
| `long_press` | menu | `on_target` | Comportement de l'appui long sur le bouton haut. |
| `long_press_action` | action | vide | Action libre, affichée seulement si `long_press` = `custom`. |
| `transition` | nombre (s) | `0.5` | Durée de transition marche/arrêt et variation. |
| `step_pct` | nombre (%) | `10` | Variation de luminosité par appui court et par pas de maintien. |
| `hold_delay` | nombre (s) | `0.4` | Intervalle entre deux pas de luminosité pendant un maintien (0,1 à 2 s). Plus la valeur est faible, plus la variation est rapide. |

### Comportements disponibles

Les deux menus (`short_press` et `long_press`) proposent les mêmes choix :

| Valeur | Effet |
|---|---|
| `toggle_target` | Bascule la cible 1. |
| `toggle_target_2` | Bascule la cible 2. |
| `on_target` | Allume la cible 1. |
| `on_target_2` | Allume la cible 2. |
| `off_target` | Éteint la cible 1. |
| `off_target_2` | Éteint la cible 2. |
| `on_both` | Allume les cibles 1 et 2. |
| `off_both` | Éteint les cibles 1 et 2. |
| `master` | Maître de pièce : si **au moins une** lumière des cibles 1 et 2 est allumée, éteint les deux ; sinon allume la cible 1. |
| `custom` | Exécute l'action personnalisée correspondante (`short_press_action` ou `long_press_action`). |

Par défaut : appui court = `toggle_target`, appui long = `on_target`
(comportement des versions précédentes).

### Comportement des boutons

| Bouton | Appui court | Maintien |
|---|---|---|
| Haut | Comportement `short_press` (`on_press_release`) | Comportement `long_press` (`on_hold`) |
| + | +1 pas de luminosité | +1 pas toutes les `hold_delay` secondes |
| − | −1 pas de luminosité | −1 pas toutes les `hold_delay` secondes |
| Hue (bas) | Non utilisé | Non utilisé |

- Si la cible est éteinte : `+` l'allume à hauteur d'un pas ; `−` n'a aucun effet.
- Descendre jusqu'à 0 % éteint la cible (comportement natif de Home Assistant).
- La variation `+/−` agit toujours sur la cible 1.
- Si `target_2` est vide, les comportements qui la visent ne font rien ; `master`
  et `on_both`/`off_both` se limitent alors à la cible 1.
- Le premier appui long peut être ignoré le temps que Zigbee2MQTT publie la
  découverte du déclencheur `on_hold` ; maintenir une seconde fois si nécessaire.

### Exemple : pièce avec plafonnier et appliques

| Input | Valeur |
|---|---|
| `target` | Plafonnier (`light.salon_lampe_plafonier`) |
| `target_2` | Zone `salon` (appliques, ajoutées plus tard) |
| `short_press` | `master` |
| `long_press` | `on_both` |

Ainsi : un appui court éteint plafonnier + appliques si quelque chose est
allumé, sinon il n'allume que le plafonnier ; un appui long allume les deux.

### Prérequis

- Home Assistant **2026.11.0** ou plus récent (sections d'inputs et affichage
  conditionnel `visible`).
- Télécommande appairée via **Zigbee2MQTT 2.x** (device triggers MQTT autodécouverts).
- Les événements utilisés sont `on_press_release`, `on_hold`, `up_press`,
  `up_hold`, `up_press_release`, `up_hold_release`, `down_press`, `down_hold`,
  `down_press_release` et `down_hold_release`.
  Si votre firmware n'émet que `*_press`, adaptez les `subtype` du blueprint.
- Le maintien démarre à la réception du premier `*_hold` ; un appui court
  applique donc toujours exactement un pas.

## Mise à jour d'un blueprint

1. Le blueprint est modifié puis poussé sur la branche `main`.
2. Dans Home Assistant : **Paramètres → Automatisations et scènes → Blueprints**.
3. Menu ⋮ du blueprint → **Ré-importer le blueprint**.
4. Si nécessaire, recharger les automatisations
   (**Outils de développement → YAML → Automatisations**).

Les automatisations existantes récupèrent la nouvelle version automatiquement,
sans reconfiguration, tant que les clés d'input restent compatibles.

## Compatibilité

- Les clés d'input sont une API publique. La v6 remplace l'input éphémère
  `hold_target` de la v5 par `target_2` (même rôle, généralisé) ; il n'était
  pas utilisé par défaut dans les automatisations v5. Ensuite, les clés ne sont
  plus renommées ni supprimées : les nouveaux inputs sont ajoutés avec une
  valeur par défaut.
- `source_url` reste stable : c'est l'ancre de mise à jour.
- Un blueprint ne peut être importé que depuis un dépôt **public**.

## Licence

[MIT](LICENSE)
