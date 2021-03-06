---
type: post
title: "Octopress, des tentacules bien huilées"
date: 2013-10-25T18:22:00
comments: true
categories: 
  - Octopress
  - Ruby
  - Sous le capot
  - Retour sur expérience
---

Octopress est un moteur de blog assez récent à destination de gens un minimum techniques.
C'est l'outil qui est utilisé pour le blog sur lequel vous vous trouvez actuellement. 
Du coup, je vais essayer de vous faire un retour à chaud sur le sujet.

L'article va partir du basique (l'installation et le *"blogging"*) pour ensuite aller vers du plus technique où on décortiquera complètement le framework.
Du coup, n'hésitez pas à vous arrêter dès que le niveau de granularité atteint vous suffit.
<!--more-->

Un blog pas comme les autres
============================
J'ai configuré par le passé plusieurs Wordpress/Drupal/Joomla et essayé de les utiliser en vain. 
J'ai même essayé d'utiliser une plateforme on-line de blog telle que overblog.
On peut surement faire des choses merveilleuses avec un Wordpress c'est juste que 
ça m'a toujours laissé comme un gout d'usine à gaz pas pratique en fond de bouche.

Du coup quand j'ai vu [Octopress](http://octopress.org/) je me suis jeté dessus. 
Laissez moi vous montrer comment celui-ci se présente.

Le principe
-----------

> "*Octopress est un framework de blog très simple permettant en l'espace de 30 minutes d'avoir un blog complètement statique en ligne.*"

Pour faire une liste de ce que j'aime dans octopress:

