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

import pandas as pd
import json

perfil = json.load(open('./bases/perfil_investidor.json'))
transacoes = pd.read_csv('./bases/transacoes.csv')
historico = pd.read_csv('./bases/historico_atendimento.csv')
produto = json.load(open('./bases/produtos_financeiros.json'))

contexto = f"""
CLIENTE: {perfil['nome']},{perfil['idade']} anos,perfil{perfil['perfil_investidor']}
OBJETIVO: {perfil['objetivo_principal']}
PATRIMÔNIO: R$ {perfil['patrimonio_total']} | RESERVA: R$ {perfil['reserva_emergencia_atual']}

TRANSAÇÕES RECENTES:
{transacoes.to_string(index=False)}

ATENDIMENTOS ANTERIORES:
{historico.to_string(index=False)}

PRODUTOS DISPONÍVEIS:
{json.dumps(produtos, indent=2,ensure_ascii=False)}
"""
SYSTEM_PROMPT = """Você é o Doutor Finanças especializado em Previdência Privadas. 

OBJETIVO:
Orientar os usuários de quais as opções que temos no mercado de acordo com o seu perfil.

REGRAS:

Sempre baseie suas respostas nos dados fornecidos
Nunca invente informações financeiras
Se não souber algo, admita e ofereça alternativas
"""
