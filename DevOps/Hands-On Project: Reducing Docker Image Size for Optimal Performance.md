# "Hands-On Project: Reducing Docker Image Size for Optimal Performance"

## Docker 이미지 최적화 실습
이 시나리오는 기본적인 Docker 이미지를 최적화하기 전과 후의 변화를 비교하기 위한 실습입니다. 동일한 애플리케이션을 사용하되, 최적화된 전략을 적용하여 이미지 크기, 빌드 속도, 성능 등을 비교할 수 있습니다.

#### Node.js 애플리케이션 생성
```
npx express-generator myapp
cd myapp
npm install
```

### Part 1: 최적화 전 Docker 이미지
#### Dockerfile 최적화전
```
# Node.js 이미지 사용
FROM node:14

# 앱 디렉토리 설정
WORKDIR /usr/src/app

# 앱 의존성 설치
COPY package*.json ./
RUN npm install

# 앱 소스 복사
COPY . .

# 포트 설정
EXPOSE 3000

# 애플리케이션 실행
CMD ["npm", "start"]

```

#### .dockerignore 최적화전
```
node_modules
npm-debug.log
```

#### 빌드 및 실행
```
docker build -t myapp:unoptimized .
docker run -d -p 3000:3000 myapp:unoptimized
```

#### 분석
```
docker images myapp:unoptimized
```

```
username@servername:~/myapp$ docker images myapp:unoptimized
REPOSITORY   TAG           IMAGE ID       CREATED         SIZE
myapp        unoptimized   57defb863c1e   2 minutes ago   923MB
```

### Part 2: 최적화 후 Docker 이미지
이제 최적화 기법들을 적용하여 이미지 크기와 성능을 개선합니다.

#### 1. 최소 베이스 이미지 사용: node:14-alpine으로 변경하여 가벼운 베이스 이미지 사용.
#### 2. 다단계 빌드: 빌드 단계와 실행 단계를 분리하여 불필요한 파일 제거.
#### 3. 레이어 최소화: RUN 명령어 결합으로 레이어 수를 줄임.
#### 4. .dockerignore 사용: 불필요한 파일을 제외.

#### 최적화 후 Dockerfile
```
# Build 단계
FROM node:14-alpine AS build

# 앱 디렉토리 설정
WORKDIR /usr/src/app

# 의존성 설치
COPY package*.json ./
RUN npm install

# 소스 코드 복사 및 빌드
COPY . .
RUN npm run build

# Production 단계
FROM node:14-alpine AS production

# 앱 디렉토리 설정
WORKDIR /usr/src/app

# Build 단계에서 빌드된 파일 복사
COPY --from=build /usr/src/app/dist ./dist

# Production 환경에서의 의존성 설치
COPY package*.json ./
RUN npm ci --production

# 포트 설정
EXPOSE 3000

# 애플리케이션 실행
CMD ["npm", "start"]
```
#### 최적화 후 .dockerignore
```
node_modules
npm-debug.log
Dockerfile
.dockerignore
dist/
.git
.gitignore
README.md
```
#### 최적화 후 빌드 및 실행
```
docker build -t myapp:optimized .
docker run -d -p 3000:3000 myapp:optimized
```
#### 분석
```
docker images myapp:optimized
```

### Part 3: 비교 및 결과
1. **이미지 크기 비교**:
    - **최적화 전**: 약 900MB
    - **최적화 후**: 약 100MB
    → 이미지 크기가 **약 8배** 감소
2. **레이어 수 비교**:
    - **최적화 전**: 많은 레이어 (각 `RUN` 명령마다 레이어 생성)
    - **최적화 후**: 적은 레이어 (명령어 결합으로 레이어 수 감소)
3. **빌드 속도 및 배포 속도**:
    - 최적화된 이미지가 훨씬 빠르게 빌드되고 배포됩니다. 용량이 작아 배포 속도와 네트워크 전송 속도가 향상됩니다.
4. **보안 및 관리**:
    - 최소한의 종속성 및 불필요한 파일 제거로 보안성이 향상되고, 관리가 더 쉬워집니다.
