# Stage 1: Build stage
FROM eclipse-temurin:21-jdk-jammy AS build

# Thiết lập thư mục làm việc
WORKDIR /app

# Copy các tệp cấu hình Maven Wrapper trước để tận dụng cache
COPY .mvn/ .mvn/
COPY mvnw pom.xml ./

# Cấp quyền thực thi cho mvnw
RUN chmod +x mvnw

# Tải dependencies (layer này sẽ được cache trừ khi pom.xml thay đổi)
RUN ./mvnw dependency:go-offline -B

# Copy mã nguồn
COPY src ./src

# Build ứng dụng (skip tests để build nhanh hơn, hoặc bỏ -DskipTests nếu cần)
RUN ./mvnw clean package -DskipTests

# Stage 2: Runtime stage
# Sử dụng Ubuntu (Jammy) thay vì Alpine vì Playwright hỗ trợ thư viện hệ thống tốt nhất trên Ubuntu
FROM mcr.microsoft.com/playwright:v1.49.0-jammy

# Thiết lập JDK 21 cho môi trường runtime (vì image Playwright gốc là Node-based)
RUN apt-get update && apt-get install -y openjdk-21-jre-headless dumb-init && rm -rf /var/lib/apt/lists/*

# Thiết lập thư mục làm việc
WORKDIR /app

# Copy tệp JAR từ stage build
COPY --from=build /app/target/*.jar app.jar

# Tạo user không phải root để bảo mật
RUN groupadd -r spring && useradd -r -g spring spring
RUN chown -R spring:spring /app
USER spring:spring

# Cấu hình Port
ENV PORT=8080
EXPOSE $PORT

# Playwright thường tải trình duyệt vào thư mục home, hãy đảm bảo app biết chỗ tìm
ENV PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1

ENTRYPOINT ["dumb-init", "--"]

CMD ["java", "-XX:+UseZGC", "-XX:MaxRAMPercentage=75.0", "-Dserver.port=${PORT}", "-jar", "app.jar"]
