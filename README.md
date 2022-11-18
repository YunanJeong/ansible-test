# ansible-test

# 설치
- [앤서블 설치 및 시작(심플)](https://league-cat.tistory.com/376)

## Python
    $ pip install ansible
    $ pip install ansible==2.10.7
- pip으로 설치. Ansible 버전따라 syntax가 다르므로, 버전 별 관리시 유용
- 파이썬 패키지 취급이므로, pyenv 등 격리환경에 종속될 수 있다.

## Ubuntu
    $ sudo apt update
    $ sudo apt install ansible
- root 환경에 설치되므로 ansible 커맨드에 `sudo` 필요 가능성

## CentOS
    $ yum install -y ansible
- Ansible과 CentOS 모두 RedHat회사에서 관리하므로, Ansible은 CentOS 기준 문서가 많다. 그러나 설치방법에 관계없이 용법이 거의 동일하므로 자신에게 편한 방법을 택하면 된다.
- 참고: 다수 개발자들에게 익숙한 Ubuntu로 CentOS의 점유율이 옮겨가는 중이다. CentOS는 서버 영역에서 많이 쓰이지만, 클라우드 서비스, DevOps, IaC 개념 등 발달로 서버 구축시 개발 비중이 커졌기 때문이다.

## MacOS
    $ brew install ansible

# 용어 및 개념
- [앤서블 시작하기(개념 등 설명 좋음)](https://wikidocs.net/130113)
- Control Node
    - Ansible이 설치 및 실행되는 호스트
    - 다른 호스트들을 관리하는 호스트
- Managed Node
    - 관리 대상 호스트
- Inventory (hosts)
    - Control Node의 `/etc/ansible/hosts` 파일
    - hosts 파일에 Managed Node들의 정보를 기술해야 함(public ip 및 분류 등)
    - 설치 후 디폴트 파일 및 경로가 자동생성되지 않으면 직접 생성 필요
    - 이후 ansible실행시 해당 파일 인식은 자동
- Module
- Task
- Playbook

# Ansible 사전 준비 작업
## 머신 준비(EC2로 시작)
- 인스턴스에 ssh, icmp 허용하는 보안그룹 필요
- Ansible 관련 블로그에서 초기설정시 핑테스트 예제를 많이 보여주는데, icmp 허용안하면 Fail 된다.
- 참고: [AWS EC2로 앤서블 테스트(초기 설정 상세함)](https://jojoldu.tistory.com/432)

## root, user 권한 관련사항
- 블로그들을 보면, 계정권한에 대해 자세히 설명이 없는데, 다음과 같이 진행하면 된다.
    - Control Node(Ansible 설치 및 실행 노드)에서는 모든 작업을 root 권한으로 처리 (`sudo`)
    - Managed Node에서는 모든 작업을 user 권한으로 처리
- 항상 이런 건 아닌데,
    - Ansible은 설정파일이 루트경로 `/etc/ansible`에 있다보니 root 권한으로 작업해야 편한 경우가 많다.
    - Managed node 쪽은 반드시 user 권한으로 작업한다.
        - Control Node가 Managed Node의 일반 user로 접근하기 때문
        - authorized_keys 파일도 일반권한으로 되어있어야 함

## SSH 설정
0. Control Node로 쓸 호스트에 Ansible 설치
    - `$ sudo ansible localhost -m ping`(ansible로 로컬에 핑 테스트)로 정상설치 확인
1. Control Node에서 `/etc/ansible/hosts` 파일 생성
```
# /etc/ansible/hosts
# 다음과 같이 Managed Host들의 ip를 적어준다.
# hosts에서 #으로 주석쓰기 가능
192.168.0.2
192.168.0.3
```

2. Control Node에서 `$ sudo ssh-keygen -t rsa` (root 권한으로 해줘야 나중에 편하다)
    - private key, public key 각각 파일 생성됨
    - `-t rsa`: 암호화 방식 rsa 선택을 의미
    - `Enter file in which to save the key (...): `라는 문구가 나오는데,
        - 파일명 포함 filepath 입력자리다. `/root/.ssh/ansible-rsa` 정도로 써주자.
        - 디폴트 파일명은 rsa
        - 디폴트 경로는 위 괄호안에 나옴 (root면 `/root/.ssh/`, user면 `~/.ssh/`)
    - `Enter passphrase: `: key의 비밀번호를 추가하고 싶을 때 사용. 보통 그냥 넘어감.

3. public key(pub 파일)의 내용 "텍스트 전체"를 복사하여, Managed Node의 `~/.ssh/authorized_keys`에 추가
    - `/root/.ssh/authorized_keys` 아니다.
    - authorized_key 기존 내용의 "다음줄"에 입력하면 된다.
    - 다음과 같이 redirect 하면 편하다.
    ```
    $ echo {...pub 내용...} >> ~/.ssh/authorized_keys
    ```
4. Control Node에서 `$ sudo ansible all -m ping`으로 정상동작 확인
    - `etc/ansible/hosts`의 모든 node에 핑테스트
    - ssh 설정이 잘못되면 fail
    - ssh, icmp 네트워크 차단되어있으면 fail
    - Managed Node의 `~/.ssh/authorized_keys` 권한이 root면 fail


## SSH 설정 관련 사항
- Ansible은 ssh로 다른 호스트들을 관리한다. ssh 사전 설정이 필요한데, Ansible 관련 글들은 이 부분을 너무 간략히 설명한다.
- ssh 설정시 헷갈리는 부분 or 알면 좋은 내용을 여기 정리한다.
- 참고: [ssh 상세설명](https://danthetech.netlify.app/Backend/configure-ssh-key-based-authentication-on-a-linux-server)

### 비대칭키 활용(private key와 public key)
- ssh 접속인증방법은 다양
    - 비대칭키: **Ansbile ssh의 일반적 채택 방식**
    - userid-password 로그인 인증: 상대적으로 보안이 약함
- 비대칭키는 생성 시부터 private key, public key의 쌍으로 구성됨
    - private key는 클라이언트에,
    - public key는 서버에 배치되는 용도다.
- 인증절차는 다음과 같다.
    1. 클라이언트는 서버에 접속요청하면서 자신의 key(private)를 제시
    2. 서버는 제시받은 key(private)와 자신의 key(public)를 비교
    3. 두 key가 매칭되는 쌍이 맞으면 접속 허가
- private key는 열쇠와 같은 것이므로, 보안 및 보관이 매우 중요
- 따라서 Ansible 사전준비시 다음과 같은 작업이 필요
    1. ssh접속용 비대칭키를 생성하고,
    2. private key 파일을 Control Node(클라이언트)에 배치
    3. public key 파일을 Managed Node(서버)에 배치


