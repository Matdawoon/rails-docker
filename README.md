# README

Comment créer un environnement de développement Ruby on Rails avec Docker (Rails 5.x)
Dans cet article, je vais vous présenter comment créer un environnement de développement pour la série Ruby on Rails (ci-après, rails) 5.x à l'aide de Docker.

Si vous souhaitez créer un environnement Docker sur Rails 6.x, veuillez consulter ici.
Quels sont les avantages de la création d'un environnement avec Docker?
La plupart des débutants sur rails l'installent généralement directement sur leur machine pour créer un environnement de développement. Cependant, si vous trébuchez en raison d'un environnement subtil (tel que la version Ruby ou les variables d'environnement), Je pense qu'il faudra beaucoup de temps pour le dépanner.

Par conséquent, en créant un environnement de rails sur l'environnement virtuel avec Docker Un comportement égal peut être obtenu dans n'importe quel environnement de développement sur n'importe quelle machine, évitant les erreurs dues à des environnements différents. C'est aussi un avantage que vous pouvez réinitialiser une fois au pire lorsque vous dérangez trop l'environnement.

De plus, lors de la tentative de publication de l'application créée vers l'extérieur en tant qu'environnement de production Vous pouvez l'exécuter en tant qu'application stable en minimisant la différence entre l'environnement de développement et l'environnement de production.  Vous pouvez trouver difficile de créer et de publier un environnement virtuel avec Docker, Récemment, même les services cloud tels que AWS ont des services tels que ECS qui peuvent exécuter des conteneurs Docker tels quels. Il devient également plus facile de publier l'environnement créé avec Docker.

Conditions préalables
Docker installé sur votre machine

Si vous n'avez pas installé Docker, téléchargez le programme d'installation de Docker sur le site officiel et installez-le. → Docker Official

Créez également un dossier de travail arbitraire. Ici, nous allons créer un dossier appelé "rails-docker" et y travailler.

$ mkdir rails-docker
$ cd rails-docker
Préparation des fichiers liés à Docker
Préparez les quatre fichiers vides suivants dans rails-docker. Si vous utilisez VScode, vous pouvez le créer en créant un nouveau fichier, Vous pouvez également utiliser la commande tactile depuis le terminal Mac.

.
├── Dockerfile #Créer un nouveau
├── Gemfile #Créer un nouveau
├── Gemfile.lock #Créer un nouveau
└── docker-compose.yml #Créer un nouveau
Explication du fichier Docker
Docker crée le conteneur à partir d'un fichier appelé Dockerfile. Une image Docker (modèle du conteneur) sera créée.

Tout d'abord, écrivez le texte intégral du Dockerfile. La version de Ruby utilisée est 2.7.0.

FROM ruby:2.7.0
RUN apt-get update -qq && apt-get install -y build-essential nodejs
RUN mkdir /app
WORKDIR /app
COPY Gemfile /app/Gemfile
COPY Gemfile.lock /app/Gemfile.lock
RUN bundle install
COPY . /app
Bien que ce soit un bulletin, j'expliquerai la signification de chacun.

FROM ruby:2.7.0
Parmi les conteneurs publics installés par Ruby, nous allons extraire celui dont la version Ruby est 2.7.0.
RUN apt-get update -qq && apt-get install -y build-essential nodejs --Installation des packages nécessaires pour exécuter Rails
RUN mkdir /app --Créez un répertoire d'application pour créer des fichiers de projet Rails
WORKDIR /app --Spécifier un répertoire de travail
COPY Gemfile /app/Gemfile
Déplacez le Gemfile sur votre machine (PC) dans le répertoire de travail du conteneur et rendez-le disponible à partir du conteneur
Similaire à Gemfile.lock
RUN bundle install
Installation par lots de gemmes répertoriées dans Gemfile
COPY . /app
Copiez tous les fichiers du dossier où se trouvent les fichiers Docker dans le répertoire de l'application à l'intérieur du conteneur
Copie des fichiers nécessaires à l'exécution de Rails à inclure dans le conteneur
Explication de Gemfile
Dans Ruby, vous pouvez définir la gemme que vous souhaitez utiliser dans votre environnement avec un fichier appelé Gemfile.

Un joyau est une bibliothèque Ruby. Le gem est géré par un système de gestion de paquets pour Ruby appelé RubyGems, et peut être installé via la commande gem fournie par RubyGems.

source 'https://rubygems.org'
gem 'rails', '5.2.1'
J'expliquerai encore un peu.

source 'https://rubygems.org' --Spécifiez la source de téléchargement de gem
gem 'rails', '5.2.1'
Spécifiez le nom du package et la version des rails, qui est la gemme à installer.
Cette fois, la version rails spécifie 5.2.1.
Vous pouvez spécifier n'importe quelle version, mais veuillez noter que le Webpacker et le fil sont nécessaires après les rails 6
Ici, les rails de la série 5 sont utilisés pour une vérification facile.
Si vous exécutez la commande "bundle install" dans le répertoire où se trouve le Gemfile, Vous pouvez installer le gem défini selon la définition dans le Gemfile.

Pour le fichier Docker que j'ai mentionné précédemment, copiez le Gemfile dans le répertoire de travail du conteneur. Vous avez spécifié d'exécuter l'installation de l'ensemble dans ce dossier.

Explication de Gemfile.lock
En fait, Gemfile.lock n'a pas besoin d'écrire quoi que ce soit au début. Ce n'est pas un fichier que vous modifiez directement Après avoir exécuté l'installation du bundle basée sur Gemfile Les gemmes installées seront désormais répertoriées dans Gemfile.lock.

Pour l'utiliser, reportez-vous à Gemfile.lock pour connaître le gem et la version installés. Vous pouvez également exécuter l'installation groupée à partir de Gemfile.lock Gemfile.lock est utilisé lorsque vous souhaitez recréer exactement le même environnement ou lorsque vous développez avec un grand nombre de personnes.

N'est-ce pas juste un Gemfile? En complément de ceux qui pensaient que c'était tranchant, l'une des raisons est que Gemfile peut réellement être écrit de cette manière.

gem 'rails', '~> 5.2.1'
Dans ce cas, bien que la version des rails installés par l'installation groupée soit 5.2.1 ou supérieure La version réelle qui peut être installée (publiée) dépend de l'heure.

Cependant, dans Gemfile.lock, le gem lui-même et la version spécifique réellement installée lors de la tentative d'installation selon le Gemfile, Il enregistre même les packages qui ont été installés pour des besoins accessoires.

Pour résumer un peu, Gemfile est une "liste des gemmes nécessaires à l'application". D'un autre côté, Gemfile.lock contiendra des "informations sur le gem réellement installé suite au suivi du Gemfile".

Explication de docker-compose.yml
Ce fichier yml est utilisé par Docker Compose.

Docker Compose est destiné aux applications constituées de plusieurs conteneurs Il s'agit d'un outil de gestion tel que la création d'une image Docker et le démarrage / l'arrêt de chaque conteneur.

Dans Docker Compose, les définitions de plusieurs conteneurs, y compris les options de construction de Docker et de démarrage du conteneur, sont écrites dans un fichier appelé docker-compose.yml. Vous pouvez l'utiliser pour créer une image Docker ou démarrer un conteneur.

Si vous souhaitez exécuter Rails seul comme un environnement de développement, vous pouvez simplement utiliser Dockerfile. En plus du serveur d'applications pour l'exécution de Rails Un serveur Web pour accepter l'accès d'Internet lors de la publication, Nous pouvons également préparer un serveur de base de données pour stocker et traiter les données.

docker-compose.yml

version: '3'
services:
  web:
    build: .
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    volumes:
      - .:/app
    ports:
      - 3000:3000
    depends_on:
      - db
    tty: true
    stdin_open: true
  db:
    image: mysql:5.7
    volumes:
      - db-volume:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: password
volumes:
  db-volume:
Étant donné que les rails nécessitent également une base de données, nous allons créer un conteneur pour le serveur de base de données en utilisant MySQL ainsi que le serveur Web. Bien que cela ne soit pas expliqué en détail ici, le serveur de base de données (conteneur) avec lequel les rails sont liés est spécifié par son nom dans "depend_on" dans la colonne appelée web.

Lancement des rails
Créer un projet de rails (nouveaux rails)
Exécutez la commande docker-compose suivante dans le dossier (dossier rails-docker) où se trouve docker-compose.yml.

$ docker-compose run web rails new . --force --database=mysql
Je vais vous expliquer comment lire la commande, y compris les options ↓

--docker-compose: Utilisation d'un outil appelé docker-compose --run: exécutez la commande suivante --web: dans un conteneur Web --rails new: Créer un nouveau projet avec des rails

.: Pour le répertoire courant ---- force: écraser les fichiers existants (ici Gemfile, Gemfile.lock) ---- database = mysql: Cependant, MySQL est utilisé pour la base de données rails.
Cela a le sens.

Si vous exécutez cette commande en une ligne, le traitement prendra quelques minutes, alors attendez patiemment en mangeant du chocolat.

Exécuter la construction
Installation de gemme ajoutée à Gemfile, Et exécutez build pour amener le fichier créé dans le conteneur.

$ docker-compose build
Modifier le fichier de paramètres de base de données
Une fois la construction terminée, modifiez les paramètres dans le fichier de base de données utilisé par les rails. Le fichier cible est ** database.yml ** dans le répertoire config.

config/database.yml

default: &default
  adapter: mysql2
  encoding: utf8
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password: password #ajouter à
  host: db #Changement
Le mot de passe doit correspondre au mot de passe spécifié dans docker-compose.yml.

docker-compose.yml

...
    environment:
      MYSQL_ROOT_PASSWORD: password #Match ici
...
En outre, le champ hôte définit le nom du conteneur MySQL.

Démarrer le conteneur
Pour démarrer le conteneur, exécutez la commande suivante:

$ docker-compose up -d
docker-compose up est une commande pour démarrer un conteneur basé sur docker-compose.yml Il peut être démarré en arrière-plan avec l'option "-d".

Créer une base de données
À l'heure actuelle, le conteneur vient de démarrer et la base de données n'a pas été créée. Exécutez la commande suivante pour créer la base de données.

$ docker-compose run web bundle exec rails db:create
C'est un processus pour exécuter des rails db: create (= créer une nouvelle base de données) dans le conteneur Web exécutant des rails.

Confirmation de démarrage du serveur de développement Rails
Vous avez démarré avec succès le serveur de développement de rails. Entrez ** localhost: 3000 ** dans la barre d'adresse de votre navigateur et vérifiez le démarrage.

image.png

Avec les rails, vous avez terminé lorsque vous voyez l'écran familier.

Arrêter le conteneur
Pour arrêter le serveur de développement dans un lot, exécutez la commande suivante.

$ docker-compose down
Si vous souhaitez le redémarrer, exécutez docker-comopose up -d.

