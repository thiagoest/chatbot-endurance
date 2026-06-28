# 🏃‍♂️🚴‍♀️ Coach Virtual de Endurance com GenAI e Streamlit

Um relatório técnico sobre escolhas de LLMs, arquitetura leve de software e os desafios reais de deploy na nuvem.

## 📝 Introdução

### Objetivo da Atividade

O objetivo deste projeto foi desenvolver e implantar um Chatbot Inteligente baseado em Inteligência Artificial Generativa utilizando um modelo open source disponibilizado pela NVIDIA. A persona escolhida foi um Treinador de Endurance (atividade esportiva de longa duração), capaz de simular um treinador especializado em corrida e ciclismo. A meta principal foi unir o poder de processamento de linguagem natural dos Grandes Modelos de Linguagem (LLMs) com uma interface de usuário rápida, segura e de baixo custo, consolidando conceitos práticos de engenharia de prompt, controle de estado, segurança de credenciais e DevOps na nuvem.

### Visão Geral da Solução Desenvolvida

A solução consiste em uma aplicação web interativa desenvolvida em Streamlit, que atua como frontend do usuário. O "cérebro" do assistente é alimentado pelo modelo Llama 3.1 8B da Meta (disponibilizado via API de inferência da NVIDIA), parametrizado com instruções estritas (System Prompt) de segurança médica, conversão de métricas esportivas (pace em min/km, velocidade em km/h e zonas de esforço) e formatação estruturada de planilhas de treino.

### ☁️ Infraestrutura

Para a sustentação da aplicação, exploramos caminhos que simulam o ciclo de vida real de um ambiente de desenvolvimento e produção:

- Instância de Destino: vm_genai_prod (Oracle Cloud Infrastructure - OCI)

- Sistema Operacional: Ubuntu Server (Canonical)

- Recursos Computacionais: Instância Always Free da Oracle Cloud.

- Rede e Segurança: Configuração de sub-redes públicas, regras de entrada (Ingress Rules) liberando a porta 8501 para tráfego TCP externo, e controle de acesso via chaves SSH de criptografia assimétrica.

Nota de Arquitetura: Durante o projeto, não foi possível estabelecer conexão SSH com a VM da Ordacle. Portando, a implantação foi finalizada na Streamlit Community Cloud, uma plataforma serveless (PaaS) otimizada que permitiu alta disponibilidade e integração contínua (CI/CD) direta do repositório GitHub sem o overhead de gerenciar sistemas operacionais.

### 🧠 Modelo Escolhido

A escolha do LLM foi estratégica para garantir precisão técnica, respostas altamente contextualizadas e velocidade:

Nome do Modelo: meta/llama-3.1-8b-instruct (NVIDIA NIM)

Justificativa da Escolha:
O Llama 3.1 de 8 bilhões de parâmetros oferece um excelente balanço entre precisão e velocidade de resposta. A API de inferência da NVIDIA permitiu o uso gratuito para desenvolvimento com latência extremamente baixa.

Principais Características:

Context Window de 128k: Permite manter conversas longas e históricos complexos de treinamento sem perda de contexto.

Alinhamento Estrito a Instruções (System Prompt): Crucial para garantir que as diretrizes de segurança (como recomendar um médico se o usuário relatar dores ou lesões) sejam seguidas rigorosamente.

Habilidade de Formatação: Excelente capacidade de estruturar tabelas de treino em Markdown legíveis para o atleta.

### 🛠️ Desenvolvimento

#### Arquitetura da Aplicação

A aplicação adota um padrão cliente-servidor leve, baseado em estados:

``` bash
[ Usuário ] ──(Interface Web)──> [ Streamlit App ]
                                       │
                         (Leitura de Secrets.toml / Env)
                                       │
                                       ▼
 [ NVIDIA API (Llama 3.1 70B) ] <──(OpenAI SDK)
```
O controle do fluxo conversacional é gerenciado pelo ```st.session_state```, garantindo que o histórico de mensagens seja enviado a cada nova requisição para manter a coerência do diálogo.

#### Bibliotecas Utilizadas

- ```streamlit```: Construção rápida da interface e controle de estado do chat.

- ```openai```: SDK utilizado para padronizar as requisições à API da NVIDIA (compatível com o protocolo da OpenAI).

#### Estratégia de Gerenciamento de Credenciais

Segurança foi prioridade absoluta. A chave de API (NVIDIA_API_KEY) nunca foi exposta no código-fonte ou enviada ao GitHub:

- Ambiente Local: Armazenada no arquivo .streamlit/secrets.toml.

- Controle de Versão: Arquivo explicitamente ignorado pelo Git adicionando .streamlit/ no .gitignore.

- Ambiente de Produção (Cloud): Injetada de forma segura nas configurações de ambiente secretas da plataforma de hospedagem.

### 🚀 Implantação

Processo de Publicação na Oracle Cloud

O plano de deploy original foi planejado cos seguintes passos no servidor Linux:

1. Acesso via conexão SSH.

2. Atualização de pacotes e instalação do Python 3, venv e git.

3. Clone do repositório do GitHub e isolamento do ambiente virtual (pip install -r requirements.txt).

4. Criação manual do arquivo de segredos para a API da NVIDIA.

5. Liberação da porta 8501 nas políticas de segurança da subnet da Oracle.

6. Execução em background via terminal persistente (nohup streamlit run app.py &).

Porém, não foi possível estabalecer a conexão SSH e gerou um dasafio adicional para encontrar uma alternativa. A escolha foi utilizar o Streamlit Cloud da seguinte forma:

1. Acessar [Streamlit Cloud](https://share.streamlit.io/)
2. Fazer login usando a conta do GitHub
3. Selecionar a opção Create App > Deploy a public app from GitHub
4. Apontar para o repositório no GitHub
5. Acessar a URL da aplicação [Chatbot Endurance](https://chatbot-endurance-g4cwnhuyu8heilqu5zpjqm.streamlit.app/)

### 💬 Discussão

Possíveis Melhorias Futuras

- RAG (Retrieval-Augmented Generation): Integrar uma base de conhecimento com livros e artigos de fisiologia do exercício para enriquecer as respostas científicas do coach.

- Integração com APIs Esportivas: Conectar a aplicação à API do Strava ou Garmin para que o coach analise os treinos reais realizados pelo usuário e ajuste as planilhas dinamicamente.

- Banco de Dados Dedicado: Substituir a memória temporária do Streamlit por um banco PostgreSQL ou SQLite para persistir os treinos dos usuários em definitivo.

Relatório técnico desenvolvido como parte do programa de especialização em IA Generativa da UFPR.
