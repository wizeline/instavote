FROM maven as builder

WORKDIR /src
COPY ./ .
RUN mvn -B package

# RUNTIME
FROM gcr.io/distroless/java17-debian11
USER nonroot:nonroot
COPY --chmod=555 --from=builder /src/target/worker-*-jar-with-dependencies.jar /app.jar
CMD ["-Dspring.profiles.active=deployment","/app.jar"]