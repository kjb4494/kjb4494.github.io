---
title: EC2에 OpenVPN 서버 구축하기
author: kjb4494
date: 2023-10-19 14:00:00 +0900
categories: [Cloud Platform, AWS]
tags: [cloud, aws, open vpn]
---

OpenVPN은 클라우드에 연결된 개인 네트워크의 보안을 강화하거나 여러 클라우드를 연결하는 방법을 제공합니다. 그 중에서도 OpenVPN은 주로 UDP를 사용하는데, 이에 대한 이유를 시작으로 AWS EC2에서 Docker로 OpenVPN을 구축하는 방법을 소개하겠습니다.

## 왜 OpenVPN은 UDP를 사용하는가?

OpenVPN은 TCP와 UDP 두 가지 전송 프로토콜 모두를 지원합니다. 그러나 주로 UDP가 권장됩니다. UDP를 사용하는 주된 이유는 다음과 같습니다:

1. **성능**: TCP는 데이터 전송의 신뢰성을 확보하기 위해 재전송 메커니즘을 사용합니다. 만약 VPN 터널 내부의 TCP가 데이터를 재전송하고, 외부의 TCP도 재전송을 한다면 이중 재전송이 발생하게 되어 성능 저하가 발생합니다.

2. **단순성**: UDP는 연결 상태가 없기 때문에 연결 설정 및 종료 과정이 단순합니다. 이로 인해 OpenVPN 서버와 클라이언트 간의 연결이 더 빠르게 이루어집니다.

3. **유연성**: TCP는 연결이 중단될 경우 재연결 메커니즘을 가지고 있지만, 이는 때로 네트워크 지연이나 중단을 야기할 수 있습니다. 반면, UDP를 사용할 경우 OpenVPN 자체의 재연결 메커니즘을 활용할 수 있어 더 유연한 연결 관리가 가능합니다.

이러한 이유로, 대부분의 환경에서는 OpenVPN의 UDP 모드 사용이 권장됩니다.

## 준비 사항

- Docker가 설치된 AWS EC2 인스턴스.
- EC2 인스턴스를 가리키는 도메인 이름. 이 예제에서는 `openvpn.kjb4494.site`를 사용합니다.

## 단계별 가이드

### 1. OpenVPN 서버 설정

먼저 Docker를 사용하여 OpenVPN을 실행합니다. 다음 명령은 미리 만들어진 이미지 `kylemanna/openvpn`을 사용하고 도메인에 특정한 설정을 합니다.

```shell
# 설정 파일 생성
sudo docker run -v /workspace/openvpn:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://openvpn.kjb4494.site

# PKI (Public Key Infrastructure) 초기화
sudo docker run -v /workspace/openvpn:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki

# OpenVPN 서버 시작
sudo docker run -v /workspace/openvpn:/etc/openvpn -d -p 1194:1194/udp --name openvpn --cap-add=NET_ADMIN --restart=always kylemanna/openvpn
```

### 2. VPN 접속을 위한 사용자 생성

서버가 실행 중일 때, VPN에 연결하기 위한 사용자 프로필을 생성해야 합니다.

```shell
# 'admin' 사용자 생성 (비밀번호 없음)
sudo docker run -v /workspace/openvpn:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full admin nopass
```

### 3. 사용자의 설정 프로필 받기

이 단계에서는 VPN 클라이언트가 서버에 연결하기 위해 필요한 설정 및 자격 증명이 포함된 `.ovpn` 파일을 생성합니다.

```shell
# 'admin' 사용자에 대한 'admin.ovpn' 생성
sudo docker run -v /workspace/openvpn:/etc/openvpn --rm -it kylemanna/openvpn ovpn_getclient admin > admin.ovpn
```

### 4. Nginx를 사용한 헬스 체크 설정

OpenVPN 서버의 상태를 모니터링하기 위해 Nginx를 사용하여 간단한 헬스 체크 엔드포인트를 설정할 수 있습니다.

먼저 설정 파일을 생성합니다:

```shell
cat <<EOL > /etc/nginx/conf.d/openvpn.conf
server {
    listen 1194;

    location / {
        return 200 'ok';
    }
}
EOL
```

설정 파일을 생성한 후 변경 사항을 적용하기 위해 Nginx를 재시작합니다:

```shell
sudo systemctl restart nginx
```

### 5. AWS 설정

- **대상그룹 생성**:

  ![스크린샷](/assets/img/posts/2023-10-19/스크린샷%202023-10-19%20141933.png)

- **NLB 생성**:

  ![스크린샷](/assets/img/posts/2023-10-19/스크린샷%202023-10-19%20142047.png)

- **Route53 설정**:

  ![스크린샷](/assets/img/posts/2023-10-19/스크린샷%202023-10-19%20142326.png){: w="300" .normal }

### 6. OpenVPN Connect

`OpenVPN Connect`에서 3번에서 추출했던 `admin.ovpn` 파일을 import하면 프로필이 생성됩니다. 잘 연결되는지 확인합니다:

![스크린샷](/assets/img/posts/2023-10-19/스크린샷%202023-10-19%20142222.png){: w="300" .normal }

## 결론

OpenVPN은 강력하면서도 유연한 VPN 솔루션으로, AWS의 EC2 인스턴스에서 손쉽게 구축할 수 있습니다. 지금까지 Docker와 함께 OpenVPN 서버를 설정하는 방법을 다뤄보았습니다. 이런 OpenVPN 서버가 있으면 개발시 클라우드 환경에서의 보안을 강화하고, 원격 접근을 허용하는 데 큰 도움이 될 것입니다. 😄
