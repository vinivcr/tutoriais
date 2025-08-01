Perfeito, Vinícius! Vamos refazer **do zero**, agora com:

---

## ✅ ESPECIFICAÇÕES DO NOVO FLUXO

| Item                        | Valor                              |
| --------------------------- | ---------------------------------- |
| 📁 Projeto na VM            | `/home/prestapp/backend`           |
| 🌿 Branch usada             | `homolog`                          |
| 🐳 Nome da imagem Docker    | `springboot-api-homolog`           |
| 🐳 Nome do container Docker | `springboot-api-container-homolog` |
| 🔁 Trigger do deploy        | Push na branch `homolog`           |
| 🌐 Acesso via               | Cloudflare Tunnel (já configurado) |

---

# 🚧 PASSO A PASSO COMPLETO

---

## 🧱 1. Preparar a VM

Acesse sua VM e execute:

```bash
sudo apt update
sudo apt install -y openjdk-17-jdk maven docker.io git curl
sudo systemctl enable docker
sudo systemctl start docker
```

---

## 📂 2. Criar diretório `/home/prestapp/backend` e clonar o projeto

```bash
sudo useradd -m prestapp || true
sudo mkdir -p /home/prestapp/backend
sudo chown -R prestapp:prestapp /home/prestapp

# Como usuário prestapp
sudo su - prestapp
cd /home/prestapp/backend
git clone https://github.com/SEU_USUARIO/SEU_REPO.git .
git checkout homolog
```

---

## 🐳 3. Criar `Dockerfile` no projeto `/home/prestapp/backend`

```Dockerfile
# Etapa 1: Build
FROM maven:3.9.5-eclipse-temurin-17 AS build
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

# Etapa 2: Run
FROM eclipse-temurin:17-jdk
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 🤖 4. Instalar e configurar o GitHub Actions Runner na VM

### ⚙️ 4.1 Baixar e configurar o runner

```bash
mkdir ~/actions-runner-homolog && cd ~/actions-runner-homolog

curl -o actions-runner-linux-x64-2.314.1.tar.gz -L https://github.com/actions/runner/releases/download/v2.314.1/actions-runner-linux-x64-2.314.1.tar.gz
tar xzf actions-runner-linux-x64-2.314.1.tar.gz
```

---

### ⚙️ 4.2 Criar runner no GitHub

No repositório → **Settings > Actions > Runners > New self-hosted runner**

* Escolha `Linux x64`
* Copie o comando `./config.sh ...`

Execute na VM (como `prestapp`):

```bash
./config.sh --url https://github.com/SEU_USUARIO/SEU_REPO --token SEU_TOKEN
```

---

### ⚙️ 4.3 Iniciar como serviço

```bash
sudo ./svc.sh install
sudo ./svc.sh start
```

---

## 📄 5. Criar Workflow: `.github/workflows/deploy-homolog.yml`

No repositório, crie o arquivo:

```yaml
name: Deploy Homolog

on:
  push:
    branches: [ homolog ]

jobs:
  deploy:
    runs-on: self-hosted

    steps:
      - name: Clonar repositório
        uses: actions/checkout@v3

      - name: Navegar até o projeto
        run: cd /home/prestapp/backend

      - name: Parar e remover container antigo
        run: |
          docker stop springboot-api-container-homolog || true
          docker rm springboot-api-container-homolog || true

      - name: Remover imagem antiga
        run: |
          docker rmi springboot-api-homolog || true

      - name: Build imagem Docker
        run: |
          cd /home/prestapp/backend
          docker build -t springboot-api-homolog .

      - name: Subir novo container
        run: |
          docker run -d --name springboot-api-container-homolog -p 8080:8080 springboot-api-homolog
```

---

## 📦 6. Garantir permissões no diretório `/home/prestapp/backend`

Se o runner não estiver conseguindo acessar a pasta:

```bash
sudo chown -R prestapp:prestapp /home/prestapp
chmod -R 755 /home/prestapp
```

---

## 🔁 7. Deploy automático

Agora sempre que você fizer:

```bash
git push origin homolog
```

O GitHub Actions (via runner) vai:

* Atualizar o repositório em `/home/prestapp/backend`
* Compilar o projeto com Maven
* Criar nova imagem Docker: `springboot-api-homolog`
* Substituir o container por um novo: `springboot-api-container-homolog`

---

## ✅ Pronto!

Seu projeto estará rodando via Docker, acessível pela porta 8080 ou via Cloudflare Tunnel.

Se quiser, posso:

* Criar o `systemd` para rodar seu container como serviço
* Ajudar a mapear `.env` e variáveis no `docker run`
* Criar uma versão semelhante para produção (`main` branch)

Você quer que o container suba com variáveis de ambiente, `docker compose` ou `network` específica também?
