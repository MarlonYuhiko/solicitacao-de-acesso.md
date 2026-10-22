# Solicitação de Acesso

Documento referencial que define os dados, regras, estados e integração técnica utilizados na autorização de visitantes no aplicativo **Aura Smart Apartment**.

## 1. Objetivo

Esta funcionalidade é voltada para o usuário do tipo **Morador**. Ela será utilizada quando o morador desejar autorizar a entrada de uma visita cujo destino esteja dentro das dependências do condomínio que utiliza o aplicativo Aura Smart Apartment para auxiliar na gestão de entrada de pessoas.

Este documento tem como objetivo padronizar e estruturar tecnicamente essa funcionalidade, mantendo os dados e as regras de validação coerentes entre Bubble, Make e Python.

---

## 2. Dados necessários

### 2.1 Dados do morador solicitante

| Variável | Tipo | Obrigatório | Exemplo de valor que será atribuído à variável | Descrição |
|---|---|---|---|---|
| `morador_id` | string | Sim | `USR001` | Identificador único do morador |
| `torre` | string | Sim | `1`, `A`, `2`, `B`... | Torre ou bloco do morador |
| `apartamento` | string | Sim | `102`, `103`, `21`, `12`... | Apartamento do morador |

> `torre` e `apartamento` serão tratados como `string`, pois são identificadores e podem conter letras, números ou combinações.

### 2.2 Dados do visitante

| Variável | Tipo | Obrigatório | Exemplo de valor que será atribuído à variável | Descrição |
|---|---|---|---|---|
| `nome_visitante` | string | Sim | `Gustavo Soares`, `Matheus`, `Eduardo`, `Pedro`, `Marlon Infante`... | Nome do visitante |
| `tipo_documento` | string | Sim | `CNH`, `RG`, `CPF` | Tipo do documento apresentado |
| `numero_documento` | string | Sim | `123456789` | Número do documento |

### 2.3 Dados da visita

| Variável | Tipo | Obrigatório | Exemplo de valor que será atribuído à variável | Descrição |
|---|---|---|---|---|
| `data_visita` | date | Sim | `2026-10-10` | Data do dia da visita |
| `horario_entrada` | time | Sim | `12:22` | Horário inicial autorizado da visita |
| `horario_saida` | time | Sim | `18:30` | Horário final autorizado |

> A data deve seguir o formato `AAAA-MM-DD`, por exemplo: `2026-10-10`.

---

## 3. Dados de entrada da API

O `status_solicitacao` **não será enviado como entrada para a API**. A própria API será responsável por determinar se a solicitação é válida ou inválida.

Exemplo de entrada:

```json
{
  "morador_id": "USR001",
  "torre": "A",
  "apartamento": "102",
  "nome_visitante": "Gustavo Soares",
  "tipo_documento": "CNH",
  "numero_documento": "123456789",
  "data_visita": "2026-10-10",
  "horario_entrada": "12:22",
  "horario_saida": "18:30"
}
```

---

## 4. Regras de validação

A função `validar_solicitacao_acesso()` deverá verificar:

1. `morador_id` deve existir e não pode estar vazio.
2. `torre` deve existir e não pode estar vazia.
3. `apartamento` deve existir e não pode estar vazio.
4. `nome_visitante` deve existir e não pode estar vazio.
5. `tipo_documento` deve existir e não pode estar vazio.
6. `numero_documento` deve existir e não pode estar vazio.
7. `data_visita` deve estar no formato `AAAA-MM-DD`.
8. `data_visita` não pode ser anterior à data atual.
9. `horario_entrada` e `horario_saida` devem estar no formato `HH:MM`.
10. `horario_entrada` deve ser anterior ao `horario_saida`.

Caso todas as regras sejam atendidas, a API deverá retornar:

```text
APROVADA
```

Caso alguma regra não seja atendida:

```text
INVALIDA
```

A API também deverá informar o motivo do resultado.

---

## 5. Status possíveis da funcionalidade

| Status | Significado |
|---|---|
| `VERIFICANDO` | Solicitação criada no aplicativo e aguardando o resultado da validação |
| `APROVADA` | Solicitação validada e autorizada |
| `INVALIDA` | Solicitação rejeitada por não atender uma ou mais regras |
| `DENTRO` | Entrada do visitante já foi registrada pela portaria |
| `FINALIZADA` | Saída do visitante foi registrada e a visita foi encerrada |

