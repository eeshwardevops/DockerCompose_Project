FROM maven:3.9.9-eclipse-temurin-21-jammy AS BUILDER
RUN git clone https://github.com/gajulaeshwar9/vprofile-project.git
RUN cd vprofile-project && mvn install


FROM tomcat:10-jdk21 
RUN rm -rf /usr/local/tomcat/webapps/*
COPY --from=BUILDER vprofile-project/target/*.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]

