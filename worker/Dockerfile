ARG JAVA_VERSION
FROM maven:3.9.10-eclipse-temurin-${JAVA_VERSION} AS builder

WORKDIR /src
COPY ./ .
RUN mvn -B package

# RUNTIME
FROM gcr.io/distroless/java${JAVA_VERSION}-debian12
USER nonroot:nonroot
COPY --chmod=555 --from=builder /src/target/worker-*-jar-with-dependencies.jar /app.jar
CMD ["-Dspring.profiles.active=deployment","/app.jar"]