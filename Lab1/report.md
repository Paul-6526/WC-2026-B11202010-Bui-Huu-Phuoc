# Lab 1: Analyzing UE–gNB Connectivity in an OAI 5G SA Network
**Student Name:** Bui Huu Phuoc (裴有福)
**Student ID:** B11202010

---

### Checkpoint 1: Wireshark Setup
**1. OAI-5G profile is selected:**
<img width="2880" height="1716" alt="Screenshot 2026-09-23 092156" src="https://github.com/user-attachments/assets/0089d599-a981-4bbb-8080-d8538ddf19f1" />

**2. Opened capture:**
<img width="1912" height="1370" alt="Screenshot 2026-09-23 092144" src="https://github.com/user-attachments/assets/c101fe35-4e76-42d9-b163-db9f2515c0ac" />

**3. NR RRC packets after applying `nr-rrc`:**
<img width="2876" height="1688" alt="Screenshot 2026-09-23 092431" src="https://github.com/user-attachments/assets/70bedfc5-0297-4772-949c-150d724553a5" />

---

### Checkpoint 2: Basic Architecture
**1. IP Address Table**

**Filter: nr-rrc**
<img width="2880" height="1638" alt="gNB and AMF" src="https://github.com/user-attachments/assets/14985ebf-629d-427a-9514-47dee5b6d913" />
**Filter: gtp**
<img width="2878" height="1448" alt="Screenshot 2026-09-23 093333" src="https://github.com/user-attachments/assets/f6775a70-407d-456e-8dc2-6a0aeb7fc6c7" />
**Filter: icmp**
<img width="2866" height="1408" alt="Screenshot 2026-09-23 093637" src="https://github.com/user-attachments/assets/418d8626-583a-4b2a-9474-957082e0ce9f" />



| Component | IP address | Evidence from the capture |
| :--- | :--- | :--- |
| **UE PDU address** | 10.0.0.2 | Assigned in PDU Session Establishment Accept |
| **gNB** | 192.168.70.129 | Source/Destination in NGAP and GTP-U packets |
| **AMF** | 192.168.70.132 | Source/Destination in NGAP packets |
| **UPF** | 192.168.70.135 | Source/Destination in GTP-U packets |
| **Data Network** | 192.168.70.135 / Ext | Target of ICMP Echo Request/Reply from UE |

**2. Interface Table**

| Interface | Connected components | Main protocol | Purpose |
| :--- | :--- | :--- | :--- |
| **N1** | UE $\leftrightarrow$ AMF | NAS-5GS | Non-Access Stratum signaling (Registration, Session management). |
| **N2** | gNB $\leftrightarrow$ AMF | NGAP | Control plane signaling between RAN and 5G Core. |
| **N3** | gNB $\leftrightarrow$ UPF | GTP-U | User plane data tunneling between RAN and UPF. |

---

### Checkpoint 3: RRC Connection Establishment
**1. Required Screenshots**
*   **Packet 104 (RRCSetupRequest):**
   <img width="2878" height="1058" alt="Screenshot 2026-09-23 094958" src="https://github.com/user-attachments/assets/958314c6-5369-461f-a2c1-2e244658be76" />
   
*   **Packet 105 (RRCSetup):**
    <img width="2880" height="1406" alt="Screenshot 2026-09-23 095047" src="https://github.com/user-attachments/assets/33e97d48-c8c2-4a0c-837b-e580a00b52ab" />
    
*   **Packet 108 (RRCSetupComplete):**
   <img width="2880" height="1390" alt="Screenshot 2026-09-23 095632" src="https://github.com/user-attachments/assets/d9da4321-fe01-43c0-b3d0-00aa857dae60" />

**2. RRC Message Table**

| Message | Direction | Logical channel / SRB | Main purpose | Packet number |
| :--- | :--- | :--- | :--- | :--- |
| **RRCSetupRequest** | UE $\rightarrow$ gNB | UL-CCCH / SRB0 | UE requests to establish an RRC connection with the gNB. | 104 |
| **RRCSetup** | gNB $\rightarrow$ UE | DL-CCCH / SRB0 | gNB configures the SRB1 and radio resources for the UE. | 105 |
| **RRCSetupComplete** | UE $\rightarrow$ gNB | UL-DCCH / SRB1 | UE confirms successful RRC connection and encapsulates the first NAS message. | 108 |

**3. Answers to the Questions**
*   **What is the establishment cause in RRCSetupRequest?**
    The establishment cause is **mo-Signalling** (Mobile Originated Signalling).
*   **What SRB does RRCSetupRequest use? Why?**
    It uses **SRB0** because a dedicated, secure signaling connection (SRB1) has not yet been established. 
*   **Which side sends RRCSetup?**
    The **gNB** sends the RRCSetup message.
*   **Which signaling radio bearer is used after the RRC connection is established?**
    **SRB1** is used to transmit RRC messages (and piggybacked NAS messages) via DCCH.
*   **Which NAS message is carried inside RRCSetupComplete?**
    The **Registration Request** NAS message is encapsulated inside the `dedicatedNAS-Message` field.
*   **At the end of this procedure, is the UE only connected to the gNB, or is it already registered with the 5G Core? Explain.**
    The UE is **only connected to the gNB**. It is not yet registered with the 5G Core. The RRCSetupComplete message simply carries the Registration Request to the gNB, which will forward it to the AMF via NGAP.

