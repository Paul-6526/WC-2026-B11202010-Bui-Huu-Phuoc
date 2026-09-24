# Lab 0: Basic Wireshark operation and capture
**Student Name:** Bui Huu Phuoc (裴有福)
**Student ID:** B11202010

---

### 1. Wireshark installation guide (in Windows)
Installing Wireshark on my Windows laptop was pretty straightforward. I just grabbed the latest 64-bit installer from the official website and clicked through the standard setup (making sure to leave the Npcap installation checked). Once it launched, I simply double-clicked my active "Wi-Fi" interface on the home screen, and it immediately started catching packets!

<img width="2880" height="1712" alt="Screenshot 2026-09-24 100537" src="https://github.com/user-attachments/assets/7434a0a2-0005-416d-8365-65c34324ff4a" />

---

### 2. Website Packet Capture: access a public HTTPS website
**2.1 Which website did you access?**
I accessed `easylist-downloads.adblockplus.org` (a background HTTPS connection automatically established by my browser's ad-blocker extension).

**2.2 What are the IP address and port number of the website server?**
The IP address of the website server is `210.71.227.144`, and the port number is `443` (HTTPS).

**2.3 What are the IP address and source port number of your PC when initially accessing the website?**
My PC's IP address is `10.119.200.238`, and the source port number is `62419`.

**2.4 What is the process of the TCP three-way handshake?**
*   **SYN (Packet 106):** My PC (`10.119.200.238`) sends a SYN packet to the server (`210.71.227.144`) to ask for a new connection.
*   **SYN-ACK (Packet 111):** The server replies with a SYN-ACK packet, basically saying "I received your request, let's connect."
*   **ACK (Packet 113):** My PC sends a final ACK packet back to confirm. The connection is now fully established.

<img width="2880" height="1334" alt="Screenshot 2026-09-24 101608" src="https://github.com/user-attachments/assets/41e028ee-8280-46b0-9f46-2ac8957a29c0" />

---

### 3. Use the filter dns to find a DNS packet and answer the following questions:
**3.1 What is the IP address and port of the DNS server?**
The IP address of the DNS server is `168.95.1.1`, and the port number is `53` (the standard port for DNS).

**3.2 What is the domain name in this query?**
The domain name in the query is `easylist-downloads.adblockplus.org`.

**3.3 Which protocol(s) does this DNS packet use? (List the protocols from Layer 2 up to Layer 5)**
*   **Layer 2 (Link Layer):** Ethernet II
*   **Layer 3 (Network Layer):** Internet Protocol Version 4 (IPv4)
*   **Layer 4 (Transport Layer):** User Datagram Protocol (UDP)
*   **Layer 5 (Application Layer):** Domain Name System (DNS)

<img width="2880" height="1342" alt="Screenshot 2026-09-24 101854" src="https://github.com/user-attachments/assets/634078ac-3ac6-4fb9-abfe-29045fb5e391" />

---

### 4. Access an HTTP page and answer the following questions:
**4.1 Which HTTP page did you access?**
I accessed `http://www.gzxyzn.com/Article/bjrk2/1644.html`.

**4.2 What is the IP address and port of the server hosting this page?**
The server's IP address is `61.183.8.129`, and the port is `80` (Dst Port in packet 2158).

**4.3 What is the request method?**
The HTTP request method used here is `GET` (Packet 2158).

**4.4 What is the response status code, and what does it mean?**
The response status code is `200 OK` (Packet 2163). This means everything went smoothly—the server understood my request and successfully returned the webpage data.

<img width="2880" height="1656" alt="Screenshot 2026-09-24 102104" src="https://github.com/user-attachments/assets/ef504d9e-d961-47d6-8386-d319d19bac60" />

---

### 5. Bonus: Basic ICMP Ping Capture
To generate some basic network traffic and verify my capture setup, I ran a quick `ping www.wikipedia.org` command in my Command Prompt. Using the `icmp` filter in Wireshark, I could clearly see the Echo Request packets leaving my PC and the Echo Reply packets returning from the Wikipedia server.

<img width="2880" height="1490" alt="Screenshot 2026-09-23 235431" src="https://github.com/user-attachments/assets/6e7fc16b-9d09-471c-8308-c9b5dd5dde8a" />

<img width="2076" height="1230" alt="Screenshot 2026-09-23 235441" src="https://github.com/user-attachments/assets/7a7f2b6d-58e7-429f-963c-594753267a32" />

---

### 6. Link of the PCAP
https://drive.google.com/file/d/10K-vl_UZCygO4oJ9Cfa7MoZrLviaGaZ2/view?usp=drive_link
