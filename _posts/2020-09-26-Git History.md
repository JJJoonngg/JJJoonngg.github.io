---
title : Git History 에서 특정 파일 삭제하기
category : Git
tags : [git, github, git-tip]
---

Git 을 사용하다가 보면 private 일 경우에도 remote 에 올리지 말아야할 file 혹은 folder 가 존재 할 수 있다.

> 보안파일, 패스워드, 비밀번호

`.gitignore` 에 까먹고 올리지 못했다거나 소스트리의 무시 기능을 까먹고 그냥 생각 없이 project create 때 다 추가했다거나....? (내가 그랬다.)



구글링 결과 그냥 새로운 repository 를 파라는 경우도 있지만, 그동안의 commit 을 기록으로 남겨두고

그걸 포트폴리오용으로 쓰거나 history 추척이 필요한 경우에 바람직한 방법이 아니기 때문에 

좋은 명령어를 소개 하고자 한다.



```
git filter-branch -f --index-filter 'git rm --cached --ignore-unmatch 삭제하고자하는 file명' --prune-empty -- --all
```

를 입력하면 된다!



<img width = "400" src="https://user-images.githubusercontent.com/52276038/94331226-b7989a00-0005-11eb-81bf-0642ab39e18a.png">



그러고 나면 해당 화면 과 같이 Rewirte 와 커밋 들이 뜨면서 작업이 완료 된다.

로컬에서 해당 작업을 모두 마치고 나면

```
git push --force --all
```

해당 명령어 를 통해서 강제 push 를 하게되면 remote 역시 해당 file 의 history 가 삭제된다.



**가장 좋은 방법은 .gitignore 와 기타 기능들을 적극 활용하여 이런 일이 발생함을 줄이는 것이아닐까..**

