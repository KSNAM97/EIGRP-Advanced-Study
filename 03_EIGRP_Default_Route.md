# Chapter 03. EIGRP Default-Route

## 📌 개요

EIGRP에서 Default-Route를 광고하는 방법으로 `ip summary-address eigrp` 명령을 활용하여
`0.0.0.0/0` 네트워크를 요약 광고할 수 있다.

---

## EX2-4) R5 외부 통신용 Default-Route 설정

### 조건
- R5는 외부 네트워크로 통신 시 EIGRP Default-Route를 사용

### R3 설정
```cisco
interface fastethernet 0/0
 ip summary-address eigrp 100  0.0.0.0  0.0.0.0
!
```

### 정보 확인
```
R5# show ip route
     13.0.0.0/24 is subnetted, 2 subnets
C       13.13.5.0  is directly connected, Loopback0
C       13.13.15.0 is directly connected, FastEthernet0/1
     150.3.0.0/24 is subnetted, 1 subnets
C       150.3.13.0 is directly connected, FastEthernet0/0
D*   0.0.0.0/0 [90/409600] via 150.3.13.3, 00:00:06, FastEthernet0/0   ← EIGRP Default-Route
```

> `D*` = EIGRP Default Route 표시

### 통신 테스트
```cisco
R5# ping 13.13.1.1  source 13.13.5.5
R5# ping 13.13.2.2  source 13.13.5.5
R5# ping 13.13.3.3  source 13.13.5.5
R5# ping 13.13.4.4  source 13.13.5.5
```
