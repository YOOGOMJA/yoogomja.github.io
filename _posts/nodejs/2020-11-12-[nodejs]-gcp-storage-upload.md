---
layout: post
title: "Express에서 Jimp로 이미지 리사이징과 GCP Storage에 업로드하기"
subtitle: "GCP에 이미지를 업로드, Jimp를 이용해 리사이징도 해보자"
author: "yoogomja"
header-style: text
tags:
    - nodejs
    - express
    - gcp
    - jimp
    - typescript
---

임시로 만드는 프로젝트에서 `express` + `typescript`를 이용해 이미지 처리를 해야할 상황이 생겼다. 이는 유저 프로필 이미지를 위해서인데, 몇 가지 고려할 사항들이 있었다.

1. api 서버와 이미지를 같은 위치에 두지 않아야 한다
2. 이미지 업로드 시 이미지의 사이즈를 줄일 필요가 있다
3. 원형으로 사용되므로 이미지를 크롭해야한다

1번의 이유는 api 서버에 이미지를 둘 경우 관리가 어려워질뿐더러, 데이터베이스와 파일을 연관짓기가 까다로울 듯 해서였다. 2번은 너무 큰 파일이 아니더라도 어짜피 크게 보일 필요가 없는 이미지들이므로, 썸네일을 만들듯 어느정도는 이미지를 리사이즈 해도 되겠다는 판단에서였다. 3번은 정사각 이미지 이외에 업로드 되었을 때 자동으로 중앙 크롭되는 것이 처리가 편할 것 같아서라는 이유였다.

먼저 1번을 해결하기 위해서 `gcp`의 업로드 방법을 알아둘 필요가 있었다.

## 1. GCP Storage 업로드하기

스토리지에 업로드하기 위해서는 몇 가지 준비가 필요했다.

-   버킷 생성하기
-   버킷을 외부에서 접근할 수 있도록 설정하기
-   버킷에 접근할 수 있는 서비스 계정 만들기
-   서비스 계정의 key 파일 생성하기

위 과정 중, **외부에서 접근할 수 있도록 설정하기**부분은 버킷의 사용자에 `allUser`에게 보기 권한을 추가해주면 된다. 그 외에 서비스 계정을 만들어주고 그 계정의 key파일을 json으로 만들어 저장해두면 준비는 완료된다.

### 1.1. 세팅하기

만든 key파일은 서버 코드가 있는 곳에 위치시킨다. 나는 여기서 루트 폴더 아래에 `secure`라는 폴더를 만들고 거기에 키 파일을 위치 시켰다.

키 파일을 위치시키고 난 뒤에는 `node`에게 해당 파일이 어디있는지 알려주어야한다. 환경 변수로 해당 위치를 알려주면 된다.

