# Day 1 실습 정리: VPC 구축부터 Nginx 웹 배포까지

> 참고 자료: `네이버클라우드_강의자료/Day-1_실습_네이버클라우드-기초_v5_marp.pdf`
>
> 일정: 2026. 10. 31(토) 16:00 ~ 17:30 실습 · 17:30 ~ 18:00 정리. 이론은 [Day1_이론_정리.md](Day1_이론_정리.md) 참고.
>
> 강의안에 적힌 콘솔 메뉴, 상품 사양, 리전 지원 범위는 자료 작성 시점의 내용일 수 있다. 실습 당일 콘솔과 공식 안내에서 현재 설정을 확인한다.

## 1. 실습 순서

| Lab | 내용 | 구분 | 비고 |
| --- | --- | --- | --- |
| 01 | 계정 생성 · 결제 수단 · 크레딧 | 사전 준비 | 이미 가입했다면 로그인만 확인 |
| 02 | 서브 계정(Sub Account) & MFA | 선택 | |
| 03 | 콘솔 둘러보기 | 선택 | **Korea/VPC 확인은 필수** |
| 04 | 대시보드 위젯 · 북마크 | 선택 | |
| 05 | VPC · Subnet 생성 | **필수** | |
| 06 | 서버 · ACG · 공인 IP · SSH · Nginx · `index.html` | **필수** | 브라우저 확인까지 |
| 6-11 ~ 6-12 | `index_day1.html` 백업 · 기록 카드 | **필수** | Day 2 준비 |
| 06-S | Claude로 나만의 `index.html` | 선택 | Claude 계정 준비 시 |
| 07 | Object Storage 정적 웹 | 선택 | 시간 여유 시 |
| 08 | 백업 확인 · 리소스 반납 | 정리 | **서버·공인 IP는 Day 2 종료 후 반납** |

## 2. 시작 전 확인

- [ ] 브라우저: Chrome 또는 Edge 최신 버전
- [ ] `www.ncloud.com` 로그인 → 우측 상단 [Console] 진입
- [ ] 리전 **Korea**, 플랫폼 **VPC** 확인
- [ ] SSH 도구: Mac은 터미널 또는 VS Code Remote SSH, Windows는 PowerShell 또는 VS Code Remote SSH
- [ ] 리소스 이름에 본인 식별자(학번·이니셜)를 붙여 다른 학습자 리소스와 구분
- [ ] 인증키 파일을 내려받을 위치와 안전한 보관 방법 준비

### Day 1 실습 기록 카드 (개인 노트북의 메모장에만 저장)

```text
1) 서버 이름        : yau-web-svr-01
2) 서버 공인 IP     : ___.___.___.___               ← STEP 6-6
3) root 비밀번호    : ____________________          ← STEP 6-7
4) 인증키 파일 위치 : ~/Downloads/yau-key-01.pem    ← STEP 6-3
5) SSH 접속 명령    : ssh root@<공인 IP>
6) 웹 접속 주소     : http://<공인 IP>
7) 웹 파일 경로     : /var/www/html/index.html
8) 백업 파일 경로   : /var/www/html/index_day1.html ← STEP 6-11
```

이 카드는 Day 2에서 그대로 사용한다. IP·비밀번호는 채팅, 공개 문서, 저장소에 입력하지 않고 다른 사람과 공유하지 않는다.

## 3. Lab 01 — 계정 생성 및 이용 환경 준비 (사전 준비)

준비물: 본인 명의 이메일, 휴대폰(본인 인증), 신용/체크카드(결제 수단 등록). 크레딧을 지급받아도 최초 1회 카드 등록이 필요하다.

1. 포털 [회원가입] → **개인 회원** 선택(필요하면 나중에 사업자 전환)
2. 약관 동의 → 로그인 정보(이메일, 영문 대소문자·숫자·특수문자 8~16자 비밀번호)
3. 이름·휴대폰 SMS 본인 인증
4. [결제 수단 등록]에서 본인 명의 카드 등록(0원 또는 1,000원 결제 후 자동 취소)
5. [마이페이지] > [크레딧 관리]에서 강의용 크레딧 등록

