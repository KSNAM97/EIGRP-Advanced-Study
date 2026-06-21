# Chapter 06. EIGRP SIA (Stuck In Active)

## 📌 개요

**SIA**는 EIGRP가 장애 경로에 대한 새로운 경로를 계산하는 과정에서,
일정 시간 동안 Query에 대한 Reply를 받지 못해 **경로 계산이 완료되지 않는 현상**이다.

---

## 📊 EIGRP 경로 상태

| 상태 | 설명 |
|------|------|
| **Passive** | 경로가 정상적으로 계산된 안정적인 상태 (정상 상태) |
| **Active** | Successor 경로 장애 + Feasible Successor 없음 → Query 전송 중 |

---

## 🔄 SIA 발생 과정

1. EIGRP가 사용 중인 **Successor 경로의 장애를 감지**
2. Topology Table에 즉시 사용 가능한 **Feasible Successor가 없으면** Active 상태로 전환
3. Neighbor Router들에게 **Query 메시지 전송**
4. Query를 받은 Neighbor도 대체 경로가 없으면 다른 Neighbor에게 Query 전달
5. 최초 Query를 전송한 Router는 **모든 Neighbor로부터 Reply**를 받아야 경로 계산 완료
6. 기본 Active Timer인 **180초** 동안 Reply 미수신 시 **SIA로 판단**
7. Reply 미전송 Neighbor와의 **인접성 해제**, 해당 경로 삭제

---

## ⚠️ SIA 발생 주요 원인

- 저속 WAN 구간 또는 심각한 네트워크 혼잡
- 패킷 손실이 많은 불안정한 링크
- Router의 CPU 또는 메모리 사용량 증가
- Query가 너무 많은 Router로 확산되는 대규모 EIGRP 네트워크

> 📌 SIA는 반드시 장애 링크에서만 발생하는 것이 아니다.
> Query가 네트워크 전체로 확산되면 장애와 무관한 저속/과부하 Router가
> Reply를 늦게 전송하여 인접성이 단절될 수 있다.
> 따라서 SIA는 단순 회선 장애라기보다 **Query 범위가 과도하게 커졌을 때 발생하는 수렴 실패** 문제다.

---

## 🧪 SIA 재현 실습

### 토폴로지
```
13.13.5.0/24
Loopback 0 (Down 예정)
     |                                                                  150.1.13.0/24
     R5 ── Fa0/0 ── R3 ── Serial1/1 ── R1 ── Fa0/0 ── R4
                      13.1                  13.4

       ── Query ──>     ── Query ──>     ── Query ──>
                                          <── Reply ──
```

### R1 (Reply 차단 ACL 설정)
```cisco
access-list 101 permit eigrp host 150.1.13.4  host 224.0.0.10
!
interface fastethernet 0/0
 ip access-group 101 in
!
router eigrp 100
 timers active-time 1   ← SIA 빠른 확인 (Default 180초 → 1분)
!
```

### R5 (장애 유발)
```cisco
interface loopback 0
 shutdown
!
```

### SIA 발생 로그
```
*Mar  1 00:24:02.515: %DUAL-3-SIA: Route 13.13.5.0/24 stuck-in-active state in IP-EIGRP(0) 100.  Cleaning up
*Mar  1 00:24:02.523: %DUAL-5-NBRCHANGE: IP-EIGRP(0) 100: Neighbor 150.1.13.4 (FastEthernet0/0) is down: stuck in active
```

---

## 🛠️ SIA 해결 방법

### 1) Timer 조정
저속 WAN 구간처럼 정상적인 Query/Reply 처리에 시간이 오래 걸리는 환경에서 Active Timer 증가.

```cisco
router eigrp 100
 timers active-time [분]
!
```

### 2) EIGRP Stub
- Hub-and-Spoke 환경의 **Spoke Router에 설정**
- Stub Router는 자신이 **Transit Router가 아님**을 Neighbor에게 알림
- Hub Router는 Stub Router에게 **Query를 전송하지 않음**

```cisco
router eigrp 100
 eigrp stub
!
```

### 3) IP 주소 요약을 통한 SIA 해결
- EIGRP는 Topology Table에 없는 네트워크 정보를 라우팅 할 수 없음
- 주소 요약 설정 시 장애 네트워크 상세 정보가 Topology에 없으므로
  **Query를 확산하지 않고 즉시 Reply 송신**

```
        S1/0                   S1/1  S1/0                   S1/1  S1/0                   S1/1
   R1 ──────────────────── R2 ──────────────────── R3 ──────────────────── R4
   C  172.16.0.0/24        D  172.16.0.0/22         D  172.16.0.0/22         D  172.16.0.0/22
   C  172.16.1.0/24
   C  172.16.2.0/24 (Down)
   C  172.16.3.0/24
   D  172.16.0.0/22 Null0

                ── Query ──>
                172.16.2.0/24
                <── Reply ──   (요약 라우터가 즉시 응답)
                172.16.2.0/24
```

---

## 📚 정리

| 해결 방법 | 적용 위치 | 효과 |
|----------|----------|------|
| Active Timer 조정 | 전체 라우터 | Reply 대기 시간 증가 |
| EIGRP Stub | Spoke Router | Query 전파 방지 |
| 주소 요약 | 요약 Router | Query 확산 범위 축소 |

> 💡 **Best Practice**: 대규모 EIGRP 네트워크에서는 **주소 요약 + Stub 조합**으로
> Query 범위를 효과적으로 제어하는 것이 SIA 예방의 핵심이다.
