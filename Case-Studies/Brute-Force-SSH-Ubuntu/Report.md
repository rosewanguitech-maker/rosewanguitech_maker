# Brute Force Login Investigation – Linux SSH
Case Study • SOC Analyst Portfolio  
File: `/Case-Studies/Brute-Force-SSH-Ubuntu/Report.md`

---

## Table of Contents
1. Overview  
2. Objective  
3. Environment and Tools  
4. Evidence Acquisition  
5. Log Analysis  
6. Findings  
7. Recommendations  
8. Conclusion  
9. Appendix (Screenshots)

---

## 1. Overview
This report documents the investigation of a suspected brute-force attack against the SSH service on an Ubuntu Linux system. The activity was identified through abnormal authentication failures recorded in `/var/log/auth.log`. The analysis determines the attacker IP, targeted username, frequency of attempts, and behavioral patterns.

---

## 2. Objective
- Examine SSH authentication logs for signs of brute-force activity.  
- Identify attacker IP addresses.  
- Determine targeted usernames.  
- Count and analyze the failed attempts.  
- Provide SOC‑level recommendations based on findings.

---

## 3. Environment and Tools

| Component | Details |
|----------|---------|
| Host OS | Ubuntu Linux |
| Log Source | `/var/log/auth.log` |
| Tools Used | grep, awk, sort, uniq, wc |
| Event Type | SSH failed authentication (sshd) |

---

## 4. Evidence Acquisition

The following commands were used to collect and inspect logs:

### View first 10 failed password entries:
```bash
sudo grep -a "Failed password" /var/log/auth.log | head
```

### Count total failed password attempts:
```bash
sudo grep -a "Failed password" /var/log/auth.log | wc -l
```

### Identify attacker IP addresses:
```bash
sudo grep -a "Failed password" /var/log/auth.log \
| awk '{print $(NF-3)}' | sort | uniq -c | sort -nr | head
```

### Identify targeted usernames:
```bash
sudo grep -a "Failed password" /var/log/auth.log | grep "sshd" \
| awk '{for(i=1;i<=NF;i++){if($i=="for"){print $(i+1)}}}' \
| sort | uniq -c | sort -nr | head
```

### Extract ports involved:
```bash
sudo grep -a "Failed password" /var/log/auth.log | grep "sshd" \
| awk -F'port ' '{print $2}' | awk '{print $1}' \
| sort | uniq -c | sort -nr | head
```

---

## 5. Log Analysis

### 5.1 Total Failed Attempts
The system recorded:

**597 failed SSH login attempts**

This indicates automated brute-force behavior.

---

### 5.2 Attacker IP Address
Top source IP extracted:

| Count | IP Address |
|-------|------------|
| 588 | 127.0.0.1 |

*Note:* The attack originated locally. This could indicate internal testing, misconfiguration, or a script running on the host.

---

### 5.3 Username Targeted
Only one username was targeted:

| Count | Username |
|-------|----------|
| 588 | attacker_test |

This confirms a targeted brute-force pattern.

---

### 5.4 Ports Used
Multiple unique destination ports were recorded, with each port used exactly 6 times—consistent with automated brute-force tools.

Example top ports:

```
60996  
60986  
60982  
60974  
60958  
```

---

## 6. Findings

- The attack consisted of **597 total failed SSH authentication attempts**.
- A single username, **attacker_test**, was repeatedly targeted.
- All attempts originated from **127.0.0.1**.
- Attempts occurred within milliseconds, confirming automated tooling.
- Multiple source ports were used to bypass basic rate‑limiting.

This pattern matches typical brute-force attacks used to guess SSH credentials.

---

## 7. Recommendations

### 1. Implement Fail2ban or UFW rate‑limiting  
Prevents repeated login attempts.

### 2. Disable SSH password authentication  
Configure SSH to use keys only:
```
PasswordAuthentication no
```

### 3. Restrict SSH access by IP  
Use firewall rules to limit exposure.

### 4. Configure Centralized Logging  
Forward logs to SIEM platforms such as Splunk, ELK, Wazuh, or Graylog.

### 5. Enforce Strong Authentication Policies  
Strong passwords + SSH keys + MFA (where possible).

---

## 8. Conclusion
The investigation confirms an automated brute-force attack targeting `attacker_test` on the Ubuntu SSH service. Although the source IP is localhost, the behavior mirrors external brute-force attacks and should be treated seriously. Proper SSH hardening and monitoring must be implemented to mitigate future risks.

---

## 9. Appendix (Screenshots)


```
![Failed password logs](../images/failed_password_sample.png)
![Port activity](../images/ports_used.png)
```

---

End of Report
