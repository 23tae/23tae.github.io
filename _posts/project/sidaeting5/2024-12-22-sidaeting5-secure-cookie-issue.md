---
title: "로컬 환경에서 Secure 쿠키 설정이 안되는 문제"
date: 2024-12-22T11:00:00.000Z
categories: [Project, 시대팅5]
tags: [spring-boot, cookie, issue]
---

## 개요

백엔드를 배포한 이후, 프론트엔드가 로컬 환경에서 리프레시 토큰이 담긴 쿠키를 설정하지 못하는 문제가 발생했다. 이 문제는 쿠키의 `Secure`와 `SameSite` 설정과 관련이 있었다. 이를 해결한 과정을 정리한다.

![slack frontend](/assets/img/project/sidaeting5/secure-cookie-issue/slack-frontend-issue.png)
_프론트엔드 팀원이 슬랙에 남긴 이슈_

## 접근 과정

1. **최초 설정**
    
    처음에는 쿠키를 `Secure=true`와 `Domain=.uoslife.net`으로 설정했다. 그러나 프론트엔드가 **localhost**에서 실행되었을 때, 쿠키를 사용할 수 없는 문제가 발생했다.
    
    ![image.png](/assets/img/project/sidaeting5/secure-cookie-issue/dev-tools-1.png)
    _개발자 도구에서 확인한 응답 헤더의 Set-Cookie_
    > This Set-Cookie didn't specify a "SameSite" attribute and was default to "SameSite=Lax," and was blocked because it came from a cross-site response which was not the response to a top-level navigation. The Set-Cookie had to have been set with "SameSite=None" to enable cross-site usage.
    
2. **쿠키 도메인 변경**
    
    이를 해결하기 위해 **쿠키 도메인**을 `localhost`로 변경했으나, 이번에는 서버 도메인(`uosmeeting.uoslife.net`)과 쿠키 도메인이 달라져서 쿠키가 설정되지 않았다.
    
3. **SameSite=None 설정**
    
    그 후 **쿠키 도메인**을 다시 `.uoslife.net`으로 되돌리고, `SameSite=None`을 설정했다. 하지만 이번에는 "`SameSite=None`은 `Secure` 속성과 함께 사용해야 한다"는 에러가 발생했다.
    
    ![image.png](/assets/img/project/sidaeting5/secure-cookie-issue/dev-tools-2.png)
    > This attempt to set a cookie via a Set-Cookie header was blocked because it had the "SameSite=None" attribute but did not have the "Secure" attibute, which is required in order to use "SameSite=None".
    
4. **Secure 설정과 HTTPS**
    
    위 에러는 Chrome에서 `Secure=false` 설정과 `SameSite=None`이 함께 사용될 때 발생하는 제약(2020년부터 적용된 것으로 보인다) 때문이었다. 이를 해결하기 위해 `Secure=true`를 다시 설정했다. 그러나 **localhost**는 **HTTP**를 사용하고 있었기 때문에 쿠키를 설정할 수 없었다.
    

## 해결 방법

### HTTPS 적용

문제를 해결하기 위해 **mkcert**를 사용하여 **로컬 환경에서 사용할 인증서**를 생성하고, 프론트엔드 설정 파일에 HTTPS를 적용했다. 이로써 **로컬 환경에서도 HTTPS**가 적용되어 **Secure 쿠키**가 정상적으로 설정되었다.

- `vite.config.ts`

```typescript
import fs from 'fs';

export default defineConfig({
  plugins: [react()],
  server: {
    https: {
      key: fs.readFileSync('./localhost-key.pem'),
      cert: fs.readFileSync('./localhost.pem'),
    },
  },
});
```

## 참고자료

- [Get Ready for New SameSite=None; Secure Cookie Settings  \|  Google Search Central Blog  \|  Google for Developers](https://developers.google.com/search/blog/2020/01/get-ready-for-new-samesitenone-secure)
- [로컬 개발에 HTTPS 사용  \|  Articles  \|  web.dev](https://web.dev/articles/how-to-use-local-https?hl=ko)
- [로컬 개발에 HTTPS를 사용해야 하는 경우  \|  Articles  \|  web.dev](https://web.dev/articles/when-to-use-local-https?hl=ko)
- [쿠키의 SameSite 옵션 (Feat CORS)](https://gyuturn.tistory.com/74)
- [google chrome - This Set-Cookie didn't specify a "SameSite" attribute and was default to "SameSite=Lax" - Localhost - Stack Overflow](https://stackoverflow.com/questions/67821709/this-set-cookie-didnt-specify-a-samesite-attribute-and-was-default-to-samesi)
- [hhttp → http 크로스 도메인 쿠키 전송 — 방구석 코딩](https://msm1307.tistory.com/159)
- [쿠키를 활용한 인증에서 SameSite 문제 해결하기](https://velog.io/@jhbae0420/쿠키를-활용한-인증에서-SameSite-문제-해결하기)
- [프론트와 백엔드 분리했을 때 쿠키 공유 안되는 이유 (feat. Credentials 옵션)](https://velog.io/@youjung/프론트와-백엔드-분리했을-때-쿠키-공유-안될-때)