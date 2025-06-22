# Build stage
FROM eclipse-temurin:21-jdk-jammy as builder
WORKDIR /workspace

# Install Maven
RUN apt-get update && apt-get install -y maven

COPY pom.xml .
COPY src src

# Build with installed Maven
RUN mvn clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:21-jre-jammy
WORKDIR /app
COPY --from=builder /workspace/target/*.jar app.jar
EXPOSE 9090
ENTRYPOINT ["java", "-jar", "app.jar"]