---

### Checkpoint 4: RRC-to-NGAP/NAS Mapping
**1. Required Screenshots**
*   **Registration Request inside NGAP InitialUEMessage:**
    <img width="2880" height="1434" alt="Screenshot 2026-09-23 100155" src="https://github.com/user-attachments/assets/89089c4a-75aa-481a-976f-13d41280cded" />
*   **Registration Accept & Complete (Encapsulated in NGAP):**
    <img width="2880" height="1674" alt="Screenshot 2026-09-23 102342" src="https://github.com/user-attachments/assets/85d725b6-bc2c-41f9-9889-44349fe1412f" />
    *Note: The Registration Accept and Registration Complete messages are encapsulated within the InitialContextSetupRequest and UplinkNASTransport packets, respectively. They are displayed as "Encrypted data" because the NAS signaling is ciphered after the Security Mode procedure.*

**2. Mapping Table**

| Stage | Protocol message | Sender $\rightarrow$ receiver | Encapsulated information |
| :--- | :--- | :--- | :--- |
| **Radio side** | RRCSetupComplete | UE $\rightarrow$ gNB | Registration Request (NAS) |
| **Core side** | NGAP InitialUEMessage | gNB $\rightarrow$ AMF | Registration Request (NAS) |

**3. Answers to Questions**
*   **What is the role of the gNB when it transports NAS messages?**
    The gNB acts as a transparent relay. It extracts the NAS message from the RRC layer and encapsulates it into the NGAP protocol to forward it to the AMF.
*   **What is the difference between RRC and NAS signaling?**
    RRC manages the radio connection directly between the UE and gNB. NAS manages mobility and sessions directly between the UE and the 5G Core (AMF), transparent to the gNB.
*   **Is the Registration Request delivered directly from the UE to the AMF? Explain the protocol path.**
    Logically, yes. Physically, it travels from the UE to the gNB wrapped in RRC, and then from the gNB to the AMF wrapped in NGAP.
*   **Which message confirms that Registration has completed successfully?**
    The **Registration Complete** message sent from the UE to the AMF.

---

### Checkpoint 5: UE IP and User Plane
**1. Required Screenshots**
*   **PDU Session Establishment Accept / IPv4 assigned:**
  <img width="2866" height="1408" alt="Screenshot 2026-09-23 093637" src="https://github.com/user-attachments/assets/ddc384ab-4e4b-4259-92c1-c082dd8edd68" />

  <img width="2878" height="1448" alt="Screenshot 2026-09-23 093333" src="https://github.com/user-attachments/assets/ef12ec1b-12f9-45be-9036-f80ea6656859" />
  
*   **ICMP Echo Request/Reply inside GTP-U:**
   <img width="2878" height="1638" alt="Screenshot 2026-09-23 093532" src="https://github.com/user-attachments/assets/6dd678cb-2267-4ccd-8654-9cdda3b2c5c4" />

**2. Answers to Questions**
*   **What IPv4 address was assigned to the UE?**
    The assigned IPv4 address is `10.0.0.2`.
*   **How many ICMP Echo Request/Reply pairs are present?**
    There are 10 pairs of Echo Request/Reply present in the capture.
*   **What does the successful Echo Reply prove about the UE connection?**
    It proves that the UE has successfully registered with the 5G Core, established a PDU session, obtained a valid IP address, and that the user-plane GTP-U tunnel between the gNB and UPF is fully functional for external data routing.

---

## Checkpoint 6: Final Sequence Diagram
*The diagram below distinguishes Control Plane signaling (solid lines) and User Plane traffic (dotted lines).*

```mermaid
sequenceDiagram
    autonumber
    participant UE
    participant gNB
    participant AMF
    participant UPF
    participant Data Network

    %% Control Plane (Solid Lines)
    UE->>gNB: RRCSetupRequest
    gNB->>UE: RRCSetup
    UE->>gNB: RRCSetupComplete [NAS Registration Request]
    gNB->>AMF: NGAP InitialUEMessage [NAS Registration Request]
    
    AMF->>gNB: NGAP DownlinkNASTransport [Authentication/Security]
    gNB->>UE: RRC DL Information Transfer [Authentication/Security]
    UE->>gNB: RRC UL Information Transfer [Auth/Security Response]
    gNB->>AMF: NGAP UplinkNASTransport [Auth/Security Response]

    AMF->>gNB: NGAP InitialContextSetupRequest [Encrypted NAS Registration Accept]
    gNB->>UE: RRC Reconfiguration
    UE->>gNB: RRC Reconfiguration Complete
    UE->>gNB: RRC UL Information Transfer [Encrypted NAS Registration Complete]
    gNB->>AMF: NGAP UplinkNASTransport [Encrypted NAS Registration Complete]
    
    AMF<<->>UE: PDU Session Establishment (via gNB)

    %% User Plane (Dotted Lines)
    UE-->>gNB: ICMP Echo Request
    gNB-->>UPF: GTP-U [ICMP Echo Request]
    UPF-->>Data Network: ICMP Echo Request
    
    Data Network-->>UPF: ICMP Echo Reply
    UPF-->>gNB: GTP-U [ICMP Echo Reply]
    gNB-->>UE: ICMP Echo Reply
