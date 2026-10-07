# Incident 001 — SSH Password Guessing

## Summary

Repeated failed SSH authentication attempts were observed against the Ubuntu Server from the Kali Linux host.

A successful SSH login was later recorded from the same source IP.

## Environment

Source IP: 192.168.56.10  
Target IP: 192.168.56.20  
Service: SSH  
Port: TCP/22  
User: student

## Evidence

- ssh-auth.log
- sudo-commands.log
- Wireshark capture
- Nmap scan
- Splunk SIEM dashboard and triggered alert

## Indicators

Source IP: `192.168.56.10`  
Target IP: `192.168.56.20`  
Username: `student`  
Service: `SSH`  
Port: `TCP/22`

## MITRE ATT&CK

T1110.001 — Password Guessing  
T1046 — Network Service Discovery  

## Detection

This activity is covered by the following detection rules:

- `detections/sigma_ssh_password_guessing.yml`
- `detections/sigma_ssh_password_guessing_correlation.yml`
- `detections/sigma_ssh_failed_then_success.yml`

The most relevant detection for this incident is the failed-to-success correlation rule because the observed activity contains failed SSH authentication attempts followed by a successful login from the same source.

## Detection context

The activity was identified through Linux SSH authentication logs and manual log analysis.

## Splunk SIEM Detection

The Ubuntu Server sends live authentication logs from `/var/log/auth.log` to Splunk Enterprise through the Splunk Universal Forwarder.

In Splunk, the following activity is monitored:

- failed SSH authentication attempts
- successful SSH authentication
- source IP activity
- username activity
- failed-to-success authentication patterns

A scheduled alert checks the last 5 minutes of SSH authentication events and triggers when the detection query returns a suspicious pattern.

The alert was tested by generating live SSH activity from Kali Linux against the Ubuntu Server.

The test produced:

- multiple failed SSH authentication attempts
- a successful SSH login
- a triggered Splunk alert
- a matching entry in the SOC SSH Monitoring dashboard

This confirmed that the detection works on live log data rather than only on previously collected evidence.

## Timeline

14:23:31 — Failed SSH authentication for user `student` from `192.168.56.10`.

14:23:34 — Second failed SSH authentication from the same source IP.

14:23:36 — Successful SSH authentication for `student` from `192.168.56.10`.

14:30:06 — User `student` executed `sudo ls /root`.

14:30:31 — User `student` executed `sudo ss -tulpn`.

## Triage

The activity is suspicious but not enough to confirm a real attack.

Two failed SSH login attempts were followed by a successful login from the same source IP and the same user account. This could be a legitimate user entering the wrong password.

However, sudo activity was observed several minutes later, so the session should be reviewed in more detail before closing the incident.

## Verdict

Suspicious activity — further investigation required.

The observed SSH authentication pattern could be caused by a legitimate user entering the wrong password, but the successful login followed by sudo activity makes the session worth reviewing.

At this stage, there is not enough evidence to confirm account compromise.

The same authentication pattern was later reproduced in the lab and successfully detected by Splunk in real time.

## Recommendations

Review the SSH session and related system activity around the successful login.

Confirm whether the source IP `192.168.56.10` belongs to an authorized system.

Review sudo activity and commands executed after authentication.

Consider using SSH keys instead of password authentication.

Enable rate limiting or Fail2Ban to reduce repeated SSH login attempts.

If the activity is unauthorized, reset the account credentials and investigate the host for additional suspicious activity.
