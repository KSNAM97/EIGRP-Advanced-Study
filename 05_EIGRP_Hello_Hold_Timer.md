# Chapter 05. EIGRP Hello / Hold Interval 변경

## 📌 개요

### Hello-Interval
- Hello Packet을 교환하는 주기

### Hold-Time
- 정해진 Hold Interval 안에 Hello를 수신하지 못하면 EIGRP 인접성 단절

---

## 📊 기본 Timer 값

| 구간 종류 | Hello-Interval | Hold-Interval |
|----------|---------------|---------------|
| LAN (Ethernet, FastEthernet) | 5초 | 15초 |
| WAN (PPP, HDLC, F/R P2P) | 5초 | 15초 |
| WAN (F/R Multipoint, NBMA) | **60초** | **180초** |

---

## ⚙️ 설정 방법

```cisco
RTX(config)# interface [Serial | Fastethernet] [Port Number]
RTX(config-if)# ip hello-interval eigrp [AS Number] [Sec]
RTX(config-if)# ip hold-time eigrp [AS Number] [Sec]
```

---

## EX5) Frame-Relay 구간 Hello/Hold Timer 변경

### 조건
- R1, R2, R3 Frame-Relay 구간에서 Hello는 **5초**마다 교환, **15초** 안에 미수신 시 인접성 단절
- R1, R3 Frame-Relay 구간에서 Hello는 **5초**마다 교환, **15초** 안에 미수신 시 인접성 단절

### R1, R3 설정
```cisco
interface serial 1/0.123
 ip hello-interval eigrp 100 5
 ip hold-time eigrp 100 15
!
interface serial 1/1
 ip hello-interval eigrp 100 5
 ip hold-time eigrp 100 15
!
```

### R2 설정
```cisco
interface serial 1/0.123
 ip hello-interval eigrp 100 5
 ip hold-time eigrp 100 15
!
```

---

## 🔍 정보 확인

```
R1# show ip eigrp interfaces detail
IP-EIGRP interfaces for process 100

                  Xmit Queue   Mean   Pacing Time   Multicast    Pending
Interface  Peers  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
Se1/1        1        0/0        37       0/15         151           0
  Hello interval is 5 sec                                  ← 변경 확인
  Use unicast
Se1/0.123    1        0/0        39       0/15         167           0
  Hello interval is 5 sec                                  ← 변경 확인
  Use unicast
Fa0/0        1        0/0        47       0/1          188           0
  Hello interval is 5 sec
  Use multicast
```

> ⚠️ **주의**: Hello/Hold Timer는 Neighbor 간 일치할 필요는 없으나,
> 한쪽이 너무 짧으면 인접성이 불안정해질 수 있다.
