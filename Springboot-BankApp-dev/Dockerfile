#----------------------------------
# Stage 1 - Build Environment
#----------------------------------

# Use Maven with OpenJDK 17 to compile the application
FROM maven:3.8.3-openjdk-17 AS builder

# Set the working directory in the container
WORKDIR /app

# Copy all project files to the working directory
COPY . /app

# Build the project and skip tests for a faster build process
RUN mvn clean install -DskipTests=true

#--------------------------------------
# Stage 2 - Production Environment
#--------------------------------------

# Use a lightweight OpenJDK 17 image to run the application
FROM openjdk:17-jdk-alpine

# Copy the application JAR from the builder stage to the target location
COPY --from=builder /app/target/*.jar /app/target/bankapp.jar

# Expose the application port (updated to 8000)
EXPOSE 8000

# Define the command to run the application
ENTRYPOINT ["java", "-jar", "/app/target/bankapp.jar"]