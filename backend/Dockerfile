FROM openjdk:8-jre
EXPOSE 8080

# add the backend jar
ADD target/backend-*.jar /opt/

# rename to backend.jar
RUN mv /opt/backend-*.jar /opt/backend.jar

ENTRYPOINT ["java", "-Xms256m", "-Xmx512m", \
"-jar", "/opt/backend.jar"]