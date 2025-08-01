Beleza! Para criar um **Cloudflare Tunnel**, você pode usar o `cloudflared` (cliente oficial da Cloudflare) e o painel Zero Trust. Abaixo te mostro o passo a passo completo com linha de comando.

---

## ✅ Requisitos

1. Conta no Cloudflare
2. Um domínio já adicionado à sua conta (ex: `seudominio.com`)
3. `cloudflared` instalado

---

## 🛠️ Passo a Passo para Criar um Tunnel

### 1. Instale o `cloudflared`

**Linux (Debian/Ubuntu):**

```bash
sudo apt install cloudflared
```

**Windows:**
Baixe o `.exe` aqui:
👉 [https://developers.cloudflare.com/cloudflared/install/](https://developers.cloudflare.com/cloudflared/install/)

---

### 2. Faça login na sua conta Cloudflare

```bash
cloudflared login
```

Ele vai abrir o navegador e pedir para você escolher o domínio.

---

### 3. Crie o Tunnel

```bash
cloudflared tunnel create meu-tunel
```

Isso vai gerar:

* Um `Tunnel ID`
* Um `credentials file` (salvo em `~/.cloudflared/`)

---

### 4. Configure a conexão com o serviço local

Crie um arquivo `config.yml` no diretório:

```bash
~/.cloudflared/config.yml
```

Exemplo de conteúdo para expor um serviço rodando localmente na porta 8000:

```yaml
tunnel: meu-tunel
credentials-file: /home/seuusuario/.cloudflared/meu-tunel.json

ingress:
  - hostname: app.seudominio.com
    service: http://localhost:8000
  - service: http_status:404
```

> Altere o caminho do `credentials-file` conforme o seu sistema e o hostname com seu domínio real.

---

### 5. Aponte seu subdomínio para o tunnel

Adicione no painel da Cloudflare uma entrada DNS:

| Tipo  | Nome (hostname)      | Conteúdo                       |
| ----- | -------------------- | ------------------------------ |
| CNAME | `app.seudominio.com` | `uuid.tunnel.cfargotunnel.com` |

(O UUID é o Tunnel ID gerado)

---

### 6. Inicie o Tunnel

```bash
cloudflared tunnel run meu-tunel
```

Agora, seu serviço local (porta 8000) está acessível publicamente via `https://app.seudominio.com`.

---

Se quiser que ele **rode em background como serviço**, me avise que te mostro como configurar isso.

Quer fazer agora mesmo um exemplo com domínio `.trycloudflare.com` sem configurar domínio próprio?