이 때, 구글은 [문서](https://cloud.google.com/appengine/docs/flexible/nodejs/using-cloud-storage?hl=ko)에서 `GOOGLE_APPLICATION_CREDENTIALS`이라는 이름으로 환경변수에 파일 위치를 알려주면 된다고 했는데, 여기에 **상대 주소**가 적용되지 않는 듯 하다.

그래서 나는 앱이 실행되었을때 직접 환경변수를 세팅해주기로 했는데, 코드는 아래와 같다.

```typescript
// 파일 위치 설정
process.env.GOOGLE_APPLICATION_CREDENTIALS = `${process.cwd()}/secure/YOUR-FILE-NAME.json`;

// 설정을 확인
console.log(
    "google authentication installed at",
    process.env["GOOGLE_APPLICATION_CREDENTIALS"]
);
```

되도록 서버가 실행되는 초기 부분에 해당 부분을 작성해둔다. 그러면, 현재 프로세스의 루트 폴더 주소와 `secure` 경로등을 포함해서 환경변수로 등록해주고 자동으로 추후에 재사용된다.

그 후, `.env`등을 이용해 사용하는 버킷의 이름도 알려주어야 한다.

```shell
# 클라우드 스토리지 버킷 이름
GCLOUD_STORAGE_BUCKET=YOUR-BUCKET-NAME
```

위와 같이 입력해주면, `gcloud storage`패키지가 자동으로 버킷이름과 인증 관련 정보를 수집해 연결하게 된다.

### 1.2. 업로드하기

업로드를 하기 위해서는 먼저 파일을 불러올 수 있어야 한다. 문서에서 이야기한대로 여기서는 `Multer`를 이용해 `multipart/form-data`를 불러오도록 한다.

설치는 `yarn add multer --save`를 입력해 할 수 있다.

그 후, 라우터에 다음과 같은 설정이 필요하다.

```typescript
import * as Multer from "multer";
import { Router, Request, Response } from "express";
const router = Router();

const multer = Multer({
    storage: Multer.memoryStorage(),
    limits: {
        fileSize: 2 * 1024 * 1024, // no larger than 2mb, you can change as needed.
    },
});

router.post(
    "/upload",
    // 파일을 하나만 가져오고, 키 값은 profileImage로
    multer.single("profileImage"),
    (req: Request, res: Response) => {
        if (!req.file) {
            res.status(400).json({
                message: "업로드할 이미지가 없습니다",
            });
            return;
        }
        res.json("upload?");
    }
);
```

위 처럼, `multer`는 미들웨어로 사용된다. 사용할 경우, `req`에 `file`이라는 이름으로 재사용 할 수 있게된다. 이 `req.file`을 업로드하는데 사용하게 된다.

그리고 `@google-cloud/storage`라는 패키지를 미리 설치해 두어야 한다.

실제 코드는 다음과 같다.

```typescript
import { Request, Response, Router } from "express";
import * as Multer from "multer";
import { Storage } from "@google-cloud/storage";
import { format } from "util";
import { Buffer } from "buffer";

// 스토리지 초기화
const storage = new Storage();
// 버킷 초기화
const bucket = storage.bucket(process.env.GCLOUD_STORAGE_BUCKET);
// 라우터 초기화
const router = Router();

// multer 초기화
const multer = Multer({
    storage: Multer.memoryStorage(),
    limits: {
        fileSize: 2 * 1024 * 1024, // no larger than 2mb, you can change as needed.
    },
});

router.post(
    "/upload",
    // 파일을 하나만 가져오고, 키 값은 profileImage로
    multer.single("profileImage"),
    (req: Request, res: Response) => {
        if (!req.file) {
            res.status(400).json({
                message: "업로드할 이미지가 없습니다",
            });
            return;
        }

        // 이미지 업로드 준비
        // 로그인 한 사용자의 이름을 따도록 함
        const blob = bucket.file(
            `${req.user.username}/${Date.now()}-${req.file.originalname}`
        );
        // 이미지 업로드 스트림 생성
        const blobStream = blob.createWriteStream();

        // 에러 핸들링
        blobStream.on("error", (err) => {
            console.log("?");
            res.status(500).json({
                message: "업로드 중 오류가 발생했습니다",
                error: JSON.parse(JSON.stringify(err)),
            });
        });

        // 종료 처리
        blobStream.on("finish", () => {
            const publicUrl = format(
                `https://storage.googleapis.com/${bucket.name}/${blob.name}`
            );
            // 최종적으로 업로드 프로세스가 완료되는 시점
            res.status(200).json({
                message: "업로드 성공",
                url: publicUrl,
            });
        });

        // 업로드 스트림 실행
        blobStream.end(req.file.buffer);
    }
);
```

만약 이를 미들웨어로 분리한다면, `req`객체에 `publicUrl`을 담아서 `next()`해주면 다음 함수에서 접근해 사용할 수 있다.

## 2. 리사이즈 하기

위 작업을 할 때, 미리 리사이즈 하고 크롭을 해주어야 했다. 여기서는 몇 가지 패키지가 제시되었는데, 가장 최근 커밋이 있는 `jimp`를 사용하기로 했다. 설치는 `yarn add jimp --save`해주면 된다.

사용법은 아래와 같다.

```typescript
import { Request, Response, NextFunction } from "express";
import * as Multer from "multer";
import * as jimp from "jimp";

const router = Router();
const CROPPED_IMG_SIZE = 200; // 200 * 200으로 자르기로 함
const multer = Multer({
    storage: Multer.memoryStorage(),
    limits: {
        fileSize: 2 * 1024 * 1024, // no larger than 2mb, you can change as needed.
    },
});

