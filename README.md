BoB 10기에서 진행한 수리카타 룰셋 만들기.

# 1. 환경 설정
#### 피해자: 우분투 20.04 LTS, 공격자: 칼리 리눅스(kali linux), virtual box(VM)를 이용
두 VM 모두 NAT 네트워크 인터페이스를 추가해 공격 패킷이 오고갈 수 있도록 네트워크를 설정했다.
칼리 리눅스에서 CVE를 부여받은 공격 중 일부를 메타스플로잇(metasploit)을 이용해 우분투 서버를 공격했다.

# 2. 서버 설치
공격에 성공하기 위해서는 ubuntu 커널단에서 막지 않는지를 확인하고 보안 패치가 이루어지기 전 버전의 서버들을 찾아 설치할 필요가 있다.

# 3. 메타스플로잇 이용
msfconsole로 메타스플로잇 열 수 있다. 메타스플로잇을 연 후에

  search cve:연도-식별번호
 
를 통해 사용할 수 있는 공격을 알려준다. 연도까지만 치거나 키워드만 쳐도 관련된 목록들이 나온다.
실제 공격을 할 때 자주 사용하는 msfconsole 명령어들은 아래와 같다.

  show options  // 설정 가능한 옵션들을 보여줌.
  show payloads  // 공격에 이용 가능한 페이로드를 보여줌.
  info  // 해당 CVE에 대한 설명을 보여줌.
  set <변수> <값>  // 공격에 설정할 값들을 지정해줄 수 있음. RHOST IP 등을 지정해주는데 사용.
  edit  // 소스 코드를 보여줌. 소스 코드 수정을 원한다면 /usr/share/metasploit-framewokr 디렉터리 아래에서
        // sudo 명령어를 통해 에디터를 실행해줘야 함.


# 4. 파일 설정
#### slowloris나 apache, tomcat DoS 공격 등이 성공했는지 정확히 판단하기 위해 여러 설정 파일 변경 필요
* apache2.4 같은 경우 slowloris를 /etc/apache2/mods-enabled/reqtimeout.conf라는 설정 파일에서 RequestReadTimeout header=10을 통해 막을 수 있다. 10초 이내로 헤더를 전부 다 보내지 않을 경우 차단하겠다는 설정으로 이 숫자를 늘리거나 해당 줄을 주석처리하면 된다. RUDY 공격과 같은 경우는 같은 파일의 RequestReadTimeout body=20을 바꿔주면 된다.
* apache2.4 같은 경우 동시 최대 연결 가능한 클라이언트 수를 /etc/apache2/mods-enabled/mpm_event.conf라는 설정 파일에서 MaxRequestWorkers라는 변수를 통해 설정한다. DoS공격을 원활히 진행하기 위해서 이를 바꾸거나 메타스플로잇의 세션 연결 수를 바꿔줘야 한다.
