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
RUN ./gradlew -Pprod clean bootJar --no-daemon

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
ENV SPRING_DATASOURCE_URL="jdbc:postgresql://localhost:5432/blog"
ENV SPRING_DATASOURCE_USERNAME="<override-me-in-k8s-secret>"
ENV SPRING_DATASOURCE_PASSWORD="<override-me-in-k8s-secret>"

RUN chown appuser:appuser /app/app.jar
USER appuser
EXPOSE 8080
ENTRYPOINT ["java", ${JAVA_OPTS}, "-jar", "app.jar"]
