# 🔐 PortSwigger Web Security Academy — SQL Injection

![Status](https://img.shields.io/badge/status-em%20andamento-yellow)
![Tema](https://img.shields.io/badge/tema-SQL%20Injection-red)
![Licença](https://img.shields.io/badge/uso-educacional-blue)

Repositório dedicado às minhas anotações, lógica de resolução e aprendizados práticos adquiridos durante a realização dos laboratórios da [PortSwigger Web Security Academy](https://portswigger.net/web-security), com foco em vulnerabilidades de **SQL Injection**.

O objetivo deste material é documentar, de forma didática, tanto a **perspectiva do atacante** (como a falha é explorada) quanto a **perspectiva do defensor** (como corrigi-la), servindo como material de estudo e portfólio.

---
## 📖 Sobre este repositório

Cada laboratório documentado aqui segue um padrão fixo de organização, contendo:

1. **Descrição do Cenário** — contexto da aplicação vulnerável e a query executada no backend.
2. **Objetivo (Perspectiva de Ataque)** — o que precisa ser alcançado para resolver o laboratório.
3. **Resolução** — o payload utilizado e o passo a passo da exploração.
4. **Explicação Teórica** — por que o payload funciona, analisando a query final interpretada pelo banco.
5. **Mitigação e Prevenção (Perspectiva de Defesa)** — como um desenvolvedor deveria corrigir a falha.

---

## 🧨 O que é SQL Injection?

SQL Injection (SQLi) é uma vulnerabilidade que permite a um atacante interferir nas consultas que uma aplicação faz ao seu banco de dados. Isso ocorre quando dados fornecidos pelo usuário são inseridos diretamente em uma query SQL sem a devida sanitização ou parametrização, permitindo:

- Leitura de dados não autorizados (bypass de filtros e lógica de negócio);
- Extração de informações sensíveis de outras tabelas;
- Modificação ou exclusão de dados;
- Em cenários mais graves, execução de comandos no servidor.

---

## 🧪 Laboratórios Resolvidos

### 🛠️ Lab 1: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

#### 📝 Descrição do Cenário

O filtro de categorias de produtos da aplicação é vulnerável a SQL Injection na cláusula `WHERE`. Internamente, a aplicação executa a seguinte consulta:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

#### 🔓 Objetivo (Perspectiva de Ataque)

Explorar a vulnerabilidade para quebrar a lógica da consulta e forçar a aplicação a exibir **todos** os produtos do banco de dados, incluindo os produtos não lançados (`released = 0`).

#### 🚀 Resolução (Exploração na Prática)

A exploração foi feita manipulando diretamente o parâmetro `category` na URL, utilizando apenas o navegador:

```
https://<LAB-ID>.web-security-academy.net/filter?category=Gifts'%20OR%201=1--
```

#### 🧠 Explicação Teórica

Ao injetar o payload, a consulta final interpretada pelo banco de dados passou a ser:

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```

A quebra da lógica ocorreu devido a três fatores combinados:

- **Aspa simples (`'`)** — Fecha prematuramente a string de texto da categoria pesquisada (`'Gifts'`), encerrando o valor esperado pela aplicação.
- **`OR 1=1`** — Introduz uma condição logicamente universal (sempre verdadeira), forçando o banco a retornar todas as linhas da tabela, independentemente da categoria.
- **Comentário SQL (`--`)** — Comenta todo o restante da query original (`AND released = 1`), eliminando a restrição que ocultava os produtos não lançados.

#### 🛡️ Mitigação e Prevenção (Perspectiva de Defesa)

Para eliminar essa classe de vulnerabilidade, os desenvolvedores **nunca** devem concatenar ou injetar entradas do usuário diretamente em strings SQL.

A solução padrão de mercado é o uso de **Consultas Parametrizadas (Prepared Statements)**, nas quais o valor do usuário é tratado sempre como dado, nunca como código:

```sql
-- O banco de dados pré-compila a estrutura da query utilizando marcadores (?)
SELECT * FROM products WHERE category = ? AND released = 1
```

Boas práticas adicionais:

- Utilizar ORMs ou bibliotecas de acesso a dados que já implementem parametrização por padrão;
- Aplicar o princípio do **menor privilégio** para o usuário do banco de dados usado pela aplicação;
- Validar e restringir o formato dos inputs esperados (allowlist), quando aplicável;
- Realizar testes de segurança (SAST/DAST) e revisões de código focadas em pontos de entrada de dados externos.

---

## ▶️ Como usar este material

Este repositório não contém código executável — é um material de estudo em Markdown. Para acompanhar:

1. Clone o repositório:
   ```bash
   git clone https://github.com/<seu-usuario>/<seu-repositorio>.git
   ```
2. Navegue pelas seções deste README ou pela pasta `labs/`;
3. Recomendado: pratique os laboratórios diretamente na [PortSwigger Web Security Academy](https://portswigger.net/web-security), que é gratuita.

---

## ⚠️ Aviso Legal / Ética

Todo o conteúdo deste repositório tem **finalidade exclusivamente educacional**, baseado em laboratórios oficiais e legais da PortSwigger, projetados especificamente para prática de segurança ofensiva.

As técnicas aqui descritas **não devem ser aplicadas** em sistemas, aplicações ou infraestruturas para as quais você não possua autorização explícita. Testes de segurança não autorizados em sistemas de terceiros são ilegais em praticamente todas as jurisdições.

---

## 📎 Referências

- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [PortSwigger — SQL Injection](https://portswigger.net/web-security/sql-injection)
- [OWASP — SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)

---

## 👤 Felipe Santos

Documentação e resolução dos laboratórios mantidas como parte dos meus estudos em segurança ofensiva (Pentest / AppSec).

Sinta-se à vontade para abrir *issues* ou sugestões de melhoria na documentação.
