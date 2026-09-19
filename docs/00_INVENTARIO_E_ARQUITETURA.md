# PRW Protheus - Engenharia Reversa Completa
## Recanto da Saudade Cemitério Parque Ltda

**Data de Análise:** 19/09/2026  
**Status:** ⚠️ BLOQUEADORES CRÍTICOS ENCONTRADOS  
**Prioridade de Ação:** IMEDIATA (próximas 48h)

---

## CHECKPOINT 01 — INVENTÁRIO TÉCNICO

### 1.1 Visão Geral do Sistema

| Aspecto | Descrição |
|---------|-----------|
| **Domínio** | Financeiro (Receitas, Contratos, Importações) |
| **Plataforma** | TOTVS Protheus (ERP) |
| **Linguagem** | AdvPL (.PRW) |
| **Total Linhas** | 45.819 linhas de código |
| **Arquivos** | 13 programas .PRW |
| **Banco Dados** | Protheus nativo (tabelas financeiras) |
| **Organização** | Recanto da Saudade |
| **CNPJ** | 04.709.150/0001-20 |
| **Objetivo** | Migração de sistema legado para Protheus |

### 1.2 Mapeamento de Arquivos

#### ✅ PROGRAMAS FUNCIONAIS
| Arquivo | Linhas | Função | Status |
|---------|--------|--------|--------|
| **SUPORTE.PRW** | 171 | Manutenção SX5/SZ5 | ✅ OK |
| **FUNCOES.PRW** | 4.691 | Biblioteca de helpers | ✅ OK |
| **CONTRATO.PRW** | 4.143 | Gestão contratos U00/U03 | ✅ OK |
| **RIMPC003.PRW** | ~ | Transações receitas | ✅ OK |
| **RECP002.PRW** | ~ | Processamento SE1 | ✅ OK |
| **RECP015.PRW** | ~ | Importação dados | ✅ OK |
| **RECP017.PRW** | ~ | Validação receitas | ✅ OK |

**Total Funcional:** 7 programas ✅

#### ❌ PROGRAMAS COM PROBLEMAS

**CRÍTICO — Não Executa:**
- **RECP016.PRW** | ~7.500 linhas
  - Função: Gerador de Pedidos Vendas (SC5) a partir de SE1
  - Erro linha 693: `nEXT` (sintaxe inválida, deveria ser `NEXT`)
  - Variáveis não inicializadas: `_cAliasTmp`, `_cPastaQry`, `nTotReg`, `cCadPrb`, `nPedItem`
  - **Ação:** Correção prioritária (próximas 48h)

**MONOLÍTICO — Refatoração Necessária:**
- **RECP001.PRW** | 11.471 linhas
  - Função: Importação grandes volumes SE1/SA1
  - Problema: Validações misturadas com lógica de transformação
  - Recomendação: Dividir em 3 módulos (Validação/Transformação/Execução)

- **SYSEXTRATOR.PRW** | 11.418 linhas
  - Função: Filtros/consultas financeiras avançadas
  - Problema: Monolítico, difícil manutenção
  - Recomendação: Dividir por funcionalidade

**AVISOS — Impacto Médio:**
- **VIRIMPCLI.PRW**
  - Código SQL comentado e queries incompletas
  - Hardcoding filial "01" (impossibilita reutilização)

- **VIRIMPFIN.PRW**
  - Mesmo padrão: SQL comentado
  - Hardcoding de parametrização

#### 📊 ESTATÍSTICAS

- Total Programas: **13** 
- Funcionais: **7** (54%)
- Com Avisos: **2** (15%)
- Monolíticos: **2** (15%)
- Críticos (não executa): **1** (8%)
- Código Duplicado: **~3.200 linhas** (importação em múltiplos programas)
- Linhas por Programa (média): **3.525**

---

## CHECKPOINT 02 — ARQUITETURA & COMPONENTES

### 2.1 Fluxo Geral

```
DADOS EXTERNOS (Legacy)
    ↓
IMPORTAÇÃO (RECP001, RECP015, VIRIMPCLI)
    ↓
VALIDAÇÃO (RECP017, FUNCOES)
    ↓
TRANSFORMAÇÃO (RIMPC003, CONTRATO)
    ↓
GERAÇÃO DE PEDIDOS (RECP016) ← ❌ BLOQUEADO
    ↓
BANCO PROTHEUS (SE1, SC5, SA1, U00, U03)
```

### 2.2 Módulos por Funcionalidade

#### **Módulo 1: Validação & Preparação**
- `RECP017.PRW` — Validações receitas
- `FUNCOES.PRW` — Funções de suporte (4.691 linhas)
- `SUPORTE.PRW` — Manutenção tabelas SX5/SZ5

