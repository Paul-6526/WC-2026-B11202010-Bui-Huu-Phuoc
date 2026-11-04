
# Lab 2: gNB TDD Traffic Analysis
**Student Name:** Bui Huu Phuoc 
**Student ID:** B11202010

---

## Task 1 — Confirm the Traffic Direction

**Direction Table:**

| Scenario | Reverse value | Direction |
| :--- | :--- | :--- |
| **A** | 1 | Downlink |
| **B** | 0 | Uplink |
| **C** | 0 | Uplink |

**Question: Does the traffic direction shown in the JSON match the scenario description? Show the proof.**
**Answer:** Yes, it matches. Scenario A is a Downlink test (`reverse: 1`), while Scenarios B and C are Uplink tests (`reverse: 0`). The screenshots below serve as proof.

*Proof Screenshots:*
*   **Scenario A:** <img width="2482" height="1264" alt="Screenshot 2026-10-07 070225" src="https://github.com/user-attachments/assets/db3d613d-5787-4cf1-a6ef-4f707a44551e" />
*   **Scenario B:** <img width="2542" height="1286" alt="Screenshot 2026-10-07 070250" src="https://github.com/user-attachments/assets/aa32b6f9-e6e3-4479-b1ee-a87ec27fb81a" />
*   **Scenario C:** <img width="2630" height="1252" alt="Screenshot 2026-10-07 070316" src="https://github.com/user-attachments/assets/21406d6b-26f3-47bf-b818-f6e356e3042c" />

---

## Task 2 — Compare Throughput

**Throughput Table:**

| Scenario | Direction | Throughput (Mbps) |
| :--- | :--- | :--- |
| **A** | Downlink | 90.65 |
| **B** | Uplink | 109.12 |
| **C** | Uplink | 124.42 |

*Proof Screenshots (sum -> bits_per_second):*
*   **Scenario A:** <img width="1390" height="1478" alt="Screenshot 2026-10-07 070609" src="https://github.com/user-attachments/assets/affa803b-7855-4075-8602-1c8bfbb2e1f4" />
*   **Scenario B:** <img width="1486" height="1502" alt="Screenshot 2026-10-07 070631" src="https://github.com/user-attachments/assets/39e6162a-95e5-490c-b4fd-c89f098a82c3" />
*   **Scenario C:**<img width="1446" height="1490" alt="Screenshot 2026-10-07 070649" src="https://github.com/user-attachments/assets/6dbce441-6bce-44ca-a810-1a04c5ed3936" />

**Answers:**
*   **What is the downlink throughput in Scenario A?** 90.65 Mbps.
*   **What is the uplink throughput in Scenario B?** 109.12 Mbps.
*   **Is the uplink throughput higher in B or C?** It is higher in Scenario C.
*   **By approximately what percentage does C improve over B?** 
    Improvement (%) = (124.42 - 109.12) ÷ 109.12 × 100 ≈ **14.02%**

---

## Task 3 — Check the Trade-off

**Packet Loss and Jitter Table:**

| Scenario | Throughput (Mbps) | Packet loss (%) | Jitter (ms) |
| :--- | :--- | :--- | :--- |
| **B** | 109.12 | 0.24% | 0.116 |
| **C** | 124.42 | 2.13% | 0.162 |

*Proof Screenshots (sum -> lost_percent and jitter_ms):*
*   **Scenario B:**<img width="1408" height="1484" alt="Screenshot 2026-10-07 070909" src="https://github.com/user-attachments/assets/3a8392a2-7191-4add-995e-f5c5f5cc153f" />
<img width="1428" height="1472" alt="Screenshot 2026-10-07 070919" src="https://github.com/user-attachments/assets/0ffe25c9-1d87-4934-818b-2305c22b53f8" />

*   **Scenario C:** <img width="1422" height="1494" alt="Screenshot 2026-10-07 070929" src="https://github.com/user-attachments/assets/214e223b-55aa-40ff-b3cc-e216c8e0c86b" />
<img width="1440" height="1462" alt="Screenshot 2026-10-07 070941" src="https://github.com/user-attachments/assets/fbac3a7d-038d-4880-add8-d3ea71999fe5" />


