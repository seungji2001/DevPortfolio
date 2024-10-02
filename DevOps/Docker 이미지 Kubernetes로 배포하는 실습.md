# Docker 이미지 Kubernetes로 배포하는 실습

### 1. Jar파일을 포함하는 Docker 이미지 생성
##### a. Dockefile 작성
``` bash
# 베이스 이미지로 OpenJDK 사용
FROM openjdk:17-jdk-alpine

# 작업 디렉터리 생성
WORKDIR /app

# JAR 파일을 컨테이너로 복사
COPY java-app.jar /app/java-app.jar

# 애플리케이션 실행 명령
ENTRYPOINT ["java", "-jar", "/app/java-app.jar"]
```
##### b. Docker 이미지 빌드
```bash
docker build -t your-dockerhub-username/your-app-name .
```
##### c. 생성된 Docker이미지를 Docker 레지스트리에 푸시
```
docker login
docker push your-dockerhub-username/java-app
```


### 3. Kubernetes로 배포

##### a. Kubernetes 배포 파일 작성
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-app
spec:
  replicas: 3  # 3개의 복제본을 생성
  selector:
    matchLabels:
      app: java-app
  template:
    metadata:
      labels:
        app: java-app
    spec:
      containers:
      - name: java-app
        image: your-dockerhub-username/java-app  # Docker 레지스트리에 푸시된 이미지
        ports:
        - containerPort: 8999  # 애플리케이션이 사용하는 포트
        resources:
          limits:
            memory: "512Mi"
            cpu: "500m"
```
##### b. minikube를 활용한 테스트
```bash
minikube start
kubectl create deployment java-app --image=your-dockerhub-username/java-app --replicas=3
```
##### c. 접근 설정 위한 ip 확인
```bash
minikube ip
curl http://<ip>:port
```
##### d. NGINX 클러스터 External IP 추가
`Nginx 배포를 외부 로드밸런서를 통해 포트 80으로 노출하는 설정(로컬 test용)`
```bash
kubectl expose deployment nginx --type=LoadBalancer --port=8999
kubectl get services
```
```bash
NAME                             READY   STATUS    RESTARTS   AGE
pod/nginx-676b6c5bbc-2xbw9       1/1     Running   0          140m
pod/nginx-676b6c5bbc-nctp6       1/1     Running   0          140m
pod/nginx-676b6c5bbc-z6rj4       1/1     Running   0          140m
pod/springapp-77cd695757-9sldw   1/1     Running   0          38m
pod/springapp-77cd695757-cmm8q   1/1     Running   0          38m
pod/springapp-77cd695757-kddzv   1/1     Running   0          38m

NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
service/kubernetes   ClusterIP      10.96.0.1       <none>          443/TCP          149m
service/nginx        LoadBalancer   10.109.14.44    10.109.14.44    80:31802/TCP     115m
service/springapp    LoadBalancer   10.97.171.219   10.97.171.219   8999:32486/TCP   29m

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx       3/3     3            3           140m
deployment.apps/springapp   3/3     3            3           38m

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-676b6c5bbc       3         3         3       140m
replicaset.apps/springapp-77cd695757   3         3         3       38m
```
`minikube의 경우 할당 받은 external-ip가 아닌 minikube ip를 통해 탐색된 ip를 external ip로 테스트한다`
##### e. Minikube에서 외부 IP 제공받기 위한 터널링
```bash
minikube tunnel  
minikube ip
kubectl get services
http://<EXTERNAL_IP>:80
```
##### d. VM에서 실행한 경우, NAT를 사용하여 포트 포워딩 하는 과정 필요
![image](https://github.com/user-attachments/assets/3f1ff956-4fdc-45a7-9b22-b253d698843d)

##### 결과
![image](https://github.com/user-attachments/assets/b7ec6777-055c-4024-95db-28f6a1c3cb97)
![image](https://github.com/user-attachments/assets/6202b051-4c7c-4f40-8cf9-81c250824513)
