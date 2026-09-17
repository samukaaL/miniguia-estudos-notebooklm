# Model Context Protocol (MCP) — Guia Prático e Arquitetura

Este repositório contém uma documentação estruturada e didática sobre o **Model Context Protocol (MCP)** [258, 411], cobrindo sua arquitetura, primitivas, mecanismos de transporte, vantagens, limitações de segurança e um glossário técnico completo.

---

## 📋 Sumário
1. [O que é o MCP?](#o-que-é-o-mcp)
2. [O Problema M x N vs. A Solução MCP (M + N)](#o-problema-m-x-n-vs-a-solução-mcp-m--n)
3. [Por que o MCP é o "USB-C da Inteligência Artificial"?](#por-que-o-mcp-é-o-usb-c-da-inteligência-artificial)
4. [Arquitetura do MCP](#arquitetura-do-mcp)
   - [Host vs. Client vs. Server](#host-vs-client-vs-server)
5. [Primitivas Fundamentais (Data Layer)](#primitivas-fundamentais-data-layer)
6. [Camada de Transporte (Transport Layer)](#camada-de-transporte-transport-layer)
7. [Fluxo de Funcionamento Passo a Passo](#fluxo-de-funcionamento-passo-a-passo)
8. [Vantagens Técnicas e Limitações](#vantagens-técnicas-e-limitações)
   - [Vantagens](#vantagens)
   - [Limitações e Desafios de Segurança](#limitações-e-desafios-de-segurança)
9. [Glossário Técnico](#glossário-técnico)

---

## 🚀 O que é o MCP?

O **Model Context Protocol (MCP)** é um protocolo aberto e padronizado, introduzido pela Anthropic em novembro de 2024, projetado para conectar modelos de linguagem (LLMs) e agentes de IA a ferramentas externas, dados de contexto e sistemas operacionais [258, 411]. 

O objetivo do MCP é fornecer uma interface homogênea para que qualquer aplicação de IA consiga interagir com múltiplos serviços de forma segura, estruturada e escalável [412, 610].

---

## 🔀 O Problema M x N vs. A Solução MCP (M + N)

* **Integrações Tradicionais ($M \times N$)**: Conectar $M$ aplicações de IA a $N$ ferramentas ou bancos de dados exigia a criação de códigos de integração (*glue code*) customizados para cada combinação [412]. Isso gerava alto custo de manutenção, duplicação de esforço e extrema fragmentação.
* **Abordagem MCP ($M + N$)**: O protocolo introduz uma camada intermediária padronizada em JSON-RPC 2.0 [45, 259]. O Host de IA implementa o protocolo uma vez, e os servidores expõem suas capacidades uma vez, reduzindo drasticamente a complexidade do ecossistema [258, 412].

---

## 🔌 Por que o MCP é o "USB-C da Inteligência Artificial"?

Assim como a interface USB-C padronizou a conexão física de periféricos (monitores, discos, teclados) a computadores sem a necessidade de cabos proprietários, o **MCP padroniza as conexões lógicas entre modelos de IA e fontes de dados externas** [258, 599]. Qualquer cliente compatível (ex: Claude Desktop, Cursor, VS Code) pode se conectar a qualquer servidor MCP (ex: GitHub, Slack, PostgreSQL) de forma imediata (*plug-and-play*) [43, 269].

---

## 🏗️ Arquitetura do MCP

O MCP adota uma arquitetura cliente-servidor estruturada em três participantes principais [42, 259]:

```text
[ Usuário ] 
    │
    ▼
[ MCP Host (ex: VS Code / Claude Desktop) ]
    │
    ├─── [ MCP Client 1 ] ─────(stdio / Streamable HTTP)─────► [ MCP Server A (Filesystem) ]
    │
    └─── [ MCP Client 2 ] ─────(stdio / Streamable HTTP)─────► [ MCP Server B (PostgreSQL) ]
```

### Host vs. Client vs. Server
* **MCP Host**: A aplicação final de IA voltada ao usuário que coordena o LLM e gerencia as conexões (ex: Claude Desktop, VS Code, Cursor) [42, 43].
* **MCP Client**: O componente interno instanciado pelo Host que mantém uma sessão de comunicação dedicada (1 para 1) com um servidor MCP específico [42, 43].
* **MCP Server**: O serviço ou processo (local ou remoto) que expõe contextos, dados legíveis e ferramentas executáveis para a IA [42, 43].

---

## 📦 Primitivas Fundamentais (Data Layer)

Os servidores MCP organizam suas capacidades em três primitivas principais [46, 50, 260]:

| Primitiva | Tipo | Descrição | Exemplo |
| :--- | :--- | :--- | :--- |
| **Resources** | Leitura | Fontes de dados e arquivos mapeados por URIs legíveis pela IA [50, 260]. | `config://app-settings` ou `file://logs/app.log` [261] |
| **Tools** | Ação | Funções executáveis com esquemas JSON Schema para a IA realizar ações ativas [50, 260]. | `get_weather(city)` ou `run_sql_query(query)` [60, 261] |
| **Prompts** | Guia | Templates de instruções pré-definidos para estruturar fluxos de trabalho [50, 260]. | `analyze_data(dataset_name)` [261] |

---

## ⚡ Camada de Transporte (Transport Layer)

A comunicação utiliza mensagens no formato **JSON-RPC 2.0** encapsuladas por dois transportes oficiais [45, 47]:

1. **`stdio` (Standard Input/Output)**:
   - Projetado para processos locais executados na mesma máquina do Host [47].
   - Comunicação rápida via fluxos padrão do sistema operacional com isolamento de processo e zero overhead de rede [47, 262].
2. **Streamable HTTP (evolução do HTTP + SSE)**:
   - Projetado para servidores remotos hospedados na nuvem [47, 262].
   - Utiliza HTTP POST e Server-Sent Events (SSE) para comunicação remota e suporta autenticação corporativa via **OAuth 2.1 + PKCE** [47, 262, 264].

---

## 🔄 Fluxo de Funcionamento Passo a Passo

1. **Descoberta**: Durante a inicialização, o MCP Client consulta o MCP Server para descobrir quais ferramentas e recursos estão disponíveis [54, 59].
2. **Seleção pelo LLM**: O usuário faz um pedido em linguagem natural. O LLM analisa os esquemas disponíveis e decide qual ferramenta invocar [68].
3. **Chamada JSON-RPC**: O cliente envia um pedido `tools/call` com os argumentos estruturados [65].
4. **Execução e Resposta**: O servidor executa a ação no sistema de destino e retorna os dados estruturados para o cliente [66, 68].
5. **Síntese Final**: O LLM interpreta a resposta e formula a mensagem final em linguagem natural ao usuário [68, 158].

---

## ⚖️ Vantagens Técnicas e Limitações

### Vantagens
* **Interoperabilidade**: Escreva o servidor uma vez e conecte-o a qualquer Host ou modelo de IA [412].
* **Desacoplamento**: Separa a inteligência do LLM da implementação das ferramentas [412].
* **Interações Bidirecionais**: Suporte a *Elicitation* (solicitação de confirmação/dados do usuário) e *Sampling* (servidor solicita conclusão do LLM) [52, 272].
* **Segurança Corporativa**: Suporte nativo a OAuth 2.1 com PKCE e Enterprise-Managed Authorization (EMA) [264, 283].

### Limitações e Desafios de Segurança
* **Estouro de Contexto (*Prompt Bloat*)**: Carregar muitas definições de ferramentas no contexto consome tokens excessivos [220, 637].
* **Queda de Precisão**: Estudos mostram que a precisão da IA na escolha de ferramentas cai significativamente quando o número de ferramentas ultrapassa **10 a 15** em modelos menores ou **20 a 30** em modelos avançados [411, 452].
* **Riscos de Injeção de Prompt**: Dados não sanitizados retornados por ferramentas podem induzir o modelo a executar comandos maliciosos (*Indirect Prompt Injection*) [441, 573].
* **Tool Poisoning & Confused Deputy**: Risco de manipuladores de ferramentas executarem ações com privilégios elevados indevidos sem o contexto de permissão estrito do usuário [355, 356].

---

## 📖 Glossário Técnico

* **MCP Host**: Aplicação principal de IA que gerencia o modelo e os clientes MCP (ex: Claude Desktop, Cursor, VS Code) [42, 43].
* **MCP Client**: Conector mantido pelo Host que estabelece sessão 1 para 1 com um MCP Server [42, 43].
* **MCP Server**: Serviço responsável por expor ferramentas, recursos e prompts para o ecossistema de IA [42, 43].
* **Resource**: Primitiva de dados de leitura mapeada por URI [50, 260].
* **Tool**: Função executável com contrato estruturado (JSON Schema) para ações ativas [50, 260].
* **Prompt Template**: Modelo de instrução reutilizável armazenado no servidor [50, 260].
* **JSON-RPC 2.0**: Protocolo de codificação de mensagens leve utilizado na camada de dados do MCP [46, 259].
* **stdio**: Mecanismo de transporte de baixa latência para servidores MCP locais [47, 262].
* **Streamable HTTP / SSE**: Mecanismo de transporte remoto baseado em HTTP para conexões na nuvem com autenticação OAuth 2.1 [47, 262, 264].