> 결제 수단이 정상 등록돼야 서버, VPC 등 실제 리소스를 만들 수 있다.

## 4. Lab 02 ~ 04 — 계정 보안과 콘솔 익히기 (선택)

### Lab 02. 서브 계정과 MFA

1. [Services] > [Management] > [Sub Account] > [Sub Accounts] → [+ 서브 계정 추가]
2. 로그인 ID·이름·이메일 입력
3. [정책 추가]에서 필요한 정책만 부여(예: VPC 서버·VPC 네트워크 관리 정책) → [저장]
4. 서브 계정 전용 로그인 URL 생성 후 사용자에게 전달
5. 전용 URL로 로그인 → 최초 비밀번호 변경 → Google OTP 등 앱으로 QR 스캔해 **MFA** 설정

보안 핵심 3요소: **정책**(어떤 서비스에 접근?) · **역할**(어떤 작업을 허용?) · **MFA**(OTP 다중 인증)

### Lab 03. 콘솔 둘러보기

- 상단 GNB: 서비스 검색창, **Platform 전환(VPC/Classic)**, **Region 선택(Korea)**
- 좌측 메뉴: Compute(Server, Auto Scaling, Kubernetes) · Storage(Block, Object) · Networking(VPC, Subnet, Load Balancer) · Database · AI Service
- Dashboard: 서버 수, 스토리지, 최근 작업 이력

> 실습은 반드시 **VPC 플랫폼, Korea 리전**에서 진행한다.

### Lab 04. 대시보드 개인화

- 대시보드 [+ 위젯 추가] → Server 현황, VPC 현황, 이용 요금·크레딧 잔액, 공지사항 위젯을 골라 드래그 배치
- 좌측 메뉴의 VPC, Server, Object Storage 옆 별표(★)로 즐겨찾기 등록

## 5. Lab 05 — VPC 및 Subnet 구축 (필수)

```
VPC: yau-vpc-01 (10.0.0.0/16, 65,536개 IP)
 ├─ Public Subnet : yau-pub-sub-01 (10.0.1.0/24) · 웹서버 · 인터넷 통신 Y
 └─ Private Subnet: yau-pri-sub-01 (10.0.2.0/24) · DB/내부 · 인터넷 통신 N
```

### STEP 5-1. VPC 생성

1. [Services] > [Networking] > [VPC] > [VPC] → [+ VPC 생성]
2. VPC 이름 `yau-vpc-01`, IP 주소 범위 `10.0.0.0/16`
3. [생성] → 상태가 **운영중**으로 바뀌는지 확인

### STEP 5-2. Public Subnet 생성

1. [VPC] > [Subnet] → [+ Subnet 생성]
2. 이름 `yau-pub-sub-01`, VPC `yau-vpc-01`, 범위 `10.0.1.0/24`, Zone `KR-1`, 용도 일반, **인터넷 통신 여부 Y(Public)**
3. [생성] → 운영중 확인

### STEP 5-3. Private Subnet 생성

1. 이름 `yau-pri-sub-01`, VPC `yau-vpc-01`, 범위 `10.0.2.0/24`, Zone `KR-1`, 용도 일반, **인터넷 통신 여부 N(Private)**
2. [생성] → 외부에서 직접 접근할 수 없는 내부 DB용 Subnet 완성

### STEP 5-4. Network ACL · Route Table 점검

- [VPC] > [Network ACL]: 기본 NACL 규칙이 Subnet에 자동 매핑되어 있는지 확인
- [VPC] > [Route Table]: `yau-pub-sub-01`의 라우팅 테이블에서 `10.0.0.0/16 → Local`(내부 통신), `0.0.0.0/0 → Internet Gateway`(인터넷 출구) 확인

### Lab 05 자주 막히는 지점

