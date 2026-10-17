# Le prompt

La scène a été produite par **Gemini 4 Argon** à partir du prompt ci-dessous, en une seule
passe. Il est reproduit intégralement, pour être réutilisable tel quel.

Deux partis pris expliquent sa longueur.

**Les données factuelles sont fournies, pas demandées.** Une première tentative demandait au
modèle de vérifier lui-même les gares par recherche web avant de coder. Il a produit un en-tête
`// DONNÉES VÉRIFIÉES` au-dessus de chiffres qu'il n'avait pas vérifiés, dont une longueur de
ligne fausse de 21 km. Fournir le jeu de données vérifié en entrée supprime entièrement ce
risque.

**Chaque exigence de performance est justifiée par son mécanisme**, pas par un symptôme. On
n'écrit pas « ne fais pas ramer la mini-carte » mais « le tracé complet est statique, ne le
redessine pas à chaque image ». Un modèle applique une contrainte qu'il comprend, il contourne
une contrainte qu'il subit.

---

## Prompt de génération

```
Tu es développeur 3D web senior. Produis une application web complète, en un seul fichier HTML
autonome : un voyage en 3D à bord du Transgabonais, de Libreville à Franceville. Public visé :
gabonais. L'exactitude des faits compte autant que la qualité du rendu.

N'effectue aucune recherche web. Toutes les données factuelles dont tu as besoin sont fournies
ci-dessous, avec leur niveau de confiance. N'en invente aucune autre.

================================================
1. DONNÉES DE RÉFÉRENCE
================================================

La ligne, selon Setrag, l'exploitant :
- 648 km d'Owendo à Franceville, 24 gares, 5 provinces traversées.
- Owendo est la gare qui dessert Libreville, à environ 7 km au sud de la ville. Son nom
  officiel est Owendo-Virié. C'est le PK 0.
- Le Train Express « Trans-Ogooué » ne dessert que les 7 gares principales. Le Train Omnibus
  « l'Équateur » dessert en plus les 17 gares secondaires. Cette application simule le
  Trans-Ogooué.
- Premier tronçon mis en service en 1978, terminus de Franceville atteint en décembre 1986.

Utilise exactement ce jeu de données, sans le modifier :

const TOTAL_KM = 648;

const STATIONS = [
  { nom: "Owendo-Virié", pk: 0,   pkSur: true,  type: "principale", province: "Estuaire",
    pop: 80000,  popSure: false,
    fait: "Gare de Libreville et PK 0 de la ligne. Siège de Setrag et atelier principal, à
           côté des ports minéralier et grumier d'où partent le manganèse et le bois." },
  { nom: "Ntoum",        pk: 35,  pkSur: false, type: "secondaire", province: "Estuaire",
    pop: 30000,  popSure: false,
    fait: "Gare secondaire desservie par le Train Omnibus l'Équateur. Pôle de cimenteries et
           de carrières, à la transition entre la région de Libreville et la grande forêt." },
  { nom: "Ndjolé",       pk: 182, pkSur: false, type: "principale", province: "Moyen-Ogooué",
    pop: 10000,  popSure: false,
    fait: "Poste fondé par Savorgnan de Brazza en 1883 au pied des premiers rapides de
           l'Ogooué. Samory Touré, almami du Wassoulou, y est mort en exil en 1900." },
  { nom: "Lopé",         pk: 290, pkSur: false, type: "principale", province: "Ogooué-Ivindo",
    pop: 2000,   popSure: false,
    fait: "Porte du Parc national de la Lopé, inscrit au patrimoine mondial de l'UNESCO.
           Mosaïque unique de forêt et de savane, mandrills et gravures rupestres." },
  { nom: "Booué",        pk: 338, pkSur: false, type: "principale", province: "Ogooué-Ivindo",
    pop: 6000,   popSure: false,
    fait: "Carrefour forestier dans une boucle de l'Ogooué, en amont des chutes. Atelier de
           maintenance de Setrag." },
  { nom: "Lastourville", pk: 484, pkSur: false, type: "principale", province: "Ogooué-Lolo",
    pop: 11000,  popSure: false,
    fait: "Sur les rives de l'Ogooué, connue des spéléologues pour sa quarantaine de grottes
           dolomitiques ornées de vestiges préhistoriques. Setrag écrit Lastourville, la
           commune s'écrit souvent Lastoursville." },
  { nom: "Moanda",       pk: 619, pkSur: true,  type: "principale", province: "Haut-Ogooué",
    pop: 45000,  popSure: false,
    fait: "Capitale du manganèse, exploité par la Comilog. Le minerai parcourt les 619 km
           jusqu'à Owendo : c'est la raison d'être économique du Transgabonais." },
  { nom: "Franceville",  pk: 648, pkSur: true,  type: "principale", province: "Haut-Ogooué",
    pop: 110000, popSure: false,
    fait: "Terminus oriental, atteint en décembre 1986. Ville des plateaux batéké, siège de
           l'université USTM et du centre de recherche CIRMF." }
];

Règles d'affichage de ces données, impératives :
- Les PK marqués pkSur: false et toutes les populations sont des ordres de grandeur non
  vérifiés. Affiche-les suivis d'un marqueur d'incertitude discret, avec une infobulle
  « valeur approximative, non vérifiée ».
- Les valeurs marquées pkSur: true sont sourcées Setrag. Affiche-les sans marqueur.
- Affiche en permanence, sous le titre : « Sélection de 8 gares sur les 24 que compte la ligne ».
- Ne génère aucun chiffre qui ne figure pas dans ce tableau.

================================================
2. SCÈNE 3D
================================================

Three.js version 0.160.0, chargé depuis jsdelivr via un importmap, avec la version épinglée
explicitement. Aucune image, texture ou modèle 3D externe : tout est généré en code, y compris
les panneaux de gare, via CanvasTexture.

Terrain. Relief procédural, traversé par une voie ferrée sinueuse définie par une courbe de
Catmull-Rom. Creuse un couloir nivelé sous la voie, avec remblais et tranchées. Prolonge la
voie de quelques centaines de mètres au-delà de Franceville, pour que le terminus ne soit pas
au bord du monde.

Biomes évolutifs le long du trajet, par coloration des sommets et par densité de végétation :
- forêt équatoriale dense d'Owendo à Ndjolé,
- mosaïque forêt-savane de la Lopé à Booué,
- forêt de vallées vers Lastourville,
- plateaux ouverts et terres latéritiques rouges de Moanda à Franceville.
Affiche le nom du paysage courant dans l'interface.

Forêt. Essences variées générées en code : fromagers à contreforts et houppier tabulaire,
palmiers, fougères de sous-bois. Exclusivement en InstancedMesh. Aucun arbre à moins de
3 unités de la voie. Densité pilotée par le biome.

Rivières et ponts. Quatre traversées majeures, dont l'Ogooué, avec un lit creusé sous chaque
pont et un tablier métallique.

Ambiance. Lumière de fin d'après-midi, dôme céleste en shader avec dégradé horizon vers zénith,
brume légère au-dessus de la canopée, tone mapping ACES Filmic. Lumière directionnelle qui suit
le train, avec ombres douces. Pense à ajouter la cible de la lumière directionnelle à la scène,
sinon les ombres ne suivront pas.

Faune. Des oiseaux posés au sol qui s'envolent au passage de la locomotive, puis reviennent.

================================================
3. TRAIN ET CAMÉRAS
================================================

Train. Locomotive jaune et verte, quatre voitures voyageurs. Position et orientation calculées
sur la courbe, voitures décalées derrière la locomotive.

Durée. Trajet complet en trois minutes environ à vitesse x1. Boutons de vitesse x1, x2, x4.

Compteur. Un bandeau affiche en permanence « PK n / 648 ». La position du train sur la courbe
se calcule par t = pk / TOTAL_KM. Il ne doit exister qu'une seule source de vérité pour la
distance : le compteur du bandeau et la fiche de gare doivent afficher exactement la même
valeur à l'arrêt, et le trajet doit se terminer très précisément au PK 648.

Caméra cabine, par défaut. Vue depuis la locomotive, avec un léger balancement sinusoïdal qui
s'atténue à l'arrêt.

Caméra drone. Bouton de bascule. OrbitControls, maxDistance d'au moins 80 pour permettre un
plan large. La caméra suit le train et s'oriente progressivement dans son axe lors des virages,
avec amortissement.

Contrôles. Un bouton pause et un bouton de retour à la gare précédente.

================================================
4. GARES ET ARRÊTS
================================================

Chaque gare est matérialisée par un quai et un panneau portant son nom, généré en CanvasTexture.

À l'approche, le train décélère progressivement, puis s'arrête devant le quai. Une fiche
s'affiche en surimpression : nom, PK, type de gare, province, population, fait marquant. Un
bouton « Repartir », avec reprise automatique après 8 secondes et décompte visible.

Distingue visuellement les gares principales des gares secondaires, dans la fiche comme sur la
mini-carte.

Mini-carte, en bas à droite : le tracé complet d'Owendo à Franceville, les points des gares, et
la progression du train. Pré-calcule le tracé complet une seule fois au démarrage dans un canvas
hors écran : il est statique, et le redessiner à chaque image coûterait plus d'une centaine de
calculs de position par frame. Par image, ne redessine que le segment parcouru et le curseur.

Fin de voyage. À Franceville, affiche un écran de fin : distance totale, nombre de gares
desservies, durée réelle du voyage, et deux boutons, « Refaire le voyage » et « Vue libre ».
Ne relance jamais le trajet automatiquement.

================================================
5. HABITANTS ET VILLAGES
================================================

La scène doit être habitée, sinon elle reste une maquette abstraite. Registre graphique stylisé
et assumé : personnages low poly, silhouettes nettes et anguleuses, proportions simplifiées. Ne
cherche surtout pas le réalisme anatomique, qui échoue systématiquement en géométrie générée par
code. Vise le diorama soigné.

Sur les quais. De 6 à 15 personnages par gare, groupés naturellement plutôt qu'alignés :
voyageurs assis sur des bagages, personnes debout qui attendent, un ou deux vendeurs ambulants
avec un plateau sur la tête, un agent en gilet de sécurité près de la voie. À l'arrivée du
train, certains se tournent vers lui et quelques-uns lèvent le bras. Au départ, deux ou trois
montent dans les voitures et disparaissent.

Le long de la voie. Des hameaux de 3 à 8 cases à toit de tôle ondulée ou de végétal, sentiers
de latérite, bananiers plantains, linge étendu, filet de fumée de cuisson. Quelques habitants
qui vaquent et, au passage du train, des enfants qui courent le long de la voie et saluent.
Hameaux plus fréquents à l'approche des gares, quasi absents au cœur de la forêt dense.

Vêtements. Couleurs vives et variées, motifs de pagne wax sur certains personnages, générés en
code sous forme de bandes et de formes géométriques. Carnations variées dans une palette de tons
bruns réalistes. Tailles variées, dont des enfants. Aucun personnage identique à un autre : fais
varier au minimum la couleur, l'échelle et la pose.

Performance, impératif. InstancedMesh et petit nombre de géométries partagées, jamais un Mesh
par personnage. Plafond de 400 personnages et 60 bâtiments dans toute la scène. Pas d'ombre
portée pour les personnages, sauf ceux du quai de la gare courante. N'anime que ce qui est à
moins de 60 unités de la locomotive. Fumée faite de quelques plans transparents, pas de
particules.

Réglage. Un commutateur « Habitants » affiche ou masque toute cette couche, avec à côté le
nombre d'objets rendus. Je dois pouvoir montrer la scène avec et sans, et comparer les images
par seconde.

Isolation. Tout le code de cette couche dans une seule section délimitée, avec un point d'entrée
unique buildPopulation(), désactivable en une ligne. Aucune autre partie du fichier ne doit en
dépendre.

Échelle. Un adulte mesure environ la hauteur de la porte d'une voiture voyageurs. Calibre sur
les dimensions du train, pas sur l'échelle de la carte, qui est symbolique.

À proscrire : personnages réalistes ou texturés façon jeu vidéo, poses figées identiques, foules
alignées au cordeau, toute représentation caricaturale. En cas de doute entre fidélité anatomique
et lisibilité graphique, choisis la lisibilité.

================================================
6. INTERFACE
================================================

Entièrement en français.
Titre discret : « Transgabonais, Libreville → Franceville ».
Sous-titre : « Train Express Trans-Ogooué ».
Mention de la sélection de 8 gares sur 24.
Bandeau PK, nom du paysage courant, boutons de vitesse, bascule de caméra, pause, gare
précédente, commutateur Habitants, compteur d'images par seconde.
Mode cinéma activé par la touche C : masque toute l'interface sauf le titre. C'est destiné à la
captation vidéo.
Esthétique sobre, lisible sur mobile, contraste suffisant sur un fond de scène clair comme
sombre.

================================================
7. PERFORMANCE ET CHARGEMENT
================================================

Objectif : 60 images par seconde sur un mobile de milieu de gamme avec la couche habitants
activée, et 60 images par seconde garanties lorsqu'elle est désactivée.

La génération du terrain et de la végétation représente des dizaines de milliers de calculs
d'élévation. Ne la fais pas en un seul bloc synchrone : cela figerait la page plusieurs secondes
sur mobile, sur un écran vide. Découpe-la en étapes séparées par des requestAnimationFrame,
derrière un écran de chargement avec une barre de progression réelle. Libellés des étapes :
« Terrain », « Voie ferrée et ponts », « Forêt équatoriale », « Gares », « Habitants et villages ».

Plafonne le pixel ratio à 2. Borne le delta de temps par image. Utilise une seule carte d'ombres
de résolution raisonnable.

================================================
8. ACCESSIBILITÉ
================================================

Ne bloque pas le zoom dans le viewport.
N'applique user-select: none qu'aux boutons, jamais à tout le document.
Un attribut aria-label sur chaque contrôle.
Respecte prefers-reduced-motion en désactivant le balancement de la caméra en vue cabine.

================================================
9. PRIORITÉS SI TU DOIS RÉDUIRE
================================================

Le fichier doit être complet et fonctionner. Si tu approches de ta limite de réponse, réduis
dans cet ordre, et dis-le explicitement à la fin :
1. d'abord la couche habitants, qui est isolée,
2. puis le mode cinéma et l'écran de fin,
3. puis la variété des essences d'arbres.
Ne réduis jamais : l'exactitude des données, la cohérence du compteur PK, les arrêts en gare,
les deux caméras et l'écran de chargement.

================================================
10. LIVRABLE
================================================

D'abord le fichier HTML complet, prêt à ouvrir dans un navigateur.
Ensuite, en dehors du code, trois sections courtes :
1. Les choix techniques structurants et pourquoi.
2. Les simplifications et approximations du modèle 3D par rapport à la réalité du Transgabonais,
   et le coût en images par seconde de la couche habitants, avec et sans.
3. Ce qu'un relecteur gabonais devrait contrôler avant publication, et toute donnée que tu as
   affichée sans en être certain.
```

