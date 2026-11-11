# Lab 3: Deploying a 5G Standalone Network Using OpenAirInterface

## 1. Goal

In this lab, I deployed a 5G Standalone (SA) network using OpenAirInterface (OAI), verified the complete connection, investigated RFsimulator communication, performed active network measurements, and troubleshooted a controlled connection failure.

---

## 3. Task 1 — Deploy and Verify the Complete 5G SA Network

**Observations & Evidence**

| Stage | Successful? | Evidence from logs |
|---|---|---|
| CN5G initialization | Yes | `docker compose ps` showed all containers in `Up (healthy)` state. |
| gNB NG Setup | Yes | gNB logs indicated successful SCTP connection and NG Setup with AMF. |
| UE synchronization | Yes | UE logs showed `UE RNTI 49f4 CU-UE-ID 1 in-sync`. |
| RRC connection establishment | Yes | UE logs showed continuous data transmission (MAC TX/RX statistics). |
| UE Registration | Yes | Successful data flow and tunnel creation implied successful registration. |
| PDU Session Establishment | Yes | Data traffic flow confirmed PDU session. |
| UE tunnel creation | Yes | `oaitun_ue1` interface was created with IP address `10.0.0.2`. |

**Screenshots showing the gNB and UE successfully connected:**

<img width="2768" height="1072" alt="Screenshot 2026-11-11 091648" src="https://github.com/user-attachments/assets/61377a9a-b5d1-49c9-a630-04cf69bd2b4d" />
<img width="2280" height="1082" alt="Screenshot 2026-11-11 091654" src="https://github.com/user-attachments/assets/ad996b58-626d-4137-ab2f-cd744ce3f92b" />
Questions:

1.  **Does a successful NG Setup guarantee that a UE can register? Explain.**
    No. A successful NG Setup only establishes the N2 control plane connection between the gNB and the AMF in the 5G Core. It does not authenticate or register any specific UE. If a UE has an invalid IMSI or fails authentication, it will be rejected even if the NG Setup is perfectly healthy.
2.  **Why can the UE establish an RRC connection but still fail Registration?**
    RRC (Radio Resource Control) operates at Layer 3 of the Radio Access Network (between UE and gNB) and handles radio link establishment. Registration is a NAS (Non-Access Stratum) procedure between the UE and the AMF. The UE must first establish an RRC connection to send its Registration Request. The AMF can then reject the NAS Registration (e.g., due to authentication failure) over the established RRC link.
3.  **What additional procedure is required after Registration before user data can be exchanged?**
    After successful Registration, the UE must complete the **PDU Session Establishment** procedure. This creates a data tunnel (GTP-U) between the gNB and the UPF, allowing the exchange of user-plane IP packets.

---

## 4. Task 2 — Investigate RFsimulator Communication

**Observations**

| Item | Observed Value |
|---|---|
| gNB process ID | **91505** (and 91506) |
| nrUE process ID | **96325** (and 96326) |
| RFsimulator transport protocol | **TCP** |
| RFsimulator port | **4043** |
| TCP connection state | **ESTAB** |
| Connected process(es) | `nr-softmodem` and `nr-uesoftmodem` |

**Screenshot showing the TCP connection:**

<img width="2880" height="1242" alt="Screenshot 2026-11-11 092132" src="https://github.com/user-attachments/assets/9546f6cd-6b7d-43b1-a708-948fdbeb3648" />
Questions:

1.  **Why does RFsimulator use a TCP connection instead of a physical antenna?**
    RFsimulator replaces the physical RF frontend (like USRPs and antennas) to allow a purely software-based simulation on a single machine or network. It encapsulates the baseband IQ samples into TCP packets, ensuring reliable delivery between the software gNB and software UE without requiring physical radio propagation.
2.  **Are the TCP packets exchanged by RFsimulator the same as the UE's application-layer IP packets?**
    No. The TCP packets carry digitized radio waveforms (IQ samples). The UE's application-layer IP packets (like ICMP Ping requests) are encapsulated *inside* the 5G protocol stack (PDCP, RLC, MAC, PHY) before being converted into these IQ samples for transport via RFsimulator.
3.  **Does using RFsimulator mean that the UE and gNB are performing no physical-layer processing? Explain.**
    No. Both the UE and gNB still perform the full 5G NR physical layer (PHY) signal processing, including modulation, demodulation, channel coding, and OFDM symbol generation. The RFsimulator only replaces the digital-to-analog/analog-to-digital conversion and over-the-air transmission stages.

---

## 5. Task 3 — Verify End-to-End Connectivity

**Observations**

| Measurement | Observed Value |
|---|---|
| UE IP address | **10.0.0.2** (`oaitun_ue1`) |
| Destination IP | **192.168.70.135** |
| Packets transmitted | **10** |
| Packets received | **10** |
| Packet loss (%) | **0%** |
| Minimum RTT (ms) | **11.811 ms** |
| Average RTT (ms) | **17.827 ms** |
| Maximum RTT (ms) | **20.867 ms** |