#### **Módulo 2: Importação**
- `RECP001.PRW` — **Monolítico (11.471 linhas)** — SE1/SA1 bulk
- `RECP015.PRW` — Importação específica
- `RECP002.PRW` — Processamento SE1
- `VIRIMPCLI.PRW` — Clientes (SQL comentado)
- `VIRIMPFIN.PRW` — Financeiro (SQL comentado)

#### **Módulo 3: Transformação & Contratos**
- `CONTRATO.PRW` — Gestão U00/U03 (4.143 linhas)
- `RIMPC003.PRW` — Transações básicas

#### **Módulo 4: Orquestração**
- `RECP016.PRW` — **❌ GERADOR PEDIDOS (CRÍTICO)**
- `SYSEXTRATOR.PRW` — **Monolítico (11.418 linhas)** — Filtros avançados

#### **Módulo 5: Biblioteca Utilitária**
- `FUNCOES.PRW` — Helpers (bem estruturado, 4.691 linhas)

### 2.3 Dependências Entre Módulos

```
FUNCOES.PRW (Biblioteca)
    ↓ (importado por)
    ├─→ RECP001.PRW (Importação)
    ├─→ RECP017.PRW (Validação)
    ├─→ RIMPC003.PRW (Transações)
    └─→ CONTRATO.PRW (Contratos)

SUPORTE.PRW (Manutenção)
    ↓ (usado por)
    └─→ RECP017.PRW (Validações)

RECP016.PRW ← depende de
    ├─→ SE1 (Títulos baixados)
    ├─→ SC5 (Pedidos vendas — saída)
    └─→ FUNCOES.PRW
```

---

## CHECKPOINT 03 — DADOS E LINHAGEM

### 3.1 Tabelas Envolvidas

#### **Entrada (Origem)**
| Tabela | Campo PK | Descrição |
|--------|----------|-----------|
| **SA1** | A1_FILIAL + A1_COD | Clientes |
| **SE1** | E1_FILIAL + E1_PREFIXO + E1_NUM + E1_PARCELA | Títulos a receber |
| **SX5** | X5_FILIAL + X5_TABELA + X5_CHAVE | Tabelas customizadas |
| **SZ5** | Variável | Parâmetros sistema |

#### **Processamento (Intermediárias)**
| Tabela | Uso | Risco |
|--------|-----|-------|
| **U00** | Contratos (customizada) | Sem FK formalizada |
| **U03** | Detalhes contratos (customizada) | Sem FK formalizada |

#### **Saída (Destino)**
| Tabela | Campo PK | Descrição | Origem |
|--------|----------|-----------|--------|
| **SC5** | C5_FILIAL + C5_NUM | Pedidos vendas | RECP016 gera |

### 3.2 Linhagem de Dados (Data Lineage)

```
Legacy System
    ↓ (arquivo/planilha)
    └─→ RECP001/RECP015 (Lê)
        ↓ (valida/transforma)
        └─→ RECP017 (Validação)
            ↓ (confirma)
            ├─→ SE1 (insere/atualiza)
            ├─→ SA1 (insere/atualiza)
            └─→ U00/U03 (cria contratos)
                ↓ (base para)
                └─→ RECP016 ❌ (gera SC5 — BLOQUEADO)
```

### 3.3 Campos Críticos Não Utilizados (Orfãos)

- [ ] Verificar se tabelas U00/U03 têm campos nunca preenchidos
- [ ] Confirmar se há colunas em SE1 que nunca são lidas
- [ ] Validar se SX5/SZ5 têm chaves sem referência

**Status:** NÃO DETERMINADO — requer inspeção completa do banco.

---

## CHECKPOINT 04 — REGRAS DE NEGÓCIO

### 4.1 Regras Extraídas do Código

#### **Validação de Importação (RECP017)**
1. Cada título (SE1) deve ter cliente (SA1) válido
2. Natureza contábil deve estar em SX5
3. Valor não pode ser zero ou negativo
4. Data de emissão ≤ data de vencimento
5. Filial deve ser "01" (hardcoded em alguns PRW)

#### **Geração de Contratos (CONTRATO.PRW)**
1. Contrato (U00) = 1 cliente + múltiplos títulos
2. Cada título = 1 linha contrato (U03)
3. Total U03 deve = soma SE1

#### **Geração de Pedidos (RECP016) ❌**
1. Seleciona SE1 com status "Baixado"
2. Cria SC5 para cada SE1
3. Valida estoque (não implementado, SQL comentado)

### 4.2 Regras de Negócio vs Técnicas

| Regra | Tipo | Evidência |
|-------|------|-----------|
| "Título deve ter cliente válido" | Negócio | RECP017 linha X |
| "Filial sempre é 01" | Técnica | Hardcoding em VIRIMPCLI, RECP016 |
| "Contrato agrupa títulos" | Negócio | CONTRATO.PRW |
| "Rollback em erro" | Técnica | RIMPC003 básico, falta cascata |