---

## Ce que ce prompt n'a pas suffi à empêcher

C'est la partie instructive. Les défauts ci-dessous sont passés **malgré une consigne explicite
dans le prompt**, ce qui dit quelque chose d'utile sur la limite de l'ingénierie de prompt.

| Le prompt exigeait | Ce qui est sorti |
|---|---|
| « Il ne doit exister qu'une seule source de vérité pour la distance » | Deux champs indépendants, `pk` et `t`, divergeant jusqu'à 39 km |
| « Le trajet doit se terminer très précisément au PK 648 » | Trajet terminé au PK 659, puis réinitialisé |
| « Trajet complet en trois minutes environ à vitesse x1 » | 4 min 10, la décélération avant chaque gare n'étant pas compensée |
| « Un adulte mesure environ la hauteur de la porte d'une voiture » | Échelle correcte, mais personnages posés du mauvais côté de la voie, à côté du quai |
| « forêt équatoriale dense d'Owendo à Ndjolé » | Arbres isolés sur des collines nues |

La leçon n'est pas que le prompt était mauvais. Elle est que **la génération de code ne se
vérifie pas en la relisant, elle se vérifie en l'exécutant**. Quatre de ces cinq défauts sont
invisibles dans le code et évidents à l'écran. Les corrections apportées sont détaillées dans
le [README](README.md).
