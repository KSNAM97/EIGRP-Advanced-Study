![Topic](https://img.shields.io/badge/Topic-Route_Filtering-blue)
![Method](https://img.shields.io/badge/Method-Prefix--list%20%7C%20Route--map-orange)
![Vendor](https://img.shields.io/badge/Vendor-Cisco-1BA0D7?logo=cisco&logoColor=white)
![Tool](https://img.shields.io/badge/Tool-GNS3-2ECC71)
![Level](https://img.shields.io/badge/Level-Advanced-red)


# EIGRP-Advanced-Study

> Cisco EIGRP 심화 개념 학습 정리 - Offset-list, Summary, SIA, Hello/Hold Timer

---

## 📖 학습 목표

본 저장소는 Cisco EIGRP(Enhanced Interior Gateway Routing Protocol)의 심화 개념을 다룹니다.
기본 EIGRP 구성을 마친 후 실무에서 자주 사용되는 경로 제어 기법과 최적화 기법을 학습합니다.

---

## 📂 Chapter

| Chapter | Title | Description |
|---------|-------|-------------|
| [Chapter 01](./01_EIGRP_Basic_Config.md) | EIGRP 기본 구성 | EIGRP AS 100, Passive-interface, Network 광고 |
| [Chapter 02](./02_EIGRP_Summary.md) | EIGRP 주소 요약 (Summary) | Manual Summarization, CIDR, 메모리 최적화 |
| [Chapter 03](./03_EIGRP_Default_Route.md) | EIGRP Default-Route | Summary-address 0.0.0.0 활용 |
| [Chapter 04](./04_EIGRP_Offset_List.md) | EIGRP Offset-list | Metric 조정을 통한 경로 제어 |
| [Chapter 05](./05_EIGRP_Hello_Hold_Timer.md) | EIGRP Hello/Hold Timer | Neighbor 유지 주기 변경 |
| [Chapter 06](./06_EIGRP_SIA.md) | EIGRP SIA (Stuck In Active) | SIA 발생 원인 및 해결 방법 |

---

## 🖧 Network Topology
![EIGRP_Topology](./topology/eigrp_topology.png)

---

## ⚙️ 사전 요구사항

- Cisco IOS 기본 명령어 이해
- Frame-Relay 기본 설정 완료 ([Frame-Relay-Basic-Study](https://github.com/KSNAM97/Frame-Relay-Basic-Study) 참고)
- EIGRP 기본 동작 원리 이해 (DUAL, Successor, Feasible Successor)

---

## 🔗 관련 저장소

- [Frame-Relay-Basic-Study](https://github.com/KSNAM97/Frame-Relay-Basic-Study)

---

## 📝 License

본 자료는 학습 목적으로 작성되었습니다.
