# Fireflow

* 플러터플로를 통해 앱을 제작하는 경우, 백엔드로서 역할을 한다.
* 플러터플로의 치명적인 단점은 debug run 과 instant reload 가 매우 느리다는 것이다. 그래서 개발이 원활하지 않다. 가능한 많은 작업을 백엔드에서 하여 프론트엔드의 개발 부담을 줄인다.


# TODO

* Firestore 보안 규칙 확인

* 코멘트를 작성 할 때, post document 또는 parent document 둘 중 하나만 입력해서 저장하면,
  * 필요한 모든 처리를 백엔드에서 한다.
  * 예) post 에 noOfComments 업데이트, comment 에 depth, list-order 업데이트, 사용자 프로필에 noOfComments 업데이트 등


# 개요

* 플러터플로에서 하기 어려운 기능(또는 쉽지 않은 기능)인데 백엔드에서는 간단히 할 수 있는 것이 있다. 또 그 반대로 플러터플로에서 쉽게 할 수 있는 기능인데, 백엔드에서는 어려운 기능이 있다. 이러한 기능의 조율을 적절히 하여 


# Firestore DB 구조


- `/settings` 에 앱/시스템 자체의 설정이다. 참고, 관리자 지정, 헬퍼 지정
- `/user_settings` 에 각 사용자별 설정을 저장한다.

# 앱 설정

- `/settings/system` 에는 시스템 설정을 지정한다. 주로 앱 버전, 헬퍼 사용자 지정 등을 할 수 있다.
- `/settings/counters` 에는 각 종 카운터가 기록된다.
  - `noOfUsers` - 총 회원 수
  - `noOfPosts` - 총 글 수 (todo)
  - `noOfComments` - 총 코멘트 수 (todo)

## 관리자 지정

- `/settings/admins` 에 `{ uid: boolean }` 형태로 관리자를 지정한다.
  - UID 를 키로 여러명의 사용자를 지정 할 수 있다.
  - 예) `/settings/admins { 9Sefounafeou384: true }` 이면, 해당 사용자는 관리자가 된다.



## 헬퍼 사용자 지정

- 헬퍼 사용자는 사용자와 상담을 하는 담당자(직원) 계정을 말한다. 예를 들면, 회원 가입을 하면, 자동으로 가입 환영 인사 메시지를 하나 보내는데, 이 때, 헬퍼 사용자 계정이 메시지를 보내는 것이다.

- 헬퍼 사용자 지정하는 방법
    - `/settings/system/ {helperUid: UID }` 를 지정하면, 그 사용자는 헬퍼 사용자가 된다.


# 사용자

- `/users` 컬렉션은 기본 사용자 문서 컬렉션인데 변경이 가능하다. 사용자 문서 컬렉션은 (변경을 해도) 이메일과 전화번호가 저장되며, 플러터플로의 기본 채팅 기능을 사용하기 위해서는 반드시 외부에 (모든 사람이 볼 수 있도록) 공개되어야 한다. 즉, 개인 정보가 노출된다.
  - 그래서, Fireflow 에서는 `/users` 를 본인만 읽을 수 있도록 하고, 대신 공개 정보는 `/users_public_data` 에 저장해서 사용한다.

- 사용자가 가입하거나 회원 정보를 수정하면, `/users/<uid>` 문서가 `/users_public_data` 로 sync 된다.
  - 이 때, 개인 정보인 `email` 과 `phone_number` 를 빼고 sync 하며, 추가적으로 `blockedUserList` 도 sync 되지 않는다.
  - 그리고 다음과 같은 추가 정보가 기록된다.
    - `isProfileComplete: boolean` - 회원 정보 중에서 `display_name`, `email`, `photo_url` 이 있으면 true 아니면 false 로 저장된다. 회원 정보 목록에서 프로필을 모두 적용한 회원 정보 목록을 할 때 사용 할 수 있다.
    - `hasPhoto` - 회원 정보에서 `photo_url` 이 있으면 true 아니면 false. 회원 정보 목록에서 사진이 있는 회원 정보 목록을 할 때 사용 할 수 있다.
    - `userDocumentReferenc` - 회원 ref
    - `updatedAt` - 회원 정보 수정한 시간

- `/users_public_data` 는 영구적으로 사용 가능하다. 공개할 정보가 있으면 이곳에 기록을 하면 되는데, 아래와 같은 추가 필드가 사용된다.
  - `likes` - 나를 좋아한 사용자 ref 배열



# 채팅


- TODO: 현재는 1:1 채팅방만 지원한다. 채팅방 아이디가 `UID-UID` 로 정해지는데, 그룹 채팅인 경우는 채팅방 아이디를 그냥 자동생성한 문서 아이디로 하고, `/chat_rooms` 의 `users` 필드에 두 명이 아닌 여러명의 사용자를 기록해서 그룹 채팅을 할 수 있도록 한다. 이 때, `new_messages` 맵에 `{uid: [count]}` 와 같이 해서, 각 사용자당 읽지 않은 메시지 수를 기록 하도록 한다. 즉, 그룹 채팅에 메시지를 보내기 위해서는 채팅방 아이디를 알아야한다. 참고로 1:1 채팅에서는 채팅방 아이디가 `UID-UID`이므로, 채팅방이 아닌 외부에서도 메시지를 보낼 수 있다. 가능한 동일한 구조를 유지하며, 1:1 채팅방과 호환이 되도록 작성한다.

- 플러터플로의 채팅 기능에는 아래와 같은 문제가 있다.
  - 개인 정보 노출. `/users` 문서에는 이메일, 전화번호 등이 들어가는데, 반드시 외부에 공개되어야지만, 채팅 기능이 동작한다.
  - 커스텀 디자인이 안된다. 플러터플로에서 제공하는 채팅 기능은 UI/UX 가 고정해져 있다. 기본 기능이 매우 빈약하며 변경 할 수 없다.

  이러한 제약 사항을 해결하고자 새로운 채팅 기능을 제작했다.


- `/chat_rooms` 에 채팅방 정보가 저장된다.
- `/chat_room_messages` 에 각 채팅 문서가 저장된다.


- 채팅 메시지를 보낼 때, 채팅방 정보가 존재하지 않으면 생성을 한다. 따라서 채팅 문서를 `/chat_room_messages` 에 생성하면, 자동으로 채팅방 정보가 업데이트 된다.
  - 이것은, 채팅방 외부에서 채팅 메시지를 보내고자 할 때에도 쉽게, 보내고자 하는 사람의 user ref 만 알면 메시지를 보낼 수 있다.
    - 예를 들어, 맨 처음 로그인을 하면, 사용자에게 웰컴 메세지를 보내거나,
    - 게시판/코멘트에서 채팅방으로 들어가지 않고, 바로 사용자에게 쪽지를 보내거나 등의 작업을 할 수 있다.



# 가입 환영 인사

- `/settings/system { helperUid: ..., welcomeMessage: ... }` 두 개의 필드가 존재하면, 새로 가입하는 사용자에게 환영 인사를 채팅으로 보낸다. 따라서 회원 가입하자 마자 (처음 사용하는 사용자에게) 새로운 채팅 메시지가 한 개 도착해 있게 된다.


