# Stage 1: Build Backend (Java) & Frontend (Node)
# Use a base image with both Node.js and JDK
FROM node:16-jdk11 as build
WORKDIR /app

# 1. Cache Node.js dependencies
COPY package.json package-lock.json ./
RUN npm install --legacy-peer-deps

# 2. Cache Gradle dependencies
COPY gradlew .
COPY gradle ./gradle
COPY build.gradle .
COPY settings.gradle .
RUN chmod +x ./gradlew
# './gradlew dependencies' might fail if frontend build is tied too early; skip if problematic
# RUN ./gradlew dependencies --no-daemon

# 3. Copy source code
COPY . .

# 4. Build the application (includes frontend build via npm task triggered by gradle)
RUN ./gradlew -Pprod clean bootJar --no-daemon -x test

# Stage 2: Runtime Environment
FROM eclipse-temurin:11-jre-focal as runtime
WORKDIR /app

# Create a non-root user
RUN addgroup --system appuser && \
    adduser --system --ingroup appuser appuser

# Copy the executable JAR from the build stage
COPY --from=build /app/build/libs/*.jar app.jar

# Define Environment Variables (defaults/placeholders, override in K8s)
ENV SPRING_PROFILES_ACTIVE="prod"
ENV JAVA_OPTS="-Xms256m -Xmx512m -Djava.security.egd=file:/dev/./urandom"
ENV EUREKA_CLIENT_SERVICEURL_DEFAULTZONE="http://localhost:8761/eureka/"
ENV SPRING_CLOUD_CONFIG_URI="http://localhost:8761"
# --- OAuth2/OIDC ---
ENV SPRING_SECURITY_OAUTH2_CLIENT_PROVIDER_OIDC_ISSUER_URI="http://localhost:9080/auth/realms/jhipster"
ENV SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_OIDC_CLIENT_ID="web_app"
ENV SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_OIDC_CLIENT_SECRET="<override-me-in-k8s-secret>"
# Add other necessary ENV vars (e.g., database if gateway uses one)

# Set ownership and switch user
RUN chown appuser:appuser /app/app.jar
USER appuser

# Expose port (default JHipster gateway port)
EXPOSE 8080

# Run the application
ENTRYPOINT ["java", ${JAVA_OPTS}, "-jar", "app.jar"]