<!-- TODO add link for markdown-->
* Une édition de posts/pages en [Markdown](http://fr.wikipedia.org/wiki/Markdown) le langage de description créé par [github](http://www.github.com).
* Tout le travail d'édition se fait hors ligne avec l'éditeur de votre choix. <!--vim-->
* Il est très simple à configurer. Il suffit de savoir éditer un tout petit peu de css (scss pour être précis) pour en faire ce que l'on veut.
* C'est simple d'utilisation.
* Une fois déployé il est entièrement statique. Pas de base de données à gérer, le moindre petit hébergement suffira à contenir votre blog 
(et parler de performances serait inconvenant).
* Votre blog est mis à jour uniquement quand vous le lui demanderez (on peut imaginer qu'on veuille publier en même temps une correction d'article et un nouvel article).
* C'est simple d'utilisation... non vraiment...
* Derrière tout ça se cache du ruby et du rake (on y reviendra plus en détail par la suite).

Voyons donc comment on met en place cet outil.

Les pré-requis (Ruby, Git, Bundler)
----------------------------------

Si vous avez déjà tout ça d'installé sur votre machine vous pouvez passer a la section suivante.
Cette partie va être la plus *"compliquée"* de l'installation.

* **Ruby**: En fonction de votre système d'exploitation son installation diffère. 

Sous linux, j'ai tendance à bien aimer rvm et a vous conseiller une installation en ce sens [Rvm.io](http://rvm.io).
```bash rvm_instal.sh 
curl -L https://get.rvm.io | bash -s stable
rvm install 1.9.3
rvm use 1.9.3
```
Si vous décidez de passer par votre gestionnaire de paquet pensez à installer aussi rubygems.

Pour le reste passez par le [rails installer](http://railsinstaller.org/fr-FR#osx). Vous aurez peut être quelques paquets inutiles d'installés mais ça simplifie grandement la tache.

* **Git**: De même. 

Sous linux votre gestionnaire de paquet fera l'affaire.

Pour le reste le site de [Git](http://git-scm.com/) vous expliqueront la procédure mieux que moi.

* **Bundler**:
Ici une simple commande:
```bash bundler_install.sh
gem install bundler
```

Et c'est tout ce qu'il vous faut. Vous pouvez passer à l'installation d'Octopress.

L'installation
--------------

L'installation est plutôt rapide.

```bash installation_octopress.sh
#On clone le projet octopress sur github.
git clone git://github.com/imathis/octopress.git mon_beau_blog
cd mon_beau_blog

#On corrige un defaut dans le repo d'octopress
echo "gem 'json', '~> 1.8.1'" >> Gemfile

#On installe les dépendances d'octopress.
bundle install

#On installe le minimum vital pour octopress.
rake install
```
  
On fait un minimum de configuration pour pouvoir déployer notre site. 
Les différentes configurations possibles sont disponibles sur http://octopress.org/docs/deploying/.

Je vais vous présenter la méthode que j'utilise. À savoir un déploiement via rsync sur un serveur.

Il suffit de modifier l'entête du fichier Rakefile

```ruby Rakefile
## -- Rsync Deploy config -- ##
# Be sure your public key is listed in your server's ~/.ssh/authorized_keys file
ssh_user       = "utilisateur@serveur"
ssh_port       = "22"
document_root  = "mettre/le/path/ou/deployer/le/site"
rsync_delete   = true # Permet d'autoriser rsync à supprimer les anciens fichiers du repertoire
rsync_args     = ""  # Any extra arguments to pass to rsync
deploy_default = "rsync"
```

Voilà! Vous avez un blog fonctionnel. On va voir comment ajouter du contenu dessus.

Publier
-------

Pour publier du contenu sur votre blog il y a trois étapes à connaitre.

#### Écrire un post

Pour écrire un post on commence par générer le fichier dans lequel on va écrire notre post:
```bash generation_post.sh
rake new_post["Mon beau post"]
```
Ça va créer le fichier "sources/\_posts/AAAA-MM-JJ-mon-beau-post.markdown".
Qu'on pourra ensuite éditer avec notre éditeur de texte favori<!--vim-->. 

Le poste se décompose en 2 parties:

* Une entête contenant un descriptif du post (titre, catégories, etc) au format YAML.
* Un corps où l'on écrira le contenu du post. Ce post sera au format [.markdown](http://fr.wikipedia.org/wiki/Markdown).
Ce choix de méthode de templating peut être changé via le Rakefile.

Des plugins sont fournis par octopress pour améliorer le rendu du post ([plugins octopress](http://octopress.org/docs/plugins/)).

Un outil pratique permet de prévisualiser notre travail.
#### Prévisualisation
Une simple tache rake vous permet de redéployer automatiquement à chaque modification de fichier.

```bash previsualisation.sh
rake preview
```

On accède au site sur l'adresse http://localhost:4000

Une fois l'article finalisé, on peut passer au déploiement.

#### Déploiement
Une fois l'article finalisé il suffit de déployer son blog.

```bash deploiement.sh
rake generate # Genere le code html/css/js dans le dossier public.
rake deploy   # Deploie le contenu du dossier public en fonction de la methode de deploiement choisi au debut.

#Ou plus concis
rake gen_deploy
```

Pour ne pas publier un article en cours d'écriture il suffit d'ajouter dans son entête YAML la ligne
```yaml
published: false
```

La customisation
----------------

#### Configurer le blog
Pour ce qui est des différents points de configuration de votre blog la majeure partie se trouve dans le fichier \_config.yml

Le fichier est plutôt bien commenté et le nom des variables parlent d'eux même.

Notez que pour ajouter google analytics il faudra exécuter cette commande
```bash
echo -e "{\a{ include google_analytics.html }}" >> source/_includes/custom/after_footer.html
```

#### Quelques thèmes

Les thèmes pour octopress sont plutôt simples à trouver et à installer. Je vais vous en présenter quelques un:

* [**Greyshade**](https://github.com/shashankmehta/greyshade): Le thème que j'utilise actuellement. 
À mon sens il nécessite quelques ajustements pour le rendre plus agréable mais ça sera pour un prochain article.
* [**Stash**](http://zespia.tw/Octopress-Theme-Slash/): Un autre thème plutôt minimaliste.
* [**FoxSlide**](https://github.com/sevenadrian/foxslide): Plus *"design"* tout en restant agréable pour les yeux.

Vous pouvez trouver une liste des thèmes sur [github](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes).

Globalement si vous vouliez juste un blog vous pouvez vous arrêter là et sauter directement à la conclusion :) 
Sinon on embraye sur un point du workflow qui m'a dérangé avec octopress, le déploiement.

Un peu d'huile dans le déploiement
==================================

À cette étape, vous devriez avoir un blog fonctionnel, déployé quelque part qui attire des milliers d'utilisateurs. 
Et un jour, votre chat décide de supprimer votre dossier octopress (faut bien que ce soit la faute de quelqu'un).

Je pense que c'est comme ça que sont nés les gestionnaires de sources. 
On va donc voir comment sauvegarder notre blog de manière un peu plus *"sure"* que sur votre PC (et éventuellement une clé usb).
On va aussi en profiter pour automatiser un peu le déploiement du blog.

Utilisons Git
-------------

Comme on l'a déjà installé (et parce que c'est très bien<!-- KIKOO MERCURIAL -->) je vous propose d'utiliser git. 
Je vous apprendrai pas ici à vous servir de git. Des [tutoriels](http://try.github.io) sur internet font ça très bien.

Votre dossier octopress est dors et déjà un repo git local. 
Vous pouvez donc déjà l'utiliser comme tel. Il va juste vous manquer un repo distant.

Pour ajouter un repo distant vous avez plusieurs choix:

* L'incontournable [github](http://github.com). 
* Votre propre serveur pour moins de partage.
* Eventuellement mon serveur [gitlab](https://git.skizzk.fr).

Ajoutez ce repo distant à vos remote:
```bash update_remote.sh
git remote rename origin octopress
git remote add origin votre_addresse_de_remote
```
Votre blog est maintenant versionné et sauvegardé sur un serveur. Cependant, on a maintenant ajouter de nouvelles commandes pour gérer son blog:
```bash 
rake generate # pour generer le site
rake deploy   # pour deployer le site
rake commit   # pour commiter le site en local
rake push     # pour envoyer notre site sur un repo distant
```
Personnellement je trouve que ça fait trop de commandes à taper et que 
le déploiement de notre blog devrait être lié à nos push puisqu'au final c'est la même unité logique fonctionnelle.

Je vous propose d'utiliser Jenkins pour ça (non pas celui [là](http://www.youtube.com/watch?v=LkCNJRfSZBU)).
Un peu de Jenkins
-----------------

[Jenkins](http://jenkins-ci.org/) est un outil d'Intégration Continue en Java. 
Il permet de créer des jobs et de les exécuter selon certains trigger tout en nous indiquant si quelque chose se passe mal.
Cet outil est vraiment pratique et simplifie un grand nombre de taches *ennuyeuses* que vous pourriez avoir a faire.
Je vous recommande d'y jeter un œil même si vous n'optez pas pour ma méthode de déploiement automatisé.

Je considère que vous avez votre propre jenkins sur un serveur (ou votre propre PC, même si c'est du coup moins pertinent).

#### Ajouter les plugins nécessaires au jenkins.

Voila la liste des plugins dont on va avoir besoin pour ce job:

1. Git plugin
2. Rvm

#### Ajouter les permissions au jenkins
On traite ici que le cas du rsync. 
Je ne sais pas quels sont les besoins des autres cas.

1. On se connecte avec l'utilisateur de jenkins sur le serveur
2. On ajoute la clé rsa du jenkins au serveur ou l'on va déployer notre blog (pour pouvoir s'y connecter sans taper de mot de passe)

#### Ajouter le job

1. On ajoute une gestion de code source par git en indiquant l'adresse du repo et la branche (a priori *master*)
2. On ajoute une scrutation periodique du gestionnaire de version. Par exemple toutes les 5 minutes:
```bash
H/5 * * * * 
```
3. On indique que l'on veut build le projet via rvm
4. On exécute un script shell
```bash Build -> Executer un script shell
bundle install
rake generate
rake deploy
```
5. On s'envoie éventuellement un mail.

Voilà, avec cette configuration à chaque *git push* votre blog sera buildé et redéployé automatiquement. Tout ça ne nous laisse qu'une question : *"Comment ca marche?"*.

Comment ça marche un poulpe?
============================

C'est très beau tout ça mais à quoi servent les différents dossiers? Par quelle magie octopress transforme du markdown en html?

Ouvrons le capot
----------------
Commençons par une petite anatomie des différents dossiers:

1. Les **plugins**: Ici se trouvent les différents plugins qui seront utilisés dans votre blog tel que *code\_block.rb* qui permet d'inclure des blocs de code dans votre blog.
2. Le dossier **public**: Ici sera généré le code compilé (donc du html, du css et du javascript).
3. Le dossier **source**: Contient les différentes pages de votre blog. Il contient tout le contenu de votre blog qui n'est pas du Sass (le html, le js, le markdown, etc...).
L'endroit où vous devriez modifier des choses sont les dossiers que vous créez vous même, le dossier \_post ainsi que le contenu du dossier *source/\_includes/custom/*
4. Le dossier **sass**: Contient ici les fichiers Sass et qui seront compilé sous forme d'un fichier css et inclus à votre blog. 
Le fichier screen.scss est le fichier principal et le resultat de sa compilation seul sera inclus.

Au final c'est bien beau tout ça mais... 
Attendez... Il est où le code?
------------------------------

Tout le code "actif" d'octopress est contenu dans son Rakefile de 300 lignes qui permet de gérer les taches *rake*.

En fait octopress est un template de site qui s'appuie sur la gem jekyll. 
Il rajoute à cette gem un certain nombre de fonctionnalités.

* Il permet d'organiser ses posts et différentes pages de manière propre.
* Il permet de fournir un template de site par défaut (html, css, js).
* Tout ça est orchestré par le Rakefile qui masque les appels à jekyll, réorganise le rendu et le déploie.

Du coup, qu'est ce que fait jekyll?

Jekyll
------

[Jekyll](http://jekyllrb.com/) est un utilitaire ecrit en ruby pour générer de l'html à partir de fichier templaté(par défaut Redcarpet et Liquid). 
Il est fait pour créer des blogs et n'inclus par défaut aucun formatage. 
Il existe un grand nombre de plugins pour ce moteur. 
Un certain nombre sont inclus par défaut dans octopress (celui qui permet de generer du css a partir de sass par exemple).

Si vous possédez un blog dans un autre format, je vous invite a en regarder la documentation. Il existe de nombreux importeur d'autres blogs. 
Et une fois que vous l'aurez converti en Jekyll, la conversion vers du Octopress sera facile.

Conclusion
==========
Avoir un blog correct vous prend maintenant 30 minutes. 
Il ne vous reste plus qu'à écrire des articles. 
Comme d'habitude, vous n'avez plus d'excuses.

Ce moteur de blog à priori destiné aux gens qui aiment un minimum toucher à la ligne de commande n'est pas "si" compliqué. 
Il permet de bloguer facilement, sans se prendre la tête avec une usine a gaz de type wordpress.
Même si vous n'êtes pas du tout technique, je vous invite à l'essayer c'est vraiment pas sorcier.
