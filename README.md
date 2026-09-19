# Recanto PRW Protheus

**Projeto:** Sistema Financeiro ERP Protheus - Recanto da Saudade Cemitério Parque Ltda  
**CNPJ:** 04.709.150/0001-20  
**Data Análise:** 19/09/2026

## 📊 Status

⚠️ **BLOQUEADO** — Correção crítica necessária antes produção

- **13 arquivos .PRW** (45.819 linhas)
- **54% código funcional** (7 de 13 programas OK)
- **RECP016 não executa** (erro sintaxe linha 693: `nEXT` → deve ser `NEXT`)
- **Variáveis não inicializadas** (RECP016)

## 🎯 Próximos Passos

1. ✅ **Ler documentação** → `00_INVENTARIO_E_ARQUITETURA.md`
2. ✅ **Corrigir RECP016** (linha 693 + declarar variáveis)
3. ✅ **Compilar** todos 13 .PRW
4. ✅ **Testar** em DEV/TST
5. ✅ **Deploy** produção

**Estimativa:** 2-3 semanas (dedicação full-time)

## 📁 Estrutura

```
recanto-prw-protheus/
├─ README.md (este arquivo)
├─ docs/
│  ├─ 00_INVENTARIO_E_ARQUITETURA.md
│  ├─ 01_DIAGRAMAS_FLUXOS.md
│  ├─ 02_MATRIZ_CRUD.md (TBD)
│  └─ ...
└─ src/
   ├─ RECP016.prw (CRÍTICO — a corrigir)
   ├─ RECP001.prw (monolítico)
   ├─ FUNCOES.prw (biblioteca — OK)
   └─ ... (outros 10 .PRW)
```

## 🔴 Bloqueadores Imediatos

| Bloqueador | Linha | Impacto |
|-----------|-------|---------|
| `nEXT` → `NEXT` | 693 | Falha sintaxe |
| Variáveis não inicializadas | vários | Runtime error |
| SQL comentado | VIRIMPCLI/VIRIMPFIN | Funcionalidade incerta |
| Hardcoding filial "01" | vários | Multi-filial impossível |

## 📊 Análise Técnica

Veja `docs/00_INVENTARIO_E_ARQUITETURA.md` para:
- 10 checkpoints completos
- Fluxo fim-a-fim (importação → validação → contratos → pedidos)
- Regras de negócio
- Segurança
- Operação & deploy

## 📈 Diagramas

8 diagramas Mermaid em `docs/01_DIAGRAMAS_FLUXOS.md`:
1. Arquitetura C4
2. Fluxo sequencial
3. Dependências entre módulos
4. ERD (Entidade-Relacionamento)
5. Estados de SE1
6. Problemas críticos (mapa de impacto)
7. Distribuição linhas de código
8. Status geral (semáforo)

## 🎓 Engenharia Reversa via PROMPT MESTRE

Análise realizada com **PROMPT MESTRE para Engenharia Reversa v91** — framework com 20 módulos especializados:

✅ Descubra antes de supor  
✅ Leia antes de alterar  
✅ Prove antes de afirmar  
✅ Rastreie antes de concluir  
✅ Documente o real antes de propor o ideal

---

**Gerado em:** 19/09/2026  
**Ferramenta:** Claude (PROMPT MESTRE)  
**Contato:** PRW Analysis Report
