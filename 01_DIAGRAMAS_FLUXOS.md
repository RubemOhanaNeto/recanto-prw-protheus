# PRW Protheus — Diagramas de Arquitetura e Fluxos

## DIAGRAMA 1: Arquitetura Geral (C4 - Contexto)

```mermaid
graph TB
    Legacy["<b>Legacy System</b><br/>(Clientes + Títulos)"]
    Protheus["<b>TOTVS Protheus</b><br/>ERP"]
    
    IMPORT["<b>Importação</b><br/>RECP001, RECP015<br/>VIRIMPCLI, VIRIMPFIN"]
    
    VALID["<b>Validação</b><br/>RECP017<br/>FUNCOES (lib)"]
    
    TRANSFORM["<b>Transformação</b><br/>CONTRATO, RIMPC003"]
    
    PEDIDOS["<b>Geração Pedidos</b><br/>RECP016 ❌"]
    
    DB[("<b>Banco Protheus</b><br/>SE1 (Títulos)<br/>SA1 (Clientes)<br/>SC5 (Pedidos)<br/>U00/U03 (Contratos)<br/>SX5/SZ5 (Config)")]
    
    Legacy -->|Arquivo| IMPORT
    IMPORT -->|Insere| DB
    IMPORT -->|Valida| VALID
    VALID -->|Confirma| TRANSFORM
    TRANSFORM -->|Cria contratos| DB
    DB -->|Lê SE1| PEDIDOS
    PEDIDOS -->|Gera SC5| DB
    
    style Legacy fill:#fff4e6
    style IMPORT fill:#ffe6e6
    style VALID fill:#ffe6e6
    style TRANSFORM fill:#e6f3ff
    style PEDIDOS fill:#ffcccc
    style Protheus fill:#e6f9ff
    style DB fill:#f0f0f0
```

---

## DIAGRAMA 2: Fluxo Detalhado (Sequência)

```mermaid
sequenceDiagram
    participant Ext as External<br/>(Legacy)
    participant RECP001 as RECP001<br/>Importação
    participant RECP017 as RECP017<br/>Validação
    participant FUNCOES as FUNCOES<br/>Lib
    participant CONTRATO as CONTRATO<br/>Contratos
    participant RECP016 as RECP016<br/>Pedidos ❌
    participant DB as Protheus DB<br/>(SE1/SA1/SC5)
    
    Ext->>RECP001: Arquivo clientes + títulos
    RECP001->>DB: INSERT SA1
    RECP001->>DB: INSERT SE1
    RECP001->>RECP017: Valida cada SE1
    RECP017->>FUNCOES: Chama helpers
    FUNCOES->>DB: Consulta SX5 (natureza)
    RECP017->>DB: UPDATE status SE1
    RECP001->>CONTRATO: Cria contratos
    CONTRATO->>DB: INSERT U00 (contrato)
    CONTRATO->>DB: INSERT U03 (detalhes)
    DB->>RECP016: SE1 com status "Baixado"
    RECP016->>DB: ❌ ERRO SINTAXE (nEXT)
    DB->>DB: Rollback manual necessário
```

---

## DIAGRAMA 3: Dependências Entre Módulos

```mermaid
graph TD
    FUNCOES["FUNCOES.PRW<br/>(4.691 linhas)<br/>Biblioteca"]
    SUPORTE["SUPORTE.PRW<br/>(171 linhas)<br/>Manutenção SX5/SZ5"]
    
    RECP001["RECP001.PRW<br/>(11.471 linhas)<br/>Importação BULK"]
    RECP002["RECP002.PRW<br/>SE1 Processing"]
    RECP015["RECP015.PRW<br/>Importação específica"]
    RECP017["RECP017.PRW<br/>Validação"]
    
    VIRIMPCLI["VIRIMPCLI.PRW<br/>Importa Clientes<br/>(SQL comentado)"]
    VIRIMPFIN["VIRIMPFIN.PRW<br/>Importa Financeiro<br/>(SQL comentado)"]
    
    RIMPC003["RIMPC003.PRW<br/>Transações básicas"]
    CONTRATO["CONTRATO.PRW<br/>(4.143 linhas)<br/>Gestão U00/U03"]
    
    SYSEXTRATOR["SYSEXTRATOR.PRW<br/>(11.418 linhas)<br/>Monolítico<br/>Filtros avançados"]
    
    RECP016["RECP016.PRW<br/>(7.500 linhas)<br/>Gerador Pedidos<br/>❌ NÃO EXECUTA"]
    
    FUNCOES -.->|import| RECP001
    FUNCOES -.->|import| RECP017
    FUNCOES -.->|import| CONTRATO
    FUNCOES -.->|import| RIMPC003
    
    SUPORTE -.->|import| RECP017
    
    RECP001 -->|popula| RECP017
    RECP015 -->|popula| RECP017
    VIRIMPCLI -->|popula| RECP017
    VIRIMPFIN -->|popula| RECP017
    
    RECP017 -->|confirma| RIMPC003
    RIMPC003 -->|cria| CONTRATO
    
    RECP002 -->|processa| RIMPC003
    
    CONTRATO -->|U00/U03| RECP016
    RECP017 -->|SE1| RECP016
    
    RECP016 -.->|depende de| SYSEXTRATOR
    
    style FUNCOES fill:#e6f3ff
    style RECP016 fill:#ffcccc
    style SYSEXTRATOR fill:#fff4e6
    style RECP001 fill:#fff4e6
    style CONTRATO fill:#e6f3ff
```

---

## DIAGRAMA 4: Modelo de Dados (ERD Simplificado)

