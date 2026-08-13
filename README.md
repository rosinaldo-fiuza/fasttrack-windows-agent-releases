# FastTrack Windows Agent — Releases

Distribuição pública de releases do FastTrack Windows Agent (cliente
desktop do CRM WhatsApp da Fibra Fácil).

O código-fonte do FastTrack (backend + Windows Agent) permanece em um
repositório privado. Este repositório existe só para hospedar os
artefatos binários e o manifesto de auto-update em um canal HTTPS
público, sem expor o código-fonte nem exigir credenciais embutidas no
Agent instalado.

Cada release publica:

- `fasttrack-agent-<versão>-win64.zip` — artefato assinado (Ed25519)
- `fasttrack-agent-<versão>-win64.zip.sha256.txt` — checksum
- `manifest.json` — manifesto consumido pelo mecanismo de auto-update do Agent

## Requisitos

- Windows 10/11 64-bit.
- Um código de enrollment válido, gerado por um supervisor/admin do
  FastTrack em `/admin/agents`. Sem esse código, o Agent instala mas não
  se conecta a nenhuma conta.

**Aviso de assinatura de código:** o executável ainda não possui
certificado Authenticode. O Windows/SmartScreen pode exibir um aviso de
"Editor desconhecido" na primeira execução — isso é esperado. Por isso a
conferência do checksum abaixo é obrigatória, não opcional.

## 1. Baixar e conferir o checksum (obrigatório)

**Nunca instale um `.zip` baixado sem conferir o checksum primeiro.**
Isso vale mesmo sem assinatura de código: o checksum publicado junto ao
pacote é a única garantia de que o arquivo baixado é exatamente o
artefato desta release, sem adulteração.

1. Baixe `fasttrack-agent-<versão>-win64.zip` e o arquivo
   `fasttrack-agent-<versão>-win64.zip.sha256.txt` na página de
   [Releases](../../releases) deste repositório.
2. Confira no PowerShell:

   ```powershell
   Get-FileHash .\fasttrack-agent-<versão>-win64.zip -Algorithm SHA256
   Get-Content .\fasttrack-agent-<versão>-win64.zip.sha256.txt
   ```

3. O hash calculado deve bater exatamente com o valor do arquivo
   `.sha256.txt` e com o campo `"sha256"` de `manifest.json`. Se não
   bater, **não instale** — descarte o arquivo e baixe novamente desta
   página de releases.

## 2. Instalar

Extraia o `.zip` e rode `install.ps1` (dentro de `build\`) a partir do
pacote extraído:

```powershell
# Sem enrollment automático (fazer depois manualmente):
.\build\install.ps1 -ZipPath .\fasttrack-agent-<versão>-win64.zip

# Com enrollment automático (código gerado em /admin/agents) e autostart:
.\build\install.ps1 -ZipPath .\fasttrack-agent-<versão>-win64.zip `
    -EnrollCode XXXX-XXXX-XX -EnableAutostart
```

O script instala em `%LOCALAPPDATA%\FastTrackAgent\app` e move
automaticamente qualquer instalação anterior para
`%LOCALAPPDATA%\FastTrackAgent\app.previous` (usado no rollback manual,
seção 6).

## 3. Enrollment manual (se não usou `-EnrollCode`)

```powershell
& "$env:LOCALAPPDATA\FastTrackAgent\app\fasttrack-agent.exe" --enroll XXXX-XXXX-XX
```

O código é gerado por um supervisor/admin em `/admin/agents` no
dashboard FastTrack e é de uso único.

## 4. Autostart

```powershell
$exe = "$env:LOCALAPPDATA\FastTrackAgent\app\fasttrack-agent.exe"
& $exe --enable-autostart   # liga
& $exe --disable-autostart  # desliga
```

## 5. Executar manualmente (sem autostart)

```powershell
& "$env:LOCALAPPDATA\FastTrackAgent\app\fasttrack-agent.exe"
```

Um ícone aparece na bandeja do sistema quando o Agent está conectado.

## 6. Rollback manual

`install.ps1` sempre guarda uma versão anterior em `app.previous`.
Para voltar a ela:

```powershell
$root = "$env:LOCALAPPDATA\FastTrackAgent"
$exe = "$root\app\fasttrack-agent.exe"

& $exe --disable-autostart
Get-Process fasttrack-agent -ErrorAction SilentlyContinue | Stop-Process

Rename-Item "$root\app" "$root\app.rolled-back"
Rename-Item "$root\app.previous" "$root\app"

& "$root\app\fasttrack-agent.exe" --enable-autostart
Start-Process "$root\app\fasttrack-agent.exe"

# depois de confirmar que reconectou:
Remove-Item -Recurse -Force "$root\app.rolled-back"
```

Para voltar a uma versão mais antiga que a imediatamente anterior,
baixe o `.zip` dessa versão nesta página de releases (conferindo o
checksum, seção 1) e rode `install.ps1` normalmente — ele já move a
instalação atual para `app.previous` antes de aplicar a antiga.

## 7. Atualização automática

O Agent verifica periodicamente (a cada ~6h, com variação aleatória) se
há uma versão nova publicada neste repositório. A atualização só é
aplicada se **todas** as verificações abaixo passarem — nenhuma delas é
opcional:

1. checksum SHA-256 do artefato baixado bate com o `manifest.json`;
2. assinatura Ed25519 do artefato é válida contra a chave pública
   embutida no Agent instalado;
3. a versão do manifesto é mais nova que a versão instalada
   (anti-downgrade);
4. o build novo passa em um self-check isolado (autentica de verdade
   contra o backend) antes de ser promovido.

Se qualquer verificação falhar, a versão atual continua rodando sem
interrupção — o Agent nunca promove um build que falhou no self-check.

A atualização automática é controlada por uma configuração do lado do
servidor (não pelo usuário do Agent) e pode estar desativada por
padrão. Enquanto estiver desativada, atualizações são manuais: repita o
processo de instalação (seções 1–2) com a versão nova.

## 8. Desinstalar

```powershell
$exe = "$env:LOCALAPPDATA\FastTrackAgent\app\fasttrack-agent.exe"
& $exe --disable-autostart
Get-Process fasttrack-agent -ErrorAction SilentlyContinue | Stop-Process
Remove-Item -Recurse -Force "$env:LOCALAPPDATA\FastTrackAgent"
```

Isso não revoga o dispositivo no servidor — para isso, um
supervisor/admin precisa revogar em `/admin/agents`.
