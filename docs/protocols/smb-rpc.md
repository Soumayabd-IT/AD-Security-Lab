# SMB & RPC

## Role in Active Directory
**SMB (Server Message Block)** is Windows' file and printer sharing protocol — it's what lets you access a shared folder over the network. **RPC (Remote Procedure Call)** lets a program execute code on a remote machine as if it were running locally. The two are closely tied together in the Windows ecosystem: many RPC calls travel over SMB, making this pair central to both remote administration and attack techniques.

## Key definitions
- **Port 445**: SMB directly over TCP (modern method, since Windows 2000).
- **Port 139**: SMB over NetBIOS (legacy method, still present for compatibility).
- **Port 135**: RPC Endpoint Mapper answers "which port is the requested service running on?".
- **Dynamic port (49152–65535)**: range where the actual RPC service responds, once negotiated via port 135.
- **Named pipe**: a communication channel that lets RPC calls travel directly over an SMB connection, without going through a dedicated RPC port.
- **Administrative share**: hidden share created automatically (`C$`, `ADMIN$`, `IPC$`) — invisible in network browsing because its name ends with `$`, but accessible with the right permissions.
- **IPC$**: special share used only for inter-process communication (named pipes), not for storing files.

## SMB versions and known vulnerabilities
| Version | Introduced with | Notable point |
|---------|------------------|----------------|
| SMBv1 | Windows NT / 2000 | Obsolete, vulnerable to EternalBlue (MS17-010), exploited by WannaCry |
| SMBv2 | Windows Vista / Server 2008 | Better performance, fixes SMBv1's major flaws |
| SMBv3 | Windows 8 / Server 2012 | Adds encryption of exchanges |

## How it works
1. The client connects to the server via SMB (port 445 or 139).
2. For an RPC call, the client first queries port 135 (Endpoint Mapper) to learn the actual port of the desired service.
3. The server replies with a dynamic port or the client uses a named pipe over SMB (`IPC$`) directly to skip this step.
4. The RPC call is executed over the established connection.

## Exploited weaknesses
- Enumeration of shares, active sessions, and users via RPC calls (SAMR, LSARPC) → [`attacks/01-recon/smb-rpc-enumeration.md`](../../attacks/01-recon/smb-rpc-enumeration.md)
- Exploitation of unpatched SMBv1 (EternalBlue) for remote code execution
- SMB relay (NTLM relay) exploiting the absence of SMB signing → [`attacks/02-credential-capture/smb-relay.md`](../../attacks/02-credential-capture/smb-relay.md)
- Lateral movement via administrative shares (`ADMIN$`, `C$`) using tools like PsExec

## Defensive takeaways
- Disable SMBv1 entirely
- Enforce mandatory SMB signing to prevent relay attacks
- Monitor for unusual connections to `IPC$` and `ADMIN$` (a lateral movement signature)

## References
-
