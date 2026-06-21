# Chapter 04. EIGRP Offset-list

## 📌 개요

- Routing Table에 등록된 특정 네트워크의 **Metric 값을 조정**하여 최적 경로를 변경하는 기능
- Offset-list는 **Metric 값의 증가만 가능** (감소 불가)
- 해당 Process 내에서 **In/Out 각 1개씩**만 적용 가능

### 설정 방법
1. ACL을 사용하여 Metric 값을 변경할 네트워크의 범위를 지정
2. EIGRP Process에서 ACL number를 지정하여 해당 네트워크에 Metric 값 변경

---

## EX1) R5 → R4 통신 시 R2 경유

### 조건
- R5의 `13.13.15.0/24` 네트워크가 R4의 `13.13.14.0/24` 네트워크로 통신 시 **R2 경유**

### 방법 1) R3에서 In 방향 적용
```cisco
# R3
access-list 1 permit 13.13.14.0  0.0.0.255
!
router eigrp 100
 offset-list 1 in 600000 serial 1/1
!
```

### 방법 2) R1에서 Out 방향 적용
```cisco
# R1
access-list 1 permit 13.13.14.0  0.0.0.255
!
router eigrp 100
 offset-list 1 out 600000 serial 1/1
!
```

### 정보 확인 (적용 전/후 비교)
```
[적용 전]
R3# show ip route eigrp
D    13.13.14.0/24 [90/2221056] via 13.13.10.1, 00:16:16, Serial1/1   ← Serial1/1

[적용 후]
R3# show ip route eigrp
D    13.13.14.0/24 [90/2733056] via 13.13.9.2, 00:00:03, Serial1/0.123 ← Serial1/0.123 (R2 경유)
```

### Traceroute 확인
```
[적용 전]
R5# traceroute 13.13.14.4 source 13.13.15.5
  1 150.3.13.3       R3 ← R5
  2 13.13.10.1       R1 ← R3 ← R5
  3 150.1.13.4       R4 ← R1 ← R3 ← R5

[적용 후]
R5# traceroute 13.13.14.4 source 13.13.15.5
  1 150.3.13.3       R3 ← R5
  2 13.13.9.2        R2 ← R3 ← R5         ← R2 경유 확인
  3 13.13.9.1        R1 ← R2 ← R3 ← R5
  4 150.1.13.4       R4 ← R1 ← R2 ← R3 ← R5
```

---

## EX2) R4 ↔ R5 통신 시 R2 경유 (다중 네트워크)

### 조건
- R4의 `13.13.4.0/24` ↔ R5의 `13.13.5.0/24` 통신 시 R2 경유
- R4의 `150.1.13.0/24` ↔ R5의 `150.3.13.0/24` 통신 시 R2 경유

### 방법 1) R1에서 In 방향 적용
```cisco
# R1
access-list 10 permit 13.13.5.0   0.0.0.255
access-list 10 permit 150.3.13.0  0.0.0.255
!
router eigrp 100
 offset-list 10 in 600000 serial 1/1
!
```

### 방법 2) R3에서 Out 방향 적용
```cisco
# R3
access-list 10 permit 13.13.5.0   0.0.0.255
access-list 10 permit 150.3.13.0  0.0.0.255
!
router eigrp 100
 offset-list 10 out 600000 serial 1/1
!
```

### 정보 확인
```
[적용 전]
R1# show ip route eigrp | include 13.13.5.0
D    13.13.5.0  [90/2323456] via 13.13.10.3, 02:19:01, Serial1/1

R1# show ip route eigrp | include 150.3.13.0
D    150.3.13.0 [90/2195456] via 13.13.10.3, 02:19:06, Serial1/1

[적용 후]
R1# show ip route eigrp | include 13.13.5.0
D    13.13.5.0  [90/2835456] via 13.13.9.2, 00:00:02, Serial1/0.123

R1# show ip route eigrp | include 150.3.13.0
D    150.3.13.0 [90/2707456] via 13.13.9.2, 00:00:04, Serial1/0.123
```

---

## 🔄 설정 삭제
```cisco
# R1
no access-list 10
!
router eigrp 100
 no offset-list 10 out 600000 serial 1/1
!

# 또는 R3
no access-list 10
!
router eigrp 100
 no offset-list 10 in 600000 serial 1/1
!
```
