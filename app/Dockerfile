# 1. 가벼운 JRE 기반 이미지 사용 (Alpine 기반)
FROM eclipse-temurin:17-jre-alpine

# 2. 작업 디렉토리 설정
WORKDIR /app

# 3. JAR 파일 복사
COPY step07_cicd-0.0.1-SNAPSHOT.jar app.jar

# 4. 실행 명령어 설정 (JVM 최적화 옵션 추가)
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]