# Born Church — Sistema de Ingressos

Sistema completo de venda de ingressos e gestão de eventos: site público + painel administrativo.

## ✅ O que está pronto

**Site público**
- Lista de eventos publicados
- Página do evento com lotes de ingresso (com controle de esgotamento)
- Inscrição individual e em grupo
- Pagamento via Asaas: PIX dinâmico (QR Code) e cartão (checkout hospedado)
- Confirmação automática de pagamento (webhook) + ingresso digital com QR Code enviado por e-mail
- Lista de espera automática: se o lote esgotar, a pessoa entra na fila e é avisada por e-mail assim que uma vaga abrir

**Painel administrativo (`/admin`)**
- Login com senha
- CRUD completo de eventos (título, datas, local, imagem, status)
- Gestão de lotes de ingresso por evento
- Lista de inscrições e status de pagamento
- Check-in com leitor de QR Code pela câmera, em tempo real

**Stack**
- Backend: FastAPI + MongoDB (Motor)
- Frontend: React + TypeScript + Vite + TailwindCSS
- Pagamento: Asaas (PIX + cartão + webhook)
- E-mail: Gmail (SMTP, grátis)

---

## 🔑 Admin padrão
- E-mail: `admin@bornchurch.com.br`
- Senha: `LEqIEUQCoAzmZv`
- **Troque essa senha assim que fizer login pela primeira vez.**

---

## 🚀 Como colocar no ar (passo a passo)

Diferente do site da igreja, esse projeto **não dá pra publicar arrastando um arquivo**. Ele precisa de um repositório no GitHub conectado à Vercel, porque tem banco de dados e recebe avisos automáticos de pagamento. São mais passos, mas cada um é simples — siga na ordem.

### Antes de começar, você vai precisar de 2 contas (as duas grátis, sem cartão):
1. **GitHub** (github.com) — onde o código fica guardado
2. **Vercel** (você já tem)

Os e-mails dos ingressos são enviados pelo Gmail — de graça, sem precisar comprar domínio nenhum.

### Passo 1 — Subir o código pro GitHub
1. Crie uma conta em github.com (se ainda não tiver)
2. Crie um novo repositório (botão verde "New")
3. Nessa página do repositório recém-criado, clique em "uploading an existing file" e arraste TODOS os arquivos e pastas desse zip (mantendo a estrutura de pastas)
4. Confirme o commit

### Passo 2 — Gerar a senha de app do Gmail (pra e-mail funcionar, de graça)
Use uma conta do Gmail (pode ser uma nova, só pra isso, ou uma que você já tem).
1. Acesse myaccount.google.com/security e ative a **Verificação em duas etapas** (se ainda não estiver ativa)
2. Depois, acesse myaccount.google.com/apppasswords
3. Crie uma senha de app (dê um nome tipo "Born Church Ingressos") e copie o código de 16 letras que aparece — você só vê ele uma vez

### Passo 3 — Importar o projeto na Vercel
1. No painel da Vercel, clique em **Add New → Project**
2. Escolha o repositório que você acabou de criar no GitHub
3. Clique em **Deploy** (o primeiro deploy pode falhar por falta das variáveis — normal, a gente ajusta no próximo passo)

### Passo 4 — Adicionar o banco de dados (MongoDB)
1. Dentro do projeto na Vercel, vá na aba **Storage**
2. Clique em **Create Database → MongoDB Atlas**
3. Siga o assistente (ele cria a conta do Atlas e conecta automaticamente, já preenchendo a variável `MONGODB_URI` pra você)

### Passo 5 — Configurar as variáveis de ambiente
Ainda no projeto, vá em **Settings → Environment Variables** e adicione (os valores já vêm prontos no arquivo `.env` deste zip — é só copiar e colar):

| Nome | Valor |
|---|---|
| `JWT_SECRET` | (copie do `.env`) |
| `SETUP_TOKEN` | (copie do `.env`) |
| `ASAAS_API_KEY` | (copie do `.env` — sua chave do Asaas) |
| `ASAAS_ENV` | `production` |
| `ASAAS_WEBHOOK_TOKEN` | (copie do `.env`) |
| `PIX_KEY` | `48.891.677/0001-93` |
| `GMAIL_ADDRESS` | o e-mail do Gmail que você usou no Passo 2 |
| `GMAIL_APP_PASSWORD` | a senha de app de 16 letras do Passo 2 |
| `EMAIL_FROM_NAME` | `Born Church` |
| `PUBLIC_BASE_URL` | o endereço do seu site na Vercel (ex: `https://born-tickets.vercel.app`) |

Depois de adicionar todas, vá na aba **Deployments** e clique em "Redeploy" no último deploy.

### Passo 6 — Criar o admin (uma única vez)
Depois que o redeploy terminar, abra no navegador (trocando pelos seus valores):
```
https://SEU-SITE.vercel.app/api/setup?token=SEU_SETUP_TOKEN
```
Isso cria o admin padrão e prepara o banco. Você só precisa fazer isso uma vez.

### Passo 7 — Configurar o webhook do Asaas
1. No painel do Asaas, vá em **Integrações → Webhooks**
2. Cadastre a URL: `https://SEU-SITE.vercel.app/api/webhooks/asaas`
3. Em "Token de autenticação", cole o mesmo valor de `ASAAS_WEBHOOK_TOKEN`
4. Marque pelo menos os eventos: `PAYMENT_CONFIRMED`, `PAYMENT_RECEIVED`, `PAYMENT_OVERDUE`

### Passo 8 — Testar
1. Acesse `https://SEU-SITE.vercel.app/admin/login` e entre com o admin padrão
2. Crie um evento, publique, crie um lote de ingresso
3. Acesse a home do site e faça uma inscrição de teste
4. Confirme que o e-mail do ingresso chega

---

## 📁 Estrutura de pastas
```
born-tickets/
├── api/                    → backend FastAPI
│   ├── index.py            → ponto de entrada
│   ├── config.py           → variáveis de ambiente
│   ├── database.py         → conexão MongoDB
│   ├── models.py           → formatos de dados
│   ├── auth.py             → login do admin
│   ├── asaas_client.py     → integração de pagamento
│   ├── email_service.py    → envio de e-mail (Gmail SMTP)
│   ├── ticket_service.py   → geração de QR Code dos ingressos
│   ├── seed.py             → alternativa ao /api/setup (rodar localmente)
│   └── routers/            → rotas da API
├── frontend/                → site (React + Vite + Tailwind)
│   └── src/
│       ├── pages/           → páginas públicas
│       └── pages/admin/     → páginas do painel
├── vercel.json              → configuração de publicação
├── .env                     → suas chaves reais (NUNCA suba pro GitHub — já protegido pelo .gitignore)
└── .env.example              → modelo em branco
```

## ⚠️ Segurança
- O arquivo `.env` tem chaves reais e já está protegido pelo `.gitignore` — confira antes de subir pro GitHub que ele não aparece na lista de arquivos enviados.
- Troque a senha do admin assim que puder.
- Depois de rodar `/api/setup` uma vez, não precisa visitar de novo.