### Fluxo esperado

```text
VERIFICANDO
    |
    v
 APROVADA
    |
    v
  DENTRO
    |
    v
FINALIZADA
```

Caso a validação falhe:

```text
VERIFICANDO
    |
    v
 INVALIDA
```

> `VERIFICANDO` representa um estado do aplicativo durante o processo. Ele não precisa ser enviado para a função de validação em Python.

---

## 6. Saída da API

### Exemplo de solicitação aprovada

```json
{
  "status_solicitacao": "APROVADA",
  "motivo": "Solicitação válida"
}
```

### Exemplo de solicitação inválida

```json
{
  "status_solicitacao": "INVALIDA",
  "motivo": "O horário de saída deve ser posterior ao horário de entrada"
}
```

---

## 7. API de validação

A validação da Solicitação de Acesso foi disponibilizada por meio de uma API em Python.

### Endpoint

```text
POST /validar
```

### URL publicada

```text
https://expotech-aura.vercel.app/validar
```

A rota utiliza o método `POST`, pois recebe os dados da solicitação em formato JSON e retorna o resultado da validação.

Abrir a URL diretamente no navegador envia normalmente uma requisição `GET`, portanto pode retornar `Method Not Allowed`. Isso não significa que a API esteja indisponível.

---

## 8. Função Python relacionada

A lógica principal segue a seguinte responsabilidade:

```python
def validar_solicitacao_acesso(solicitacao):
    pass
```

A função recebe os dados da solicitação e retorna:

- `status_solicitacao`
- `motivo`

A autenticação e o login do morador não são responsabilidade dessa função. Essa parte será tratada pelo Bubble.

---

## 9. Fluxo técnico esperado

A integração planejada para essa funcionalidade é:

```text
Morador
   |
   v
Bubble
   |
   v
Make
   |
   v
API Python / Vercel
   |
   v
Make
   |
   v
Bubble
```

### Responsabilidade de cada parte

**Bubble**
- Interface utilizada pelo morador.
- Coleta os dados da solicitação.
- Exibe o resultado e o status da solicitação.

**Make**
- Recebe os dados do Bubble.
- Envia uma requisição HTTP `POST` para a API.
- Recebe a resposta da API.
- Encaminha o resultado para o restante do fluxo.

**Python / API**
- Recebe os dados em JSON.
- Aplica as regras de validação.
- Retorna `APROVADA` ou `INVALIDA`.
- Retorna também o motivo da decisão.

**Vercel**
- Mantém a API disponível por uma URL pública para que o Make consiga acessá-la.

---

## 10. Arquivos da API

A implementação atual da API utiliza:

```text
access.py
requirements.txt
vercel.json
```

### `access.py`

Contém:
- criação da aplicação FastAPI;
- função `validar_solicitacao_acesso()`;
- regras de validação;
- endpoint `POST /validar`.

### `requirements.txt`

Declara as dependências utilizadas pela aplicação Python.

### `vercel.json`

Define a configuração necessária para executar e rotear a aplicação Python na Vercel.

---

## 11. Testes mínimos

Antes de considerar a funcionalidade concluída, devem ser testados pelo menos os seguintes cenários:

| Cenário | Resultado esperado |
|---|---|
| Todos os campos válidos | `APROVADA` |
| Campo obrigatório vazio | `INVALIDA` |
| Data anterior ao dia atual | `INVALIDA` |
| Data em formato incorreto | `INVALIDA` |
| Horário em formato incorreto | `INVALIDA` |
| Horário de saída anterior ou igual ao de entrada | `INVALIDA` |

---

## 12. Observações técnicas

- Os nomes das variáveis devem permanecer iguais entre Bubble, Make, JSON e Python.
- `numero_documento`, `torre` e `apartamento` devem ser tratados como texto.
- `data_visita` deve seguir o padrão `AAAA-MM-DD`.
- O status retornado pela API não deve ser enviado como dado de entrada.
- Alterações nos nomes dos campos ou regras devem ser comunicadas à equipe para evitar incompatibilidades.
- Este documento representa a versão atual da funcionalidade e poderá ser atualizado durante os próximos Sprints.
