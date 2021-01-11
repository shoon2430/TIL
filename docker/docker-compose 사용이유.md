# Docker-compose 사용이유



### 1. docker 실행 명령어를 일일이 입력하기가 복잡해서

```powershell
# nginx 컨테이너 실행
docker run -it nginx
```

![1_1](https://github.com/shoon2430/TIL/blob/master/images/docker/1_1.png)



```powershell
# nginx 컨테이너 실행 + 호스트의 8080 포트 연결
docker run -it -p 8080:80 nginx
```

![1_2](https://github.com/shoon2430/TIL/blob/master/images/docker/1_2.png)



```powershell
# nginx 컨테이너 실행 + 호스트의 8080 포트 연결 + 컨테이너 종료시 자동 삭제
docker run -it -p 8080:80 --rm nginx

# nginx 컨테이너 실행 + 호스트의 8080 포트 연결 + 컨테이너 종료시 자동 삭제 + 호스트의 디렉터리를 컨테이너 안에 링크
docker run -it -p 8080:80 --rm -v $(pwd):/usr/share/nginx/html/ nginx
```

![1_3](https://github.com/shoon2430/TIL/blob/master/images/docker/1_3.png)



### 2. 컨테이너끼리 연결하기 편해서

```powershell
# 준비 django-sample 이미지를 빌드합니다
git clone https://github.com/raccoonyy/django-sample-for-docker-compose.git django-sample

cd django-sample

docker build -t django-sample .
```



```powershell
# django 컨테이너 실행 + postgres 컨테이너 실행
docker run --rm -d --name django \
  -p 8000:8000 \
  django-sample

docker run --rm -d --name postgres \
  -e POSTGRES_DB=djangosample \
  -e POSTGRES_USER=sampleuser \
  -e POSTGRES_PASSWORD=samplesecret \
  postgres
```



![2_1](https://github.com/shoon2430/TIL/blob/master/images/docker/2_1.png)

다음과 같은 예시로 Django컨테이너를 먼저 띄울 경우 DB가 존재하지않아 실행시 에러가 발생한다.



```powershell
# postgres 컨테이너 실행 + django 컨테이너 실행 + 서로 연결하기
docker run --rm -d --name postgres \
  -e POSTGRES_DB=djangosample \
  -e POSTGRES_USER=sampleuser \
  -e POSTGRES_PASSWORD=samplesecret \
  postgres
​
docker run -d --rm \
  -p 8000:8000 \
  -e DJANGO_DB_HOST=db \
  --link postgres:db \
  django-sample
```

![2_2](https://github.com/shoon2430/TIL/blob/master/images/docker/2_2.png)



```powershell
docker run --rm -d --name postgres \
  -e POSTGRES_DB=djangosample \
  -e POSTGRES_USER=sampleuser \
  -e POSTGRES_PASSWORD=samplesecret \
  postgres
  
docker run -d --rm \
  -p 8000:8000 \
  -e DJANGO_DB_HOST=db \
  --link postgres:db \
  django-sample
```



### 3. 특정 컨테이너끼리만 통신할 수 있는 가상 네트워크 환경을 편리하게 관리하고 싶어서

```powershell
# postgres 컨테이너 실행 + django1 컨테이너 연결
docker run --rm -d --name postgres \
  -e POSTGRES_DB=djangosample \
  -e POSTGRES_USER=sampleuser \
  -e POSTGRES_PASSWORD=samplesecret \
  postgres
​
docker run -d --rm --name django1 \
  -p 8000:8000 \
  -e DJANGO_DB_HOST=db \
  --link postgres:db \
  django-sample
```

![3_1](https://github.com/shoon2430/TIL/blob/master/images/docker/3_1.png)



```powershell
# postgres 컨테이너는 호스트의 다른 컨테이너들이 모두 접근할 수 있음
docker run -d --rm --name django2 \
  -p 8001:8000 \
  -e DJANGO_DB_HOST=db \
  --link postgres:db \
  django-sample
```

![3_2](https://github.com/shoon2430/TIL/blob/master/images/docker/3_2.png)



postgres 컨테이너 + django1 컨테이너만 통신할 수 있는 가상 네트워크 만들기

```powershell
# 도커 네트워크 살펴보기
docker network ls
```

```powershell
# 도커 네트워크 생성하기
docker network create --driver bridge web-service

docker network ls
```

![3_3](https://github.com/shoon2430/TIL/blob/master/images/docker/3_3.png)



```powershell
# 컨테이너 실행하기
docker run --rm -d --name postgres \
  --network web-service \
  -e POSTGRES_DB=djangosample \
  -e POSTGRES_USER=sampleuser \
  -e POSTGRES_PASSWORD=samplesecret \
  postgres

docker run -d --rm --name django1 \
  --network web-service \
  -p 8000:8000 \
  -e DJANGO_DB_HOST=db \
  --link postgres:db \
  django-sample

docker run -d --rm --name django2 \
  -p 8001:8000 \
  -e DJANGO_DB_HOST=db \
  --link postgres:db \
  django-sample
```

![3_4](https://github.com/shoon2430/TIL/blob/master/images/docker/3_4.png)

> - docker의 네트워크 모드 종류
>   - bridge: 해당 네트워크 안에서만 통신 가능
>   - host: 호스트와 똑같은 네트워크 환경
>   - none: 아무 네트워크도 사용하지 않음



### 4. 이 모든 것을 간단한 명령어로 관리하고 싶어서

도커만으로 컨테이너를 생성할 경우

```powershell
# 실행 명령어와 종료 명령어
docker network create --driver bridge web-service

docker run --rm -d --name postgres \
  --network web-service \
  -p 5432:5432 \
  -e POSTGRES_DB=djangosample \
  -e POSTGRES_USER=sampleuser \
  -e POSTGRES_PASSWORD=samplesecret \
  postgres

docker run -d --rm --name django1 \
  --network web-service \
  -p 8000:8000 \
  -e DJANGO_DB_HOST=db \
  --link postgres:db \
  django-sample

docker kill django1 postgres

docker network rm web-service
```



도커 컴포즈로 컨테이너를 실행할 경우

```yaml
version: '3'

volumes:
  postgres_data: {}

services:
  db:
    image: postgres
    volumes:
      - postgres_data:/var/lib/postgres/data
    environment:
      - POSTGRES_DB=djangosample
      - POSTGRES_USER=sampleuser
      - POSTGRES_PASSWORD=samplesecret

  django:
    build:
      context: .
      dockerfile: ./compose/django/Dockerfile-dev
    volumes:
      - ./:/app/
    command: ["./manage.py", "runserver", "0:8000"]
    environment:
     - DJANGO_DB_HOST=db
    depends_on:
      - db
    restart: always
    ports:
      - 8000:8000
```

![4_1](https://github.com/shoon2430/TIL/blob/master/images/docker/4_1.png)



