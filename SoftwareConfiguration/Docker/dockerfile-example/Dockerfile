# syntax=docker/dockerfile:1

FROM openjdk:19
WORKDIR /app
COPY simple-project.jar ./simple-project.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/simple-project.jar"]