| 증상 | 원인 | 해결 |
| --- | --- | --- |
| 메뉴에 VPC가 안 보임 | Platform이 Classic | 콘솔 상단 Platform을 VPC로 변경 |
| Subnet 생성 시 VPC 목록에 없음 | VPC가 아직 생성중 | 1~2분 기다린 뒤 새로고침 |
| "IP 주소 범위가 올바르지 않습니다" | VPC 대역 밖 값·오타 | `10.0.1.0/24`를 복사해 붙여넣기 |
| Public/Private를 잘못 선택 | 생성 후 변경 불가 | 잘못 만든 Subnet 삭제 후 다시 생성 |
| VPC를 2개 이상 만듦 | 버튼 중복 클릭 | 쓰지 않는 VPC 삭제(Subnet이 없을 때만 가능) |

완료 기준: `yau-vpc-01`, `yau-pub-sub-01` 상태가 모두 **운영중**

## 6. Lab 06 — Linux 서버 생성, SSH 접속, Nginx 웹 배포 (필수)

로드맵: **OS 선택(Ubuntu 22.04) → 스펙(vCPU 2, 4GB, Public Subnet) → 인증키(.pem) → ACG(22, 80) → 공인 IP → SSH & Nginx 배포**

> 주의: 반드시 **Public Subnet(`yau-pub-sub-01`)**을 선택해야 공인 IP를 붙일 수 있다. `.pem` 파일은 관리자 비밀번호를 확인하는 유일한 수단이므로 삭제하지 않는다.

### STEP 6-1 ~ 6-5. 서버 생성

1. [Services] > [Compute] > [Server] > [Server] → [+ 서버 생성]
2. **이미지**: Ubuntu 탭 → Ubuntu Server 22.04 LTS
3. **서버 설정**: VPC `yau-vpc-01`, Subnet `yau-pub-sub-01`, Generation 2, 시간 요금제, Standard(vCPU 2, 메모리 4GB), 서버 이름 `yau-web-svr-01`
4. **인증키**: 새로운 인증키 생성 → 이름 `yau-key-01` → [인증키 생성 및 다운로드]. 재다운로드가 불가하므로 안전하게 보관한다.
5. **네트워크 접근(ACG)**: [신규 ACG 생성] → 이름 `yau-web-acg`, 인바운드 규칙 2개

   | 프로토콜 | 접근 소스 | 포트 | 목적 |
   | --- | --- | --- | --- |
   | TCP | `0.0.0.0/0` | 22 | SSH 관리 접속 |
   | TCP | `0.0.0.0/0` | 80 | HTTP 웹 접속 |

   강의안 예시는 출발지를 `0.0.0.0/0`으로 열지만, SSH(22)는 가능한 경우 **본인 공인 IP로 제한**한다. 실습 종료 후 불필요한 외부 공개 규칙을 정리한다.

6. **최종 확인**(Ubuntu 22.04 / vCPU 2 · 4GB · 50GB / VPC·Subnet / 인증키 / ACG) → [서버 생성]
7. 상태: 설정중 → 부팅중 → **운영중**

### STEP 6-6. 공인 IP 신청 및 연결

1. [Compute] > [Server] > [Public IP] → [+ 공인 IP 신청]
2. 적용 서버 `yau-web-svr-01` 선택 → [확인]
3. 발급된 IP를 기록 카드 2)에 기록

### STEP 6-7. root 비밀번호 확인

1. [Server] 목록에서 `yau-web-svr-01` 선택
2. [서버 관리 및 설정 변경] > [관리자 비밀번호 확인]
3. `yau-key-01.pem` 업로드 → [비밀번호 확인] → 표시된 root 비밀번호를 기록 카드 3)에 저장

### STEP 6-8. SSH 원격 접속

권장: 비밀번호 방식(Mac · Windows 공통)

```bash
ssh root@<공인 IP>
```

1. `Are you sure you want to continue connecting (yes/no/[fingerprint])?` → `yes`
2. `password:` → root 비밀번호 붙여넣기(Mac ⌘+V / Windows 마우스 오른쪽 클릭) → Enter. 화면에 아무것도 안 보여도 정상이다.
3. 프롬프트가 `root@yau-web-svr-01:~#`로 바뀌면 접속 성공

