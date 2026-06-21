# Chapter 01. EIGRP 기본 구성

## 📌 개요

EIGRP(Enhanced Interior Gateway Routing Protocol)는 Cisco에서 개발한 Advanced Distance Vector 라우팅 프로토콜이다.

### 주요 특징
- AS(Autonomous System) 번호 기반 동작
- DUAL(Diffusing Update Algorithm) 알고리즘 사용
- Successor / Feasible Successor 개념 사용
- Composite Metric (Bandwidth + Delay 기반)

---

## 🎯 실습 조건

- EIGRP Routing Protocol을 사용하여 모든 네트워크간 통신되도록 설정
- AS = 100, 자동 요약(auto-summary) 기능은 사용하지 않음
- EIGRP 라우팅 업데이트가 필요한 Interface로만 EIGRP Packet이 송신되어야 함

---

## ⚙️ 설정

### R1
```cisco
router eigrp 100
 no auto-summary
 passive-interface default
 no passive-interface serial 1/1
 no passive-interface serial 1/0.123
 no passive-interface fastethernet 0/0
 network 13.13.1.0  0.0.0.255
 network 13.13.10.0 0.0.0.255
 network 13.13.11.0 0.0.0.255
 network 13.13.9.0  0.0.0.255
 network 150.1.13.0 0.0.0.255
!
```

### R2
```cisco
router eigrp 100
 no auto-summary
 passive-interface default
 no passive-interface serial 1/0.123
 network 13.13.2.0  0.0.0.255
 network 13.13.12.0 0.0.0.255
 network 13.13.9.0  0.0.0.255
!
interface serial 1/0.123
 no ip split-horizon eigrp 100
!
```

> ⚠️ **Frame-Relay Hub Router**에서는 `no ip split-horizon eigrp 100` 설정 필수
> (Spoke ↔ Spoke 간 라우팅 정보 전달을 위함)

### R3
```cisco
router eigrp 100
 no auto-summary
 passive-interface default
 no passive-interface serial 1/1
 no passive-interface serial 1/0.123
 no passive-interface fastethernet 0/0
 network 13.13.3.0  0.0.0.255
 network 13.13.10.0 0.0.0.255
 network 13.13.13.0 0.0.0.255
 network 13.13.9.0  0.0.0.255
 network 150.3.13.0 0.0.0.255
!
```

### R4
```cisco
router eigrp 100
 no auto-summary
 passive-interface default
 no passive-interface fastethernet 0/0
 network 13.13.4.0  0.0.0.255
 network 150.1.13.0 0.0.0.255
 network 13.13.14.0 0.0.0.255
!
```

### R5
```cisco
router eigrp 100
 no auto-summary
 passive-interface default
 no passive-interface fastethernet 0/0
 network 13.13.5.0  0.0.0.255
 network 150.3.13.0 0.0.0.255
 network 13.13.15.0 0.0.0.255
!
```

---

## 🔍 정보 확인

### Neighbor 인접성 확인
```cisco
R1# show ip eigrp neighbors   [인접성 3개 확인]
R2# show ip eigrp neighbors   [인접성 2개 확인]
R3# show ip eigrp neighbors   [인접성 3개 확인]
R4# show ip eigrp neighbors   [인접성 1개 확인]
R5# show ip eigrp neighbors   [인접성 1개 확인]
```

### 출력 예시
```
R1# show ip eigrp neighbors
IP-EIGRP neighbors for process 100
H   Address         Interface       Hold Uptime   SRTT   RTO  Q  Seq
0   150.1.13.4      Fa0/0             14 00:01:09  542  3252  0  20
1   13.13.10.3      Se1/1            154 00:01:21   11   200  0  78
2   13.13.9.2       Se1/0.123        154 00:01:33   43   258  0  37
```

### Routing Table 확인
```cisco
R1# show ip route eigrp
R2# show ip route eigrp
R3# show ip route eigrp
R4# show ip route eigrp
R5# show ip route eigrp
```
