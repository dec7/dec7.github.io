---
layout: post
title: "Raspberry Pi setting"
description: "Raspberry Pi setting"
date: 2020-06-08
tags: [raspberrypi,setting]
comments: true
share: true
---

## 구성 요소
|제품|수량|가격|링크|
|--|--|--|--|
|raspberry pi 4 Model B (rev 1.2) CM2711, quad core Cortex-A72 (ARM v8) 64-bit SoC @ 1.5GHz / 4G|4|321,200|[link](https://www.devicemart.co.kr/goods/view?no=12234534&NaPm=ct%3Dkb6f150k%7Cci%3Dcheckout%7Ctr%3Dppc%7Ctrx%3D%7Chk%3D64aac6eeb204c534ed21a1fc7b4f0b6039093976)|
|iptime PoE8000 8포트 PoE LAN 기가비트 스위칭 허브|1|89,000|[link](https://smartstore.naver.com/ks1st/products/4863848850?NaPm=ct%3Dkb6f5pqx%7Cci%3Dcheckout%7Ctr%3Dppc%7Ctrx%3D%7Chk%3D645ea5303b81a96261b3e542722523d78c2d4a83)|
|HDMI to Micro HDMI 젠더 15cm|1|1,900|[link](https://smartstore.naver.com/bscom/products/4756877897?NaPm=ct%3Dkb6f6x42%7Cci%3Dcheckout%7Ctr%3Dppc%7Ctrx%3D%7Chk%3D72ed7c8c4be0a4dc9a2fc2467ee572d491775fec)|
|USB C타입 꺽임 케이블 20cm|10|20,700|[link](https://smartstore.naver.com/bscom/products/4634998787?NaPm=ct%3Dkb6f7shh%7Cci%3Dcheckout%7Ctr%3Dppc%7Ctrx%3D%7Chk%3D215cfdd23430bc68ab2916e9a86d9c23f80bfb2b)|
|USB 3.1 젠더(C타입) 측면 USB 2.0 A(M) 상향 25cm NT627|5|15,000|[link](https://smartstore.naver.com/alltem/products/4316036880?NaPm=ct%3Dkb6fvemm%7Cci%3Dcheckout%7Ctr%3Dppc%7Ctrx%3D%7Chk%3D13e7110504d7a50fbe1b08997a85002e803e5649))
|라즈베리파이 투명 아크릴케이스 4층|1|16,500|[link](https://smartstore.naver.com/ntrex/products/4610813013?NaPm=ct%3Dkb6f9bci%7Cci%3Dcheckout%7Ctr%3Dppc%7Ctrx%3D%7Chk%3D568d2ebc94eda9e72fe70bb260292b1441fcad55)|
|실리콘파워 microSDXC Class10 Superior UHS-I U3 A2 V30 64GB |4|52,350|전자랜드|
|CAT7 STP 슬림랜선 0.3m|6|14,100|[link](https://smartstore.naver.com/bscom/products/388113168?NaPm=ct%3Dkb6fch3d%7Cci%3Dcheckout%7Ctr%3Dppc%7Ctrx%3D%7Chk%3D51b0d379093ac5cd192c662381b0c072463b9f28)|

## network 구성
### 장비
- 주 공유기: iptiime A8004t
- 보조 공유기: iptime A3004ns-m

### 장비 구성
- 주 공유기 gateway 설정
- 보조 공유기 무선 멀티 브릿지 설정
- raspberrybi 연결할 스위치와 유선 연결

### 장비 설정
- 주 공유기
  - iptime VPN 적용
  - 보조 공유기에 고정IP할당
  - raspberry pi에 고정IP할당

### 참조
- [link](https://comterman.tistory.com/815)

## raspberry pi 설정
### raspbian os 설치
- Raspberry Pi OS (32-bit) Lite / May 2020 Ver / 2020-05-27
  - [link](https://www.raspberrypi.org/downloads/raspberry-pi-os/)
  - [link](https://geeksvoyage.com/raspberry%20pi4/installing-os-for-pi4/)
- Etcher
  - raspbian image 복사 프로그램
  - [link](https://etcher.io/)

### raspi-config
```
sudo raspi-config
```
- ssh, hostname, locale, timezome 변경
- 고정IP 설정
  - ```sudo vi /etc/dhcpcd.conf```
  - [link](https://withcoding.com/46)
- reboot  

- vi 설치

## kubernetes 설치
### 구성
- raspberry pi 1ea controller
- raspberry pi 3ea workers

### 설정
- swap file 비활성화 
```
sudo dphys-swapfile swapoff
sudo dphys-swapfile uninstall
sudo systemctl disable dphys-swapfile
```

- docker 설치 
```
sudo curl -sL get.docker.com |  sh
sudo usermod -aG docker pi
```

- kubenetes 리포 추가 
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - 
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list 
```

- boot 설정 변경
```
sudo vi /boot/cmdline.txt

# 추가
ipv6.disable=1 cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1
```

- 업데이트
```
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
```

- kubenetes admin 설치
```
sudo apt-get install -y kubeadm
```

- iptables 링크 변경
```
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy 
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy 
sudo update-alternatives --set ebtables /usr/sbin/ebtables-legacy
```

### 참고
- [link](https://github.com/codesqueak/k18srpi4)
- [link](https://medium.com/finda-tech/overview-8d169b2a54ff)