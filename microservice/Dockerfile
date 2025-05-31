# syntax=docker/dockerfile:1

# → Etapa de construcción: jalar código y generar JAR
FROM maven:3.9.7-eclipse-temurin-21 AS build
WORKDIR /workspace/app

# Copiamos únicamente pom.xml primero: esto permite cachear dependencias si sólo cambia el código
COPY pom.xml .

# Carpeta src completa
COPY src ./src

# Compilar sin tests para agilizar
RUN mvn -q -B package -DskipTests

# → Etapa de runtime: imagen más ligera sólo con JRE
FROM eclipse-temurin:21-jre
ENV JAVA_OPTS="-XX:+UseContainerSupport"
WORKDIR /app

# Copiamos el JAR compilado desde la etapa “build”
COPY --from=build /workspace/app/target/*-SNAPSHOT.jar app.jar

# Exponemos el puerto 8080
EXPOSE 8080

# Comando de entrada
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]
