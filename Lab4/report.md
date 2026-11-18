# Lab 4 — 5G CU/DU Split Deployment and F1-C/F1-U Analysis

## 1. Goal
The objective of this lab is to deploy a 5G Standalone (SA) network by replacing the monolithic gNB with a split architecture consisting of a Central Unit (gNB-CU) and a Distributed Unit (gNB-DU). We capture and analyze the F1-C (control plane) and F1-U (user plane) traffic using Wireshark to understand the functional division in an O-RAN architecture, and finally verify end-to-end user connectivity.

---

## 3. Task 1 — Inspect the split configurations

**Observations:**
Based on the provided configuration files, the setup parameters are as follows:
*   **CU F1 address:** `127.0.0.3`
*   **DU F1 address:** `127.0.0.4`
*   **DU SSB NR-ARFCN (absoluteFrequencySSB):** `630048` (3450.72 MHz)
*   **Bandwidth:** `106` PRBs

**Evidence (Configuration Check):**
<img width="2880" height="1452" alt="Screenshot 2026-11-18 085622" src="https://github.com/user-attachments/assets/6d1af268-3eb3-49c2-9f3e-f3629c991bcb" />

**Questions & Answers:**

1.  **Which protocol layers mainly reside in the CU, and which reside in the DU?**
    The CU (Central Unit) mainly hosts the higher-layer protocols: RRC, SDAP, and PDCP. The DU (Distributed Unit) handles the lower-layer, real-time protocols: RLC, MAC, and High-PHY.
2.  **What is the purpose of the F1 interface?**
    The F1 interface connects the CU and the DU. It allows the separated functional blocks of the radio access network to communicate, carrying both control signaling (F1-C) and user data (F1-U).
3.  **Why must the UE radio frequency be compatible with the DU rather than taken blindly from the previous lab?**
    In a split architecture, the DU is the entity physically transmitting the radio signal (or simulating it via RFsimulator). Therefore, the UE must tune its frequency to match the DU's `absoluteFrequencySSB` (3450.72 MHz). If we reused Lab 3's monolithic frequency (3619.20 MHz), the UE would fail to synchronize because it would be listening on the wrong channel.

---

## 4. Task 2 — Deploy the split 5G SA network

**Evidence of Deployment:**

*   **gNB-CU Startup and AMF Connection:**
    <img width="2880" height="1446" alt="Screenshot 2026-11-18 090829" src="https://github.com/user-attachments/assets/b6305f7b-0d86-4d44-9f27-b6837d4fd662" />
*   **gNB-DU Startup and RFsimulator MAC Stats:**
    <img width="2880" height="1450" alt="Screenshot 2026-11-18 090852" src="https://github.com/user-attachments/assets/c1d57023-45f6-4b7c-bac0-7d438621ff31" />
*   **nrUE Connected (MAC Logs at DU side):**
   <img width="2880" height="1438" alt="Screenshot 2026-11-18 091226" src="https://github.com/user-attachments/assets/50730a2a-8729-4203-9cfa-50e2b82a7375" />
   
*   **Successful Ping & UE Tunnel Check (0% packet loss):**
    <img width="2880" height="1454" alt="Screenshot 2026-11-18 091751" src="https://github.com/user-attachments/assets/ca7a0cbb-6e6c-4740-9e07-4e9c774a1511" />

**Observations:**
*   **UE IP:** `10.0.0.2` (`oaitun_ue1`)
*   **Transmitted/Received packets:** `30 / 30`
*   **Packet loss:** `0% packet loss`
*   **Average RTT:** `16.379 ms`

---

## 5. Task 3 — Capture F1-C and F1-U in a single PCAP

The PCAP file was successfully captured on the loopback interface (`lo`) of the Azure VM. Since the Azure Cloud Shell GUI download feature encountered issues, the file `lab4-f1-complete.pcap` was securely downloaded to the local Windows machine using the Command Prompt (`scp` command) for Wireshark analysis.

**Evidence (SCP Download via CMD):**
<img width="2880" height="1700" alt="Screenshot 2026-11-18 093255" src="https://github.com/user-attachments/assets/ea8123a3-c700-4dd7-ada0-99e06278ea1c" />

---

## 6. Task 4 — Analyze F1-C (F1AP / SCTP)

**Evidence (F1 Setup Request/Response over SCTP):**
<img width="2880" height="1718" alt="Screenshot 2026-11-18 093536" src="https://github.com/user-attachments/assets/dae1ed54-5c38-472e-9058-b5220de8ed7f" />
*Packet numbers: 5 (F1SetupRequest, DU->CU) and 7 (F1SetupResponse, CU->DU).*

