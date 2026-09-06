# LDAP (Lightweight Directory Access Protocol)

## Role in Active Directory
LDAP is the standard protocol used to query and administer the Active Directory directory: searching for users, groups, computers, organizational units, and reading their attributes. Unlike Kerberos, which is used to authenticate to a service, LDAP is used to read or modify the directory database itself — but accessing LDAP itself requires prior authentication.

## Key definitions
- **Port 389**: LDAP in clear text (unencrypted by default).
- **Port 636**: LDAPS, LDAP encrypted via TLS/SSL.
- **Bind**: the authentication step against the LDAP server before any query.
  - **Simple bind**: distinguished name (DN) + password, sent in clear text if no TLS.
  - **SASL bind (GSSAPI)**: authentication via Kerberos, without transmitting the password.
  - **Anonymous bind**: no authentication — disabled by default since Windows Server 2003, but sometimes re-enabled by mistake.
- **Object class**: the type of object in the directory (`user`, `computer`, `group`, `organizationalUnit`...).
- **servicePrincipalName (SPN)**: attribute stored on service accounts, readable via LDAP — this is the basis for Kerberoasting enumeration.
- **LDAP filter**: query syntax (RFC 4515) used to target specific objects, e.g. `(&(objectClass=user)(servicePrincipalName=*))` to list every account with an SPN.

## How it works
1. **Bind** — the client authenticates (simple, SASL/Kerberos, or anonymous if allowed).
2. **Search request** — the client sends a query with a base DN, a scope, and a filter.
3. **Response** — the server returns the matching objects with the requested attributes.
4. **Unbind** — the session is closed.

## Exploited weaknesses
- Enumeration possible with any valid domain account, even without privileges → [`attacks/01-recon/ldap-enumeration.md`](../../attacks/01-recon/ldap-enumeration.md)
- SPN attribute readable by any authenticated user → feeds Kerberoasting → [`attacks/03-kerberos-attacks/kerberoasting.md`](../../attacks/03-kerberos-attacks/kerberoasting.md)
- Simple bind in clear text (port 389 without TLS) → passwords can be intercepted on the network
- BloodHound builds its attack graph almost entirely from LDAP queries → [`attacks/05-privilege-escalation/bloodhound-mapping.md`](../../attacks/05-privilege-escalation/bloodhound-mapping.md)

## Defensive takeaways
- Disable anonymous bind (check it hasn't been re-enabled by mistake)
- Enforce LDAP signing and channel binding (LDAPS/StartTLS) to prevent LDAP relay and password interception
- Monitor for abnormal volumes of LDAP requests, especially filters targeting SPNs (a reconnaissance signature)

## References
-
