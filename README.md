SVG pilotable par le réseau
===========================

Ce projet permet de manipuler des images svg chargées sur un serveur, et fournit une API côté client pour envoyer des 
requêtes de modifications de l'image simplement.

Si vous le souhaitez, vous pouvez générer la documentation du projet avec doxygen (dispo sous ce nom avec apt) en vous 
plaçant à la raçine du projet et en tappant la commande `doxygen Doxyfile`. Pour la consulter, ouvrez le fichier 
`doc > html > index.html`.

Ce document, fait office d'un manuel, pour un résumé du travail éffectué par rapport au sujet, veuillez vous référer au 
fichier [TRAVAIL_EFFECTUE.md](TRAVAIL_EFFECTUE.md).

Sommaire
--------
1. [Compilation](#i---compilation)
2. [Creation d'une image svg pilotable](#ii---creation-dune-image-svg-pilotable)
3. [Lancer un serveur](#iii---lancer-un-serveur)
4. [Utilisation du client simple GUI ou CLI](#iv---utilisation-du-client-simple-gui-ou-cli)
5. [Créer son propre client](#v---crer-son-propre-client)





I - Compilation
===============

Voir [COMPILING.md](COMPILING.md)





II - Creation d'une image svg pilotable
=======================================

Tout d'abord, il faut faut une image svg normale. Vous pouvez la creer avec un logiciel comme inkscape ou bien à la
main.

Ensuite pour rendre certains attributs pilotables on va utiliser la balise `driven`, qui ne fait pas partie de la norme 
svg. La balise `driven` contient deux attributs au minimum :
* `target`, qui doit contenir le nom de l’attribut dont on veut contrôler la valeur. Cet attribut est pris parmi les 
  attribut de la balise parente de la balise `driven` que l'on est en train de définir.
* `by`, qui est le nom de l’entrée utilisée pour contrôler cet attribut (on l'appelera **nom du driven** tout au long de 
  ce document).
  
##### Quelques règles supplémentaire lors de la définition de drivens :

* Le nom des drivens ont des restrictions, mais uniquement ceux que l'on spécifie dans l'attribut `by`, car on verra plus
  tard que le serveur peut créer des drivens supplémentaires lors du chargement de l'image qui ne respecte pas forcemment 
  ces règles (voir section sur les [attributs composés](#attributs-composites)).  
  Donc, un nom de driven ne peut comporter que des lettres (minuscules et majuscules), des chiffres et le caractère _ et 
  doit obligatoirement commencer par une lettre.  
  Cela permet de pouvoir differencier les noms de driven des valeurs caculéés facilement (voir section sur les 
  [valeurs calculées](#valeurs-calcules)).
* Deux drivens ne peuvent avoir le même nom.

**Exemple :**
```svg
<circle cx="50" cy="25" r="20" fill="yellow">
    <driven target="cx" by="sun_x"/>
    <driven target="cy" by="sun_y"/>
</circle>
```
Cela crée un cercle de couleur jaune dont la position est modifiable avec les driven portant les noms `sun_x` pour 
manipuler l'attribut `cx` et `sun_y` pour manipuler l'attribut `cy`.

Pour voir des exemples complet de svg pilotable, vous pouvez consulter les images svg disponible dans le dossier 
[ressources](resources).

Extentions
----------
1. [typage des drivens](#1---typage-des-driven)
2. [animation/transitions](#2---transitions)
3. [attributs composites](#3---attributs-composites)
4. [valeurs calculées](#4---valeurs-calcules) [pas implémenté]


### 1 - typage des driven

pour spécifier un "type" aux attributs driven il faut dans une balise driven ajouter l'attribut `type="valeur"`.

**Exemple :**
```svg
<circle cx="250" cy="125" r="100" stroke="yellow" opacity="0.5" fill="orange" stroke-width="5">
    <driven target="cx" by="sun_x" type="intInterval(0, 300)"/>
    <driven target="cy" by="sun_y" type="intInterval(0, 300)"/>
    <driven target="opacity" by="sun_op" type="doubleInterval(0, 1)"/>
</circle>
```
Lors de la réception d'une nouvelle valeur par le serveur, l'image
est mise a jour uniquement si elle respecte la condition imposée
par l'attribut type, sinon la valeur reçue est ignorée.


#####valeurs possibles pour l'attribut type :
 
| valeur                     | condition de validation de la valeur
|---------------------------:|:--------------
| int ou int()               | Doit être un entier.
| double ou double()         | Doit être un double.
| intInterval(start,end)     | Doit être un entier entre **start** et **end** inclus.
| doubleInterval(start,end)  | Doit être double entre **start** et **end** inclus.
| color ou color()           | Doit être une couleur sous forme hexadecimale. <br/> (ex: #FFFFFF, #f6a, ...)
| list('el1', 'el2',...)     | Doit faire partie de la liste `el1, el2, ...` chaque élément doit être une chaine de caractère.
| regex('val')               | Doit correspondre a l'expression régulière **val**. <br/> **Val** doit etre écrite avec la [syntaxe ECMAScript](https://www.cplusplus.com/reference/regex/ECMAScript/).
| compound ou compound()     | voir section [attributs composites](#3---attributs-composites).

Les chaines de caractères doivent êtrent spécifiées entre ' '.
Attention a bien échaper les ' à l'interieur des chaines de caractères ex : 'l\\'orage' 

### 2 - Transitions
Une transition permet à un attribut de passer d'une valeur a une autre de façon fluide.

Pour indiquer un délai de transition lors de la mise a jour d'un attribut piloté,
il suffit de rajouter l'attribut `delay="val"` a la balise driven concerné, val 
doit être une durée en milisecondes.

**ATTENTION**
L'attribut `delay` ne marche que lorsque l'attribut `type`à été spécifié 
sur la balise.  


#####`delay` est compatible avec les types suivants:
* int
* double
* intInterval
* doubleInterval
* color
* compound (quand la cible est un attribut `points` uniquement)

exemple:
```svg
<circle cx="250" cy="125" r="100" fill="orange">
    <driven target="cx" by="sun_x" type="int" delay="2000"/>
</circle>
```
Ici, Lors d'un changement d'abscisse du cercle se fera de façon fluide en 2000 millisecondes soit 2 secondes.

### 3 - Attributs composites

Un attribut composite est un attribut qui se compose ou peux se composer de plusieurs valeurs que l'on peux distinguer 
les unes des autres. On a notemment les attributs **style** et **points**.

**Exemple :**
```svg
<g id="maison">
    <rect x="60" y="65" style="width: 30; height: 23; fill: grey;"/>
    <polyline points="60 65 75 55 90 65" stroke="black" fill="red" stroke-width="1"/>
</g>
```

Pour identifier un attribut composite on ajoute a la balise `driven` qui le cible l'attribut `type="compound"`.
Ensuite l'application identifie automatiquement le type d'attribut composite, selon le nom de l'attribut cible du driven.
Le fait de devoir spécifier `type="composite"` permet de pouvoir garder le comportement par défaut en ne le mettant pas.

Les attributs composites sont traités de la manière suivante :
* Pour un attribut **style**, pour chaque sous attribut, un driven sans type sera créé ayant le nom 
  `PREFIX:NOM_SOUS_ATTRIBUT` où PREFIX est la valeur de l'attribut `by`.
  
  Les sous attributs définis dans style **ne peuvent avoir de type et donc pas de transition non plus**.  
  De plus la manipulation de cet attribut est moins performante que si on definissait les sous attributs dans leurs 
  propres attributs donc pour des mises à jour très fréquentes, préférez toujours definir un driven sur un attribut 
  dédié.
  
  **Exemple :**
  ```svg
  <circle cx="-110" cy="200" r="100" fill="orange" style="stroke: yellow; stroke-width: 5">
      <driven target="style" by="cstyle" type="compound"/>
  </circle>
  ```
  Les driven résultans seront donc `cstyle:stroke` pour modifier le sous attribut `stroke` et `cstyle:sroke-width` 
  pour le sous attribut `sroke-width`.
  
* Pour **points**, même pricipe, pour chaque point de l'attribut (c.à.d pour chaque couple de valeurs x et y), deux 
  drivens seront créés avec le nom `PREFIX:INDICEx` et `PREFIX:INDICEy`, où `PREFIX` est la valeur de l'attribut `by`, 
  `INDICE` est l'indice du points, il est détérminé selon l'ordre de définition des points dans la balise `points` de gauche à droite
  en partant de l'indice 0, et enfin on ajoute a la fin `x` ou `y` pour symboliser respectivement l'abscisse ou l'ordonnée.  
  Les drivens ainsi générés seront tous de type **int**.
  
  [pas encore implémenté] Le **delay est compatible** avec ce type d'attribut composite.
  
  **Exemple :**
  ```svg
  <polyline points="60 65 75 55 90 65">
      <driven target="points" by="line" type="compound" delay="500"/>
  </polyline>
  ```
  Les driven résultans seront donc `line:0x`, `line:0y`, `line:1x`, `line:1y` `line:2x`, `line:2y` pour respectivement 
  modifier les points avec les coordonées (60,65), (75,55) (90,65) avec un temps de transistion de 500 millisecondes.  
  
  Si on veut changer les coordonées du deuxième point (75,55) et les mettre à (75,40) au serveur, il faut envoyer la 
  valeur `40` pour le driven `line:1y`.  
  
  
**ATTENTION**, les drivens générés par un driven de type compound lors du chargement de l'image ne sont **pas utilisables
dans les valeurs calculées**.

### 4 - Valeurs calculées
 [pas implémenté]

Pour piloter un attribut par une valeur calculée, il faut simplement spécifier l'opération a effectuer dans l'attribut
`by` et ainsi, chaque variable dans l'attribut `by` deviendra un driven en elle même.  
Les types des drivens générés sont **déduit automatiquement** en fonction de l'opération (voir tableau des
opérations réalisabes ci-dessous).  
Le resultat d'une valeur calculée sera toujours un **double** et le **delay est donc compatible** avec les valeurs 
calculées.

##### tableau des opérations disponibles (d, d1,d2 sont des drivens/opérandes):

 opération |type des opérandes|type du résultat |description
:---------:|:----------------:|:---------------:|:----------:
  d1 - d2  |      double      |      double     | soustraction
  d1 + d2  |      double      |      double     | addition
  d1 * d2  |      double      |      double     | multiplication
  d1 / d2  |      double      |      double     | division
  d1 % d2  |      double      |      double     | modulo
Autres opérations a venir.
  
On peux utiliser les parenthèses pour former des expressions plus complexes.

**Exemple :**

```svg
<circle cx="-110" cy="200" r="100" fill="orange">
    <driven target="cy" by="(input1 - input2) * input3"/>
</circle>
```
Donnera les drivens `input1`, `input2` et `input3` qui sont de type double , et on peut les réutiliser dans d'autres 
attributs `by`.





III - Lancer un serveur
=======================

Pour cela utilisez la commande :
```shell script
./svgserver chemin_image_svg
```
L'executable se trouve à la racine du projet après la compilation.

Sans préciser le port, c'est le port 6789 qui sera utilisé.

Vous pouvez utilisez l'image maison.svg qui se trouve dans le dossier ressources pour faire des tests.

Si vous voulez un port en particuler précisez l'option `-p port`  
Exemple :
```shell script
./svgserver -p 7856 resources/maison.svg
```




IV - Utilisation du client simple GUI ou CLI
============================================

#### client en ligne de commandes

Pour cela utilisez la commande : 
```shell script
./svgclientCLI [addresse_ip]
```
L'executable se trouve à la racine du projet après la compilation.

Sans argument, addresse_ip sera 127.0.0.1 (localhost)

Si vous voulez un port en particuler précisez l'option `-p port`  
Exemple :
```shell script
./svgclientCLI -p 7856 192.168.0.16
```

Une fois lancé, vous pouvez tappez la commande `help` pour une description
des commandes disponibles.

#### client graphique

Pour cela utilisez la commande : 
```shell script
./svgclientGUI
```
L'executable se trouve à la racine du projet après la compilation.

Pour acceder à la page de modifcation des drivens il faut renseigner l'adresse IP et le Port.



Sur la page de modification on dispose de quatre boutton.

1.ajouter : Une fois le driven selectionner grace à un combox (Select), il faut l'ajouter dans la liste
                  des elements qui vont etre envoyer au serveur pour la mis à jour des attributs.

2.On a la possibilité avant d'envoyer au serveur les attributs qui vont etre mis à jour
d'effacer cette liste grace au boutton Annuler MAJ, ou de les consulter (liste des Mis à jour).

3.envoyer : envoyer au serveur les mis à jours souhaiter.



V - Créer son propre client
===========================

Vous pouvez utiliser notre API qui simplife la tache.
Pour cela, vous devez utiliser la classe RemoteSVGHandler, veuillez vous
réferer a la documentation de cette classe pour plus d'informations.