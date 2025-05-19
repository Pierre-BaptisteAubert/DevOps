# DevOps - Pierre-Baptiste Aubert
## TP 19/05/2025
### Questions
#### 1-1 For which reason is it better to run the container with a flag -e to give the environment variables rather than put them directly in the Dockerfile?

It is better to use -e to pass environment variables at runtime rather than putting them in the Dockerfile, because it makes the image more generic, reusable, and secure.

#### 1-2 Why do we need a volume to be attached to our postgres container?
We need to attach a volume to our Postgres container to persist the database data outside the container's filesystem.

#### 1-3 Document your database container essentials: commands and Dockerfile.

1- Create the flask-app with a Dockerfile an init folder with the SQL query
In the Dockerfile :
-put the base image ``FROM postgres:17.2-alpine``
-put a command lign to copy all the query in ``/docker-entrypoint-initdb.d``

2- Build the image with : ``docker build -t pbaubert/myfirstdatabase .``

3- Create the network for the connection between the frontend and backend : ``docker network create app-network``

4- Run the container with : ``docker run -p 8888:5000 --name my-postgres-container --network app-network -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd -v 'C:\Users\utilisateur\Desktop\Pb\EPF\S7\DevOps\DevOps\TP1-19-05-2025\datapostgres:/var/lib/postgresql/data' pbaubert/myfirstdatabase``

5- Pull adminer image

6- Run adminer container :  ``docker run -p 8090:8080 --net=app-network --name=adminer -d adminer``

 