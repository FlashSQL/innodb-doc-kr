# InnoDB 8.0 Reference Manual

[MySQL 공식 레퍼런스 메뉴얼](https://dev.mysql.com/doc/refman/8.0/en/)에서 제공하는
내용 중, InnoDB 파트만을 분리하여 한글 번역본으로 제공합니다. InnoDB와 밀접히
연결된 경우에는 MySQL 파트에 대한 자료도 업로드 될 수 있습니다. 

MySQL/InnoDB와 관련된 추가적인 문서, 연구 자료나 논문 등도 업로드 될 수 있습니다. 

## Setup local server with Hexo

> 로컬 서버에서 테스트하기 위해서는 `npm`이 필요합니다. 

본 리포지토리를 clone 한 뒤 `yarn` 혹은 `npm i`를 실행하여 필요한 패키지들을 설치합니다.
`hexo`가 설치되어 있지 않은 경우 `npm i hexo-cli -g`를 통해 hexo 커맨드라인 툴을 설치합니다. 
모든 설치 작업이 정상적으로 완료된 경우 `hexo server`를 통해 로컬 서버를 실행하고,
웹 브라우저에서 `http://localhost:4000`을 통해 접속합니다. 

### Github page로 deploy 하기

작업이 완료된 이후 `hexo deploy -g` 를 통해 github로 deploy 한다.