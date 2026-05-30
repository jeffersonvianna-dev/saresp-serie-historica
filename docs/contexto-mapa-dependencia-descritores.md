# Contexto: Mapa de dependência entre descritores — SARESP 9º ano (Matemática)

> Documento de handoff. Cole/anexe numa próxima conversa para retomar o trabalho
> com todo o contexto e o caminho sugerido.

## 1. Objetivo
Construir um **mapa de dependência (pré-requisito) entre os descritores** da avaliação de
Matemática do 9º ano — "o que vem antes do quê" — e, sobre esse mapa, **tornar visível
onde o ensino mais falha**.

## 2. Dados disponíveis
- Microdados do SARESP, **Matemática, 9º ano**, **~300 mil estudantes** da rede de SP.
- Questões com **vínculo por descritor** (matriz de referência).
- **Importante:** cada descritor = **verbo (taxonomia de Bloom) + conteúdo**. A dependência
  tem, portanto, DOIS eixos: (a) conteúdo (ex.: racionais → proporção → porcentagem) e
  (b) cognitivo/Bloom (ex.: identificar → aplicar → analisar).
- Os microdados são sensíveis e **não devem ir para o repositório** (o `.gitignore` já
  exclui bases brutas). A análise roda localmente; o repo guarda scripts e resultados
  agregados/visualizações.

## 3. Estado do repositório (em 2026-05-30)
- É um **dashboard HTML estático** de série histórica SARESP agregada (2011–2025):
  `index.html` + `data/dashboard-data.js`. Não há granularidade de descritor.
- **Não existe** ainda: microdados, Q-matriz (questão→descritor), scripts/ETL, notebooks,
  nem análise de dependência.
- Branch de trabalho: `claude/descriptor-dependency-mapping-IW7bV`.

## 4. Decisões já tomadas
- **Abordagem: HÍBRIDA** — estrutura pedagógica (matriz/currículo) validada empiricamente
  pelos dados.
- O usuário quis **primeiro entender a literatura/modelos** antes de fixar o formato do mapa.
- Foco adicional pedido: **diagnóstico de "onde mais falhamos no ensino"**, não só a estrutura.
- **Automação ponta a ponta sem outro especialista humano** é viável (ver seção 6).

## 5. Modelos da literatura (resumo de 1 linha cada)
1. **AHM (Attribute Hierarchy Method)** — desenha o grafo de pré-requisitos entre atributos
   (hierarquias linear/convergente/divergente). É o "mapa mental" como hipótese.
   (Leighton, Gierl & Hunka, 2004)
2. **Q-matriz + CDM (DINA / G-DINA)** — formaliza o vínculo questão→descritor e estima o
   domínio por descritor de cada aluno. (Tatsuoka 1983; de la Torre 2011)
3. **Knowledge Space Theory / Learning Spaces** — estados de conhecimento válidos e relações
   de precedência ("surmise"); base do ALEKS. (Doignon & Falmagne, 1985)
4. **POKS (Partial Order Knowledge Structures)** — **infere o grafo de pré-requisitos
   diretamente das respostas** (probabilidades condicionais). (Desmarais et al.)
5. **HDCM (Hierarchical Diagnostic Classification Models)** — **testa estatisticamente** se
   uma hierarquia de pré-requisitos se sustenta nos dados (comprime 2^k perfis → poucos
   sem perder ajuste). (Templin & Bradshaw, 2014)
6. **Escala TRI / learning progressions** — ancora cada descritor no ponto de proficiência
   onde P(acerto)≈65%; a ordem na escala dá a ordenação empírica de pré-requisito. Leitura
   já usada no Brasil (SARESP/SAEB).

Encaixe: (1)+(2) montam estrutura e ligação aos itens; (6) dá a ordenação empírica;
(4)+(5) validam/corrigem as setas com os 300 mil; (3) é a base teórica de trajetória.

## 6. Arquitetura proposta (automatizável, sem outro recurso humano)
- **Camada 1 — Pedagógica (o LLM/Claude faz):** lê o texto de cada descritor, separa
  verbo (→ Bloom) e conteúdo (→ área/tópico) e gera a **hipótese de pré-requisitos**
  (grafo inicial, incluindo setas entre áreas). Saída **auditável**: tabela
  `descritor → Bloom, área, pré-requisitos propostos, justificativa`.