키 파일 방식(Mac/Linux):

```bash
cd ~/Downloads
chmod 400 yau-key-01.pem
ssh -i yau-key-01.pem root@<공인 IP>
```

`Permission denied (publickey)`가 나오면 비밀번호 방식으로 접속한다.

| 증상 | 원인 | 해결 |
| --- | --- | --- |
| `Connection timed out` | ACG 22번 없음 · 공인 IP 미연결 · 학교망 차단 | ACG Inbound 22 확인 → Public IP 적용 서버 확인 → 휴대폰 핫스팟으로 재시도 |
| `Permission denied, please try again.` | 비밀번호 오입력 | 기록 카드에서 다시 복사해 한 번만 붙여넣기 |
| 비밀번호가 입력이 안 됨 | 표시되지 않을 뿐 정상 입력 중 | 붙여넣고 Enter |
| `REMOTE HOST IDENTIFICATION HAS CHANGED` | 같은 IP로 예전에 다른 서버 접속 | `ssh-keygen -R <공인 IP>` 후 재접속 |
| `UNPROTECTED PRIVATE KEY FILE!` | pem 권한이 너무 열림 | `chmod 400 yau-key-01.pem` 또는 비밀번호 방식 |
| ssh 명령을 찾을 수 없음(Windows) | OpenSSH 클라이언트 미설치 | 설정 > 앱 > 선택적 기능 > OpenSSH 클라이언트 설치 |

### STEP 6-9. Nginx 설치 및 실행

서버 터미널(`root@yau-web-svr-01:~#`)에서 순서대로 입력한다.

```bash
apt-get update -y            # 패키지 목록 업데이트
apt-get install -y nginx     # Nginx 설치
systemctl start nginx        # 서비스 시작
systemctl enable nginx       # 부팅 시 자동 실행
systemctl status nginx       # 상태 확인: active (running)
```

- 설치 중 "Which services should be restarted?" 화면이 나오면 Tab → `<Ok>` → Enter
- 브라우저에서 `http://<공인 IP>` 접속 → **"Welcome to nginx!"** 확인. HTTPS가 아니라 HTTP 주소를 사용한다.

### STEP 6-10. 나만의 웹 페이지 배포

```bash
cat << 'EOF' > /var/www/html/index.html
<!DOCTYPE html><html><head><meta charset="UTF-8"><title>영남대 Agentic AI</title></head>
<body style="text-align:center; padding:50px; font-family:sans-serif; background:#f0f4f8;">
  <div style="background:white; padding:35px; border-radius:14px; box-shadow:0 4px 10px rgba(0,0,0,0.1); display:inline-block;">
    <h1 style="color:#0288d1;">영남대학교 Agentic AI 강의 과정</h1>
    <p style="font-size:18px; color:#475569;">VPC 사설망 위에 Linux Ubuntu 가상 서버 & Nginx 웹 배포 성공!</p>
    <p style="font-size:14px; color:#94a3b8;">Server Hostname: yau-web-svr-01 | Port: 80 (HTTP)</p>
  </div>
</body></html>
EOF
```

확인:

```bash
ls -l /var/www/html/
cat /var/www/html/index.html
```

브라우저에서 `http://<공인 IP>`를 강력 새로고침(Windows Ctrl+F5 / Mac ⌘+Shift+R)하고, 휴대폰(LTE/5G)으로도 접속해 본다. "영남대학교 Agentic AI 강의 과정" 카드가 보이면 필수 목표 달성이다. 한글 표시를 위해 `<meta charset="UTF-8">`을 포함한다.

### 웹 접속이 안 될 때

