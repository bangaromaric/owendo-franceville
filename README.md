# Owendo vers Franceville, un voyage ferroviaire en 3D

Voyage interactif en 3D sur la ligne ferroviaire gabonaise, d'Owendo-Virié au PK 0
jusqu'à Franceville au PK 648, avec arrêt et fiche documentaire à chaque gare.

Démo : https://owendo-franceville.web.app

Une page HTML autonome, sans build ni dépendance locale. Three.js est chargé depuis un CDN,
tout le reste est généré en code : relief, voie, ponts, forêt, train, gares, habitants,
panneaux de gare. Aucune image ni modèle 3D externe.

## Pourquoi ce dépôt

La scène a été produite par **Gemini 4 Argon** à partir d'un prompt unique, puis auditée et
corrigée. L'intérêt n'est pas la démo : c'est l'écart entre un code qui tourne du premier coup
et un code juste. Le tableau des défauts plus bas est le vrai contenu de ce dépôt.

Le prompt complet est dans [PROMPT.md](PROMPT.md), avec la liste des défauts passés malgré une
consigne explicite.

## Les données

Source de référence : [Setrag](https://setrag.eramet.com/), exploitant de la ligne.
La ligne fait **648 km**, compte **24 gares** et traverse **5 provinces**. Le train express ne dessert que
les 7 gares principales ; le train omnibus dessert en plus les 17 gares secondaires. Cette
démo reprend le parcours de l'express, plus Ntoum.

| Gare | PK | Type | Province | Population |
|---|---|---|---|---|
| Owendo-Virié | 0 | principale | Estuaire | 80 000 ~ |
| Ntoum | 35 ~ | secondaire | Estuaire | 30 000 ~ |
| Ndjolé | 182 ~ | principale | Moyen-Ogooué | 10 000 ~ |
| Lopé | 290 ~ | principale | Ogooué-Ivindo | 2 000 ~ |
| Booué | 338 ~ | principale | Ogooué-Ivindo | 6 000 ~ |
| Lastourville | 484 ~ | principale | Ogooué-Lolo | 11 000 ~ |
| Moanda | 619 | principale | Haut-Ogooué | 45 000 ~ |
| Franceville | 648 | principale | Haut-Ogooué | 110 000 ~ |

Le signe `~` marque une valeur approximative et non vérifiée. Il apparaît tel quel dans
l'interface, avec une infobulle. Seuls les PK d'Owendo, Moanda et Franceville sont sourcés.
**Les populations ne le sont pas** et demandent une relecture avant d'être citées ailleurs.

L'exploitant écrit « Lastourville », la commune s'écrit généralement « Lastoursville ». La
graphie retenue est celle de la gare.

## Ce que l'audit a relevé

Défauts constatés en exécutant la page et en relisant le code, sur les versions successives
produites par le modèle.

| Défaut | Symptôme | Cause |
|---|---|---|
| Longueur de ligne fausse de 21 km | PK final à 669 au lieu de 648 | Donnée inventée, sous un commentaire affirmant « DONNÉES VÉRIFIÉES » |
| Deux sources de vérité pour la distance | Jusqu'à 39 km d'écart entre le compteur et la fiche de gare | Champs `pk` et `t` maintenus indépendamment |
| Train télescopé au départ | Les 5 véhicules superposés en un seul à Owendo | Position des voitures bornée par `Math.max(0, ...)`, faute de voie en amont du PK 0 |
| Voyageurs du mauvais côté de la voie | Quai vide, personnages debout sur la terre battue | `lookAt` construit l'axe X local comme l'opposé du vecteur latéral, et le décalage était appliqué dans le mauvais sens |
| Torse blanc dupliqué en gare | Deux torses superposés sur chaque voyageur à l'arrêt | `InstancedMesh` porteur d'ombre sans couleur d'instance, torse principal jamais masqué |
| Durée réelle 40 % au-dessus de l'annonce | 4 min 10 au lieu des 3 min visées | Décélération sur 18 km avant chaque gare, non compensée dans la vitesse de croisière |
| Mini-carte recalculée à chaque image | Jusqu'à 120 calculs de position par image pour un tracé statique | Aucune mise en cache du fond de carte |

Corrections d'aspect apportées dans la foulée : densité de la forêt portée de 4 200 à 11 000
tirages sur la section de forêt dense, houppiers des fromagers bâtis en trois strates au lieu
d'un disque unique, palmiers en cône inversé plutôt qu'en cône de conifère, luminosité relevée
sans perdre l'ambiance de fin d'après-midi, et cadrage de la caméra de cabine revu.

## Lancer en local

Un simple `file://` ne suffit pas : la page utilise un module ES et un import map.

```bash
python -m http.server 8000
# puis http://127.0.0.1:8000/index.html
```

## Déployer

Le site Firebase visé est déclaré dans `firebase.json`. Seul le projet reste à sélectionner,
`.firebaserc` n'étant pas versionné :

```bash
firebase use <votre-projet-firebase>
firebase deploy --only hosting
```

Pour partager une version de travail sans toucher au domaine public :

```bash
firebase hosting:channel:deploy preview --expires 7d
```

## Commandes de la démo

| Action | Commande |
|---|---|
| Changer de caméra | bouton Caméra, cabine ou drone |
| Vitesse | x1, x2, x4 |
| Pause, gare précédente | boutons dédiés |
| Afficher ou masquer les habitants | bouton Habitants, avec le nombre d'objets rendus |
| Mode cinéma, masque l'interface | touche `C` |

## Limites connues

- La géographie est évocatrice, pas cartographique : les méandres de la voie et le relief sont
  procéduraux, seules les distances au PK sont respectées.
- Les populations affichées sont des ordres de grandeur non sourcés.
- La vue drone peut démarrer à l'intérieur d'une colline, son placement initial ne tient pas
  compte du relief. Un mouvement de souris suffit à en sortir.

## Mention

Projet personnel, à but documentaire et pédagogique, sans aucun lien ni affiliation avec
l'exploitant de la ligne ou avec les entreprises citées. Les noms commerciaux éventuellement
mentionnés appartiennent à leurs titulaires respectifs et ne figurent ici qu'au titre de la
source des données. Aucune signalétique, aucun logo et aucune identité visuelle d'un tiers
n'est reproduit dans la démo.

## Licence

MIT.
