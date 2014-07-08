# QueueMetrics Agent page Web RTC compatibility matrix

This document is a summary of compatibility tests between QueueMetrics WebRTC embedded softphone and several browsers and/or Asterisk versions/distro
Is not intended to be exaustive but as work in progress. Please feel free to update if you have more details on that. Thanks.
Tested with QueueMetrics 14.06.2

## Compatibility matrix

|Browser/Setup | Chrome 35.0.1916.153 m (HTTP) | Chrome 35.0.1916.153 m (HTTPS) | Firefox 30.0 (HTTP) | Firefox 30.0 (HTTPS) | IE11 11.0.9600.17126 (HTTP) | IE11 11.0.9600.17126 (HTTPS) |
| :----------: | :---------------------------: | :----------------------------: | :-----------------: | :------------------: | :-------------------------: | :--------------------------: |
| Asterisk 11.10.2 | OK (1)                    |  OK                            | KO: registration OK but no calls (2)| KO | KO: registration OK but no calls (3) | KO |
| Asterisk 12.3.0  | OK (1)                    |  OK                            | KO: registration OK but no calls (4)| KO | KO: registration OK but no calls (4) | KO |
| Elastix 2.4.0 (Asterisk 11.7.0) | OK (1)     |  OK                            | KO: registration OK but no calls | KO | KO: registration OK but no calls | KO |
| FreePBX 5.211.65-14 32bit (Asterisk 11.10.2)| KO (No audio) | KO (No audio)   | KO: registration OK but not calls (6)| KO  | KO: registration OK but no calls (3) | KO |
| FreePBX BETA-6.12.65-13 32bit (Asterisk 12.3.2) | OK (1) | OK                 | KO: registration OK but not calls (7)| KO  | KO: registration OK but no calls (5) | KO |


```
(1) - Chrome asks for sharing microphone on each call
(2) - chan_sip.c:10509 process_sdp: Rejecting secure audio stream without encryption details: audio 60349 UDP/TLS/RTP/SAVPF 109 0 8 101
(3) - _ast_realloc: Memory Allocation Failure in function __ast_websocket_read at line 456 of res_http_websocket.c
(4) - chan_sip.c:10638 process_sdp: Can't provide secure audio requested in SDP offer
(5) - _ast_realloc: Memory Allocation Failure in function __ast_websocket_read at line 504 of res_http_websocket.c
(6) - chan_sip.c:10509 process_sdp: Rejecting secure audio stream without encryption details: audio 57539 UDP/TLS/RTP/SAVPF 109 0 8 101
(7) - chan_sip.c:10648 process_sdp: Can't provide secure audio requested in SDP offer
```
