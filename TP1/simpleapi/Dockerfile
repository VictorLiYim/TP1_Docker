FROM eclipse-temurin:21-jdk-alpine AS javaapp-build

WORKDIR /opt/javaapp

RUN apk add --no-cache maven

COPY pom.xml .
COPY src ./src

RUN mvn package -DskipTests

#Run
FROM eclipse-temurin:21-jre-alpine
WORKDIR /opt/javaapp
COPY --from=javaapp-build /opt/javaapp/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]