---

## CHECKPOINT 05 — FLUXOS CRÍTICOS

### 5.1 Fluxo Principal (Fim-a-Fim)

**Escopo:** Do upload de dados ao Protheus até geração de pedidos

1. **ENTRADA**
   - Arquivo/planilha com clientes + títulos

2. **IMPORTAÇÃO (RECP001.PRW)**
   - 11.471 linhas
   - Lê clientes → insere SA1
   - Lê títulos → insere SE1

3. **VALIDAÇÃO (RECP017.PRW)**
   - Valida cada SE1
   - Confirma referências cruzadas

4. **CONTRATO (CONTRATO.PRW)**
   - Cria U00 + U03 para rastreabilidade

5. **PEDIDOS (RECP016.PRW)** ❌ **NÃO EXECUTA**
   - Deveria ler SE1 com status "Baixado"
   - Gerar SC5 (pedidos vendas)
   - **Bloqueador:** Erro sintaxe linha 693

### 5.2 Casos de Erro

| Cenário | Tratamento Atual | Recomendação |
|---------|------------------|--------------|
| Cliente inexistente | Falha durante INSERT | Validar ANTES de inserir |
| Natureza ausente | Falha durante INSERT | Validar ANTES |
| SE1 duplicado | Sem verificação | Adicionar constraint UNIQUE |
| Estoque insuficiente | SQL comentado | Implementar ou remover |
| Rollback em lote | Básico | Implementar cascata (U00→U03→SC5) |

---

## CHECKPOINT 06 — SEGURANÇA

### 6.1 Análise Defensiva

#### **Autenticação**
- ✅ Protheus nativo garante login
- ⚠️ Não há auditoria de quem rodou cada PRW
- 🔴 Logs podem ser insuficientes

#### **Autorização**
- ✅ Protheus tem perfis de acesso
- 🔴 Hardcoding de filial "01" ignora multi-filial
- ⚠️ Sem validação de permissão de criação de SC5

#### **Criptografia**
- ✅ Conexão Protheus é segura (padrão)
- ⚠️ Se importa de arquivo externo, verificar origem

#### **Injeção SQL**
- ⚠️ Código AdvPL com SQL comentado = risco potencial
- 🔴 Sem sanitização aparente em entradas

#### **Dados Sensíveis**
- ⚠️ Valores financeiros em SE1
- ⚠️ Dados clientes em SA1
- ⚠️ Sem máscara aparente em logs

### 6.2 Riscos Identificados

| Risco | Severidade | Recomendação |
|-------|-----------|--------------|
| Hardcoding filial "01" | ALTO | Parametrizar |
| SQL comentado não revisado | ALTO | Decisão: executar ou remover |
| Sem cascata rollback | ALTO | Implementar |
| Variáveis não inicializadas | CRÍTICO | Corrigir (RECP016) |
| Erro sintaxe `nEXT` | CRÍTICO | Corrigir (RECP016) |
| Sem auditoria PRW | MÉDIO | Adicionar log de execução |

---

## CHECKPOINT 07 — QUALIDADE & TESTES

### 7.1 Cobertura de Testes

| Programa | Testes Identificados | Status |
|----------|----------------------|--------|
| FUNCOES.PRW | Presumível (biblioteca) | ⚠️ Não comprovado |
| RECP001.PRW | Não encontrado | ❌ |
| RECP017.PRW | Não encontrado | ❌ |
| CONTRATO.PRW | Documentado em código | ⚠️ Manual |
| RECP016.PRW | Não executável | ❌ Bloqueado |

### 7.2 Fluxos Críticos Sem Teste

1. ✅ **Importação com erro de validação** — deve rejeitar lote?
2. ✅ **Contrato com títulos conflitantes** — como procede?
3. ✅ **Geração pedidos com estoque zero** — aborta ou avisa?
4. ✅ **Rollback parcial** — dados inconsistentes?

### 7.3 Recomendações de Teste

**Fase DEV/TST (Próximas 2 semanas):**

```
Cenário: Importação válida (clientes + títulos OK)
Resultado esperado: SE1 + SA1 + U00/U03 criados

Cenário: Cliente duplicado na importação
Resultado esperado: Erro ou update?

Cenário: Título com natureza ausente
Resultado esperado: Rejeição

Cenário: Gerar pedidos (após RECP016 corrigido)
Resultado esperado: SC5 criado para cada SE1 "Baixado"
```

---

## CHECKPOINT 08 — OPERAÇÃO & DEPLOY

### 8.1 Procedimento Atual (Observado)