| 증상 | 원인 | 해결 |
| --- | --- | --- |
| 계속 로딩 후 시간 초과 | ACG에 80번 없음 | `yau-web-acg` Inbound에 TCP 80 추가 |
| "사이트에 연결할 수 없음 / 연결 거부" | Nginx 미실행 | `systemctl restart nginx` → `systemctl status nginx` |
| "보안 연결이 지원되지 않음" | 브라우저가 https로 접속 | 주소를 `http://`로 직접 입력 |
| 여전히 "Welcome to nginx!" | 캐시 · 파일 저장 실패 | 강력 새로고침 → `cat /var/www/html/index.html` 확인 |
| 한글이 깨짐 | `<meta charset="UTF-8">` 누락 | STEP 6-10 블록을 처음부터 다시 붙여넣기 |
| 터미널에 `>`만 계속 나옴 | 마지막 `EOF` 줄이 빠짐 | `EOF` 입력 후 Enter(안 되면 Ctrl+C 후 재시도) |
| `E: Could not get lock ...` | 부팅 직후 자동 업데이트 진행 중 | 1~2분 뒤 같은 명령 다시 실행 |

### STEP 6-11. `index_day1.html` 백업 (Day 2 필수)

```bash
cp /var/www/html/index.html /var/www/html/index_day1.html
ls -l /var/www/html/
```

`index.html`, `index_day1.html`, `index.nginx-debian.html`이 보이면 완료. 브라우저에서 `http://<공인 IP>/index_day1.html`로도 확인한다. Day 2에서 `index.html`을 수정하다 문제가 생기면 이 백업으로 되돌린다.

### STEP 6-12. Day 2 필수 정보 최종 점검

- [ ] 기록 카드에 서버 공인 IP · root 비밀번호 기록
- [ ] `yau-key-01.pem` 파일 위치 확인 및 안전한 보관
- [ ] `http://<공인 IP>` 예제 페이지 확인
- [ ] `http://<공인 IP>/index_day1.html` 확인
- [ ] 기록 카드를 개인 노트북에 저장
- [ ] **서버·공인 IP를 반납하지 않음**

Day 2 재접속 흐름: 콘솔에서 서버 상태 확인(운영중이 아니면 [서버 시작]) → `ssh root@<공인 IP>` → root 비밀번호 → `/var/www/html/index.html` 수정 → 문제가 생기면 `index_day1.html`로 복원

## 7. Lab 06-S — Claude로 나만의 `index.html` 배포 (선택)

1. `claude.ai` 로그인 후 프롬프트 입력(내 정보로 바꾸기)

   ```text
   나만의 자기소개 웹페이지를 index.html 파일 하나로 만들어줘.
   - 이름: 홍길동 / 소속: 영남대학교 ○○학과 / 관심 분야: AI, 클라우드
   - 오늘 네이버클라우드 Ubuntu 서버에 Nginx로 배포했다는 문구 포함
   - 외부 파일(이미지, CSS, JS 링크) 없이 HTML 한 파일 안에 스타일까지 모두 포함
   - <meta charset="UTF-8"> 포함, 밝고 깔끔한 카드형 디자인
   - 코드만 출력해줘
   ```

   프롬프트에 root 비밀번호, 공인 IP, 개인 연락처 같은 민감 정보는 넣지 않는다.

2. 백업 확인 후 nano로 교체

   ```bash
   ls -l /var/www/html/index_day1.html   # 백업이 있는지 먼저 확인
   > /var/www/html/index.html             # 기존 내용 비우기
   nano /var/www/html/index.html
   ```

   Claude 코드 붙여넣기 → 저장 Ctrl+O → Enter → 종료 Ctrl+X

3. 브라우저 강력 새로고침. 예제 페이지로 되돌리려면:

   ```bash
   cp /var/www/html/index_day1.html /var/www/html/index.html
   ```

