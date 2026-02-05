# ---------- Build Stage ----------
FROM maven:3.9.9-eclipse-temurin-17 AS builder

WORKDIR /build

COPY pom.xml .
RUN mvn -B -q -e -C -DskipTests package || true

COPY src ./src
RUN mvn -B clean package -DskipTests

# ---------- Runtime Stage ----------
FROM eclipse-temurin:17-jre-jammy

WORKDIR /app

RUN groupadd -r appgroup && useradd -r -g appgroup appuser

COPY --from=builder /build/target/*.jar app.jar

USER appuser

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
