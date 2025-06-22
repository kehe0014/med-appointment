FROM eclipse-temurin:21-jre-jammy

WORKDIR /app

# Copie directe du fichier JAR
COPY target/Appointment-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 9090

ENTRYPOINT ["java", "-jar", "app.jar"]
