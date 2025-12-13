# Giai đoạn 1: Build ứng dụng
FROM maven:3.9.6-eclipse-temurin-21-jammy AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B
COPY src ./src
RUN mvn clean package -DskipTests

# Giai đoạn 2: Chạy ứng dụng
# Sử dụng Ubuntu Jammy vì nó tương thích tốt nhất với các lệnh apt-get install-deps
FROM eclipse-temurin:21-jre-jammy
WORKDIR /app

# Cài đặt các thư viện hệ thống cần thiết cho Playwright
RUN apt-get update && apt-get install -y \
    libglib2.0-0 \
    libnss3 \
    libnspr4 \
    libdbus-1-3 \
    libatk1.0-0 \
    libatk-bridge2.0-0 \
    libcups2 \
    libdrm2 \
    libx11-6 \
    libxcomposite1 \
    libxdamage1 \
    libxext6 \
    libxfixes3 \
    libxrandr2 \
    libgbm1 \
    libxcb1 \
    libxkbcommon0 \
    libpango-1.0-0 \
    libcairo2 \
    libasound2 \
    && rm -rf /var/lib/apt/lists/*

# Copy file jar từ bước build
COPY --from=build /app/target/*.jar app.jar

# Biến môi trường quan trọng
ENV PORT=8080
# Cho phép Playwright tự tải trình duyệt nếu chưa có trong image này
ENV PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=0 

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
