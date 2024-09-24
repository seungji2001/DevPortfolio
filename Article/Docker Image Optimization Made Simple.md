# Docker 이미지 최적화

### 1. 최소 베이스 이미지 선택
Alpine Linux나 Scratch와 같은 경량 베이스 이미지를 사용
```
FROM nginx:alpine
```
### 2.단일 책임 원칙 적용
각 서비스를 개별 이미지로 구성한 후, **Docker Compose**나 **Kubernetes**로 서비스 통합 후 관리
### 3. 다단계 빌드 사용
빌드 시 필요한 종속성과 Production단계로 나누어 이미지 크기를 축소
```
# Build 단계
FROM node:14-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production 단계
FROM node:14-alpine as production
WORKDIR /app
COPY --from=build /app/package*.json ./
RUN npm ci --production
COPY --from=build /app/dist ./dist
CMD ["npm", "start"]
```
### 4.레이어 최소화
RUN 명령어를 결합
```
RUN apt-get update && \
apt-get install -y git && \
apt-get clean && \
rm -rf /var/lib/apt/lists/*
```
### 5. .dockerignore 파일 사용
.dockerignore 파일을 만들어 불필요한 파일이나 디렉토리를 Docker 빌드 과정에서 제외
### 6. 구체적인 태그 사용
베이스 이미지나 종속성을 가져올 때는 latest 대신 특정 버전 태그를 사용하여 예측 가능한 환경을 유지
```
FROM nginx:<tag>
```
### 7. Dockerfile 명령어 최적화
패키지 설치 시 특정 버전을 사용 및 불필요한 종속성을 제거

### 8. 아티팩트 압축
애플리케이션이 생성하는 바이너리나 정적 파일을 Docker 이미지에 복사하기 전에 압축하여 빌드 속도와 이미지 크기 축소

### 9. 이미지 레이어 분석
docker history와 docker inspect 명령어를 사용하여 이미지 레이어를 분석하고 불필요한 파일이나 명령을 제거

### 10. Docker 이미지 정리
docker system prune 명령어로 사용하지 않는 이미지, 컨테이너, 볼륨, 네트워크를 정리하여 디스크 공간을 확보 및 성능을 향상

### 11. 캐싱 활용
Docker의 빌드 캐시를 최대한 활용하기 위해 Dockerfile을 구조화 및 자주 변경되는 명령어는 파일의 마지막에 배치하여 캐시 무효화를 최소화

### 12. 보안 스캐닝
Docker 보안 스캔 도구를 사용해 이미지의 취약점을 식별하고 수정

### 13. 더 작은 대안 사용
가능한 경우 더 작은 도구나 라이브러리를 사용하세요. 예를 들어, BusyBox나 Microcontainers와 같은 경량 대안을 고려

### 14. 설치 후 정리
패키지 설치 후 생성된 임시 파일과 캐시를 제거하여 이미지 크기 축소

### 15. Docker Squash 사용
Docker Squash를 사용하여 여러 레이어를 병합해 이미지 크기 축소

[Reduce the size of the Docker Image](https://overcast.blog/docker-image-optimization-tips-tricks-6a17f687162b)
