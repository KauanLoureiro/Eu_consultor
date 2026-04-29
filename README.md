# 🛒 SQL & Python: Cases de Consultoria de Dados

![Python](https://img.shields.io/badge/Python-3.12-blue?logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-2.x-150458?logo=pandas&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Local-336791?logo=postgresql&logoColor=white)
![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-ORM-red)
![Status](https://img.shields.io/badge/Status-Finalizado-green)

---

## 💡 Sobre o Projeto

Este repositório documenta minha jornada em um **"Estágio Simulado de Engenharia de Dados"**.

Para fugir dos tutoriais guiados e enfrentar problemas reais, utilizei uma IA atuando como **Gestor de Dados Fictício**. Em cada case, assumo o papel de **Engenheiro de Dados** contratado por uma empresa de um setor diferente. Meu objetivo é simples e direto: **receber dados brutos e sujos, e entregá-los limpos, modelados e prontos para consumo**.

Os dados que produzo servem a dois times downstream:

- 👨‍💼 **Time de Análise de Dados** — que consome a camada silver e gold para extrair insights de negócio via SQL.
- 🤖 **Time de Ciência de Dados** — que utiliza os dados estruturados e modelados na camada gold para treinar modelos de churn, previsão de demanda, score de risco, entre outros.

A dinâmica de cada case consiste em:

- Receber um dataset propositalmente **"sujo"** — datas em múltiplos formatos, valores monetários inconsistentes, duplicatas, nulos e capitalização variada.
- Realizar a **limpeza e transformação** com Pandas e carregar na camada silver do PostgreSQL.
- Responder **perguntas de negócio reais** com SQL, simulando as demandas que chegariam dos times de Analytics e Data Science.

---

## 🎯 Habilidades Treinadas

Este projeto foi estruturado para desenvolver especificamente três competências core de Engenharia de Dados:

| Habilidade | O que é praticado |
|---|---|
| 🐼 **Limpeza com Pandas** | Tratamento de nulos, padronização de tipos, imputação, remoção de duplicatas, regex em strings |
| 🏅 **Arquitetura Medalhão** | Ingestão na camada raw → transformação → entrega na camada silver no PostgreSQL |
| 🗄️ **SQL Analítico** | GROUP BY, CTEs, Window Functions, JOINs entre subqueries, análise temporal |

---

## 🛠 Tecnologias Utilizadas

- **Linguagem:** Python
- **Manipulação de Dados:** Pandas, NumPy
- **Banco de Dados:** PostgreSQL (local)
- **Conexão Python ↔ Banco:** psycopg2, SQLAlchemy

---

## 🏗 Arquitetura do Projeto — Camadas Medalhão

Todos os cases seguem a mesma arquitetura de dados em camadas:

```
┌─────────────────────────────────────────────────────────────┐
│  🥉 BRONZE                                                  │
│  CSV sujo recebido do sistema legado da empresa             │
│  Sem nenhuma transformação — dado original preservado       │
└───────────────────────────┬─────────────────────────────────┘
                            │  Python + Pandas
                            │  (limpeza, padronização, imputação)
┌───────────────────────────▼─────────────────────────────────┐
│  🥈 SILVER                                                  │
│  Dados limpos e estruturados                                │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            |
┌───────────────────────────▼─────────────────────────────────┐
│  🥇 GOLD                                                    │
│  Dados Prontos para geração de insights e atuação em modelos│
└───────────────────────────┬─────────────────────────────────┘
                            |
               ┌────────────┴────────────┐
               │                         │
┌──────────────▼──────────┐ ┌────────────▼──────────────────┐
│  👨‍💼 Analytics           │ │  🤖 Data Science             │
│  Perguntas de negócio   │ │  Modelos de churn,            │
│  respondidas via SQL    │ │  previsão, score de risco...  │
└─────────────────────────┘ └───────────────────────────────┘
```

Cada case possui seu próprio **schema** no PostgreSQL, garantindo isolamento total:

```sql
case_1.vendas_silver      -- Moda Fácil Ltda.
case_2.consultas_silver   -- VidaMais Clínicas S.A.
...
```

---

## 📂 Detalhes dos Cases

### 1️⃣ Case: Varejo de Moda — Moda Fácil Ltda.

🏢 **Contexto:** Rede de lojas físicas presente em várias capitais brasileiras. O time de dados recebeu um dump do sistema legado de vendas dos últimos 2 anos e precisava limpar, modelar e responder perguntas estratégicas para a diretoria.

- **Desafio Técnico:** Dataset com datas em 3 formatos distintos (`YYYY-MM-DD`, `DD/MM/YYYY`, `YYYY/MM/DD`), preços com prefixo `R$` e vírgula decimal, capitalização inconsistente em produto/cidade/cliente, quantidades negativas e ~15 duplicatas embaralhadas. Imputação de preços nulos pela **média do próprio produto**.
- **ETL:** Limpeza com Pandas → geração de IDs únicos com **UUID v7** → carga via `COPY` no PostgreSQL com schema dedicado `case_1`.
- **SQL:** 5 perguntas de negócio respondidas via queries, incluindo **Window Function** (`SUM OVER`) para faturamento acumulado e **CTEs duplas** para análise trimestral.

---

### 2️⃣ Case: Saúde — VidaMais Clínicas S.A.

🏢 **Contexto:** Rede de clínicas médicas multiespecialidade. O time de gestão precisava entender performance de médicos, receita por convênio e padrões de cancelamento para orientar decisões estratégicas.

- **Desafio Técnico:** Datas em 3 formatos, valores monetários sujos, status de consulta em múltiplas capitalizações (`"realizada"`, `"REALIZADA"`), avaliações fora do range válido (0, 6, -1), nomes de médicos sem prefixo de título e ~18 duplicatas, além de uso de regex para replaces mais escaláveis.
- **ETL:** Padronização de categorias, validação de range de avaliações, limpeza de strings → carga na camada silver no schema `case_2`.
- **SQL:** 5 perguntas respondidas com dificuldade crescente — de `GROUP BY` simples até **CTEs + JOIN entre subqueries** para comparar perfis de pacientes.

---

### 3️⃣ Case: Distribuição de Insumos Agrícolas — Distribuidora Atalaia 🌿

🏢 **Contexto:** Distribuidora de insumos agrícolas com atuação no Norte e Nordeste do Brasil. Pela primeira vez no projeto, os dados chegaram em **duas tabelas relacionadas** — clientes e pedidos — exigindo modelagem relacional real antes de qualquer análise. O time de Data Science sinalizou interesse em usar a camada silver para futuros modelos de **predição de churn**.

- **Desafio Técnico:** Dataset com estrutura relacional: `clientes.csv` (40 registros, `cliente_id` como PK) e `pedidos.csv` (100 registros, `cliente_id` como FK). Sujeiras incluíam e-mails com capitalização inconsistente (`MARIA.SOUZA@...`), telefones sem padrão, cidades com case misto (`"são paulo"` vs `"São Paulo"`), 3 registros sem telefone, `cliente_id = 0` e `cliente_id = 99` como **FKs órfãs** (não existem em clientes), status inválido `"-1"` no pedido 1049 e `data_entrega` nula em pedidos em andamento e cancelados.
- **ETL:** Limpeza e padronização nas duas tabelas → validação de integridade referencial (remoção de FKs órfãs) → carga relacional na camada silver no schema `case_3`.
- **SQL:** 5 perguntas com dificuldade crescente — do `SELECT` + `WHERE` básico até **CTEs + análise de churn**, passando por `Window Functions` com `RANK` e `PARTITION BY` por vendedor.
- **Insight de Negócio:** A query para preparação para analise de churn com clientes com ao menos 2 pedidos em 2023 sem atividade nos últimos 90 dias — entregando uma lista acionável para o time comercial acionar campanhas de retenção, e uma base estruturada para o time de DS iniciar um modelo de score de risco.

---

Autor: Kauan Amorim
linkedin: https://www.linkedin.com/in/kauanamorim1/