**Answers:**
*   **Does C have higher uplink throughput than B?** Yes.
*   **Does C have lower packet loss than B?** No, C has higher packet loss (2.13% vs 0.24%).
*   **What is the trade-off when changing from B to C?**
    Scenario C has **higher** throughput, but **higher** packet loss than Scenario B. Therefore, the balanced TDD setting improves **uplink throughput**, but it may reduce **connection reliability**.

---

## Task 4 — Read SIB1 and Draw the TDD Configuration

### 4.3 Record the TDD Parameters
**TDD Parameters (pattern1):**

| Item | DL-heavy: A/B | Balanced: C |
| :--- | :--- | :--- |
| **Configuration capture filename** | tdd-dl-heavy-sib1.pcapng | tdd-balanced-sib1.pcapng |
| **SIB1 packet number** | 2 | 2 |
| **referenceSubcarrierSpacing (kHz)** | 30 kHz | 30 kHz |
| **dl-UL-TransmissionPeriodicity (ms)** | 5 ms | 2.5 ms |
| **nrofDownlinkSlots** | 7 | 2 |
| **nrofDownlinkSymbols** | 6 | 6 |
| **nrofUplinkSlots** | 2 | 2 |
| **nrofUplinkSymbols** | 4 | 4 |
| **Is pattern2 present?** | No | No |

*Proof Screenshots (SIB1 tdd-UL-DL-ConfigurationCommon):*
*   **DL-heavy (A/B):** <img width="2880" height="1614" alt="Screenshot 2026-11-04 093955" src="https://github.com/user-attachments/assets/efb3a0ab-1646-4e30-a0ab-cdbf02abcee3" />
*   **Balanced (C):** <img width="2880" height="1628" alt="Screenshot 2026-11-04 094152" src="https://github.com/user-attachments/assets/b64610f0-f70f-4cb8-adda-dc873dd2cd23" />
### 4.4 Draw the TDD Time Allocation

The tables below illustrate the TDD time allocation derived from SIB1. The reference subcarrier spacing (SCS) is 30 kHz, meaning each slot duration is 0.5 ms and contains 14 OFDM symbols. 
*Note: **D** = Downlink, **U** = Uplink, **F** = Flexible.*

**1. DL-heavy Configuration (Scenarios A & B)**
*Repeating duration: 5 ms (10 slots)*
<img width="2574" height="635" alt="Screenshot_20261104_103958_Samsung Notes" src="https://github.com/user-attachments/assets/c9048c77-0b09-4b40-a6d4-ed73b351057e" />

**2. Balanced Configuration (Scenario C)**
*Repeating duration: 2.5 ms (5 slots). To fill the common 5 ms axis, the pattern repeats twice.*
<img width="2554" height="667" alt="Screenshot_20261104_104541_Samsung Notes" src="https://github.com/user-attachments/assets/d554bdce-04e3-4ad6-918e-a09f00044fde" />

**Per-sequence Configuration Table:**

| TDD setting | Repeating duration (ms) | Total reference slots | DL symbols | UL symbols | Flexible symbols | Configured UL symbol share (%) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **DL-heavy** | 5 | 10 | 104 | 32 | 4 | 22.86% |
| **Balanced** | 2.5 | 5 | 34 | 32 | 4 | 45.71% |

**Common 5 ms Comparison Table:**

| TDD setting | Repeating duration (ms) | Repetitions in 5 ms | DL symbols in 5 ms | UL symbols in 5 ms | Flexible symbols in 5 ms |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **DL-heavy** | 5 | 1 | 104 | 32 | 4 |
| **Balanced** | 2.5 | 2 | 68 | 64 | 8 |

**Question: Can two patterns have the same UL symbol count per period but provide different numbers of UL symbols per unit time? Explain using your two tables.**
**Answer:** Yes. Both patterns have exactly 32 UL symbols per repeating period. However, because the Balanced setting repeats every 2.5 ms (twice within a 5 ms interval), it provides a total of 64 UL symbols per 5 ms. In contrast, the DL-heavy setting repeats only once every 5 ms, providing just 32 UL symbols in the same timeframe.

