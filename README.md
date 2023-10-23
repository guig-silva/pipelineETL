**Contexto:** Sou é um cientista de dados no Santander e recebi a tarefa de envolver seus clientes de maneira mais personalizada. O objetivo é usar o poder da IA Generativa para criar mensagens de marketing personalizadas que serão entregues a cada cliente.

O desafio consistiu em:

1. O trabalho é consumir o endpoint `GET https://sdw-2023-prd.up.railway.app/users/{id}` (API da Santander Dev Week 2023) para obter os dados de cada cliente.
2. Depois de obter os dados dos clientes, deveria-se utilizar a API do ChatGPT (OpenAI) para gerar uma mensagem de marketing personalizada para cada cliente. Essa mensagem deveria enfatizar a importância dos investimentos.
3. Uma vez que a mensagem para cada cliente estivesse pronta, essas informações seriam enviadas de volta para a API, atualizando a lista de "news" de cada usuário usando o endpoint `PUT https://sdw-2023-prd.up.railway.app/users/{id}`.

(E)XTRAÇÃO: Extraí uma lista de IDs de usuário a partir do arquivo CSV. Para cada ID, é necessário uma requisição GET para obter os dados do usuário correspondente.

import pandas as pd

df = pd.read_csv('SDW2023.csv')
user_ids = df['UserID'].tolist()
print(user_ids)

import requests
import json

def get_user(id):
  response = requests.get(f'{sdw2023_api_url}/users/{id}')
  return response.json() if response.status_code == 200 else None

users = [user for id in user_ids if (user := get_user(id)) is not None]
print(json.dumps(users, indent=2))

(T)RANSFORMAÇÃO: A API do OpenAI GPT-4 é utilizada para gerar uma mensagem de marketing personalizada para cada usuário.

import openai

openai.api_key = openai_api_key

def generate_ai_news(user):
  completion = openai.ChatCompletion.create(
    model="gpt-4",
    messages=[
      {
          "role": "system",
          "content": "Você é um especialista em markting bancário."
      },
      {
          "role": "user",
          "content": f"Crie uma mensagem para {user['name']} sobre a importância dos investimentos (máximo de 100 caracteres)"
      }
    ]
  )
  return completion.choices[0].message.content.strip('\"')

for user in users:
  news = generate_ai_news(user)
  print(news)
  user['news'].append({
      "icon": "https://digitalinnovationone.github.io/santander-dev-week-2023-api/icons/credit.svg",
      "description": news
  })

(L)OAD: A lista de "news" de cada usuário na API com a nova mensagem gerada.

def update_user(user):
  response = requests.put(f"{sdw2023_api_url}/users/{user['id']}", json=user)
  return True if response.status_code == 200 else False

for user in users:
  success = update_user(user)
  print(f"User {user['name']} updated? {success}!")