```mermaid
erDiagram
    SA1 ||--o{ SE1 : "tem títulos"
    SA1 ||--o{ U00 : "tem contratos"
    U00 ||--o{ U03 : "tem detalhes"
    SE1 ||--o{ U03 : "relaciona-se"
    SE1 ||--o{ SC5 : "gera pedidos"
    SX5 ||--o{ SE1 : "natureza"
    SZ5 ||--o{ SE1 : "parâmetros"
    
    SA1 {
        string A1_FILIAL PK
        string A1_COD PK
        string A1_NOME
        string A1_EMAIL
        string A1_TIPO
    }
    
    SE1 {
        string E1_FILIAL PK
        string E1_PREFIXO PK
        string E1_NUM PK
        string E1_PARCELA PK
        string E1_CLIENTE FK
        string E1_NATUREZA FK
        decimal E1_VALOR
        date E1_EMISSAO
        date E1_VENCTO
        string E1_STATUS
    }
    
    U00 {
        string U00_FILIAL PK
        string U00_CONTRNO PK
        string U00_CLIENT FK
        date U00_DATCON
        decimal U00_TOTAL
    }
    
    U03 {
        string U03_FILIAL PK
        string U03_CONTRNO PK
        int U03_ITEM PK
        string U03_REFSE1 FK
        decimal U03_VALOR
    }
    
    SC5 {
        string C5_FILIAL PK
        string C5_NUM PK
        string C5_CLIENT FK
        date C5_EMISSAO
        string C5_STATUS
    }
    
    SX5 {
        string X5_FILIAL PK
        string X5_TABELA PK
        string X5_CHAVE PK
        string X5_DESCRI
    }
    
    SZ5 {
        string Z5_PARAM
        string Z5_VALOR
    }
```

---

## DIAGRAMA 5: Estados de SE1 (Título)

```mermaid
stateDiagram-v2
    [*] --> Criado: RECP001 insere
    
    Criado --> Validado: RECP017 OK
    Criado --> Rejeitado: RECP017 KO
    
    Validado --> Processado: RIMPC003
    Processado --> EmContrato: CONTRATO cria U00/U03
    
    EmContrato --> Baixado: Operação manual
    
    Baixado --> EmPedido: RECP016 gera SC5
    EmPedido --> Concluido: SC5 processado
    
    Rejeitado --> [*]
    Concluido --> [*]
    
    style Criado fill:#e6f3ff
    style Validado fill:#e6f3ff
    style Processado fill:#e6f3ff
    style EmContrato fill:#e6f3ff
    style Baixado fill:#fff4e6
    style EmPedido fill:#ffcccc
    style Concluido fill:#e6e6e6
    style Rejeitado fill:#ffcccc
```

---

## DIAGRAMA 6: Problemas Críticos (Mapa de Impacto)

```mermaid
graph TD
    ERRO1["RECP016 linha 693<br/>nEXT em vez de NEXT<br/>❌ FALHA SINTAXE"]
    
    ERRO2["Variáveis não inicializadas<br/>_cAliasTmp, _cPastaQry<br/>nTotReg, cCadPrb, nPedItem<br/>❌ RUNTIME ERROR"]
    
    BLOQ["❌ RECP016 NÃO EXECUTA"]
    
    IMPACTO1["SE1 nunca chega a SC5<br/>Pedidos não são gerados<br/>Fluxo interrompido"]
    
    IMPACTO2["Impossível gerar pedidos<br/>Deve ser manual"]
    
    DECIS["Necessária decisão<br/>de negócio"]
    
    ERRO1 --> BLOQ
    ERRO2 --> BLOQ
    BLOQ --> IMPACTO1
    BLOQ --> IMPACTO2
    IMPACTO1 --> DECIS
    
    DECIS --> OPC1["Opção 1:<br/>Corrigir RECP016<br/>(recomendado)"]
    DECIS --> OPC2["Opção 2:<br/>Gerar SC5 manual"]
    DECIS --> OPC3["Opção 3:<br/>Usar outro programa"]
    
    style ERRO1 fill:#ffcccc
    style ERRO2 fill:#ffcccc
    style BLOQ fill:#ff6666
    style IMPACTO1 fill:#ffcccc
    style IMPACTO2 fill:#ffcccc
    style DECIS fill:#fff4e6
    style OPC1 fill:#ccffcc
    style OPC2 fill:#fff4e6
    style OPC3 fill:#fff4e6
```

---

## DIAGRAMA 7: Linhas por Programa (Análise de Complexidade)

```mermaid
pie title Distribuição de Linhas de Código
    "RECP001 (Importação)" : 11471
    "SYSEXTRATOR (Filtros)" : 11418
    "FUNCOES (Biblioteca)" : 4691
    "CONTRATO (Contratos)" : 4143
    "RECP016 (Pedidos)" : 7500
    "Outros (RECP002, 015, 017, etc)" : 6596
```

---

## DIAGRAMA 8: Status de Execução (Semáforo)

```
┌─────────────────────────────────────────────────────────┐
│                    STATUS GERAL                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Compilação:              ❌ FALHA (RECP016)           │
│  Execução Importação:     ✅ OK                        │
│  Execução Validação:      ✅ OK                        │
│  Execução Contratos:      ✅ OK                        │
│  Execução Pedidos:        ❌ BLOQUEADO                 │
│                                                         │
│  Testes Unitários:        ❌ Não encontrados           │
│  Testes Integração:       ❌ Não encontrados           │
│  Documentação:            ❌ Ausente                   │
│                                                         │
│  RECOMENDAÇÃO: NÃO PROSSEGUIR PARA PRODUÇÃO            │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## PRÓXIMOS DOCUMENTOS RECOMENDADOS

- `02_MATRIZ_CRUD.md` — Mapeamento detalhado
- `03_BANCO_DE_DADOS.md` — Tabelas e constraints
- `04_RECOMENDACOES.md` — Roadmap