| 증상 | 해결 |
| --- | --- |
| 화면이 깨지거나 일부만 보임 | 코드가 잘려 붙여넣어짐 → 처음부터 다시 |
| 맨 위에 ```` ```html ```` 글자가 보임 | nano에서 해당 줄 삭제 후 저장 |
| nano에서 빠져나오지 못함 | Ctrl+X → Y → Enter |

## 8. Lab 07 — Object Storage 정적 웹 호스팅 (선택)

Object Storage는 서버 없이 HTML·CSS·JS·이미지 같은 정적 파일을 직접 호스팅할 수 있는 객체 스토리지다.

1. [Services] > [Storage] > [Object Storage] → 최초 1회 이용 신청
2. [+ 버킷 생성]: 이름 `yau-web-<학번>`(전 세계 유일), 리전 Korea
3. 버킷 [접근 권한 관리]: 전체 사용자(Anonymous)에 **조회(Read)만** 허용. **쓰기(Write) 권한은 절대 부여하지 않는다.**
4. 내 PC에서 간단한 `index.html`을 만들어 [파일 올리기]로 업로드
5. [정적 웹사이트 호스팅] 사용 체크, 인덱스 문서 `index.html` → 저장
6. 발급된 엔드포인트 URL(예: `https://<버킷이름>.kr.object.ncloudstorage.com/`)로 접속 확인

## 9. Lab 08 — 백업 확인 및 리소스 반납

| 자원 | Day 1 수업 후 | 반납 시점 |
| --- | --- | --- |
| 서버 `yau-web-svr-01` | 유지(정지 여부는 강사 안내) | Day 2 실습 종료 후 |
| 공인 IP | 유지 | Day 2 실습 종료 후 |
| Object Storage 버킷(Lab 07 진행 시) | 정리 가능 | Day 1 종료 시 |
| VPC · Subnet · ACG | 유지 | Day 2 실습 종료 후 |

과금 주의 리소스: **공인 IP**(연결돼 있어도 시간당 비용) · **서버**(정지 상태여도 디스크 보관료) · **Object Storage**(남은 파일). 유지하는 동안 크레딧에서 차감되므로 [마이페이지]에서 잔액을 확인한다.

반납 절차(강사 안내 후, Day 2 종료 뒤):

1. **공인 IP**: [Compute] > [Server] > [Public IP] → 선택 → [공인 IP 연결 해제] → [공인 IP 반납]
2. **서버**: [Server] → `yau-web-svr-01` 선택 → [서버 정지] → 정지 확인 후 [서버 반납] → 서버 이름 확인 후 [반납]
3. **Object Storage**: 버킷 안 파일을 **먼저 모두 삭제**한 뒤(파일이 남아 있으면 삭제 차단) [버킷 삭제]
4. 콘솔에서 남은 유료 리소스가 없는지 최종 확인

## 10. FAQ

| 단계 | 흔한 문제 | 빠른 확인 |
| --- | --- | --- |
| Lab 03 | 메뉴에 VPC가 없음 | 상단 Platform을 VPC로 변경 |
| Lab 05 | Subnet 생성 시 VPC가 안 보임 | VPC가 운영중인지 확인 → 새로고침 |
| Lab 06 | 서버가 Private Subnet에 생성됨 | 서버 상세의 Subnet 확인 → 다시 생성 |
| Lab 06 | SSH timed out | ACG 22 · Public IP 적용 서버 확인 |
| Lab 06 | SSH Permission denied | root 비밀번호 다시 복사, 키 방식이면 올바른 키인지 확인 |
| Lab 06 | 브라우저 접속 안 됨 | ACG 80 · `systemctl status nginx` · `http://` 확인 |
| Lab 06 | 기본 Nginx 페이지만 보임 | `index.html` 내용 확인 후 강력 새로고침 |
| 6-11 | 백업 파일이 없음 | `cp` 명령 다시 실행 → `ls -l /var/www/html/` |

## 11. Day 2 준비물

- [ ] 실습 기록 카드(서버 공인 IP, root 비밀번호)
- [ ] 반납하지 않은 서버 `yau-web-svr-01`(운영중 또는 정지)
- [ ] `/var/www/html/index.html` + `/var/www/html/index_day1.html`
- [ ] `yau-key-01.pem`(root 비밀번호를 다시 확인해야 할 때 필요)

Day 2 시작 시 첫 명령:

```bash
ssh root@<공인 IP>
ls -l /var/www/html/
```