1. ✅ Compila PRW em DEV
2. ⚠️ Copia para PROD (manual?)
3. ❌ Executa RECP016 → falha
4. ❌ Sem rollback automático
5. ❌ Sem log centralizado

### 8.2 Runbook Proposto

**Pré-Requisitos:**
- [ ] Backup completo SE1/SA1/SC5/U00/U03
- [ ] 2 horas de janela de manutenção
- [ ] DBA disponível para rollback

**Passos:**

1. **Corrigir RECP016 (30 min)**
   - Linha 693: nEXT → NEXT
   - Declarar/inicializar variáveis
   - Testes unitários

2. **Compilar (15 min)**
   - Compilar 13 PRW em DEV
   - Sem erros
   - Sem warnings critérios

3. **Deploy (30 min)**
   - Backup PROD
   - Copiar .APO (compiled) para PROD
   - Documentar checksum

4. **Validação (30 min)**
   - Testar RECP001 (importação)
   - Testar RECP017 (validação)
   - Testar CONTRATO (contratos)
   - Testar RECP016 (pedidos) — **PRIMEIRA EXECUÇÃO**

5. **Rollback (15 min, se necessário)**
   - Restaurar backup
   - Comunicar stakeholders

**Total:** 2h

### 8.3 Monitoramento Pós-Deploy

```
Diariamente:
- [ ] Verificar SE1 recém-criados (último dia)
- [ ] Verificar erros em log Protheus
- [ ] Confirmar SC5 gerados

Semanalmente:
- [ ] Auditoria importação (valores, quantidades)
- [ ] Validar linhagem U00→U03→SC5
- [ ] Revisar rollbacks (se houver)
```

---

## CHECKPOINT 09 — DOCUMENTAÇÃO EXISTENTE

### 9.1 Documentação Encontrada

| Tipo | Encontrado | Qualidade |
|------|-----------|-----------|
| **Código comentado** | Sim (SQL em VIRIMPFIN) | ⚠️ Incompleto |
| **Diagramas** | Não | ❌ |
| **Runbook** | Não | ❌ |
| **Dicionário de dados** | Não | ❌ |
| **Matriz CRUD** | Não | ❌ |
| **Testes documentados** | Não | ❌ |

### 9.2 Documentação Necessária (Gerada)

- ✅ **Este relatório** — Engenharia Reversa
- ✅ **Diagrama de fluxo** — Mermaid
- ✅ **ERD** — Tabelas + relacionamentos
- ✅ **Matriz CRUD** — Qual PRW cria/lê/atualiza/deleta
- ✅ **Dicionário de dados** — Campos críticos
- ✅ **Testes recomendados** — Casos de teste
- ✅ **Runbook** — Deploy e operação

---

## CHECKPOINT 10 — CONCLUSÕES & PRÓXIMOS PASSOS

### ✅ O QUE ESTÁ BOM

1. **Código bem estruturado em FUNCOES.PRW** (4.691 linhas, biblioteca)
2. **Lógica de contratos clara** em CONTRATO.PRW
3. **Validações presentes** em RECP017.PRW
4. **7 de 13 programas funcionais** (54% código saudável)

### ❌ O QUE PRECISA CORRIGIR (URGENTE)

| Prioridade | Ação | Prazo |
|-----------|------|-------|
| 🔴 CRÍTICO | Corrigir RECP016 (nEXT→NEXT, variáveis) | 48h |
| 🔴 CRÍTICO | Compilar sem erros | 48h |
| 🟡 ALTO | Remover hardcoding filial "01" | 1 semana |
| 🟡 ALTO | Refatorar RECP001 (11.471 linhas → 3 módulos) | 2 semanas |
| 🟡 ALTO | Refatorar SYSEXTRATOR (11.418 linhas) | 2 semanas |
| 🟡 ALTO | Revisar SQL comentado (executar ou remover?) | 3 dias |
| 🟢 MÉDIO | Criar testes unitários | 3 semanas |
| 🟢 MÉDIO | Implementar rollback cascata | 2 semanas |

### 📋 RECOMENDAÇÃO GERAL

**NÃO PROSSEGUIR com produção até:**

1. ✅ RECP016 corrigido e testado
2. ✅ Compilação sem erros
3. ✅ SQL comentado revisado (decisão negócio)
4. ✅ Testes DEV/TST executados

**Estimativa total até produção:** 2-3 semanas (se dedicação full-time)

---

**Documento continuado em:**
- `01_BANCO_DE_DADOS.md` — Tabelas e linhagem detalhada
- `02_DIAGRAMA_FLUXOS.md` — Mermaid diagrams
- `03_MATRIZ_CRUD.md` — Mapeamento CRUD
- `04_RECOMENDACOES.md` — Roadmap detalhado