**Screenshot of the Ping result:**

<img width="2880" height="1236" alt="Screenshot 2026-11-11 092226" src="https://github.com/user-attachments/assets/349f0399-aaf8-4377-aded-df28e6c6c70f" />

**Questions:**

1.  **Why must Ping use the `oaitun_ue1` interface in this experiment?**
    The `oaitun_ue1` interface represents the 5G tunnel (PDU session) connected to the UPF. Forcing Ping to use this interface ensures the ICMP traffic is routed through the complete 5G SA protocol stack rather than the host machine's default Ethernet or Wi-Fi network.
2.  **Does successful Registration guarantee successful user-plane connectivity?**
    No. Registration only authenticates the UE and establishes a signaling connection (Control Plane). User-plane connectivity requires the successful creation of a PDU Session (establishing GTP-U tunnels) and proper IP routing configuration.
3.  **What does a successful Ping prove, and what does it not prove about the simulated 5G system?**
    A successful Ping proves that the entire 5G software protocol stack (Control Plane and User Plane) is functioning correctly from end to end. It does not prove how the system would perform under real-world conditions like physical RF interference, mobility (handovers), fading, or hardware latency.

---

## 6. Task 4 — Investigate the Effect of Packet Size

**Measurements**

| Scenario | ICMP Payload | Avg. RTT (ms) | Max. RTT (ms) | Packet Loss (%) |
|---|---:|---:|---:|---:|
| A | 56 bytes | **18.281** | **23.920** | **0%** |
| B | 512 bytes | **19.771** | **23.802** | **0%** |
| C | 1200 bytes | **22.832** | **26.748** | **0%** |

*RTT Difference = Average RTT(C) - Average RTT(A) = 22.832 - 18.281 = **4.551 ms***

**Screenshots of Scenario A, B, and C:**

<img width="2254" height="1210" alt="Screenshot 2026-11-11 092353" src="https://github.com/user-attachments/assets/bf7b64a8-9947-4ce2-aac5-3f6b15ecc29e" />
<img width="2222" height="1276" alt="Screenshot 2026-11-11 092530" src="https://github.com/user-attachments/assets/1fc3b074-c3b0-4db7-964f-dc6ef86ade96" />
<img width="2262" height="1260" alt="Screenshot 2026-11-11 092610" src="https://github.com/user-attachments/assets/4b9ffe21-240f-44e8-9862-8bb43240c72e" />


**Questions:**

1.  **Which scenario has the highest average RTT?**
    Scenario C (1200 bytes) had the highest average RTT (**22.832 ms**).
2.  **Does increasing packet size always increase RTT in your measurements?**
    Yes, the average RTT increased sequentially from 56 bytes (18.281 ms) to 512 bytes (19.771 ms) to 1200 bytes (22.832 ms).
3.  **Why might a larger packet require more transmission or processing resources?**
    Larger packets may require fragmentation at the MAC/RLC layers, occupying more physical resource blocks (PRBs) over the simulated air interface. Furthermore, processing larger payloads takes more CPU cycles for encapsulation and decryption at both the UE and UPF.
4.  **What factors other than packet size could affect RTT in a software-based 5G network?**
    In a software-simulated environment, RTT is highly dependent on the host machine's CPU load, operating system thread scheduling, Docker network bridge overhead, and the latency introduced by the RFsimulator's TCP transport.
5.  **Can these results be interpreted as real over-the-air 5G latency? Explain.**
    No. These measurements represent the processing time of the OAI software stack and the host's OS networking on a single CPU. Over-the-air 5G latency would involve actual radio wave propagation, physical layer hardware delays, and varying RF channel conditions, which are completely abstracted away in RFsimulator.

---

## 7. Task 5 — Diagnose a Controlled UE Registration Failure

**Observations**

| Item | Observation |
|---|---|
| Modified parameter | Changed `--uicc0.imsi` to `001010000000099` |
| UE synchronized with gNB? | **Yes** |
| RRC connection established? | **Yes** |
| Registration successful? | **No** |
| Error or rejection evidence | AMF logs showed the UE status as `5GMM-DEREGISTERED` for IMSI `001010000000099`. |
| Likely cause | Authentication failure because IMSI `099` is not provisioned in the core network's subscriber database (UDR/MySQL). |

**Evidence of Registration Failure (AMF Log):**

<img width="2880" height="1276" alt="Screenshot 2026-11-11 092910" src="https://github.com/user-attachments/assets/0c610a43-47e2-4ed6-96ab-c98c568dccc4" />

**Questions:**

1.  **Why can the UE potentially synchronize with the gNB even when the IMSI is incorrect?**
    Synchronization (detecting SSB signals) is a purely physical-layer process that does not require any identity verification. It happens before any signaling messages are exchanged.
