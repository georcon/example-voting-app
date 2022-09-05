FROM maven:3.6.1-jdk-8-alpine

WORKDIR /app

COPY . .

RUN mvn compile

RUN mvn package -DskipTests
RUN mv target/worker-jar-with-dependencies.jar /run/worker.jar
RUN rm -rf *
CMD java -jar /run/worker.jar

