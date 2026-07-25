# Base de Conhecimento

## Dados Utilizados

Descreva se usou os arquivos da pasta `data`, por exemplo:

| Arquivo | Formato | Utilização no Agente |
|---------|---------|---------------------|
| `historico_atendimento.csv` | CSV | Contextualizar interações anteriores |
| `perfil_investidor.json` | JSON | Personalizar recomendações |
| `produtos_financeiros.json` | JSON | Sugerir produtos adequados ao perfil |
| `transacoes.csv` | CSV | Analisar padrão de gastos do cliente |

> [!TIP]
> **Quer um dataset mais robusto?** Você pode utilizar datasets públicos do [Hugging Face](https://huggingface.co/datasets) relacionados a finanças, desde que sejam adequados ao contexto do desafio.

---

## Adaptações nos Dados

> Você modificou ou expandiu os dados mockados? Descreva aqui.

[Sua descrição aqui]

---

## Estratégia de Integração

### Como os dados são carregados?
[ex: Os JSON/CSV são carregados no início da sessão e incluídos no contexto do prompt]

### Como os dados são usados no prompt?
Serão consultados dinamicamente

---

## Exemplo de Contexto Montado

import json
import requests
import pandas as pd
import streamlit as st

# ==========================
# CONFIGURAÇÕES
# ==========================

OLLAMA_URL = "http://localhost:11434/api/generate"
MODELO = "gpt-oss"   # Altere para o nome do modelo instalado, se necessário

# ==========================
# CARREGAMENTO DOS DADOS
# ==========================

try:
    with open("./bases/perfil_investidor.json", encoding="utf-8") as f:
        perfil = json.load(f)

    with open("./bases/produtos_financeiros.json", encoding="utf-8") as f:
        produtos = json.load(f)

    transacoes = pd.read_csv("./bases/transacoes.csv")
    historico = pd.read_csv("./bases/historico_atendimento.csv")

except Exception as e:
    st.error(f"Erro ao carregar os arquivos: {e}")
    st.stop()

# ==========================
# CONTEXTO DO CLIENTE
# ==========================

contexto = f"""
CLIENTE
Nome: {perfil['nome']}
Idade: {perfil['idade']} anos
Perfil de Investidor: {perfil['perfil_investidor']}

OBJETIVO PRINCIPAL
{perfil['objetivo_principal']}

PATRIMÔNIO
Patrimônio Total: R$ {perfil['patrimonio_total']}
Reserva de Emergência: R$ {perfil['reserva_emergencia_atual']}

TRANSAÇÕES RECENTES
{transacoes.to_string(index=False)}

HISTÓRICO DE ATENDIMENTOS
{historico.to_string(index=False)}

PRODUTOS DISPONÍVEIS
{json.dumps(produtos, indent=2, ensure_ascii=False)}
"""

# ==========================
# PROMPT DO SISTEMA
# ==========================

SYSTEM_PROMPT = """
Você é o Doutor Finanças, especialista em Previdência Privada.

Sua função é orientar clientes utilizando exclusivamente os dados fornecidos.

Regras:

- Baseie todas as respostas no contexto enviado.
- Nunca invente informações financeiras.
- Caso alguma informação esteja ausente, informe isso ao usuário.
- Seja claro, objetivo e didático.
- Quando adequado, apresente vantagens e desvantagens das opções.
- Utilize linguagem simples.
"""

# ==========================
# FUNÇÃO DE CONSULTA AO OLLAMA
# ==========================

def perguntar(pergunta: str) -> str:

    prompt = f"""
{SYSTEM_PROMPT}

==============================
CONTEXTO DO CLIENTE
==============================

{contexto}

==============================
PERGUNTA DO CLIENTE
==============================

{pergunta}
"""

    try:
        resposta = requests.post(
            OLLAMA_URL,
            json={
                "model": MODELO,
                "prompt": prompt,
                "stream": False
            },
            timeout=120
        )

        resposta.raise_for_status()

        dados = resposta.json()

        return dados.get(
            "response",
            "Não foi possível obter uma resposta do modelo."
        )

    except requests.exceptions.ConnectionError:
        return (
            "Não foi possível conectar ao Ollama.\n\n"
            "Verifique se ele está em execução com:\n\n"
            "ollama serve"
        )

    except requests.exceptions.Timeout:
        return "A resposta demorou mais que o esperado."

    except Exception as e:
        return f"Ocorreu um erro: {e}"

# ==========================
# INTERFACE STREAMLIT
# ==========================

st.set_page_config(
    page_title="Doutor Finanças",
    page_icon="💰",
    layout="wide"
)

st.title("💰 Doutor Finanças")
st.subheader("Seu Educador Financeiro em Previdência Privada")

st.write(
    "Faça perguntas sobre Previdência Privada com base no perfil do cliente."
)

pergunta = st.chat_input(
    "Digite sua dúvida sobre Previdência Privada..."
)

if pergunta:

    with st.chat_message("user"):
        st.write(pergunta)

    with st.chat_message("assistant"):

        with st.spinner("Consultando o especialista..."):

            resposta = perguntar(pergunta)

            st.write(resposta)



"""
