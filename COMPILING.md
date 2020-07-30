SVG Pilotable par réseau
========================

liste des dépendances:
* gtk+-3.0
* cairo 
* librsvg2
* tinyxml2
* cbor

ATTENTION
--------

Certaines librairies utilisés par ce projet nécéssitent des versions
**supérieures** aux versions téléchargebles avec apt.


voici la liste de ces librairies :

| librairie | version minimale | disponibilité avec apt
|-----------|:----------------:| :-------:
| tinyxml2  |7.1               | pas sur apt
| cairo     |1.16.0            | Ubuntu >= 19
| librsvg2  |2.46              | pas sur apt

----------------------
guide d'installation
---------------------
Lancez les commandes une par une et ajoutez les éventueles dépendances
requises demandes lors des ./configure ou ./autogen.sh

* tinyxml2 (v7.1)

  ```shell script
  cd <dossier_ou_cloner_le_depot>
  git clone https://github.com/leethomason/tinyxml2.git
  cd tinyxml2
  git checkout tags/7.1.0
  make
  sudo make install
  ```
   
* cairo (v1.16.0)

    Téléchargez l'archive cette version [ici](https://www.cairographics.org/releases/cairo-1.16.0.tar.xz)
    et l'extraire dans le dossier de votre choix.

  ```shell script
  cd </chemin/vers/le/dossier/extrait>
  ./configure
  make
  sudo make install
   ```

* librsvg2 (v2.46.3)

    **Vous devez avoir installé cairo 1.16 avant.**

   ```shell script
  sudo apt install autoconf gettext gtk-doc-tools autopoint libtool rustc cargo libcroco3-dev automake itstool libxml2-dev libgirepository1.0-dev libpango1.0-dev
  echo 'PATH="$PATH:/usr/lib/x86_64-linux-gnu/gdk-pixbuf-2.0"' >> ~/.profile
    
  cd <dossier_ou_cloner_le_depot>
  git clone https://gitlab.gnome.org/GNOME/librsvg.git
  cd librsvg
  git checkout tags/2.46.3
  ./autogen.sh
  make
  sudo make install
   ```
  
----------------------
compilation et lancement
---------------------
placez vous dans le dossier contenant le fichier CMakeList.txt (racine du dépôt) et lancez les commandes suivantes :

   ```shell script
  sudo apt install cmake # si vous n'avez pas cmake d'installé
  cmake .
  make
  ```

Faites notemment attention lors du lancement de la commande cmake que cette cette commande indique que tout les modules
ont bien été trouvé. Si lors du lancement du serveur ou d'un des clients il y a un problème c'est soit qu'une des 
dépendances a mal été installé, soit qu'une des dependances n'est pas a la bonne version car normalement il n'y a 
pas d'erreurs qui puisse empêcher le lancement.
    
Les executables générés sont placés a la racine du projet.
L'executable du serveur s'appelle **svgserver** et 
L'executable d'un client de base en CLI s'appelle **svgclientCLI**
