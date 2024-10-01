<!-- 목차 -->


## 1. WSL2 Ubuntu IP 고정사용: 포드포워딩

이 가이드는 WSL2 Ubuntu의 IP 주소가 재부팅 시 변경되는 문제를 해결하고, 고정 IP처럼 사용하는 방법을 설명합니다. Windows의 포트 포워딩과 작업 스케줄러를 활용하여 재부팅 후에도 WSL2 Ubuntu에 접속할 수 있도록 합니다.

### 1. WSL IP 확인

WSL2 Ubuntu의 IP 주소는 Windows의 IP 주소와 다릅니다.  두 IP 주소를 확인합니다.

**Windows PowerShell (관리자 권한):**

```powershell
ipconfig
```

* IPv4 주소 (192.168.x.x 형태)를 확인합니다.

**WSL2 Ubuntu 터미널:**

```bash
ifconfig
```

* eth0 또는 다른 네트워크 인터페이스의 inet 주소 (172.x.x.x 형태)를 확인합니다. 이 주소는 재부팅 시 변경될 수 있습니다.

### 2. 포트 포워딩 (초기 설정)

외부에서 WSL2 Ubuntu에 접속하려면 Windows에서 포트 포워딩을 설정해야 합니다.  Nginx를 예시로 사용합니다.

**WSL2 Ubuntu 터미널:**

```bash
sudo apt update
sudo apt install nginx
sudo systemctl start nginx
# sudo service nginx start
```

**Windows PowerShell (관리자 권한):**

```powershell
# 기존 규칙 삭제 (필요한 경우)
netsh interface portproxy delete v4tov4 listenport=80 listenaddress=0.0.0.0

# 포트 포워딩 규칙 추가. <WSL_IP>는 WSL2 Ubuntu의 IP 주소로 변경.
netsh interface portproxy add v4tov4 listenport=80 listenaddress=0.0.0.0 connectport=80 connectaddress=<WSL_IP>

# 규칙 확인
netsh interface portproxy show all
```

* `listenport`: Windows에서 수신할 포트 번호 (예: 80)
* `connectaddress`: WSL2 Ubuntu의 IP 주소
* `connectport`: WSL2 Ubuntu에서 수신할 포트 번호 (예: 80 - Nginx 기본 포트)

브라우저에서 Windows IP 주소(`ipconfig`로 확인한 주소)로 접속하여 Nginx 시작 페이지가 표시되는지 확인합니다.

### 3. PowerShell 스크립트 작성

재부팅 시 변경되는 WSL2 Ubuntu의 IP 주소에 맞춰 포트 포워딩 규칙을 자동으로 업데이트하는 PowerShell 스크립트를 작성합니다.

**`wsl_portforward.ps1` 파일 생성 및 아래 내용 추가:**

```powershell
# 기존 포트 포워딩 규칙 삭제
netsh interface portproxy delete v4tov4 listenport=80 listenaddress=0.0.0.0

# WSL2 Ubuntu IP 주소 가져오기
$wsl_ip = wsl hostname -I | Out-String
# $wsl_ip = (wsl hostname -I).Split()[0] | Out-String # 도커 IP 주소 제외
$wsl_ip = $wsl_ip.Trim()

# 새로운 포트 포워딩 규칙 추가
netsh interface portproxy add v4tov4 listenport=80 listenaddress=0.0.0.0 connectport=80 connectaddress=$wsl_ip
```

### 4. 스크립트 실행 권한 설정 및 테스트

**Windows PowerShell (관리자 권한):**

```powershell
# 스크립트 실행 권한 설정
Set-ExecutionPolicy RemoteSigned -Force

# 스크립트 실행
C:\wsl_portforward.ps1 # 스크립트 파일 경로

# 포트 포워딩 규칙 확인
netsh interface portproxy show all
```

스크립트 실행 후, 브라우저에서 Windows IP 주소로 접속하여 Nginx 시작 페이지가 표시되는지 확인합니다.

### 5. 작업 스케줄러 등록

Windows 시작 시 스크립트가 자동으로 실행되도록 작업 스케줄러에 등록합니다.

1. "작업 스케줄러" 검색 후 실행.
2. "작업 만들기..." 클릭.
3. "일반" 탭에서 작업 이름 설정 및 "최고 권한으로 실행" 체크.
4. "트리거" 탭에서 "새로 만들기..." 클릭 후 "시작할 때" 선택.
5. "동작" 탭에서 "새로 만들기..." 클릭 후 "프로그램 시작" 선택.
    * "프로그램/스크립트": `powershell` 입력.
    * "인수 추가(선택)": `-File "C:\wsl_portforward.ps1"` 입력 (스크립트 파일 경로).
6. "조건" 및 "설정" 탭은 기본 설정 사용.
7. "확인" 클릭하여 작업 생성.

### 6. 재부팅 후 테스트

Windows를 재부팅한 후, 브라우저에서 Windows IP 주소로 접속하여 Nginx 시작 페이지가 표시되는지 확인합니다.  정상적으로 표시된다면 WSL2 Ubuntu를 고정 IP처럼 사용하는 설정이 완료된 것입니다.
 