### 4.5 Connect SIB1 to the Performance Results

| Scenario | Reference TDD setting | JSON traffic direction | Configured UL symbol share (%) | Throughput (Mbps) | Packet loss (%) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **A** | DL-heavy | Downlink | 22.86% | 90.65 | - |
| **B** | DL-heavy | Uplink | 22.86% | 109.12 | 0.24% |
| **C** | Balanced | Uplink | 45.71% | 124.42 | 2.13% |

**Answers:**
1.  **Which decoded TDD parameters differ between DL-heavy and balanced?**
    The `dl-UL-TransmissionPeriodicity` (5 ms vs. 2.5 ms) and `nrofDownlinkSlots` (7 vs. 2) differ between the configurations.
2.  **Which configuration provides more configured UL symbols per unit time?**
    The Balanced configuration provides more UL symbols per unit time (64 UL symbols per 5 ms compared to 32 UL symbols for DL-heavy), as demonstrated by its higher UL symbol share (45.71% vs 22.86%).
3.  **How does that difference help explain the B/C throughput comparison?**
    More UL symbols provide the UE with more transmission opportunities (physical time resources) within the same timeframe, which explains why the uplink throughput in Scenario C is significantly higher than in Scenario B.
4.  **Why can A and B use the same SIB1 TDD configuration even though their traffic directions differ?**
    SIB1 broadcasts the common TDD slot structure for the entire cell, which dictates the fixed time-domain resources for all UEs simultaneously, regardless of whether an individual UE is currently performing uplink or downlink operations.

---

## Task 5 — Is the TDD Setting Suitable?

**TDD Suitability Table:**

| Scenario | TDD setting | Traffic direction | Is the setting suitable for this direction? Why? |
| :--- | :--- | :--- | :--- |
| **A** | DL-heavy | Downlink | Yes. More DL resources provide more downlink transmission opportunities, making it highly suitable. |
| **B** | DL-heavy | Uplink | No. A DL-heavy setting limits heavy uplink traffic because there are very few UL resources available. |
| **C** | Balanced | Uplink | Yes. A balanced setting gives uplink traffic more transmission opportunities than the DL-heavy setting in this lab. |

**Question: Why might Scenario C achieve better uplink throughput than Scenario B? Refer to your SIB1 diagrams and measured results.**
**Answer:** Based on the SIB1 diagrams, Scenario C's "Balanced" setting allocates 64 UL symbols every 5 ms, whereas Scenario B's "DL-heavy" setting only allocates 32 UL symbols. This doubling of uplink transmission opportunities directly allows Scenario C to achieve a higher measured uplink throughput (124.42 Mbps) compared to Scenario B (109.12 Mbps).

---

## Task 6 — Find GTP-U Information

**GTP-U Information Table:**

| Item | Value |
| :--- | :--- |
| **Packet number** | 1831 |
| **Outer source IP** | 192.168.70.129 |
| **Outer destination IP** | 192.168.70.134 |
| **Inner source IP** | 10.0.0.6 |
| **Inner destination IP** | 192.168.70.135 |
| **TEID** | 0x00000011 (17) |

*Proof Screenshot:*
<img width="2880" height="1488" alt="Screenshot 2026-10-07 072051" src="https://github.com/user-attachments/assets/59955341-5781-4e02-8e29-f53bde4ea074" />

<img width="2880" height="1494" alt="Screenshot 2026-10-07 072040" src="https://github.com/user-attachments/assets/f9498f7f-42c8-4c38-990d-ff66cac3b42a" />

---

## Task 7 — TCP Retransmissions

**TCP Retransmissions Table:**

| Scenario | Displayed TCP retransmissions |
| :--- | :--- |
| **A** | 14 |
| **B** | 12 |
| **C** | 12 |

