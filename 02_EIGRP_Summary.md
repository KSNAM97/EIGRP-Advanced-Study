# Chapter 02. EIGRP 주소 요약 (Summarization)

## 📌 CIDR (Classless Inter-Domain Routing)

- 무 Class 형식으로 IP address에 대해서 Class 기반이 아닌 **SubnetMask 기반**으로 동작
- Routing Table을 줄임으로써 **메모리 소모량 감소** 및 **Lookup Delay 최소화**
- **Flapping 현상**에 의한 문제를 주소 요약을 사용하여 해결
- EIGRP Summary는 **Outbound interface**에서 설정

---

## EX2-1) Loopback 121 생성 및 EIGRP 광고

### 조건
- R4에 Loopback 121을 생성한 후 `121.160.8.0/24 ~ 121.160.15.0/24` 대역 IP 할당
- Loopback 121에 포함된 네트워크를 EIGRP로 라우팅 업데이트 (단 `network` command는 **한줄만 사용**)

### R4 설정
```cisco
interface loopback 121
 ip address 121.160.8.4   255.255.255.0
 ip address 121.160.9.4   255.255.255.0 secondary
 ip address 121.160.10.4  255.255.255.0 secondary
 ip address 121.160.11.4  255.255.255.0 secondary
 ip address 121.160.12.4  255.255.255.0 secondary
 ip address 121.160.13.4  255.255.255.0 secondary
 ip address 121.160.14.4  255.255.255.0 secondary
 ip address 121.160.15.4  255.255.255.0 secondary
!
router eigrp 100
 network 121.160.8.0  0.0.7.255
!
```

### Wildcard 계산
```
121.160.8.0  ~ 121.160.15.0  (8개 네트워크)
121.160.00001 000.00000000
121.160.00001 111.11111111
-------------------------
Wildcard = 0.0.7.255
```

### 정보 확인
```
R5# show ip route eigrp | include 121
     121.0.0.0/24 is subnetted, 8 subnets
D       121.160.9.0  [90/2349056] via 150.3.13.3, 00:00:05, FastEthernet0/0
D       121.160.8.0  [90/2349056] via 150.3.13.3, 00:00:05, FastEthernet0/0
D       121.160.11.0 [90/2349056] via 150.3.13.3, 00:00:05, FastEthernet0/0
D       121.160.10.0 [90/2349056] via 150.3.13.3, 00:00:05, FastEthernet0/0
D       121.160.13.0 [90/2349056] via 150.3.13.3, 00:00:05, FastEthernet0/0
D       121.160.12.0 [90/2349056] via 150.3.13.3, 00:00:05, FastEthernet0/0
D       121.160.15.0 [90/2349056] via 150.3.13.3, 00:00:05, FastEthernet0/0
D       121.160.14.0 [90/2349056] via 150.3.13.3, 00:00:05, FastEthernet0/0
```

---

## EX2-2) R3 → R5 수동 요약 (13.13.0.0/16)

### 조건
- R3은 R5에게 `13.13.0.0/16` 범위로 수동 요약하여 업데이트

### R3 설정
```cisco
interface fastethernet 0/0
 ip summary-address eigrp 100  13.13.0.0  255.255.0.0
!
```

### 정보 확인 (메모리 비교)
```
[요약 전]
eigrp 100   8 networks   19 subnets   1944 overhead   3672 bytes

[요약 후]
eigrp 100   8 networks   10 subnets   1296 overhead   2448 bytes  ← 메모리 절감
```

---

## EX2-3) R3 → R5 Loopback 121 요약 (121.160.8.0/21)

### 조건
- R3은 R5에게 `121.160.8.0/24 ~ 121.160.15.0/24` 네트워크를 하나로 요약

### Subnet Mask 계산
```
121.160.00001 000.00000000  ← 121.160.8.0
121.160.00001 001.00000000  ← 121.160.9.0
121.160.00001 010.00000000  ← 121.160.10.0
121.160.00001 011.00000000  ← 121.160.11.0
121.160.00001 100.00000000  ← 121.160.12.0
121.160.00001 101.00000000  ← 121.160.13.0
121.160.00001 110.00000000  ← 121.160.14.0
121.160.00001 111.11111111  ← 121.160.15.0
--------------------------
255.255.11111 000.00000000  =  121.160.8.0/21 (255.255.248.0)
```

### R3 설정
```cisco
interface fastethernet 0/0
 ip summary-address eigrp 100  121.160.8.0  255.255.248.0
!
```

### 정보 확인
```
R5# show ip route eigrp
     13.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
D       13.13.0.0/16 [90/409600] via 150.3.13.3, 00:00:03, FastEthernet0/0
     121.0.0.0/21 is subnetted, 1 subnets
D       121.160.8.0 [90/2349056] via 150.3.13.3, 00:00:03, FastEthernet0/0  ← 요약된 단일 네트워크
```

```
R3# show ip protocol
  Address Summarization:
    121.160.8.0/21 for FastEthernet0/0
      Summarizing with metric 2323456
    13.13.0.0/16 for FastEthernet0/0
      Summarizing with metric 128256
```

---

## 🔄 설정 삭제
```cisco
# R3
interface FastEthernet0/0
 no ip summary-address eigrp 100 121.160.8.0 255.255.248.0
 no ip summary-address eigrp 100 13.13.0.0 255.255.0.0
!
```
