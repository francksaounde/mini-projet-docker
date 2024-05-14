
# Mini-project docker

#### Project carried out by Saounde Franck 

## Introduction
In this document we answer questions relating to the mini project on docker as part of the devops 19 Eazytraining bootcamp.

## Build and test :

To build the api image, let's go to the directory containing the corresponding Dockerfile     

```bash 
cd mini-projet-docker/
git clone https://github.com/diranetafen/student-list.git
cd student-list/simple_api/
```
Once we're in the right directory 
let's update the Dockerfile following the recommendations in the statement :

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

Let's now check the list of available images and start creating the image       
which we call *api_image*, from the Dockerfile we've just modified.     

Since we're in the directory containing the Dockerfile, we indicate the current context with a dot (‘.’)                          


![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/build_api_image.png)

Once the image has been created, check the list of images again,     
We can see that it has been updated:     
 

![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/confirmation%20build%20api_image.png)


We can now launch an api container based on this new image.      
Let's name our new container *api_app*.        
At the same time, mount the current directory (${PWD}) as a volume and associate it with the container's /data directory.       
The command to launch our container is then:     

```bash
docker run -d --name api_app -v ${PWD}:/data -p 5000:5000 api_image
```

Having launched the container in the background (thanks to the -d option in the command), let's look at the list of all our containers (-a option)    
And check the logs to confirm that our single container has been launched and is ready to listen:        

![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/logs%20d%C3%A9marrage%20container%20api_app.png)
 

We're now going to make a request to the container using the ‘curl’ command.       

We modify the command provided in the statement by adding the address of the host - which is our virtual machine -   
on which the container is running (hence localhost), and we specify port 5000 which has been exposed.     
We can now see the information for registered students:         


![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/curl%20api_app.png)             

 


## Infrastructure As Code :

We edit a docker-compose.yml file to deploy our two services.     
Let's give a few comments on its contents:

- We indicate the version of the docker-compose ```version: "3.8"```

##### Web Service

- *web_app* is the name of the frontend container ```container_name: web_app```

- The *web_app* container runs on the *php:apache* image ```image: php:apache```

-Apache is a web server that uses port 80 on the host  
We are therefore forced to use port 80 at container level   
We choose to associate it with external port 80,    
Hence: ```ports "80:80"```    

- We specify the user *toto* and its password *python*,   
as supplied by the ‘curl’ command used earlier and as mentioned in the python code

```
- environment:
      - USERNAME=toto
      - PASSWORD=python
```

- We tell Apache to use our index.php file instead of its default page

  ```volumes: ./website:/var/www/html```
 

- We define a network in which we put the two containers.    
This will allow them to communicate:      
   ```networks: student_network```         



##### Api Service:
   
- We indicate the directory in which to find the Dockerfile to build the api image       
  ```build: context: ./simple_api```
    
- As in the previous section, we are mounting a volume        
    ```volumes:"${PWD}/simple_api:/data"```
   
- We expose port 5000 inside the container and associate it with port 5000 on the host    
    ```ports: "5000:5000"```


To sum up, here is the structure of our docker-compose.yml file

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

First, let's delete the container created earlier             
 

![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/suppression%20container%20api_app.png)



Let's update the docker-compose.yml file        
 
![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/docker%20compose%20api_app%20et%20web_app.png)


Let's update the index.php file with the name of the api container and the exposed port 
(5000 as mentioned in the docker-compose)                
 
![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/update_url_index_php.png)


Now let's run our stack with the ```docker-compose up``` command         


![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/docker-compose-up-api-and-web-app.png)


 

Let's also check that the api is accessible via our web interface.        
After clicking on the ‘List Student’ button, we have the list of registered students         


![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/ihm-api-and-web-app-confirmation.png)                    




 
## Docker Registry 

First we delete the stack launched previously,         

![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/docker-compose-down-api-and-web-app.png)


Then we create a *‘registry’* directory and place it inside. Then we create    
the docker-compose file for the registry and the interface:    

```bash 
mkdir registry
cd registry/
vi docker-compose.yml
cat docker-compose.yml
 ```                          

![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/docker-compose-registry.png)


- We put the registry and the interface on the same *registry_pozos-network-registry* network.   
- We define environment variables so that we can, among other things:
  Delete images created ```DELETE_IMAGES=true```             
- To give a title to the ```REGISTRY_TITLE=pozos-images-registry``` register     
- To define the access url for the ```NGINX_PROXY_PASS_URL=http://pozos-backend-registry:5000``` interface    
- And, of course, we expose the necessary  ```5000:5000``` and ```8090:80``` ports          
  And so on...
         
Now let's run our stack with the ```docker-compose up -d``` command         
Next, let's check that our stack has been started using the ```docker-compose ps``` command      


![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/docker-compose-ps-confirmation-registry.png)


We can also see that the *registry_pozos-network-registry* network has been created               


![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/docker-network-ls-confirmation-network-registry.png)

 
By running a test on the exposed port 8090 (as mentioned in the docker-compose file).     
The registry is displayed as empty for the moment           


 ![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/ihm-confirmation-registry-vide.png)
 

Let's push in the image created for the student management application. Let's start by checking the existing images                


![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/liste-des-images-apres-d%C3%A9marrage-registry.png)

 
Next, tag the image           

![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/docker-tag-image-a-push.png)

 
We can now push the image to the registry, always specifying port 5000         
(as specified in the associated docker-compose file)               


![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/docker-push-image-to-registry.png)
 
Once this has been done, we refresh the page and can see the change on the registry frontend:            


![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/confirmation%20ihm%20image%20pouss%C3%A9e%20sur%20regitry.png)


Clicking on the name of the image reveals a great deal of detail:            

![Alt text](https://github.com/francksaounde/student-list/blob/main/screenshots/d%C3%A9tails-image-pouss%C3%A9e-sur-registry.png)
 


## Conclusion:

To conclude, we can emphasise that this work has enabled us to review a large number of notions
covered in the Introduction to Docker module.         
We were able to delve deeper by doing some research, particularly into the use of volumes.        
The very concrete nature of this exercise also allowed us to put ourselves in conditions close to those of a business case.




