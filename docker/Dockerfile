FROM eclipse-temurin:21-jre-jammy

WORKDIR /app

# Le JAR est copié depuis le context racine du projet
ARG JAR_FILE=target/Appointment-0.0.1-SNAPSHOT.jar
COPY ${JAR_FILE} app.jar

EXPOSE 9090

ENTRYPOINT ["java", "-jar", "app.jar"]
