# Minimize Docker Image Size
![image](https://github.com/user-attachments/assets/2f9d2784-ab16-4054-8d39-930de01c1a76)

## Use Minimal Base Images
**Ubuntu**(128MB)와 같은 표준이미지보다 작은 **Alphine**(5MB)사용 방법이 있으나, Alphine의 취약점과 wget과 같은 불필요한 프로그램의 포함 
으로 **Distroless** 이미지를 사용하기도 합니다. 
```
TAG                                  SIZE
debian                               124.0MB
alpine                               5.0MB
gcr.io/distroless/static-debian11    2.51MB
```

### Minimize Layers
**Dockerfile**에서 명령어들을 결합하여 레이어 수를 줄일 수 있습니다. `RUN` 명령어를 하나로 합치면 레이어 수를 줄일
수 있으며, 이미지의 전체 크기도 줄일 수 있습니다. 
```
FROM golang:1.18 as build
...
RUN go mod download
RUN go mod verify
```
다음과 같이 변경할 수 있습니다.
```
FROM golang:1.18
...
RUN go mod download && go mod verify
```
이 경우 이미지의 최적화와 불필요한 레이어의 생성을 막아 이미지의 크기를 줄일 수 있습니다.

### Multi-Stage Builds
빌드 환경을 최종 이미지에서 분리할 수 있습니다. 애플리케이션을 빌드할 때 필요한 여러 파일이나
도구는 빌드 단계에서만 사용되며, 최종 실행 단계에는 포함되지 않게 하는 방법입니다. 

빌드시는 많은 파일과 툴이 필요하지만, 실제 컨테이너 실행시는 꼭 필요한 실행파일과 몇 가지 종속성
만 있으면 되기 때문입니다.

```
# 애플리케이션 빌드이 단계에서는 필요한 모든 개발 도구와 라이브러리를 사용하여
# 애플리케이션을 빌드하고, 최종적으로 실행 파일을 생성합니다.

FROM golang:1.20-alpine AS build  # 첫 번째 단계: Go 애플리케이션 빌드

FROM golang:1.20-alpine AS build  # 첫 번째 단계: Go 애플리케이션 빌드
WORKDIR /build                    # 작업 디렉토리 설정
COPY go.mod go.sum ./             # 종속성 파일 복사
RUN go mod download && go mod verify  # 종속성 다운로드 및 검증
COPY . .                          # 나머지 소스 코드 복사
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o run .  # 실행 파일 빌드

# 두 번째 단계 (최종 실행 환경) 빌드된 실행 파일을 distroless 이미지로 복사
FROM gcr.io/distroless/static-debian11
WORKDIR /app
COPY --from=build /build/run .
CMD ["/app/run"]
```

### Docker Slim
 Docker 이미지의 내용을 분석하고, 불필요한 부분을 제거하여 이미지를 압축할 수 있습니다. 예시 명령어는 nginx 이미지를 Docker Slim으로 최적화하는 방법을 보여줍니다.
 ```
docker-slim build --sensor-ipc-mode proxy --sensor-ipc-endpoint 172.17.0.1 --http-probe=false nginx;
```


---- 
 [Reduce the size of the Docker Image](https://faun.pub/reduce-the-size-of-the-docker-image-e6895b653419)
 <br/>
 Docker 이미지의 효율적인 배포와 자원 사용 최소화를 위한 방법에 대해 정리한 글입니다. 이 글은 Docker 사용 시 성능 향상 및 자원 최적화를 목표로 여러 전략을 적용하려는 저자의 고민과 해결 방안을 다룹니다.
 
