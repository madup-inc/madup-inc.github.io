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

### 글 업로드 방법

1. `_posts` 폴더에 `YYYY-MM-DD-{글제목}.md` 파일을 생성한다.  
1. 글의 첨부 이미지는 `uploads/{글제목}` 경로에 저장한다.  
1. md 파일에 이미지를 첨부할 때는 아래 코드 사용을 권장한다.  
   `{% include image.html img="{글제목}/{이미지명.확장자}" caption={이미지설명} %}`

### 참고

- [Setting up your GitHub Pages site locally with Jekyll](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/)
- [etoile 테마 사용법](https://docs.unbound.studio/etoile-writer-blogger-jekyll-theme/s)
