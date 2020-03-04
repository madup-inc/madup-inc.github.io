# [**MadTech :: 매드업 기술 문서**](https://madup-inc.github.io)

## 환경 설정

깃헙 페이지를 이용한 기술 블로그. 마크다운으로 작성한 문서를 정적 파일로 변경하기 위해 루비와 지킬을 이용한다.
(참고: [Setting up your GitHub Pages site locally with Jekyll](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/))

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


## Images

- 썸네일은 assets/images/thumbnail 경로에 {글제목}을 파일명으로 저장한다.
- 글에 첨부하는 이미지는 assets/images/{글제목} 경로에 저장한다.


## Disqus

- _config.yml 파일에서 직접 등록한다.  
