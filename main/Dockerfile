# ── Build stage ───────────────────────────────────────────────────────────────
# Uses the Maven wrapper bundled in the repo so no external Maven install is needed.
FROM eclipse-temurin:17-jdk-alpine AS build
WORKDIR /workspace

# Copy wrapper and POM first so dependency layer is cached separately from source.
COPY .mvn/ .mvn/
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline -q

# Copy source and build the fat JAR (tests are run in CI before the image is built).
COPY src/ src/
RUN ./mvnw clean package -DskipTests -q

# ── Runtime stage ─────────────────────────────────────────────────────────────
# Minimal JRE image — no JDK, no build tools.
FROM eclipse-temurin:17-jre-alpine

# Run as a non-root user for security hardening.
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app
COPY --from=build /workspace/target/main-0.0.1-SNAPSHOT.jar app.jar

USER appuser
EXPOSE 8080

# Spring Boot fat JAR is self-contained; no extra arguments needed for the default (H2) profile.
# For production (RDS), inject SPRING_PROFILES_ACTIVE=prod and the DB_* env vars at runtime.
ENTRYPOINT ["java", "-jar", "app.jar"]

