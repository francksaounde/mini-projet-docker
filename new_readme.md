
Mini projet docker

Nom : Saounde Franck
Github : 

Introduction
Dans ce document nous répondons aux questions relatives au mini projet sur docker dans le cadre du bootcamp devops session 19 de Eazytraining.

Build and test :
cd mini-projet-docker/
git clone https://github.com/diranetafen/student-list.git
cd student-list/simple_api/
En suivant les recommandations de l’énoncé, mettons à jour le fichier Dockerfile par les lignes qui suivent :
FROM python:3.8-buster
LABEL maintainer="pozos-dsi"
WORKDIR /
COPY student_age.py .
RUN apt update -y && apt install python-dev python3-dev \
    libsasl2-dev python-dev libldap2-dev libssl-dev -y
COPY requirements.txt .
RUN pip3 install -r /requirements.txt
EXPOSE 5000
CMD ["python3", "./student_age.py"]

On vérifie la liste des images disponibles et on lance la création de l’image api_image à partir du Dockerfile qu’on vient de modifier. Puisqu’on est dans le répertoire qui contient le Dockerfile on indique le contexte courant par ‘.’
 
Une fois l’image créée on vérifie de nouveau la liste des images, et on constate qu’elle a été mise à jour :
 

On peut donc lancer un container pour l’api à base de cette image, 
Appelons-le « api_app » :
(docker run -d --name api_app -v ${PWD}:/data -p 5000:5000 api_image)
Ayant lancé le container en arrière-plan (grâce à l’option -d dans la commande), nous regardons la liste de tous nos containers (option -a) et nous interrogeons les logs pour confirmer que notre unique container est bien lancé et prêt à écouter 
 

Nous allons maintenant effectuer une requête vers le container via la commande ‘curl’
Nous modifions la commande fournie dans l’énoncé en ajoutant le nom d’adresse de l’hôte qui est notre machine virtuelle sur laquelle tourne le container (d’où localhost), et nous spécifions le port 5000 qui a été exposé. On peut donc voir les informations des deux étudiants enregistrés
 

Infrastructure As Code :
Voici la structure de notre fichier docker-compose.yml
version: "3.8"
services:
  web:
    container_name: web_app
    image: php:apache
    depends_on:
      - api
    ports:
      - "80:80"
    environment:
      - USERNAME=toto
      - PASSWORD=python
    volumes:
      - ./website:/var/www/html
    networks:
      - student_network
  api:
    container_name: api_app
    build:
      context: ./simple_api
    volumes:
      - "${PWD}/simple_api:/data"
    ports:
      - "5000:5000"
    networks:
      - student_network
networks:
    student_network:
Au préalable, supprimons le container créé précédemment
 

Mettons à jour le fichier docker-compose.yml
 

Mettons à jour le fichier index.php avec le nom de notre container pour api et le port exposé (tel que mentionné dans le docker-compose
 

Lançons maintenant notre stack par la commande 
	  docker-compose up 

 

Vérifions aussi que l’api est accessible via notre interface web
 
Docker Registry 
Au préalable nous supprimons la stack lancée précédemment, 
 
Pour le registre et l’interface nous créons le fichier docker-compose suivant :
mkdir registry
cd registry/
vi docker-compose.yml
cat docker-compose.yml
 

Lançons maintenant notre stack par la commande 
	  docker-compose up -d
 
Nous constatons aussi que le réseau « registry_pozos-network-registry» a été créé
 
En faisant un test sur le port exposé  8090, comme mentionné dans le fichier docker-compose on a bien l’affichage du registre pour le moment vide
 

Poussons-y l’image créée pour l’application de gestion des étudiants
	Commençons par vérifier les images existantes
 
Ensuite, faisons d’abord un tag sur l’image
 
Nous pouvons désormais faire un push de l’image sur le régistre, précisant toujours le port 5000 comme précisé dans le fichier docker-compose associé
 
Une fois cela fait, nous actualisons la page et pouvons constater le changement sur le frontend du régistre :
 
En cliquant sur le nom de l’image, on obtient beaucoup de détails
 

Pour conclure, nous pouvons souligner que ce travail nous a permis de revoir un grand nombre de notions abordées dans le cadre du module sur l’introduction à Docker. Nous avons pu approfondir en faisant des recherches notamment sur l’utilisation des volumes. 
Le caractère très concret de cet exercice permet aussi de se mettre dans des conditions proches de celles d’un cas d’entreprise.




