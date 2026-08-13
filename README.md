# FastTrack Windows Agent — Releases

Distribuição pública de releases do FastTrack Windows Agent (cliente
desktop do CRM WhatsApp da Fibra Fácil).

O código-fonte do FastTrack (backend + Windows Agent) permanece em um
repositório privado. Este repositório existe só para hospedar os
artefatos binários e o manifesto de auto-update em um canal HTTPS
público, sem expor o código-fonte nem exigir credenciais embutidas no
Agent instalado (WINAGENT-F11C).

Cada release publica:

- `fasttrack-agent-<versão>-win64.zip` — artefato assinado (Ed25519)
- `fasttrack-agent-<versão>-win64.zip.sha256.txt` — checksum
- `manifest.json` — manifesto consumido pelo mecanismo de auto-update do Agent
