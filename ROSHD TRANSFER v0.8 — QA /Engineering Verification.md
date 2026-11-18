ROSHD TRANSFER v0.8 — QA / Engineering Verification
====================================================

BUILD HOST — VERIFIED
- Native x86 PE32 and x64 PE32+ GUI executables compiled successfully.
- PE subsystem version is 6.01 (Windows 7 target); both executables contain the ROSHD icon resource.
- Windows system API imports verified, including SetFilePointerEx, GetFileTime, FlushFileBuffers and SetFileAttributesW.
- No Python, CMD, Node, Qt, VCRuntime or child-process launcher dependency in the executable import table/source.
- Single-instance mutex is shared by x86 and x64 builds.

AUTOMATED TESTS — PASSED
- CRC32: 17,713 lengths, including the standard CRC32 test vector.
- Fastpack: 1,007 compression/decompression cases using 1 MiB blocks.
- Loopback framing/integrity: 128 MiB compressible and 1,024 MiB incompressible data; CRC matched.
- 400 MiB resume reference test: connection deliberately dropped after 3 MiB; receiver retained the partial file; second connection resumed from the stored offset; CRC state was continued; final ACK was deliberately lost; third connection returned the idempotent completed status without retransmitting the file.
- Static source checks: resume offset + CRC state handshake, retry loop, no artificial bandwidth cap, durable receiver writes, worker handles joined at shutdown, socket shutdown, and no child-process launch.

IMPORTANT TEST SCOPE
- The 400 MiB interrupted-transfer test is a Python loopback reference-protocol simulation. It validates the designed framing/resume behavior but does NOT execute the native Win32 application.
- The 1 GiB loopback test uses the same fastpack compressor but is not a real LAN/Wi-Fi throughput test.
- This environment has no Windows 7/10 runtime or Wine; native EXE startup, GUI behavior, firewall interaction, actual network interruption/reconnection, and Task Manager process exit have not been verified on physical Windows PCs.

REQUIRES REAL-PC VALIDATION
1. Windows 10 x64 sender -> Windows 7 x86 receiver, 400 MiB or larger file.
2. Interrupt LAN/Wi-Fi after 10–20% and restore it; confirm progress resumes from the saved offset.
3. Repeat with Wi-Fi sender and wired LAN receiver, then reverse direction.
4. Close the application during an active transfer; confirm the process disappears from Task Manager after shutdown and no child CMD remains.
5. Reopen the app, reselect the same source file, and confirm it resumes the receiver's retained partial file.
6. Send multiple files consecutively and confirm the queue advances after each completed file.

KNOWN LIMITATIONS / SAFETY
- Payload is not encrypted. Use trusted private LAN only; do not send confidential files over public networks.
- Both PCs must run v0.8; v0.7 and v0.8 discovery/protocol identifiers are intentionally incompatible.
- Automatic retry continues until the user cancels or closes the app. If the receiver is offline, the active queue remains waiting.
- If the sender application is restarted, the source file must be re-added to the queue; the receiver-side partial file is retained and can resume.
- Resume identity is based on sender username, device name, file name, size and last-write timestamp; final CRC32 verifies the complete content.
- Actual throughput depends on network quality, router, disk and CPU. No artificial speed limit is implemented.
