## 🦥 `devbelly Devlog`

📎 **블로그 바로 가기**

[`https://devbelly.github.io/`](https://devbelly.github.io/)

---

### fork 주의사항 ★

가끔 fork해서 사용하시는 분들이 있는데 편하게 사용하셔도 됩니다!  
나름 열심히 만들었으니.. fork 하실 때 `star` 하나만 눌러주시면 뿌듯할 것 같네요 :)

**아래 사항들을 꼭 변경해주셔야 저의 analytics에 반영이 되지 않습니다!!!**  
(제 블로그 구글 analytics에 다른 분들 게시물이 가끔 보이는데... 서로 민망하잖아요ㅎㅎ)

1. \_config.yml 변경

```yml
google_site_verification: "직접 추가하시거나 삭제해주세요"
bing_site_verification:
yandex_site_verification:
naver_site_verification: "직접 추가하시거나 삭제해주세요"
```

```yml
# Analytics
analytics:
  provider: "google-gtag"
  # false (default), "google", "google-universal", "google-gtag", "custom"
  google:
    tracking_id: "직접 추가하시거나 삭제해주세요"
    anonymize_ip: # true, false (default)
```

2. 아래 4개 파일 삭제

- google9f6~~~.html
- feed.xml
- naverd967c~~~.html
- robots.txt

**Comment 설정**

1. utterances 셋팅

- [utterances](https://github.com/apps/utterances) 접속
- repository 선택 후 install
- 이 Jekyll Theme에서는 셋팅 방법이 다름.
- Enable Utterances script를 기억

2. \_config.yml 수정

```yml
# 1번의 script에서 issue_term과 theme을 아래와 같이 작성
# 1번의 script와 _config.yml이 다를 경우만 수정
comments:
  provider: "utterances"
  utterances:
    theme: "github-light" # "github-dark"
    issue_term: "pathname" # pathname은 post.md 파일 이름으로 연결됨
```

3. 본인 github에 public repo 생성 (repo명 : blog-comments)

4. \_includes/comments-providers/utterances.html 수정

```html
var script = document.createElement('script'); script.setAttribute('src',
'https://utteranc.es/client.js'); script.setAttribute('repo',
'본인아이디/blog-comments'); # 3에서 만들었던 레포지토리로 수정
```

5. comment 기능은 issue를 tracking하는 것이기 때문에 issue 관련 permission이 있다면 허용

---

### 개발 기록

[VER1.0]
![devbelly github blog main](/assets/images/posts_img/readme/blog-main-ver1.png)

[VER2.0]
![devbelly github blog main](/assets/images/posts_img/readme/blog-main-ver2.png)

- logo 변경
- 카테고리 디자인 변경
- font family, size 변경
- 메인 컬러 변경

[VER2.1]
![devbelly github blog main](/assets/images/posts_img/readme/ver2_1_main.png)

- 카테고리 정리
- favicon 변경

<br>

> 🌴 **목차**

┌ `Algorithm`  
├ `C++`  
├ `Python`  
├ `Git`  
├ `GitHub Blog`  
├ `Maching Learning`  
└ `Web`
