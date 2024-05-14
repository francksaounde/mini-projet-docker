
# Mini projet docker

#### Projet réalisé par Saounde Franck 

## Introduction
Dans ce document nous répondons aux questions relatives au mini projet sur docker dans le cadre du bootcamp devops 19 Eazytraining.

## Build and test :

Pour la construction de l'image de l'api, plaçons-nous dans le répertoire contenant le Dockerfile correspondant

```bash 
cd mini-projet-docker/
git clone https://github.com/diranetafen/student-list.git
cd student-list/simple_api/
```
Une fois dans le bon répertoire, 
mettons à jour le fichier Dockerfile en suivant les recommandations de l’énoncé :

```bash 
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
```

Vérifions maintenant la liste des images disponibles et lançons la création de l’image       
que nous appelons *api_image*, à partir du Dockerfile qu’on vient de modifier.     

Puisqu’on est dans le répertoire qui contient le Dockerfile on indique le contexte courant par un point (‘.’)
 
Une fois l’image créée, vérifions de nouveau la liste des images,     
Nous constatons qu’elle a été mise à jour :     
 

Nous pouvons donc lancer un container pour l’api à base de cette nouvelle image.      
Nommons notre nouveau container *api_app*.        
Par la même occasion montons le répertoire courant (${PWD}) comme volume et associons-le au répertoire /data du container.       
La commande de lancement de notre container est alors:     

```bash
docker run -d --name api_app -v ${PWD}:/data -p 5000:5000 api_image
```

Ayant lancé le container en arrière-plan (grâce à l’option -d dans la commande), regardons la liste de tous nos containers (option -a)    
Et interrogeons les logs pour confirmer que notre unique container est bien lancé et prêt à écouter:        
 

Nous allons maintenant effectuer une requête vers le container via la commande ‘curl’.       

Nous modifions la commande fournie dans l’énoncé en ajoutant l’adresse de l’hôte -qui est notre machine virtuelle-   
sur laquelle tourne le container (d’où localhost), et nous spécifions le port 5000 qui a été exposé.         

On peut donc voir les informations des étudiants enregistrés:         
 

## Infrastructure As Code :

Nous éditons un fichier docker-compose.yml pour déployer nos deux services.     
Donnons quelques commentaires de son contenu:

- Nous indiquons la version du docker-compose ```version: "3.8"```

##### Service web

- *web_app* désigne le nom du container frontend ```container_name: web_app```

- Le container *web_app* tourne à base de l'image *php:apache* ```image: php:apache```

- Apache est un serveur web qui utilise le port 80 de l'hôte  
Nous sommes donc contraints d'utiliser le port 80 au niveau du container   
Nous choisissons de lui associer le port externe 80,    
D'où: ```ports "80:80"```    

- Nous indiquons l'utilisateur *toto* et son mot de passe *python*,   
tels que fournis par la commande "curl" précédemment utilisée et tel que mentionné dans le code python

```
- environment:
      - USERNAME=toto
      - PASSWORD=python
```

- Nous indiquons à apache d'utiliser notre fichier index.php à la place de sa page par défaut

  ```volumes: ./website:/var/www/html```
 

- Nous définissons un réseau dans lequel nous mettons les deux containers.    
Ceci pour leur permettre de communiquer:      
   ```networks: student_network```         



##### Service api:
   
- Nous indiquons le répertoire dans lequel trouver le Dockerfile pour construire l'image de l'api       
  ```build: context: ./simple_api```
    
- Comme dans la partie précédente, nous montons un volume        
    ```volumes:"${PWD}/simple_api:/data"```
   
- Nous exposons le port 5000 à l'intérieur du container et l'associons au port 5000 de l'hôte    
    ```ports: "5000:5000"```


En définitive, voici la structure de notre fichier docker-compose.yml

```bash 
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
```

Au préalable, supprimons le container créé précédemment
 

Mettons à jour le fichier docker-compose.yml
 

Mettons à jour le fichier index.php avec le nom du container de l'api et le port exposé 
(5000 tel que mentionné dans le docker-compose)
 

Lançons maintenant notre stack par la commande ```docker-compose up```         
Ensuite vérifions que notre stack est bien démarrée par la commande ```docker-compose ps```
 

Vérifions aussi que l’api est accessible via notre interface web.        
Après avoir cliqué sur le bouton "List Student" nous avons bien la liste des étudiants enregistrés


 
## Docker Registry 

Au préalable nous supprimons la stack lancée précédemment, 


Ensuite nous créons un répertoire *'registry'* et nous plaçons à l'intérieur. Puis nous créons    
le fichier docker-compose pour le registre et l’interface:    


```bash 
mkdir registry
cd registry/
vi docker-compose.yml
cat docker-compose.yml
 ```



- Nous mettons le registry et l'interface dans le même réseau *registry_pozos-network-registry*.   
- Nous définissons des variables d'environnement pour pouvoir entre autres:
  Supprimer les images créées ```DELETE_IMAGES=true```             
- Pour donner un titre au régistre ```REGISTRY_TITLE=pozos-images-registry```     
- Pour définir l'url d'accès à l'interface ```NGINX_PROXY_PASS_URL=http://pozos-backend-registry:5000```    
- Et bien entendu, nous exposons les ports nécessaires  ```5000:5000``` et ```8090:80```          
  Etc...
         
Lançons maintenant notre stack par la commande ```docker-compose up -d```
 
Nous constatons aussi que le réseau *registry_pozos-network-registry* a été créé
 
En faisant un test sur le port exposé  8090 (comme mentionné dans le fichier docker-compose).     
On a bien l’affichage du registre pour le moment vide
 

Poussons-y l’image créée pour l’application de gestion des étudiants. Commençons par vérifier les images existantes
 
Par la suite, faisons un tag sur l’image
 
Nous pouvons désormais faire un push de l’image sur le régistre, précisant toujours le port 5000            
(comme précisé dans le fichier docker-compose associé)
 
Une fois cela fait, nous actualisons la page et pouvons constater le changement sur le frontend du régistre :
 
En cliquant sur le nom de l’image, on obtient beaucoup de détails:
 

## Conclusion:

Pour conclure, nous pouvons souligner que ce travail nous a permis de revoir un grand nombre de notions
abordées dans le cadre du module sur l’introduction à Docker.         
Nous avons pu approfondir en faisant des recherches notamment sur l’utilisation des volumes.        
Le caractère très concret de cet exercice permet aussi de se mettre dans des conditions proches de celles d’un cas d’entreprise.




