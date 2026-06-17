## [매드업 블로그](https://madup-inc.github.io)

### 환경 설정

설치:

```
$ gem install bundler
$ bundle install
```

개발 서버 구동:

```
$ jekyll serve
```

[개발 서버](http://127.0.0.1:4000)에 접속하면 블로그를 확인할 수 있다.

Docker 사용 시:

호스트에 Ruby를 설치하지 않고 `ruby:2.7` 컨테이너로 실행한다. 이미지 경로가 로컬 기준으로 잡히도록 `127.0.0.1`에 바인딩한다.

```
$ docker run --rm -v "$PWD":/srv -w /srv \
  -v madup-blog-bundle:/usr/local/bundle ruby:2.7 \
  bash -lc "gem install bundler -v 2.3.10 && bundle _2.3.10_ install"

$ docker run --rm --network host -v "$PWD":/srv -w /srv \
  -v madup-blog-bundle:/usr/local/bundle ruby:2.7 \
  bundle _2.3.10_ exec jekyll serve --host 127.0.0.1 --port 4000
```

### 글 업로드 방법

1. `_posts` 폴더에 `YYYY-MM-DD-{글제목}.md` 파일을 생성한다.  
1. 글의 첨부 이미지는 `uploads/{글제목}` 경로에 저장한다.  
1. md 파일에 이미지를 첨부할 때는 아래 코드 사용을 권장한다.  
   `{% include image.html img="{글제목}/{이미지명.확장자}" caption={이미지설명} %}`
1. md 파일에 영상 첨부시에는 아래 코드 사용이 가능하다.
   `{% include video.html src="{글제목}/{이미지명.mp4}" caption="예시" %}`

### 참고

- [Setting up your GitHub Pages site locally with Jekyll](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/)
- [etoile 테마 사용법](https://docs.unbound.studio/etoile-writer-blogger-jekyll-theme/s)
