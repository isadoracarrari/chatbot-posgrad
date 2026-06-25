# 🤖 Como implantei um Chatbot com IA Generativa Open Source na Oracle Cloud

*Um relato prático sobre infraestrutura, modelos de linguagem e o que aprendi no caminho.*

---

## Introdução

Este projeto foi desenvolvido como atividade prática da disciplina de Produtos de GenAI, com o objetivo de implantar um chatbot baseado em Inteligência Artificial Generativa utilizando um modelo open source disponibilizado pela NVIDIA.

A solução consiste em um assistente especializado em Engenharia de Prompt, desenvolvido em Python com Streamlit e publicado em uma máquina virtual na Oracle Cloud Infrastructure (OCI), acessível publicamente via navegador.

---

## Infraestrutura

A aplicação foi hospedada em uma instância da Oracle Cloud Infrastructure com as seguintes configurações:

| Recurso | Configuração |
|---|---|
| Shape | VM.Standard.E2.1.Micro |
| OCPU | 1 |
| Memória RAM | 1 GB |
| Armazenamento | Block storage |
| Rede | 0.48 Gbps |
| Sistema Operacional | Ubuntu 24.04 LTS |
| Região | Brazil East (São Paulo) |

A VM faz parte do tier gratuito da Oracle Cloud (Always Free), o que torna a solução viável para projetos acadêmicos e prototipação sem custos.

---

## Modelo Escolhido

**Llama 3.3 70B Instruct** — disponibilizado pela Meta via API da NVIDIA (integrate.api.nvidia.com).

### Por que esse modelo?

- É um modelo open source de alta performance com 70 bilhões de parâmetros
- Otimizado para tarefas de diálogo e seguimento de instruções
- Disponível gratuitamente via NVIDIA NIM API, sem necessidade de infraestrutura de GPU própria
- Desempenho comparável a modelos proprietários maiores em tarefas de chat e raciocínio

### Principais características

- Arquitetura: transformer autoregressivo
- Modalidade: text-to-text
- Suporte multilíngue, incluindo português
- Forte em raciocínio, geração de código e conhecimento geral

---

## Desenvolvimento

### Arquitetura da aplicação

A aplicação segue uma arquitetura simples e eficiente:

```
Usuário (navegador)
      ↓
Streamlit (frontend + servidor web)
      ↓
chatlas (cliente de chat)
      ↓
NVIDIA API (meta/llama-3.3-70b-instruct)
```

O histórico de conversa é mantido em memória via `st.session_state` do Streamlit, sem necessidade de banco de dados.

### Bibliotecas utilizadas

| Biblioteca | Finalidade |
|---|---|
| `streamlit` | Interface web e servidor da aplicação |
| `chatlas` | Cliente de chat para integração com APIs de LLMs |
| `openai` | SDK base utilizado pelo chatlas para chamadas à API |
| `python-dotenv` | Gerenciamento de variáveis de ambiente |

### Estratégia de gerenciamento de credenciais

A API key da NVIDIA é armazenada em um arquivo `.env` na raiz do projeto, carregada em tempo de execução via `python-dotenv`. O arquivo `.env` está listado no `.gitignore`, garantindo que a chave nunca seja exposta no repositório público.

---

## Implantação

### Processo de publicação na Oracle Cloud

1. Conexão na VM via SSH com chave privada
2. Instalação de dependências do sistema (`python3-venv`, `git`)
3. Clone do repositório GitHub na VM
4. Criação do ambiente virtual Python e instalação das dependências
5. Criação manual do arquivo `.env` na VM com a API key
6. Liberação da porta 8501 via `iptables` na VM
7. Liberação da porta 8501 via Ingress Rule na Security List da VCN (Oracle Cloud)
8. Execução da aplicação em background com `nohup`

### Principais desafios encontrados

**Configuração da conta NVIDIA:** O processo de criação de conta no NVIDIA NGC apresentou um erro de `CANNOT_MERGE` devido a um vínculo pré-existente com uma conta institucional. A solução foi acessar diretamente o portal `build.nvidia.com` já autenticado para gerar a API key.

**Liberação de portas na Oracle Cloud:** A OCI exige liberação de porta em dois níveis: no firewall do sistema operacional (`iptables`) e na Security List da VCN. Esquecer qualquer um dos dois impede o acesso externo.

**Persistência do processo:** O Streamlit encerra quando a sessão SSH é fechada. A solução foi utilizar `nohup` para manter o processo rodando em background de forma independente da sessão.

---

## Discussão

### Lições aprendidas

- Infraestrutura em nuvem exige atenção em múltiplas camadas de segurança de rede — o firewall do SO e as regras da VCN são independentes e ambos precisam estar configurados
- O tier gratuito da Oracle Cloud é surpreendentemente capaz para aplicações leves de IA, especialmente quando o processamento pesado é delegado a uma API externa
- Manter credenciais fora do repositório é inegociável, mesmo em projetos acadêmicos — boas práticas de segurança devem ser cultivadas desde o início

### Possíveis melhorias futuras

- Adicionar autenticação de usuário para controlar o acesso à aplicação
- Implementar persistência de histórico de conversas em banco de dados
- Configurar HTTPS com certificado SSL para eliminar o aviso de "conexão não segura"
- Usar `systemd` ou `supervisor` para gerenciar o processo de forma mais robusta que `nohup`
- Adicionar monitoramento de uptime e alertas automáticos

---

## Como executar localmente

```bash
# Clone o repositório
git clone https://github.com/isadoracarrari/chatbot-posgrad.git
cd chatbot-posgrad

# Crie e ative o ambiente virtual
python3 -m venv .venv
source .venv/bin/activate  # Linux/Mac
# .venv\Scripts\activate   # Windows

# Instale as dependências
pip install -r requirements.txt

# Crie o arquivo .env com sua chave
echo "NVIDIA_API_KEY=sua_chave_aqui" > .env

# Execute a aplicação
streamlit run app.py
```

Acesse em: `http://localhost:8501`

---

## Acesso à aplicação

🌐 **URL pública:** http://150.230.84.196:8501

📁 **Repositório:** https://github.com/isadoracarrari/chatbot-posgrad

---

*Desenvolvido como atividade prática da disciplina de Produtos de GenAI.*
