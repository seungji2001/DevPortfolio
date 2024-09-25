# Trivy
Trivy는 컨테이너 이미지뿐만 아니라, 파일 시스템, Git 리포지토리, Dockerfile, Kubernetes 매니페스트 등에서 보안 취약점과 잘못된 설정을 검사하는 도구입니다.

## Trivy 장점
다양한 CI 파이프라인 도구와 통합이 가능하여 Travis CI, CircleCI, Jenkins, GitLab CI 등과 연동할 수 있습니다.
## 지원 플랫폼
Ubuntu, CentOS, MacOS와 같은 여러 리눅스 배포판에서 실행될 수 있습니다.
## Trivy 설치
Ubuntu에서 Trivy를 설치하는 방법으로, 필요한 패키지 설치부터 GPG 키 추가, 리포지토리 등록, 그리고 실제 Trivy 설치 명령어가 포함되어 있습니다.

### 설치 명령어
Ubuntu 18.04에서 Trivy를 설치하기 위한 명령어
```
$ sudo apt-get install wget apt-transport-https gnupg lsb-release
$ wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
$ echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
$ sudo apt-get update
$ sudo apt-get install trivy
```

## Trivy 설치 이후 취약점 스캔
### 취약점 스캔 실행
Trivy가 설치된 후, Docker 이미지를 대상으로 보안 취약점 스캔을 수행할 수 있습니다.
```
$ trivy image nginx:latest
```
스캔 결과 많은 취약점이 발견될 수 있으며, 가벼운 **Alpine**을 사용하는것이 더 나을 수 있다고 기사에서 언급되었습니다.
### 설정 파일에서 보안 취약점 발견
Dockerfile, Kubernetes 매니페스트 등의 구성 파일에서도 보안 취약점을 탐지할 수 있습니다. 
```
$ trivy config <config_dir>
```