**Evidence (UE-associated F1AP messages):**
<img width="2880" height="1496" alt="Screenshot 2026-11-18 093830" src="https://github.com/user-attachments/assets/9136cf56-cfae-41dc-acef-03404c53f22a" />
*Explanation of messages:*
*   **Packet 44 (InitialULRRCMessageTransfer):** The UE's initial RRC Setup Request is forwarded from the DU to the CU.
*   **Packet 45 (DLRRCMessageTransfer):** The CU generates the RRC Setup message and sends it down to the DU to be transmitted to the UE.
*   **Packet 61 (UEContextSetupRequest):** The CU requests the DU to establish UE-specific resources (like F1-U TEIDs) for data transport.

**Questions & Answers:**

1.  **Which node initiates F1 Setup, and which acknowledges it?**
    The DU initiates the connection by sending the `F1 Setup Request`. The CU acknowledges it by sending the `F1 Setup Response`.
2.  **Explain the difference between SCTP association establishment and F1 Setup.**
    SCTP association is a transport-layer (Layer 4) handshake (INIT, INIT_ACK, COOKIE_ECHO, COOKIE_ACK) that ensures a reliable connection exists between the two IP endpoints. F1 Setup is an application-layer (F1AP) procedure that runs *over* the established SCTP connection, where the DU and CU exchange 5G-specific configuration data (e.g., served cells, capabilities).
3.  **How can a NAS Registration Request be transported over F1-C?**
    A NAS message (like Registration Request) is encapsulated inside a control-plane RRC message (e.g., RRC Setup Complete) by the UE. When the DU receives this RRC message over the air, it encapsulates it into an `ULRRCMessageTransfer` F1AP message and forwards it to the CU over the F1-C interface. The CU then extracts the NAS message and forwards it to the AMF over the N2 (NGAP) interface.
4.  **What is the role of UE Context Setup?**
    It establishes UE-specific signaling and user-plane transport resources between the CU and DU. Crucially, it sets up the F1-U Tunnel Endpoint Identifiers (TEIDs) so that user data can be routed correctly between the two nodes.
5.  **Does F1 Setup alone prove successful UE Registration? Explain.**
    No. F1 Setup only establishes the network infrastructure connection between the gNB-CU and gNB-DU. It happens before any UE even tries to connect. UE Registration is a completely separate procedure that occurs later between a specific UE and the 5G Core Network (AMF).

---

## 7. Task 5 — Analyze F1-U (GTP-U / UDP)

**Evidence (GTP-U T-PDU):**
<img width="2880" height="1630" alt="Screenshot 2026-11-18 094333" src="https://github.com/user-attachments/assets/1f7cb67d-f1ee-4dd6-bf2e-4f9b1e6ba97e" />
*Observation: The screenshot shows a G-PDU (T-PDU) carrying user data from 127.0.0.4 to 127.0.0.3, with TEID: 0xb03b116b (2956661099).*

**Questions & Answers:**

1.  **Which transport protocol carries F1-U?**
    F1-U is carried by the UDP protocol (port 2152).
2.  **What does a GTP-U TEID identify?**
    A TEID (Tunnel Endpoint Identifier) identifies a specific logical tunnel for a specific PDU session belonging to a specific UE. It ensures that user data packets are routed to the correct user context.
3.  **Which direction is 127.0.0.4 → 127.0.0.3 in this setup?**
    Since 127.0.0.4 is the DU (radio side) and 127.0.0.3 is the CU (core side), this represents the **Uplink** direction.
4.  **How would you distinguish F1-U from N3 in a packet capture?**
    Both use GTP-U over UDP port 2152. To distinguish them, you must look at the outer IP addresses. F1-U traffic flows between the DU and CU (e.g., 127.0.0.4 and 127.0.0.3). N3 traffic flows between the CU and the 5GC UPF (e.g., 192.168.70.x).
5.  **Why do F1-C and F1-U use different transport protocols?**
    F1-C uses SCTP because control-plane signaling requires guaranteed, reliable, and in-order delivery. F1-U uses UDP because user-plane data prioritizes speed and low latency; lower overhead is preferred, and any packet loss is handled by higher-layer protocols (like TCP inside the UE's application).

---

## 8. Task 6 — Compare monolithic and split gNB

| Feature | Lab 3: Monolithic gNB | Lab 4: CU/DU Split |
| :--- | :--- | :--- |
| **Number of RAN processes** | 1 (nr-softmodem) | 2 (CU and DU) |
| **Location of RRC/PDCP** | gNB | gNB-CU |
| **Location of RLC/MAC/PHY** | gNB | gNB-DU |
| **F1-C interface present?** | No | Yes |
| **F1-U interface present?** | No | Yes |
| **UE connected to the same 5GC?** | Yes | Yes |
| **End-to-end Ping works?** | Yes | Yes |

**Explanation:**
Splitting a monolithic gNB into a CU and DU only changes the internal architecture and distribution of the Radio Access Network (RAN). The external interfaces provided to the UE (the NR-Uu radio interface) and to the 5G Core Network (N2 and N3) remain completely unchanged. Because the UE is unaware of how the base station is structured internally, it still needs to perform the exact same synchronization, registration, and PDU session establishment procedures to access the network.