2.  **Which network function is involved in UE authentication?**
    The **AUSF** (Authentication Server Function) and the **UDM** (Unified Data Management), which cross-references the IMSI and secret keys stored in the UDR/MySQL database.
3.  **Why does changing an IMSI affect Registration but not the RFsimulator TCP connection itself?**
    The RFsimulator TCP connection only transports raw IQ samples representing the physical RF channel between the UE and gNB. The IMSI is an application-layer (NAS) identity evaluated much higher up in the protocol stack at the Core Network, so it has no bearing on the underlying physical simulation link.
4.  **What evidence proves that restoring the correct configuration solved the problem?**
    After restoring the IMSI to `001010000000001`, the AMF logged successful registration, the `oaitun_ue1` interface received an IP address again, and ICMP Ping packets to `192.168.70.135` were successfully transmitted and received.

---

## 8. Task 7 — Explain the Relationship Between Simulation and Real 5G

| Feature | OAI RFsimulator | Physical 5G System |
|---|---|---|
| gNB and UE protocol stack | Yes | Yes |
| OFDM signal processing | Yes | Yes |
| Physical antennas | No | Yes |
| RF hardware impairments | Not inherently | Yes |
| Wireless propagation effects | Not inherently | Yes |
| 5G Core connectivity | Yes | Yes |

**Questions:**

1.  **Which parts of the 5G system remain functional when physical RF hardware is replaced with RFsimulator?**
    The entire upper protocol stack (RRC, PDCP, RLC, MAC) and the core network functions (AMF, SMF, UPF, etc.) remain fully functional. Even the digital signal processing parts of the physical layer (PHY) operate identically.
2.  **Why is RFsimulator useful for testing the 5G protocol stack?**
    It isolates protocol-level bugs from physical hardware issues. Researchers can test signaling, core network integration, and high-layer behaviors in a controlled, low-cost environment without dealing with unpredictable RF interference or requiring expensive USRP hardware.
3.  **Why might latency and throughput differ between RFsimulator and a real 5G deployment?**
    RFsimulator relies on the host PC's CPU and TCP stack to transport baseband data, which can become a processing bottleneck, often limiting throughput and adding variable latency. Real 5G deployments use dedicated hardware accelerators (FPGAs/ASICs) that process radio signals with microsecond precision and massive parallelism.
4.  **What would need to change if you wanted to test a physical UE instead of OAI nrUE?**
    You would need to replace the RFsimulator configuration on the gNB side with real RF frontend hardware (like a USRP B210), attach physical antennas, and ensure the gNB software is configured to drive the USRP over a high-speed interface (e.g., USB 3.0 or 10GbE) instead of using the `--rfsim` flag.

---

## 9. Final Analysis

1.  **Describe the complete process from starting the OAI 5GC to successful user-plane connectivity.**
    First, the 5GC microservices initialize and establish internal communication via the NRF. The gNB starts and connects its control plane to the AMF (NG Setup) and user plane to the UPF. The nrUE synchronizes with the gNB, establishes an RRC connection, authenticates with the AMF using its IMSI, registers to the network, and requests a PDU session. Finally, a GTP-U tunnel is created, granting the UE an IP address (`oaitun_ue1`) to route user data.
2.  **Explain the difference between an RFsimulator TCP connection and a 5G PDU Session.**
    The RFsimulator TCP connection is a low-level, simulation-specific transport mechanism that carries baseband IQ samples between the software UE and gNB, replacing physical radio waves. A 5G PDU Session is a standardized network tunnel established between the UE and the UPF in the core network to transport actual user application data (like IP packets) across the 5G infrastructure.
3.  **Based on your measurements, how did packet size affect RTT and packet loss?**
    In my measurements, increasing the ICMP payload size resulted in a noticeable increase in Average RTT, rising from 18.281 ms for 56 bytes to 22.832 ms for 1200 bytes (a difference of 4.551 ms). However, packet size had no impact on reliability, as all tests consistently showed 0% packet loss in the stable simulated environment.
4.  **Describe the registration failure you introduced and explain how you diagnosed and recovered from it.**
    I introduced a failure by modifying the UE's IMSI to `...099`, an identity not provisioned in the core network's database. I diagnosed this by observing the AMF logs, which clearly showed the UE state as `5GMM-DEREGISTERED`, confirming an authentication rejection. Recovery was achieved by restoring the IMSI to the provisioned value (`...001`) and restarting the nrUE, which restored end-to-end connectivity.
5.  **Identify two limitations of this simulated environment and explain how they might affect experimental results.**
    One major limitation is the lack of physical RF channel impairments (like fading, multipath, and interference), which means performance results will appear artificially perfect and not reflect real-world coverage challenges. Another limitation is that all nodes (Core, gNB, UE) share the same host CPU and OS scheduler, meaning that high processing loads can introduce variable software-induced latency (jitter) that doesn't exist in distributed, hardware-accelerated commercial networks.
