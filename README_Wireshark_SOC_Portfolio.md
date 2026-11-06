# SOC Analyst Portfolio: Wireshark Network Traffic Investigations

![Focus](https://img.shields.io/badge/Focus-SOC%20%7C%20Blue%20Team-1f6feb)
![Tool](https://img.shields.io/badge/Tool-Wireshark-1679A7)
![Platform](https://img.shields.io/badge/Platform-Kali%20Linux-557C94)

## 1. Project Overview

This repository documents practical network-traffic investigations completed in the **Wireshark SOC Labs** on the NetworkWalks training platform.

The exercises focus on examining packet captures, understanding network conversations, identifying suspicious activity, and extracting evidence that can support a security operations investigation.

The work includes attacker-traffic identification, TCP stream analysis, HTTP plaintext credential exposure, brute-force investigation, and SIP/VoIP call analysis. Screenshots supplied with the project are retained as assessment and investigation evidence.

> **Important:** This is a training-lab portfolio. The observations below are limited to what is visible in the supplied screenshots. Where a screenshot does not expose a final answer or complete packet details, the report identifies that limitation rather than guessing.

## 2. Objectives

- Inspect packet captures and interpret source/destination communication.
- Use Wireshark display filters to isolate relevant traffic.
- Follow TCP streams to reconstruct application-layer conversations.
- Recognise risks associated with credentials sent over unencrypted HTTP.
- Investigate repeated authentication failures and possible brute-force activity.
- Inspect SIP signalling and VoIP call metadata.
- Record findings clearly and distinguish packet evidence from assumptions.

## 3. Lab Environment and Tools

| Component | Use |
|---|---|
| Wireshark | Packet capture inspection, display filtering, stream following and protocol analysis |
| Kali Linux | Linux environment shown in the lab screenshots |
| NetworkWalks Wireshark SOC Labs | Guided practical training exercises |
| Protocols visible in the exercises | TCP, HTTP, DNS, FTP, SIP, UDP and related network protocols |

## 4. Investigation Workflow

My general workflow for these exercises was:

1. **Understand the scenario** — identify the activity or question the lab asks me to investigate.
2. **Capture or load traffic** — work with the supplied lab capture.
3. **Establish context** — review endpoints, protocols, ports, packet counts and timing.
4. **Filter the traffic** — narrow the packet list to relevant conversations or events.
5. **Inspect evidence** — examine packet details and, where appropriate, follow the TCP stream.
6. **Determine the finding** — answer the investigation question using packet evidence.
7. **Document limitations** — avoid treating an indicator as proof when the capture does not support that conclusion.

## 5. Lab Investigation Reports

### Lab 3 — Identify the Attacker's IP

**Objective:** Investigate unusual traffic reported by a staff member and identify the IP address associated with the activity described in the lab.

**Methodology and evidence visible in the screenshots:**
- Reviewed TCP traffic between hosts in the `192.168.10.0/24` network.
- Examined SYN traffic using the display filter shown in the screenshot: `tcp.flags.syn == 1`.
- Inspected repeated TCP reset packets directed to `192.168.10.66`.
- The screenshot shows packets with source `192.168.10.20`, destination `192.168.10.66`, and the TCP RST flag set.

**Display filter visible in the evidence:**

```wireshark
ip.dst == 192.168.10.66 && tcp.flags.reset == 1
```

**Analyst observation:** The displayed packets show repeated TCP reset activity from `192.168.10.20` toward `192.168.10.66`. This is a traffic pattern worth investigating in the context of the lab scenario.

**Finding boundary:** The screenshot supports the packet addresses and RST pattern above. It does not, by itself, establish that the source host is malicious. Attribution should be based on the full capture and surrounding context.

**Assessment evidence:** Lab 3 result screenshot shows **69/100**.

---

### Lab 4 — Follow TCP Stream

**Objective:** Use TCP stream reconstruction to inspect a conversation between a manager's computer and a company server.

**Methodology:**
- Filtered the capture to focus on TCP traffic and relevant service ports.
- Identified the TCP conversation shown in the packet list.
- Used Wireshark's **Follow → TCP Stream** function to reconstruct the application data exchanged in that conversation.

**Evidence visible in the screenshot:** The exercise asks for the stream number of a chat. The screenshot shows the task and packet list, but does not clearly expose a verified final stream number.

**Finding:** TCP stream reconstruction is useful for reviewing a complete conversation rather than interpreting isolated packets. The exact requested stream number is intentionally not stated here because it is not legible in the supplied evidence.

---

### Lab 5 — Follow TCP Stream, Part 2

**Objective:** Investigate web requests made by a staff member and locate information hidden within a web page or response.

**Methodology and evidence visible in the screenshots:**
- Used the HTTP request filter `http.request`.
- Reviewed requests to the intranet host `intranet.company.local`.
- Selected the request for `/download/report.txt`.
- Followed the TCP stream to inspect the HTTP response body.

**Display filter:**

```wireshark
http.request
```

**Observed HTTP response details:**
- Request: `GET /download/report.txt`
- Host: `intranet.company.local`
- Response: `HTTP/1.1 200 OK`
- Content type: `text/plain`
- Server header: `Apache/2.4.57 (Ubuntu)`
- Response body includes a quarterly report, total sales value of `48200`, preparer `Nadia`, and status `Final`.

**SOC relevance:** Following a stream can reveal application-layer content that is not obvious from the packet list. In a real investigation, exposed business information should be handled as potentially sensitive data and reported through the approved process.

**Assessment evidence:** Lab 5 result screenshot shows **97/100**.

---

### Lab 6 — HTTP Plaintext Credentials

**Objective:** Investigate an HTTP login exchange and identify the authentication details and related request/response evidence in the lab capture.

**Methodology and evidence visible in the screenshots:**
- Reviewed the capture using Wireshark's protocol hierarchy statistics.
- The protocol hierarchy screenshot shows HTTP traffic in the capture, alongside TCP, DNS, FTP and other protocols.
- The assessment result confirms that the protocol, method, login page, signing PC, number of attempts, username, password, packet details and fix were assessed.

**Security significance:** HTTP does not encrypt application data in transit. If a login form submits credentials over ordinary HTTP, those credentials may be readable to someone who can capture the traffic.

**Defensive recommendations:**
- Enforce HTTPS using modern TLS.
- Redirect HTTP requests to HTTPS and enable HSTS where appropriate.
- Avoid transmitting credentials in cleartext.
- Monitor for authentication activity over insecure protocols.

**Finding boundary:** The supplied result screenshot confirms an **88/100** assessment score, but the actual username, password and exact number of attempts are not readable in the evidence provided. They are therefore not reproduced here.

---

### Lab 7 — Brute Force Catch

**Objective:** Investigate repeated failed sign-ins reported by a server owner and identify evidence relevant to a possible brute-force attempt.

**Methodology and evidence visible in the screenshots:**
- Opened Wireshark's **Statistics → Protocol Hierarchy** view.
- Reviewed the protocol distribution in the capture.
- The visible hierarchy includes IPv4, TCP, HTTP, DNS, FTP and other protocols.
- The assessment tasks cover the protocol, target, attacker, username, failure code, failed logins, success code, working password and attack type.

**SOC investigation approach:**
1. Identify the authentication protocol and relevant service.
2. Establish the target host and the suspected source host.
3. Examine repeated authentication attempts and their timestamps.
4. Compare failed and successful authentication responses.
5. Determine whether the pattern is consistent with brute-force behaviour.
6. Record the relevant source, destination, account and event evidence.

**Finding boundary:** The assessment screenshot confirms that all ten assessment items were marked correct, but it does not display the underlying credential values, IP addresses or exact attempt count. Those values are not guessed or published in this report.

**Assessment evidence:** Lab 7 result screenshot shows **100/100**.

---

### Lab 8 — VoIP Call Replay

**Objective:** Examine VoIP traffic, identify call signalling information and inspect the call represented in the packet capture.

**Methodology and evidence visible in the screenshots:**
- Used the SIP display filter `sip`.
- Reviewed SIP `INVITE`, `100 Trying`, `180 Ringing`, `200 OK` and `BYE` messages.
- Inspected the SIP endpoints shown in the packet list.
- Opened Wireshark's VoIP Calls window to review the call summary.

**Display filter:**

```wireshark
sip
```

**Observed call metadata:**
- SIP endpoints shown: `192.168.10.45` and `192.168.10.52`.
- The INVITE line shows `sip:202@192.168.10.52`.
- The call summary shows a completed call with a duration of approximately 25 seconds and 811 packets.

**SOC relevance:** SIP signalling can provide useful evidence about call setup, endpoints, call completion and timing. A complete replay investigation should also inspect the relevant media stream and compare it with the lab's stated scenario.

**Finding boundary:** The screenshots show SIP signalling and a completed call summary. They do not independently establish that the call was malicious or prove that replay occurred.

**Assessment evidence:** Lab 8 result screenshot shows **99/100**.

## 6. Assessment Summary

| Lab | Exercise | Recorded result |
|---|---|---:|
| 3 | Identify the Attacker's IP | 69/100 |
| 4 | Follow TCP Stream | No score visible in supplied screenshots |
| 5 | Follow TCP Stream, Part 2 | 97/100 |
| 6 | HTTP Plaintext Credentials | 88/100 |
| 7 | Brute Force Catch | 100/100 |
| 8 | VoIP Call Replay | 99/100 |

The scores above are transcribed from the supplied NetworkWalks result screenshots. They represent lab assessment results, not professional certification.

## 7. Wireshark Filters Practised

The following filters are visible in, or directly supported by, the supplied screenshots:

| Filter | Purpose |
|---|---|
| `tcp.flags.syn == 1` | Display TCP packets with the SYN flag set |
| `ip.dst == 192.168.10.66 && tcp.flags.reset == 1` | Display TCP reset packets destined for the specified host |
| `http.request` | Display HTTP requests |
| `sip` | Display SIP signalling packets |

Filters should be adapted to the capture and investigation question. A filter narrows the evidence; it does not independently prove malicious activity.

## 8. Skills Demonstrated

- Wireshark packet analysis
- Network traffic investigation
- TCP/IP and protocol analysis
- TCP stream reconstruction
- HTTP request and response inspection
- Recognition of plaintext credential exposure risks
- Authentication-event investigation
- Brute-force attack investigation methodology
- SIP and VoIP signalling analysis
- Evidence-based technical reporting

## 9. Evidence Folder

Recommended repository structure:

```text
wireshark-soc-analyst-portfolio/
├── README.md
└── evidence/
    ├── lab-03-attacker-ip/
    ├── lab-04-follow-tcp-stream/
    ├── lab-05-hidden-web-content/
    ├── lab-06-http-plaintext-credentials/
    ├── lab-07-brute-force-catch/
    └── lab-08-voip-call-replay/
```

Place the relevant screenshots in each lab folder. Use clear filenames such as `lab-07-score.png` or `lab-08-sip-packets.png`. Do not upload passwords, private tokens, personal data or unrelated browser information.

## 10. Learning Outcomes

These exercises strengthened my ability to move from a broad network-security scenario to a focused packet-level investigation. I practised narrowing traffic with display filters, reconstructing TCP conversations, reviewing application-layer protocols and identifying evidence relevant to authentication and VoIP investigations.

An important lesson from these labs is that a SOC analyst must distinguish between an observable network indicator and a confirmed security incident. Findings should be supported by packet evidence, documented clearly and escalated with appropriate context.

## 11. Disclaimer

All activities documented here were completed in an educational lab environment for cybersecurity learning and practical skill development. The screenshots are provided as evidence of the exercises and assessment results. No claim is made that these lab exercises represent a production-network incident or a professional certification.

---

**Author:** Jonathan Favour  
**Portfolio focus:** SOC Analysis | Blue Team Security | Network Traffic Analysis  
**Primary tool:** Wireshark  
**Training platform:** NetworkWalks
