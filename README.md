# CVE-2023-27997 Vulnerability Assessment Tool

Safely detect whether a FortiGate SSL VPN instance is vulnerable to CVE-2023-27997 based on response timing. See the [full write-up](https://bishopfox.com/blog/cve-2023-27997-vulnerability-scanner-fortigate) at the Bishop Fox blog, including a complete walkthrough of the methodology behind this tool.

<div align="center">
    <img src="https://s3.us-east-2.amazonaws.com/s3.bishopfox.com/prod-1437/Images/channels/blog/Content/MicrosoftTeams-image-22.png" width="400px" />
</div>


## Description

CVE-2023-27997 is a heap-based buffer overflow in FortiGate's SSL VPN component which has been demonstrated to be exploitable for pre-authentication RCE. Since this a memory corruption bug, we to be able to detect vulnerable versions without crashing the `sslvpnd` process and disconnecting active users. 

This tool will send 800 requests to the vulnerable URL path. Half of the requests will have an invalid length, and will therefore be rejected by newer FortiGate versions; this creates a measurable timing difference  (around 250 microseconds on our devices while testing) between requests with valid and invalid lengths, which we can detect using a bit of math.

The request size and data length fields are specifically chosen such that, on vulnerable devices, memory corruption will only affect areas in the heap in which there is no in-use data. This guarantees that the SSL VPN process will not crash.

## Getting Started

### Install

```
$ git clone https://github.com/BishopFox/CVE-2023-27997-check
$ cd CVE-2023-27997-check
$ python3 -m venv venv
$ source venv/bin/activate
$ python3 -m pip install -r requirements.txt
```

### Usage

Scan the SSL VPN at `https://<IP>:<PORT>`. The target may return with status `Vulnerable`, `Patched`, or `Unknown`. The scan will usually complete in under 30 seconds—but due to the large number of requests being sent, it may take up to 5 minutes in some circumstances.

```
$ python3 CVE-2023-27997-check.py <IP> <PORT>
```

In the following example, `example.com:10443` is vulnerable to CVE-2023-27997.

```
$ python3 CVE-2023-27997-check.py example.com 10443
Checking https://example.com:10443
Vulnerable
```

Note that this tool relies on timing information measured over the internet, which is subject to noise. The tool will warn you when results may be wrong. 

```
$ python3 CVE-2023-27997-check.py example.com 10443
Checking https://example.com:10443
WARNING: Low confidence results.
Patched

$ python3 CVE-2023-27997-check.py example.com 10443
Checking https://example.com:10443
WARNING: Low confidence results.
Unknown 
```

### Troubleshooting

If you're seeing `WARNING: Low confidence results`, consider trying to reduce internet noise by using a VPS that's as geographically close to the target server as possible.

## Back matter

### Legal disclaimer

Usage of this tool for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state, and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program.

### See also
- [FortiGuard Advisory](https://www.fortiguard.com/psirt/FG-IR-23-097)
- [LEXFO writeup](https://blog.lexfo.fr/xortigate-cve-2023-27997.html)
