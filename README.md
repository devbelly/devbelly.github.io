## ๐ฆฅ `devbelly Devlog`

๐ **๋ธ๋ก๊ทธ ๋ฐ๋ก ๊ฐ๊ธฐ**

[`https://devbelly.github.io/`](https://devbelly.github.io/)

---

### fork ์ฃผ์์ฌํญ โ

๊ฐ๋ forkํด์ ์ฌ์ฉํ์๋ ๋ถ๋ค์ด ์๋๋ฐ ํธํ๊ฒ ์ฌ์ฉํ์๋ ๋ฉ๋๋ค!  
๋๋ฆ ์ด์ฌํ ๋ง๋ค์์ผ๋.. fork ํ์ค ๋ `star` ํ๋๋ง ๋๋ฌ์ฃผ์๋ฉด ๋ฟ๋ฏํ  ๊ฒ ๊ฐ๋ค์ :)

**์๋ ์ฌํญ๋ค์ ๊ผญ ๋ณ๊ฒฝํด์ฃผ์์ผ ์ ์ analytics์ ๋ฐ์์ด ๋์ง ์์ต๋๋ค!!!**  
(์  ๋ธ๋ก๊ทธ ๊ตฌ๊ธ analytics์ ๋ค๋ฅธ ๋ถ๋ค ๊ฒ์๋ฌผ์ด ๊ฐ๋ ๋ณด์ด๋๋ฐ... ์๋ก ๋ฏผ๋งํ์์์ใใ)

1. \_config.yml ๋ณ๊ฒฝ

```yml
google_site_verification: "์ง์  ์ถ๊ฐํ์๊ฑฐ๋ ์ญ์ ํด์ฃผ์ธ์"
bing_site_verification:
yandex_site_verification:
naver_site_verification: "์ง์  ์ถ๊ฐํ์๊ฑฐ๋ ์ญ์ ํด์ฃผ์ธ์"
```

```yml
# Analytics
analytics:
  provider: "google-gtag"
  # false (default), "google", "google-universal", "google-gtag", "custom"
  google:
    tracking_id: "์ง์  ์ถ๊ฐํ์๊ฑฐ๋ ์ญ์ ํด์ฃผ์ธ์"
    anonymize_ip: # true, false (default)
```

2. ์๋ 4๊ฐ ํ์ผ ์ญ์ 

- google9f6~~~.html
- feed.xml
- naverd967c~~~.html
- robots.txt

**Comment ์ค์ **

1. utterances ์ํ

- [utterances](https://github.com/apps/utterances) ์ ์
- repository ์ ํ ํ install
- ์ด Jekyll Theme์์๋ ์ํ ๋ฐฉ๋ฒ์ด ๋ค๋ฆ.
- Enable Utterances script๋ฅผ ๊ธฐ์ต

2. \_config.yml ์์ 

```yml
# 1๋ฒ์ script์์ issue_term๊ณผ theme์ ์๋์ ๊ฐ์ด ์์ฑ
# 1๋ฒ์ script์ _config.yml์ด ๋ค๋ฅผ ๊ฒฝ์ฐ๋ง ์์ 
comments:
  provider: "utterances"
  utterances:
    theme: "github-light" # "github-dark"
    issue_term: "pathname" # pathname์ post.md ํ์ผ ์ด๋ฆ์ผ๋ก ์ฐ๊ฒฐ๋จ
```

3. ๋ณธ์ธ github์ public repo ์์ฑ (repo๋ช : blog-comments)

4. \_includes/comments-providers/utterances.html ์์ 

```html
var script = document.createElement('script'); script.setAttribute('src',
'https://utteranc.es/client.js'); script.setAttribute('repo',
'๋ณธ์ธ์์ด๋/blog-comments'); # 3์์ ๋ง๋ค์๋ ๋ ํฌ์งํ ๋ฆฌ๋ก ์์ 
```

5. comment ๊ธฐ๋ฅ์ issue๋ฅผ trackingํ๋ ๊ฒ์ด๊ธฐ ๋๋ฌธ์ issue ๊ด๋ จ permission์ด ์๋ค๋ฉด ํ์ฉ

---

### ๊ฐ๋ฐ ๊ธฐ๋ก

[VER1.0]
![devbelly github blog main](/assets/images/posts_img/readme/blog-main-ver1.png)

[VER2.0]
![devbelly github blog main](/assets/images/posts_img/readme/blog-main-ver2.png)

- logo ๋ณ๊ฒฝ
- ์นดํ๊ณ ๋ฆฌ ๋์์ธ ๋ณ๊ฒฝ
- font family, size ๋ณ๊ฒฝ
- ๋ฉ์ธ ์ปฌ๋ฌ ๋ณ๊ฒฝ

[VER2.1]
![devbelly github blog main](/assets/images/posts_img/readme/ver2_1_main.png)

- ์นดํ๊ณ ๋ฆฌ ์ ๋ฆฌ
- favicon ๋ณ๊ฒฝ

<br>

> ๐ด **๋ชฉ์ฐจ**

โ `Algorithm`  
โ `C++`  
โ `Python`  
โ `Git`  
โ `GitHub Blog`  
โ `Maching Learning`  
โ `Web`
