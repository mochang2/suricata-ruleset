BoB 10기에서 진행한 수리카타 룰셋 만들기.

# 1. 환경 설정
#### 피해자: 우분투 20.04 LTS, 공격자: 칼리 리눅스(kali linux), virtual box(VM)를 이용
두 VM 모두 NAT 네트워크 인터페이스를 추가해 공격 패킷이 오고갈 수 있도록 네트워크를 설정했다.
칼리 리눅스에서 CVE를 부여받은 공격 중 일부를 메타스플로잇(metasploit)을 이용해 우분투 서버를 공격했다.

# 2. 서버 설치
공격에 성공하기 위해서는 ubuntu 커널단에서 막지 않는지를 확인하고 보안 패치가 이루어지기 전 버전의 프로그램들을 찾아 설치할 필요가 있다.

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
* apache2.4 같은 경우 slowloris를 _/etc/apache2/mods-enabled/reqtimeout.conf_ 라는 설정 파일에서 RequestReadTimeout header=10을 통해 막을 수 있다. 10초 이내로 헤더를 전부 다 보내지 않을 경우 차단하겠다는 설정으로 이 숫자를 늘리거나 해당 줄을 주석처리하면 된다. RUDY 공격과 같은 경우는 같은 파일의 RequestReadTimeout body=20을 바꿔주면 된다.
* apache2.4 같은 경우 동시 최대 연결 가능한 클라이언트 수를 _/etc/apache2/mods-enabled/mpm_event.conf_ 라는 설정 파일에서 MaxRequestWorkers라는 변수를 통해 설정한다. DoS공격을 원활히 진행하기 위해서 이를 바꾸거나 메타스플로잇의 세션 연결 수를 바꿔줘야 한다.
* 수리카타 룰 파일의 위치는 _/var/lib/suricata/rules/suricata.rules_ 에 있으며, 인터페이스를 지정하거나 로그 파일 등의 위치를 지정할 수 있는 설정 파일 위치는 _/etc/suricata/suricata.yaml_ 이다. alert나 drop등의 정책으로 기본적으로 로그가 찍히는 위치는 _/var/log/suricata/fast.log_ 이다.

# 5. 수리카타 모드 변경
#### 단순히 수리카타 설치만 하고 drop 룰을 적용하면 패킷이 차단되지 않음. 모드의 변경이 필요.
수리카타에는 여러 가지 모드들이 있는데 그 중 대표적으로 많이 사용되는 모드가 IDS모드와 IPS모드이다. 수리카타를 설치하고 아무 설정을 건드리지 않거나, 명령어를 입력하지 않으면 패킷에 대한 로그를 남기지만(IDS) 직접 차단시키지는 못한다.  
IPS 모드로 동작하기 위해서는 가장 쉬운 방법이 NFQUEUE를 이용하는 것이다. NFQUEUE에 대한 설명과 NFQUEUE를 이용하는 것 이외에 수리카타를 IPS 모드로 동작시키고 싶다면 다음 문서를 참조하면 된다. <https://suricata.readthedocs.io/en/suricata-6.0.0/setting-up-ipsinline-for-linux.html#setting-up-ips-with-netfilter> </br>
<https://suricata.readthedocs.io/en/suricata-6.0.0/configuration/suricata-yaml.html#nfq>  
  
아래 명령어는 NFQUEUE를 이용하여 수리카타를 IPS 모드로 동작시키게 하는 명령어이다. 공식문서에 따라 yaml 파일 설정을 바꾼 후 순서대로 입력하면 된다.

        sudo iptables -I INPUT -j NFQUEUE  // input chain NFQUEUE 정책 실행
        sudo iptables -I OUTPUT -j NFQUEUE  // output chain NFQUEUE 정책 실행
        sudo suricata -c /etc/suricata/suricata.yaml -q 0  // 수리카타 설정 파일을 불러와서 실행
        
참고로 공격을 시도했을 때 로그에 wdrop이 남으면 IPS 모드가 제대로 설정이 안 되었다는 뜻이다. wdrop은 would drop으로 drop 흉내를 내지만 패킷의 흐름을 방해하진 않는다. 하지만 완벽히 drop시킨다는 뜻은 또 아니다.
        
# 6. 수행한 공격과 차단 룰셋
* CVE:2007-6750(Slow HTTP DoS)
* CVE:2020-13160(anydesk format string vulnerability)
* CVE:2014-0050(apache, tomcat fileupload DoS)
* CVE:2018-12613(phpmyadmin LFI RCE)
* CVE:2020-10199(nexus repo manager RCE)

공격에 대한 자세한 설명과 룰셋, 소스코드는 아래 노션에 정리함.  
<https://www.notion.so/IDS-IPS-283460d27f1446af8fbeb61da3e3fa76>    

# 7. .pcap(linux 환경) 재전송
공격 코드가 없어도 pcap 파일에서 캡처된 패킷 그대로 replay되게끔 할 수 있다. 우선 replay하고 싶은 패킷을 wireshark 등을 통해 pcap 파일로 만들어야 한다. 이후 아래 명령어를 통해 dummy interface를 추가한다.

        sudo ip link add dum0 type dummy  // dummy interface의 이름이 dum0
        sudo ifconfig dum0 up
        sudo ip link del dum0             // 추가된 interface를 삭제하는 명령어

tcpreplay 명령어를 이용해 해당 inteface로 패킷을 전송하게끔 한다. tcpreplay는 기본으로 있는 패키지가 아니므로 sudo apt install tcpreplay를 해줘야 한다.

        sudo tcpreplay -i dum0 xxx.pcap   // 절대경로, 상대경로 모두 가능
        
위와 같은 방식으로 진행하면 dum0 interface에 똑같은 패킷이 재전송되고, wireshark로 패킷을 잡으면 똑같은 패킷들이 똑같은 시간동안 잡힌다.

# 8. Suricata 사용 이유

- 오픈소스.
- multi-threading을 사용하기 때문에 high performance를 기대할 수 있음.
- Snort가 갖고 있던 대부분의 기능을 갖고 있고 Snort의 기능 외에도 Protocol Identification, HTTP Normalizer & Parser, File Identification 등의 기능이 추가됐음.