*Proof Screenshots:*
*   **Scenario A:** <img width="2880" height="1712" alt="Screenshot 2026-10-07 072721" src="https://github.com/user-attachments/assets/6db21a8f-6b7d-44b2-8a27-57c6e76ea4a8" />
*   **Scenario B:** <img width="2880" height="1710" alt="Screenshot 2026-10-07 072740" src="https://github.com/user-attachments/assets/fc8b68dd-5053-4f12-8345-deac44047234" />
*   **Scenario C:** <img width="2880" height="1714" alt="Screenshot 2026-10-07 072814" src="https://github.com/user-attachments/assets/4c3b0857-cbb3-4903-9ac7-f18ceda72702" />

**Question: Do these TCP retransmissions directly represent lost UDP performance packets?**
**Answer:** No. They belong to the TCP control connection used by iperf3 for signaling. The actual throughput test uses UDP, so these TCP retransmissions do not directly represent the lost UDP performance packets.

---

## Task 8 — RTT and Packet Interval

**TCP RTT Example:**

| Scenario | Example TCP ACK RTT |
| :--- | :--- |
| **A** | 485.000 microseconds |

*Proof Screenshot:* <img width="2880" height="1416" alt="Screenshot 2026-10-07 073120" src="https://github.com/user-attachments/assets/24f61f59-0fc8-4fa5-bb58-e8a8a680910f" />

**Packet Interval Example:**

| Flow | Packet numbers | Time interval |
| :--- | :--- | :--- |
| **UDP** | 1825 - 1826 | 49.905 milliseconds |

*Proof Screenshot:* <img width="2880" height="1282" alt="Screenshot 2026-10-07 073258" src="https://github.com/user-attachments/assets/27853a17-0929-455a-b6dc-30680ad11c24" />
<img width="2880" height="1486" alt="Screenshot 2026-10-07 073313" src="https://github.com/user-attachments/assets/22868a8b-78a0-4437-a502-41a3f832ddeb" />


---

## Final Questions

1.  **Does Scenario A use a TDD setting that matches its traffic direction?**
    Yes, Scenario A is a downlink test and uses a "DL-heavy" TDD setting. This matching configuration provides ample transmission opportunities for the downlink traffic.
2.  **Why is Scenario B expected to have limited uplink resources?**
    Scenario B is an uplink test but operates under a "DL-heavy" TDD configuration. This means the majority of slots are reserved for downlink, leaving very few slots available for the UE to transmit uplink data.
3.  **How much does uplink throughput change from B to C?**
    The uplink throughput improves significantly by approximately **14.02%** (from ~109.12 Mbps in Scenario B to ~124.42 Mbps in Scenario C).
4.  **What negative result is visible in C?**
    While throughput increased, Scenario C experienced a much higher **packet loss rate (2.13%)** compared to Scenario B (0.24%), along with a slight increase in jitter.
5.  **Why can increasing UL slots improve uplink throughput?**
    Increasing UL slots provides the UE with more physical time resources (transmission opportunities) within each radio frame to send data, directly translating to a higher achievable data rate.
6.  **Is adding more UL slots always the best solution? Explain using throughput and packet loss.**
    No. As seen in Scenario C, while adding UL slots (Balanced setting) increased the throughput, it also caused a severe spike in packet loss. If packet loss becomes too high, the overall user experience will degrade significantly despite the higher raw bandwidth.
7.  **Suggest one simple follow-up test that could improve throughput without producing too much packet loss.**
    A follow-up test could involve tuning the UDP sending rate (bandwidth parameter) of iperf3 to closely match the actual capacity of the "Balanced" TDD link, preventing buffer overflows at the gNB that lead to packet loss.
8.  **What can the configuration captures tell you that the iperf3 JSON cannot? Why are separately recorded configuration captures not direct proof of the configuration during an earlier test?**
    Configuration captures reveal the actual physical layer TDD slot structure (DL/UL ratio) broadcasted by the gNB via SIB1, whereas JSON only shows application-layer throughput. Separately recorded captures are not direct proof because network parameters could theoretically have been modified between the time the SIB1 was recorded and when the iperf3 test was executed.
