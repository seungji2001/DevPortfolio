# Docker Volume

**Docker volume** 사용을 통해 컨테이너 삭제 또는 재시작 시 내부 저장 데이터 사라지는 문제를 해결 할 수 있으며, 여러 컨테이너가 동일한 볼륨 공유 가능
하도록 해줍니다. 또한, 호스트와의 데이터 공유 가능하며, 컨테이너 삭제 후에도 볼륨은 독립적으로 유지가 가능합니다.

**volume 생성**
```
docker volume create myvolume
```
**volume 정보**
```
username@servername:~/step04Storage$ docker volume inspect myvolume
[
    {
        "CreatedAt": "2024-09-25T01:33:46Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/myvolume/_data",
        "Name": "myvolume",
        "Options": null,
        "Scope": "local"
    }
]
```

**볼륨 목록 보기**
```
 docker volume ls
```

**볼륨으로 생성된 디렉터리 확인**
```
sudo ls /var/lib/docker/volumes/<생성된볼륨명>/_data
```
해당 경로 안에 기존 파일 copy 가능

**호스트와 컨테이너의 디렉터리 연결할 컨테이너 구동(생성)**
```
docker run --name mynginxserver -d -v <생성된볼륨명>:/usr/share/nginx/html -p 8081:80 nginx
docker run --name mynginxserver -d -v <생성된볼륨명>:/usr/share/nginx/html -p 8080:80 nginx
```
두개의 컨테이너를 생성한 후 같은 볼륨에 접근가능한지 확인을 위해 두개의 컨테이너를 생성
**curl 또는 웹브라우저로 host 에서 file 수정 및 docker와의 동기화 확인**
```
curl http://127.0.0.1:8081/html 접속 확인
curl http://127.0.0.1:8080/html 접속 확인
```


