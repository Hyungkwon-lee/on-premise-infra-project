# on-premise-infra-project
1. 개요
온프레미스 인프라 환경에서 네트워크 장비(라우터/스위치)의
관리자 인증을 중앙에서 통제하기 위해 RADIUS 서버를 구축하였다.

기존에는 장비별 로컬 계정을 사용하여 관리했기 때문에
계정 관리 및 보안 통제가 어려운 문제가 존재했다.

2. 문제 상황
네트워크 장비마다 로컬 계정 개별 관리
관리자 권한 통제 어려움
접근 로그 중앙 관리 불가
보안 감사 대응 어려움

3. 해결 방법

RADIUS 기반 AAA(Authentication, Authorization, Accounting) 구조를 적용하여
중앙 인증 시스템을 구축

4. 주요 설정
- 주요 구성
RADIUS 서버: 192.168.20.12
네트워크 장비: Router / L3 Switch
인증 방식: ID / Password 기반 + 권한 제어


네트워크 장비 설정 (Cisco)
aaa new-model

aaa group server radius RADIUS_GROUP
 server 192.168.20.12 auth-port 1812 acct-port 1813

aaa authentication login RAD_AUTH group RADIUS_GROUP local
aaa authorization exec RAD_AUTH group RADIUS_GROUP local
aaa accounting exec RAD_ACCT start-stop group RADIUS_GROUP

radius-server host 192.168.20.12 auth-port 1812 acct-port 1813 key admin123

ip radius source-interface Vlan20

line vty 0 4
 login authentication RAD_AUTH
 transport input ssh

 RADIUS 서버 설정
 # /etc/raddb/clients.conf
client HQ-DSW3 {
 ipaddr = 192.168.20.4
 secret = admin123
}

# /etc/raddb/users
testuser Cleartext-Password := "testpass"
 Service-Type = Login-User,
 Cisco-AVPair = "shell:priv-lvl=15"

 5. 결과
네트워크 장비 관리자 인증을 중앙 서버로 통합
사용자별 권한(Level 15) 제어 가능
인증 및 접근 로그 기록 가능
보안 정책 일관성 확보

. 트러블슈팅
❗ 문제: RADIUS 인증은 되지만 권한이 적용되지 않음
원인:
사용자 privilege level 설정 누락
해결:
RADIUS 서버에서 Cisco-AVPair = "shell:priv-lvl=15" 추가
결과:
관리자 권한 정상 적용

7. 배운 점
AAA 구조를 통해 인증 / 인가 / 로그를 분리할 수 있음을 이해
네트워크 장비 보안에서 중앙 인증의 중요성 체감
실제 운영 환경에서 계정 관리 방식 설계 경험 확보
