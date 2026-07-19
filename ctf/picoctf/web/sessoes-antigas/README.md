# 🍪 Sessões Antigas — picoCTF 2026

<p align="center">
  <img src="https://img.shields.io/badge/Categoria-Explora%C3%A7%C3%A3o%20da%20Web-blue" />
  <img src="https://img.shields.io/badge/Dificuldade-F%C3%A1cil-brightgreen" />
  <img src="https://img.shields.io/badge/Evento-picoCTF%202026-orange" />
  <img src="https://img.shields.io/badge/Status-Resolvido%20%E2%9C%94-success" />
</p>

> **Autor do desafio:** David Gaviria
> **Vulnerabilidade:** Gerenciamento inadequado de expiração de sessão (Session Hijacking)

---

## 📖 Descrição do desafio

O controle adequado do tempo limite de sessão é crucial para a segurança
das contas de usuário. Se um usuário fizer login em um computador público
ou compartilhado, mas não encerrar a sessão explicitamente, e as datas de
expiração da sessão estiverem configuradas incorretamente, a sessão pode
permanecer ativa indefinidamente.

Isso permite que um invasor, usando o mesmo navegador posteriormente,
acesse a conta do usuário sem precisar de credenciais.

<p align="center">
  <img width="768" height="505" alt="Descricao" src="https://github.com/user-attachments/assets/fc5caff2-c9aa-4d51-8c6d-c80ad1dbc949" />

</p>

---

## 🗺️ Metodologia

```
Reconhecimento → Login como usuário comum → Ler comentários → 
Descobrir /sessions → Roubar cookie do admin → Capturar a flag
```

---

## 🔎 Passo a passo

### 1️⃣ Reconhecimento — Tela de login

A aplicação apresenta um sistema de login padrão, sem nada explorável
à primeira vista.

<p align="center">
  <img width="615" height="351" alt="paginalogin" src="https://github.com/user-attachments/assets/51d47ab7-28f1-460f-b594-28eeea22018e" />

</p>

---

### 2️⃣ A pista escondida nos comentários

Após criar uma conta e logar como usuário comum, a homepage exibe uma
seção de comentários. Um deles, de `mary_jones_8992`, entrega a pista:

> 💬 *"Hey I found a strange page at /sessions"*

<p align="center">
  <img width="803" height="649" alt="Informacao" src="https://github.com/user-attachments/assets/7c255d39-58e4-4096-abf8-a85b598cbfc0" />

</p>

📌 **Lição:** sempre vasculhar conteúdo gerado por usuários (comentários,
posts, bios) — é um esconderijo clássico de pistas em CTFs.

---

### 3️⃣ Endpoint `/sessions` exposto

Acessando `dolphin-cove.picoctf.net:PORTA/sessions`, o servidor devolve —
**sem qualquer autenticação** — a lista completa de sessões ativas:

```
1) session:Z9CpjHKYG9UVz1MdGGB0Uzbj4D4TH3UsjmMvu9fmZ5Y, {'_permanent': True, 'key': 'admin'}
2) session:kGMQVdtaRSyAmdaVpQNAYxJ_YSFc2Y1ECFU76PraOEE, {'_permanent': True, 'key': 'Mancomputer'}
```

<p align="center">
  <img width="777" height="130" alt="sessions" src="https://github.com/user-attachments/assets/c57f7cf4-9129-41b5-848a-dd65ce418f24" />

</p>

🚨 O campo `'_permanent': True` confirma a causa raiz: essas sessões
**nunca expiram** — inclusive a do usuário `admin`.

---

### 4️⃣ Sequestro de sessão (session hijacking)

Com o token do `admin` em mãos, bastou:

1. Abrir **DevTools** (`F12`) → aba **Application** → **Cookies**
2. Localizar o cookie `session`
3. Substituir o valor pelo token roubado do admin
4. Recarregar a página

<p align="center">
 <img width="615" height="319" alt="token" src="https://github.com/user-attachments/assets/95b9c375-eae0-4cbb-b039-567af7a57f71" />

</p>

O servidor aceitou o cookie forjado como válido, sem qualquer verificação
adicional — autenticando o navegador diretamente como `admin`.

---

### 5️⃣ 🏁 Flag capturada!

<p align="center">
  <img width="593" height="196" alt="bandeira" src="https://github.com/user-attachments/assets/f141e7dc-f47a-4ed2-a8dd-b279d471a514" />

</p>

```
picoCTF{***************************}
```

---

## 🧠 Causa raiz

| Falha | Impacto |
|---|---|
| Sessões marcadas como `_permanent: True` sem expiração | Tokens antigos continuam válidos indefinidamente |
| Endpoint `/sessions` exposto publicamente, sem autenticação | Vazamento direto de tokens de qualquer usuário, incluindo admins |

## 🛡️ Como corrigir

- ⏱️ Definir expiração real para cookies de sessão (ex.: 15–30 min de inatividade)
- 🔒 Nunca expor endpoints de debug/administração de sessão em produção
- 🚪 Invalidar sessões no servidor ao fazer logout — não depender só do cliente
- 🍪 Usar cookies com `HttpOnly`, `Secure` e `SameSite`
- 🔁 Rotacionar identificadores de sessão após autenticação

## 🧰 Ferramentas utilizadas

- Navegador + DevTools (`Application → Cookies`)
- Nenhuma ferramenta de interceptação de tráfego foi necessária — a
  vulnerabilidade foi encontrada por navegação manual e leitura de
  conteúdo exposto pela própria aplicação

---

<p align="center"><i>Writeup elaborado para fins educacionais — desafio legítimo da plataforma picoCTF / CyLab Security Academy.</i></p>
