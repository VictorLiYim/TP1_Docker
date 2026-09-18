# TP 1 DOCKER Victor LI YIM

## DATABASE PostgreSQL

Les scripts sql sont copiés avec **COPY** dans /docker-entrypoint-initdb.d/
PostgreSQL exécute automatiquement ces scripts quand la base de données est initialisée pour la première fois.

Création de l'image :
```bash
docker build -t my-postgres .
docker network create app-network
```
Lancement PostgreSQL
```bash
docker run --name my-postgres \
--network app-network \
-e POSTGRES_DB=db \
-e POSTGRES_USER=usr \
-e POSTGRES_PASSWORD=pwd \
-v ~/postgres-data:/var/lib/postgresql/data \
-d my-postgres
```

**Adminer**

Adminer fournit une interface web pour accéder et gérer la base de donnée PostgreSQL.
Les paramètres de connexion sont les suivants :
System: PostgreSQL
Server: my-postgres
Username: usr
Password: pwd
Database: db

```bash
docker run \
-p "8090:8080" \
--network app-network \
--name adminer \
-d \
adminer
```

Adminer est accessible via:

http://localhost:8090

**Question 1**

Il vaut mieux utiliser -e car cela évite de mettre des informations sensibles, comme les mots de passe, directement
#dans le Dockerfile. Cela permet aussi de changer les variables d\'environnement sans modifier l\'image
#Docker.

**Question 2**

Sans volume les données sont stockées dans le conteneur, donc si on supprime le conteneur les données le sont avec.
Stocker les données dans le volume permet d'écarter les données du cycle de vie du conteneur

## Backend API

**Question 3**

On utilise le multistage build car ça permet de séparer la compilation et l'exécution ce qui nous permettra de rendre 
l'image finale plus légère.

**Les différentes étapes du Dockerfile (dans simpleapi)**

- Compilation: FROM eclipse-temurin:21-jdk-alpine AS javaapp
- Définition de dossier de travail : WORKDIR /opt/javaapp
- Installation du Maven : RUN apk add --no-cache maven
- Copie les dossiers dans le conteneur : COPY pom.xml ., COPY src ./src
- Compilation du projet pour génération du .jar : RUN mvn package -DskipTests
- Étape du Run : FROM eclipse-temurin:21-jre-alpine
- Dossier de travail : WORKDIR /opt/javaapp
  - Récupération du .jar : COPY --from=javaapp-build /opt/javaapp/target/*.jar app.jar
- Port exposé : EXPOSE 8080
- Lancement de l'application ENTRYPOINT ["java", "-jar", "app.jar"]

Comment le tester :
depuis le dossier simpleapi (cd simpleapi) lancez les commandes: 
- docker build -t javaapp .
- docker run --rm -p 8080:8080 javaapp
- Test basique : http://localhost:8080
- Pour tester le paramètre dans la fonction greeting : http://localhost:8080/?name=Victor

L'API retournera un fichier json de ce type :
{
"id": 1,
"content": "Hello, Victor!"
}

### Connecter le backend à PostgreSQL avec Docker

Le backend Spring Boot et PostgreSQL doivent être connectés au même réseau Docker.

Le réseau utilisé est `app-network`.

PostgreSQL est lancé avec :

```bash
docker run --name my-postgres \
  --network app-network \
  -e POSTGRES_DB=db \
  -e POSTGRES_USER=usr \
  -e POSTGRES_PASSWORD=pwd \
  -d my-postgres
```
Ensuite on lance le backend avec:

```bash
docker run --rm \
  --network app-network \
  -p 8080:8080 \
  javaapp
```
Dans application.yml, le backend utilise le nom du conteneur PostgreSQL pour se connecter à la base :
```bash
spring:
datasource:
url: jdbc:postgresql://my-postgres:5432/db
username: usr
password: pwd
driver-class-name: org.postgresql.Driver
```

Le backend peut ainsi communiquer avec PostgreSQL grâce au réseau Docker app-network.
Api : http://localhost:8080
Exemple d'endpoint : http://localhost:8080/departments/IRC/students