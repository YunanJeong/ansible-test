# ansible-test

# 참고
[앤서블 시작하기](https://wikidocs.net/130109)

# 설치
[앤서블 설치 및 시작](https://league-cat.tistory.com/376)

## Python
    $ pip install ansible
    $ pip install ansible==2.10.7
- pip으로 설치. Ansible 버전따라 syntax가 다르므로, 버전 별 관리시 유용
- 파이썬 패키지 취급이므로, pyenv 등 격리환경에 종속될 수 있다.

## Ubuntu
    $ sudo apt update
    $ sudo apt install ansible
- root 환경에 설치되므로 ansible 커맨드에 항상 `sudo` 필요

## CentOS
    $ yum install -y ansible
- Ansible과 CentOS 모두 RedHat회사에서 관리하므로, Ansible은 CentOS 기준 문서가 많다. 그러나 설치방법에 관계없이 용법이 거의 동일하므로 자신에게 편한 방법을 택하면 된다.
- 참고: 다수 개발자들에게 익숙한 Ubuntu로 CentOS의 점유율이 옮겨가는 중이다. CentOS는 서버 영역에서 많이 쓰이지만, 클라우드 서비스, DevOps, IaC 개념 등 발달로 서버 구축시 개발 비중이 커졌기 때문이다.

## MacOS
    $ brew install ansible