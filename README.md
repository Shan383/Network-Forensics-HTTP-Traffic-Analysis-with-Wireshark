# Network Forensics: HTTP Traffic Analysis with Wireshark
**Aegis Northline Technologies — GRC & Security Portfolio**

## Overview

This project demonstrates a full plaintext-HTTP forensic workflow: generating a controlled HTTP session, capturing it without altering the evidence, and reconstructing the conversation end-to-end using Wireshark and tshark. The exercise simulates the kind of packet-level investigation a GRC/IT risk analyst may need to interpret when reviewing incident evidence, SOC escalations, or audit findings tied to network activity.

**Environment:** Kali Linux VM, Apache 2.4.68, curl 8.20.0, Wireshark/tshark, loopback interface (`lo`)

## Objectives

- Stand up a local web service and generate a single, controlled HTTP request/response
- Capture the session without modifying the original evidence file
- Verify evidence integrity using SHA-256 hashing (chain of custody)
- Reconstruct and interpret the TCP three-way handshake, HTTP exchange, and connection teardown
- Explain what each protocol layer (Ethernet, IP, TCP, HTTP) contributes as forensic evidence
- Document the limitations of loopback captures (no genuine MAC-address evidence)

## Methodology

### 1. Evidence Folder Structure
A case-style folder layout was created before any traffic was generated, separating raw evidence, working copies, exported artifacts, reports, and screenshots — mirroring standard evidence-isolation practice.

### 2. Generating Controlled Traffic
Apache was installed and configured to serve a single test page. Service status and the listening port were verified before capture began, ensuring the environment was in a known-good state prior to evidence collection.

### 3. Capturing the Session
`tshark` was run on the loopback interface with a filter scoped to TCP port 80, writing directly to the evidence file. `curl --no-keepalive` was used to generate a single clean request/response cycle, avoiding persistent-connection noise in the capture.

### 4. Preserving Chain of Custody
A working copy of the capture was created with timestamps preserved, and SHA-256 hashes were generated for both the original and the copy. Identical hashes confirmed the working copy was a byte-for-byte duplicate, satisfying integrity requirements before any analysis began.

### 5. Analyzing the TCP Three-Way Handshake
Using `tcp.stream==0`, the full 10-packet conversation was isolated. The SYN → SYN-ACK → ACK sequence was mapped out with source/destination ports, initial sequence numbers, and acknowledgment numbers, establishing the session's evidentiary anchor point.

### 6. Examining the HTTP Request and Response
`http.request` and `http.response` filters isolated the GET request and the 200 OK response. Header fields (Host, User-Agent, Server, Content-Type, Content-Length, Last-Modified, ETag) were extracted directly from Wireshark's protocol tree, and **Follow TCP Stream** was used to reconstruct the full plaintext exchange in a single view.

### 7. Encapsulation and Connection Closure
Frame-level inspection showed how each layer (Ethernet → IP → TCP → HTTP) nests around the payload above it without altering it. Connection teardown was isolated using `tcp.flags.fin==1 || tcp.flags.reset==1`, confirming a clean four-step FIN/ACK closure with no reset packets — i.e., the session ended by mutual agreement rather than error.

## Key Findings

| Area | Finding |
|---|---|
| Evidence integrity | Identical SHA-256 hashes for original and working copy confirm an unaltered evidence chain |
| Handshake | Full SYN/SYN-ACK/ACK sequence captured with sequence numbers and timing, anchoring the session to specific sockets and timestamps |
| HTTP exchange | Complete plaintext request/response recovered, including headers and page content — no decryption required |
| Connection closure | Clean FIN/ACK teardown, no RST packets — graceful session termination |
| Loopback limitation | Ethernet MAC addresses appear as all-zero on `lo` traffic; genuine MAC evidence requires a two-host capture |

## Forensic Takeaways

- **HTTP is cleartext evidence.** Every header and byte of the response body is recoverable from a capture with nothing more than standard Wireshark filters — which is exactly why plaintext protocols are considered high-risk in a real environment.
- **The handshake is evidence, not just plumbing.** Initial sequence numbers and port pairings let an investigator establish who talked to whom, when, and in what order.
- **Encapsulation supports layered analysis.** Each protocol layer can be examined independently while trusting the payload it wraps remains intact.
- **Capture topology matters.** Loopback captures are useful for controlled lab work but cannot yield hardware-address evidence — a reminder to match capture location to the evidence requirements of a real case.

## Skills Demonstrated

`Network Forensics` · `Packet Analysis (Wireshark/tshark)` · `TCP/IP Fundamentals` · `Evidence Handling & Chain of Custody` · `Linux (Kali)` · `Apache & curl` · `SHA-256 Verification`

---
*Part of an ongoing hands-on GRC and security portfolio built around a fictional organization, Aegis Northline Technologies, to demonstrate practical technical skills alongside GRC/audit deliverables.*
