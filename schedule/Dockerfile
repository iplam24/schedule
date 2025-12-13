# Giai đoạn 1: Build file JAR bằng Java 21
FROM maven:3.9.6-eclipse-temurin-21-jammy AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B
COPY src ./src
RUN mvn clean package -DskipTests

# Giai đoạn 2: Chạy ứng dụng bằng Image Playwright chính chủ
# Phiên bản v1.49.0 khớp với version trong pom.xml của bạn
FROM mcr.microsoft.com/playwright/java:v1.49.0-jammy
WORKDIR /app

# Copy file jar từ bước build sang
COPY --from=build /app/target/*.jar app.jar

# Biến môi trường bắt buộc cho Railway
ENV PORT=8080
# Playwright trong image này đã có sẵn trình duyệt, không cần tải lại
ENV PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