router.post(
    "/crop", // 파일을 하나만 가져오고, 키 값은 profileImage로
    multer.single("profileImage"),
    (req: Request, res: Response) => {
        if (!req.file) {
            res.status(400).json({
                message: "크롭할 이미지가 없습니다",
            });
            return;
        }

        // 이미지 리사이즈
        // 파일 읽어오기
        const jimpImg = await jimp.read(Buffer.from(req.file.buffer));

        // 가로가 더 긴 경우
        if (jimpImg.bitmap.width > jimpImg.bitmap.height) {
            // 리사이즈 하기
            jimpImg.resize(jimp.AUTO, CROPPED_IMG_SIZE);
            // 정 가운데를 기준으로 크롭하기
            jimpImg.crop(
                // x 좌표
                jimpImg.bitmap.width / 2 - CROPPED_IMG_SIZE / 2,
                // y 좌표
                0,
                // 너비
                CROPPED_IMG_SIZE,
                // 높이
                CROPPED_IMG_SIZE
            );
        }
        // 세로가 더 긴 경우
        else {
            jimpImg.resize(CROPPED_IMG_SIZE, jimp.AUTO);
            jimpImg.crop(
                // x 좌표
                0,
                // y 좌표
                jimpImg.bitmap.height / 2 - CROPPED_IMG_SIZE / 2,
                // 너비
                CROPPED_IMG_SIZE,
                // 높이
                CROPPED_IMG_SIZE
            );
        }
        const resized = await jimpImg.getBufferAsync(req.file.mimetype);

        res.setHeader("Content-Type", "image/jpeg");
        res.setHeader("Content-Length", resized.byteLength);
        res.write(resized);
    }
);
```

위와 같이 작성하면, 리사이즈 된 이미지를 응답 받을 수 있다.

## 3. 완성하기

이 두 스텝을 합쳐서 미들웨어로 만들었는데 그 내용은 다음과 같다.

```typescript
import { Request, Response, NextFunction } from "express";
import { Storage } from "@google-cloud/storage";
import * as jimp from "jimp";
import { format } from "util";
import { Buffer } from "buffer";

const storage = new Storage();
const bucket = storage.bucket(process.env.GCLOUD_STORAGE_BUCKET);
const THUMNAIL_IMG_SIZE = 200;

export const uploadImage = async (
    req: Request,
    res: Response,
    next: NextFunction
) => {
    if (!req.file) {
        res.status(400).json({
            message: "업로드된 파일이 없습니다",
        });
        return;
    }

    // 이미지 리사이즈
    const jimpImg = await jimp.read(Buffer.from(req.file.buffer));
    if (jimpImg.bitmap.width > jimpImg.bitmap.height) {
        jimpImg.resize(jimp.AUTO, THUMNAIL_IMG_SIZE);
        jimpImg.crop(
            jimpImg.bitmap.width / 2 - THUMNAIL_IMG_SIZE / 2,
            0,
            THUMNAIL_IMG_SIZE,
            THUMNAIL_IMG_SIZE
        );
    } else {
        jimpImg.resize(THUMNAIL_IMG_SIZE, jimp.AUTO);
        jimpImg.crop(
            0,
            jimpImg.bitmap.height / 2 - THUMNAIL_IMG_SIZE / 2,
            THUMNAIL_IMG_SIZE,
            THUMNAIL_IMG_SIZE
        );
    }
    const resized = await jimpImg.getBufferAsync(req.file.mimetype);

    // 이미지 업로드 준비
    const blob = bucket.file(
        `${req.user.username}/${Date.now()}-${req.file.originalname}`
    );
    // 이미지 업로드 스트림 생성
    const blobStream = blob.createWriteStream();

    // 에러 핸들링
    blobStream.on("error", (err) => {
        console.log("?");
        res.status(500).json({
            message: "업로드 중 오류가 발생했습니다",
            error: JSON.parse(JSON.stringify(err)),
        });
    });

    // 종료 처리
    blobStream.on("finish", () => {
        const publicUrl = format(
            `https://storage.googleapis.com/${bucket.name}/${blob.name}`
        );
        req.gcpImgUrl = publicUrl;
        next();
    });

    // 업로드 스트림 실행
    blobStream.end(resized);
};
```

이렇게 작성해두고, 실제 사용하는 부분에서는 다음과 같이 사용했다.

```typescript
import { UserController } from "../controllers";
import * as Middlewares from "../middlewares";
import * as Multer from "multer";

const multer = Multer({
    storage: Multer.memoryStorage(),
    limits: {
        fileSize: 2 * 1024 * 1024, // no larger than 5mb, you can change as needed.
    },
});

router.post(
    "/:username/profile",
    // 기존 미들웨어의 인증 확인
    Middlewares.Auth.isAuthenticated,
    // 이미지 처리
    multer.single("profileImage"),
    // 리사이징 및 업로드 처리
    Middlewares.GCP.uploadImage,
    // 실제 컨트롤러 코드
    UserController.uploadProfileImage
);
```

위와 같이 사용할 경우, 파일 업로드 부분과 해당 주소를 다루는 부분을 분리 할 수 있어서 오류 제어에 도움이 되었다.
