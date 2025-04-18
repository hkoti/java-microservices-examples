# Stage 1: Build
FROM eclipse-temurin:11-jdk-focal as build
WORKDIR /app
COPY gradlew .
COPY gradle ./gradle
COPY build.gradle .
COPY settings.gradle .
RUN chmod +x ./gradlew
RUN ./gradlew dependencies --no-daemon
COPY src ./src
RUN ./gradlew -Pprod clean bootJar --no-daemon -x test

# Stage 2: Runtime
FROM eclipse-temurin:11-jre-focal as runtime
WORKDIR /app
RUN addgroup --system appuser && \
    adduser --system --ingroup appuser appuser
COPY --from=build /app/build/libs/*.jar app.jar

ENV SPRING_PROFILES_ACTIVE="prod"
ENV JAVA_OPTS="-Xms256m -Xmx512m -Djava.security.egd=file:/dev/./urandom"
ENV EUREKA_CLIENT_SERVICEURL_DEFAULTZONE="http://localhost:8761/eureka/"
ENV SPRING_CLOUD_CONFIG_URI="http://localhost:8761"
# --- MongoDB Configuration ---
# Use K8s ConfigMap/Secret to provide the correct URI, potentially including credentials
# Example K8s value: mongodb://<user>:<password>@<mongodb-service>.<namespace>.svc.cluster.local:27017/store
ENV SPRING_DATA_MONGODB_URI="mongodb://localhost:27017/store"

RUN chown appuser:appuser /app/app.jar
USER appuser
# Expose default Spring Boot port; adjust if application-prod.yml sets server.port differently
EXPOSE 8080
ENTRYPOINT ["java", ${JAVA_OPTS}, "-jar", "app.jar"]
