GitHub Pages는 GitHub를 통해 웹페이지롤 호스팅할 수 있는 서비스입니다.

github의 pages 기능을 이용하면 repository에 html 파일을 업로드하고, 이를 public으로 공개하여 특정 url로 브라우저에서 접속가능하게 해 줍니다.

일반적으로 이 기능은 GitHub의 사용자나 사용자의 프로젝트를 설명하기 위한 페이지를 만들기 위해 사용됩니다. 그도 그럴것이 GitHub Pages는 데이터베이스나 서버사이드 스크립트를 사용할 수 없어서 정적인 페이지만 로딩이 가능하기 때문입니다. 

하지만, 정적 사이트 생성기인 [http://jekyllrb-ko.github.io/ jekyll]을 이용하여 DB나 php 없이도 블로그 같은 서비스를 github에서 돌릴 수도 있습니다. 이 부분은 따로 시간을 내어 설명하도록 하겠습니다.

## 페이지 만들기

GitHub Pages는 User/Organization 페이지와 Project 페이지의 두가지 타입을 가집니다. 두 개의 페이지는 만드는 방법과, 웹에서 접근하는 방법에서 약간의 차이를 가집니다.

### User & Organization 페이지 만들기

User(Organization) 페이지의 경우 페이지를 위한 저장소를 따로 만들어야 합니다. 이때 저장소의 이름을 <code>username.github.io</code>형태로 만들어야 합니다. '''username'''은 본인의 사용자 ID를 넣습니다. 따라서 id가 sogeuni라면 github에서 보이는 repository의 이름은 <code>sogeuni/sogeuni.github.io</code>가 됩니다.

그런다음 <code>master</code> 브랜치에 컨텐츠를 올리면 자동으로 웹에 게시됩니다.

웹에서는 <code>http(s)://<username>.github.io</code>로 repository의 루트에 접근가능합니다. 루트에 index.html 파일을 작성하면 웹에서 확인이 가능할 것입니다. 웹페이지 만들기가 어렵다면 [https://help.github.com/articles/creating-pages-with-the-automatic-generator Automatic Page Generator]와 같은 온라인 도구를 이용할 수도 있습니다.

### Project 페이지 만들기

프로젝트 페이지는 특정프로젝트를 설명하고자 하는 페이지므로 User(Organization) 페이지와는 다르게 기존의 프로젝트와 같은 저장소를 사용합니다. 프로젝트 페이지의 경우 저장소의 루트에 접근하는 URL은 <code>http(s)://<user(org)name>.github.io/<projectname></code>이 됩니다.

프로젝트페이지는 User 페이지와 달리 <code>gh-pages</code> 브랜치에 컨텐으를 업로드해야 합니다. 따라서, '''master'''나 '''develop''' 브랜치에 실제 프로젝트 코드가 존재한다면, 프로젝트의 설명을 위한 index.html 페이지는 '''gh-pages''' 브랜치의 루트에 넣어야만 웹에서 볼 수 있습니다.

## GitHub Pages의 제약사항

* 한달에 100GB 이상의 트래픽이 발생하거나 100,000건 이상의 request가 발생하면 GitHub로 부터 메일이 옵니다. 따라서 트레픽을 고려하여 다른 hosting으로 이전하는 것이 좋습니다.
* 저장소의 제한이 1GB이여 초과시 역시 GitHub로부터 메일이 옵니다.
* GitHub Page는 [https://help.github.com/articles/github-terms-of-service/ GitHub 약관]이 적용됩니다.

## 함께보기

* [[맥에 jekyll 설치하기]]

## 참고자료

* https://pages.github.com/
* https://help.github.com/categories/github-pages-basics/
