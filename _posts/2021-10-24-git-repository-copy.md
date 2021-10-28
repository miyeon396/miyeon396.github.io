---
title : [Git] Repository Copy
---
### Git Repository 통으로 복사 (내 로컬에 있는게 통 복사가 됨)

1. git clone --mirror {{repo a.git}} (기존 repo = 복사할 repo)
2. cd {{repo a.git}} (project 의 .git 위치)
3. git remote set-url --push origin {{repo b.git}} (신규 repo = 새로 만들어져 올릴 주소)
4. git push --mirror
5. 일케하믄 remote가 새로 만든 remote로 바뀌니 기존거 유지하고 싶다면 source tree 내 설정 다시 확인 후 바꿔주는 작업이 필요하다.
