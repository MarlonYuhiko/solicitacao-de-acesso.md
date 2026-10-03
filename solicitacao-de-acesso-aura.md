# Solicitação de Acesso

Documento de referência técnica que define os dados, estados e regras utilizados na funcionalidade de autorização de visitantes do aplicativo Aura Smart Apartment

## 1. Objetivo

A funcionalidade de Solicitação de Acesso será utilizada pelo perfil morador** para autorizar previamente a entrada de visitantes nas dependências do condomínio.

O objetivo deste documento é padronizar os dados e as regras dessa funcionalidade, garantindo que Bubble, Make e Python utilizem a mesma estrutura de informações durante o desenvolvimento e a integração do sistema.

---

## 2. Dados necessários

### 2.1 Dados do morador solicitante

| Variável | Tipo | Obrigatório | Exemplo | Descrição |
|---|---|---|---|---|
| `morador_id` | string | Sim | `USR001` | Identificador único do morador que criou a solicitação |
| `torre` | string | Sim | `A` | Torre ou bloco associado ao morador |
| `apartamento` | string | Sim | `102` | Apartamento associado ao morador |

> `torre` e `apartamento` serão tratados como `string`, pois podem conter letras, números ou combinações, como `A`, `1`, `102A` ou `Cobertura 2`.

### 2.2 Dados do visitante

| Variável | Tipo | Obrigatório | Exemplo | Descrição |
|---|---|---|---|---|
| `nome_visitante` | string | Sim | `João Silva` | Nome do visitante autorizado |
| `tipo_documento` | string | Sim | `CIN` | Tipo do documento que será apresentado na portaria |
| `numero_documento` | string | Sim | `12345678900` | Número do documento informado |

Exemplos de `tipo_documento`:

- `CIN`
- `RG`
- `CNH`
- `PASSAPORTE`

### 2.3 Dados da visita

| Variável | Tipo | Obrigatório | Exemplo | Descrição |
|---|---|---|---|---|
| `data_visita` | date | Sim | `2026-10-10` | Data autorizada para a visita |
| `horario_entrada` | time | Sim | `12:22` | Horário inicial autorizado |
| `horario_saida` | time | Sim | `18:30` | Horário final autorizado |
| `status_solicitacao` | string | Sim | `AGUARDANDO_VALIDACAO` | Estado atual da solicitação |

> Para integração entre sistemas, datas devem utilizar preferencialmente o padrão `AAAA-MM-DD`.

---

## 3. Status possíveis

| Status | Significado |
|---|---|
| `AGUARDANDO_VALIDACAO` | Solicitação criada e aguardando validação das regras do sistema |
| `APROVADA` | Solicitação validada e autorizada |
| `INVALIDA` | Solicitação rejeitada por não atender uma ou mais regras |
| `DENTRO` | Entrada do visitante já foi registrada pela portaria |
| `FINALIZADA` | Saída do visitante foi registrada e a visita foi encerrada |

### Fluxo esperado dos status

```text
AGUARDANDO_VALIDACAO
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

Caso a solicitação não atenda às regras:

```text
AGUARDANDO_VALIDACAO
        |
        v
     INVALIDA
```

---

## 4. Regras de validação

A primeira versão da função `validar_solicitacao_acesso()` deverá verificar:

1. `morador_id` deve existir e não pode estar vazio.
2. `torre` deve existir e não pode estar vazia.
3. `apartamento` deve existir e não pode estar vazio.
4. `nome_visitante` deve existir e não pode estar vazio.
5. `tipo_documento` deve existir e não pode estar vazio.
6. `numero_documento` deve existir e não pode estar vazio.
7. `data_visita` não pode representar uma data anterior à data atual.
8. `horario_entrada` deve ser anterior a `horario_saida`.
9. Todos os campos obrigatórios devem estar presentes antes da solicitação ser aprovada.

Caso todas as regras sejam atendidas, o sistema deverá retornar:

```text
APROVADA
```

Caso alguma regra não seja atendida:

```text
INVALIDA
```

O sistema também deverá informar o motivo da invalidação.

---

## 5. Exemplo de entrada

```json
{
  "morador_id": "USR001",
  "torre": "A",
  "apartamento": "102",
  "nome_visitante": "João Silva",
  "tipo_documento": "CIN",
  "numero_documento": "12345678900",
  "data_visita": "2026-10-10",
  "horario_entrada": "12:22",
  "horario_saida": "18:30",
  "status_solicitacao": "AGUARDANDO_VALIDACAO"
}
```

---

## 6. Exemplo de saída válida

```json
{
  "status_solicitacao": "APROVADA",
  "motivo": "Solicitação válida"
}
```

## 7. Exemplo de saída inválida

```json
{
  "status_solicitacao": "INVALIDA",
  "motivo": "O horário de saída deve ser posterior ao horário de entrada"
}
```

---

## 8. Função Python relacionada

A primeira entrega em Python poderá utilizar a seguinte estrutura:

```python
def validar_solicitacao_acesso(solicitacao):
    pass
```

A função receberá uma solicitação contendo os dados definidos neste documento e retornará, no mínimo:

- `status_solicitacao`
- `motivo`

A lógica de login do usuário não será responsabilidade desta função. A autenticação do morador será realizada pelo Bubble.

---

## 9. Responsabilidade da primeira entrega em Python

A primeira entrega relacionada a esta documentação consiste em implementar a lógica de validação da solicitação de acesso.

A função deverá:

- receber os dados da solicitação;
- verificar os campos obrigatórios;
- validar data e horários;
- retornar `APROVADA` ou `INVALIDA`;
- informar o motivo quando uma solicitação for considerada inválida;
- ser testada com diferentes cenários antes da integração com Bubble e Make.

---

## 10. Observações técnicas

- Os nomes das variáveis devem permanecer padronizados entre **Bubble, Make, JSON e Python**.
- Variáveis como `numero_documento`, `torre` e `apartamento` devem ser tratadas como texto, pois são identificadores e não valores utilizados em cálculos.
- Este documento representa a primeira versão da funcionalidade e poderá ser atualizado conforme novas regras forem definidas.
- Alterações nos nomes dos campos devem ser comunicadas à equipe para evitar incompatibilidades entre as ferramentas.
