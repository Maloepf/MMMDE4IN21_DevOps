FROM eclipse-temurin:17-jdk-alpine
# Build Main.java with JDK
WORKDIR /usr/src
COPY Main.java .
RUN javac Main.java

FROM eclipse-temurin:17-jre-alpine
# Copy resource from previous stage
COPY --from=0 /usr/src/Main.class .
CMD ["java", "Main"]