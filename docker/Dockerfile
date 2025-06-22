# Build stage
FROM eclipse-temurin:21-jdk-jammy as builder
WORKDIR /workspace
COPY . .
RUN ./mvnw clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:21-jre-jammy
WORKDIR /app
ARG JAR_FILE=target/*.jar
COPY --from=builder /workspace/${JAR_FILE} app.jar
EXPOSE 9090
ENTRYPOINT ["java", "-jar", "app.jar"]