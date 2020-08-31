---
layout: post
title: "Docker 볼륨에 mongodb dump와 restore하기"
subtitle: "Docker Volume의 mongodb를 관리해보자"
author: "yoogomja"
header-style: text
tags:
    - docker
    - gcp
    - mongodb
---

# 1. docker 에서 dump하기

## 1.1. 볼륨 확인하기

`docker volume ls`

위 명령어로 저장된 볼륨의 목록을 확인할 수 있다. 그 후에는 해당 볼륨에 shell에 접근해야한다.

## 1.2. 볼륨 접근하기

`docker exec -ti 볼륨_이름 sh`

해당 볼륨의 shell로 접근한다. 여기서 `mongo` 키워드를 입력해서 `mongodb`가 설치되어있는 볼륨으로 제대로 접근했는지 한번 더 확인해본다.

## 1.3. 덤프 수행하기

`mongodump --out /backup/`

위 명령어를 이용하면, `backup`이라는 폴더에 데이터가 덤프된다. 크기에 따라 다소 시간이 소요될 수 있다.

> 만약 `mongodb alpine` 버전 등을 사용하고 있어 `mongodump`가 실행되지 않는 다면, `apk add --no-cache mongodb-tools`를 입력해 `mongodump`와 `mongorestore`를 한번에 설치하자

## 1.4. 덤프를 볼륨 외부로 복사하기

`sudo docker cp 볼륨_이름:/backup/ 원하는/위치/`

`docker`명령어로 이제 원하는 위치로 복사해오자. 해당 위치로 복사해오고 나중에 옮길 볼륨으로 복사해주어야 한다.

## 1.4.1. gcp에서 수행하는 경우

`gcp`에서 다운받아야하는경우 `gcloud`를 이용해서 직접 복사해올 수도 있지만, 온라인 콘솔에서 가져올 때에는 파일 다운로드 기능을 통해서 가져와야한다. 이때에는 직접 다운로드 받을 파일의 경로를 입력해줘야한다. 다만 이때에 폴더를 직접 가져오는 것은 안되고, 압축해 가져와야한다.

`tar -cvf 압축_파일명.tar 대상/폴더명/`

`tar`로 먼저 압축해준 후, 해당 압축 파일의 경로를 찾아서 압축 파일 경로를 알려주면 다운로드 받을 수 있다. 현재 위치는 리눅스에서는 `pwd`로 확인할 수 있다.

# 2. 다른 docker 볼륨으로 restore 하기

## 2.1. 덤프 내용을 복사하기

`docker cp 덤프/위치 대상_볼륨:/대상_위치`

위 명령어로 덤프 내용을 대상 볼륨으로 복사해준다. 복사 후에는 1에서 진행한 내용과 동일하다

## 2.2. 볼륨으로 접근하기

`docker exec -ti 볼륨_이름 sh`

해당 볼륨으로 접근한 뒤에 restore를 진행해야한다.

## 2.3. 복구하기

`mongorestore -h localhost 덤프/위치`

위 명령어를 볼륨의 쉘에서 실행해주면 덤프의 모든 내용을 복구한다.

> 만약 `mongorestore`가 없다면, 위와 동일하게 `apk add --no-cache mongodb-tools`를 입력해 `mongodump`와 `mongorestore`를 한번에 설치하자

# 참고

-   [How to Backup Docker Containered Mongo DB with Mongodump and Mongorestore](https://medium.com/faun/how-to-backup-docker-containered-mongo-db-with-mongodump-and-mongorestore-b4eb1c0e7308)

-   [github restore issue](https://github.com/mvertes/docker-alpine-mongo/pull/30/files)