- **Camada 2 — Empírica (código sobre os dados):** roda POKS + um CDM (G-DINA) e, para cada
  seta da Camada 1, emite veredito: **mantém / remove / adiciona**. Renderiza o grafo final.
- Único input humano necessário: **texto dos descritores** + **arquivo de respostas com o
  vínculo questão→descritor** (Q-matriz). O resto é LLM + código.

### Ressalvas (não impedem automação, mas registrar)
- A Camada 1 é um *prior forte*, não oráculo → por isso sai como tabela inspecionável.
- Qualidade empírica depende da Q-matriz; descritor com **poucos itens** fica fraco de
  estimar (a informação vem do nº de itens, não de alunos).
- POKS mede precedência/associação, **não causa**; setas entre áreas = "evidência de ordem".
- Verbos ambíguos (ex.: "resolver") devem ser sinalizados, não chutados em silêncio.

## 7. Diagnóstico de "onde o ensino mais falha"
Métrica central (isola a falha de ENSINO, não a falta de base):
- **Taxa de não-aprendizagem condicional** =
  `P(NÃO domina D | domina TODOS os pré-requisitos de D)`.
  Alta = a transição "vaza" mesmo com alunos preparados.
- **Volume** = nº de alunos "prontos mas que não aprenderam" = taxa × (quantos chegaram
  prontos). Prioriza ação: gargalo em habilidade básica perde muito mais gente.
- **Impacto a jusante** (próximo passo sugerido): ponderar o gargalo por quantos descritores
  dependem dele — atacar o ponto certo rende mais aprendizagem total.
- **Recorte por escola/diretoria/rede** (próximo passo): vira mapa de equidade e de
  prioridade para formação de professores.

Leitura-chave: comparar **taxa** (quão difícil) vs **volume** (quantos atinge). Um descritor
pode ter taxa altíssima mas baixo volume (poucos chegam até ele) — enquanto um gargalo
básico, de taxa "moderada", é onde mais se perde gente, por efeito cascata.

## 8. Visualizações já prototipadas (com dados SIMULADOS)
- Mapa de dependência D1–D5 + escala TRI + heatmap POKS (3 painéis).
- Mapa em escala real das 5 áreas (Números, Álgebra, Geometria, Grandezas e Medidas,
  Probabilidade e Estatística) com setas tracejadas para dependências **entre** áreas.
- Diagnóstico de falhas: nós coloridos pela taxa de não-aprendizagem + ranking de gargalos
  por volume.
- Quadro com 1 painel por modelo (AHM, Q-matriz/CDM, KST, POKS, HDCM, TRI).
- Scripts em Python + matplotlib (sem dependências além de matplotlib).

## 9. Caminho sugerido (próximos passos)
1. **Definir formato de saída** do mapa: grafo interativo (HTML, no estilo do dashboard
   atual) / diagrama estático (PNG/PDF) / Mermaid versionável / só a matriz de pré-requisitos.
2. **Camada 1 com os descritores reais:** gerar a tabela Bloom+conteúdo+pré-requisitos.
3. **Preparar os dados:** confirmar a Q-matriz (questão→descritor) e o formato das respostas.
4. **Camada 2:** rodar POKS + G-DINA; validar/corrigir as setas; (opcional) teste HDCM.
5. **Diagnóstico de falhas:** calcular taxa condicional + volume + impacto a jusante;
   (opcional) recorte por escola/diretoria.
6. **Renderizar e versionar** o mapa final e o relatório de gargalos.

## 10. O que trazer na próxima conversa
- A **lista de descritores** de Matemática 9º ano (texto completo: verbo + conteúdo).
- Uma **amostra do arquivo de respostas** (cabeçalho + algumas linhas) e o **dicionário**:
  - como cada questão se liga ao descritor (Q-matriz) e como o acerto está codificado
    (0/1, A/B/C/D + gabarito, etc.);
  - chaves de aluno/escola/diretoria/rede, se quiser recorte.
- Decisão sobre **formato do mapa** (item 9.1) e sobre **método empírico** (POKS vs G-DINA/HDCM).
- Restrições de privacidade/LGPD para tratamento dos microdados.

## 11. Decisões em aberto
- Formato do mapa (interativo / estático / Mermaid / matriz).
- Método empírico de validação (POKS, G-DINA, HDCM, ou combinação).
- Se haverá recorte por escola/diretoria desde o início.
- Onde rodar a análise dos microdados (local) e o que volta ao repo (apenas agregados).
