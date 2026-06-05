# Bussola Pública — Pipeline de Inteligência Legislativa

> Automação da coleta, classificação e distribuição de dados da Câmara dos Deputados usando Python, OpenAI Embeddings, PostgreSQL e n8n.

**Dashboard ao vivo:** [cesarramosjunior.github.io/bussula-publica](https://cesarramosjunior.github.io/bussula-publica)

---

## Contexto

Toda quarta-feira, 513 deputados decidem pautas que afetam diretamente o bolso, a empresa e a cidade dos brasileiros. Cada voto é registrado, cada proposição é catalogada, cada centavo da cota parlamentar é declarado — tudo isso é dado público, atualizado diariamente, disponível por uma API aberta e gratuita.

A **Bussola Pública** é uma consultoria fictícia de inteligência legislativa que vende monitoramento parlamentar para escritórios de advocacia, áreas de Relações Governamentais e associações setoriais. O objetivo deste projeto é automatizar a coleta e análise de dados da Câmara dos Deputados — hoje feita manualmente por dois analistas.

---

## Pipeline

```
API Câmara → Python (extração) → Pandas (transformação) → PostgreSQL/Supabase (carga)
                                                                    ↓
                                                         OpenAI Embeddings (classificação)
                                                                    ↓
                                                         n8n (relatório diário por email)
                                                                    ↓
                                                         Dashboard (GitHub Pages)
```

---

## Stack

| Tecnologia | Uso |
|---|---|
| Python + requests | Extração via API da Câmara dos Deputados |
| Pandas | Transformação e seleção de colunas |
| PostgreSQL + Supabase | Armazenamento na nuvem |
| OpenAI text-embedding-3-small | Classificação temática por similaridade semântica |
| n8n (Railway) | Automação do relatório diário |
| GitHub Pages | Dashboard público |

---

## Dados coletados

| Tabela | Registros | Descrição |
|---|---|---|
| proposicoes | 9.384 | Propostas legislativas |
| classificacoes | 9.301 | Classificação temática por embeddings |
| deputados | 513 | Cadastro dos parlamentares |
| partidos | 21 | Partidos políticos ativos |
| votacoes | 538 | Votações em plenário |
| despesas | 127 | Cota parlamentar (parcial) |

---

## Classificação por Embeddings

Cada proposição foi classificada automaticamente em um dos 10 temas usando o modelo `text-embedding-3-small` da OpenAI. O processo:

1. A ementa de cada proposição é enviada para a API da OpenAI
2. A API retorna um vetor de 1.536 números representando o significado semântico do texto
3. O mesmo processo é aplicado a cada um dos 10 temas pré-definidos
4. A similaridade de cosseno é calculada entre o vetor da proposição e o vetor de cada tema
5. O tema com maior similaridade é atribuído à proposição

**Temas monitorados:** Tributário, Saúde, Trabalho, Segurança, Educação, Infraestrutura, Direitos Civis, Meio Ambiente, Tecnologia, Economia

**Volume processado:** 9.301 proposições classificadas em ~12,5 horas de processamento

---

## Automação com n8n

Workflow publicado no n8n (hospedado no Railway) que roda todo dia às 06h:

```
Cron (06h) → Query PostgreSQL → Code (monta HTML) → Gmail (envia relatório)
```

O relatório diário inclui o total de proposições por tema e é enviado automaticamente para o email configurado.

O workflow exportado está disponível em [`Fluxo-Bussula-Pública.json`](./Fluxo-Bussula-Pública.json).

---

## Estrutura do Repositório

```
bussola-publica/
├── explorar_api.ipynb            # Etapa 1 — Exploração da API
├── extracao_proposicoes.ipynb    # Etapa 2 — Extração de proposições
├── extracao_deputados.ipynb      # Etapa 2 — Extração de deputados
├── extracao_partidos.ipynb       # Etapa 2 — Extração de partidos
├── extracao_votacoes.ipynb       # Etapa 2 — Extração de votações
├── extracao_despesas.ipynb       # Etapa 2 — Extração de despesas
├── transformacao.ipynb           # Etapa 3 — Transformação e carga no Supabase
├── ia_classificacao.ipynb        # Etapa 4 — Classificação por embeddings
├── carga_classificacao.ipynb     # Etapa 4 — Carga dos resultados no banco
├── Fluxo-Bussula-Pública.json    # Etapa 5 — Workflow n8n exportado
├── index.html                    # Dashboard público (GitHub Pages)
├── .gitignore
└── README.md
```

---

## Como rodar

### Pré-requisitos

```
pip install requests pandas python-dotenv sqlalchemy psycopg2-binary openai numpy
```

### Variáveis de ambiente

Crie um arquivo `.env` na raiz:

```
DATABASE_URL=postgresql://postgres:SENHA@db.seu-projeto.supabase.co:5432/postgres
OPENAI_API_KEY=sua-chave-aqui
```

### Ordem de execução

```
1. explorar_api.ipynb
2. extracao_*.ipynb (qualquer ordem)
3. transformacao.ipynb
4. ia_classificacao.ipynb
5. carga_classificacao.ipynb
```

---

## Limitações conhecidas

- Despesas coletadas parcialmente (~8 deputados dos 513)
- A classificação cobre 9.301 das 9.384 proposições (83 perdidas por quedas de conexão)
- A ingestão incremental ainda é executada manualmente — a automação completa do pipeline (extração + classificação + carga) via n8n é a próxima evolução do projeto
- Foreign keys não criadas por ausência de PRIMARY KEY explícita nas tabelas

---

## Fonte dos dados

- **API da Câmara dos Deputados:** [dadosabertos.camara.leg.br](https://dadosabertos.camara.leg.br)
- **Documentação:** [dadosabertos.camara.leg.br/swagger/api.html](https://dadosabertos.camara.leg.br/swagger/api.html)

---

*Projeto Integrador — Pós-Graduação em Dados | Junho de 2026*
