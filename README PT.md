# Pi Network - Python server-side package

Esta é uma solução baseada em Python gerada pelo usuário para a Pi Network que você pode usar para integrar a plataforma de aplicativos da Pi Network com um aplicativo de backend Python.

pi_python.py é a biblioteca e pi_python_test.py é para testar a biblioteca.

## Exemplo

1. Inicialize o SDK e insira seus dados secretos
```python
""" Dados Secretos """
api_key = "Digite Aqui Sua Chave de API"
wallet_private_seed = "SecretWalletSeed"

""" Inicialização """
pi = PiNetwork()
pi.initialize(api_key, wallet_private_seed, "Pi Testnet")
```

2. Crie um pagamento A2U

Certifique-se de armazenar seus dados de pagamento em seu banco de dados. Veja um exemplo de como você pode acompanhar os dados.
Considere este um exemplo de tabela de banco de dados.

| uid | product_id | amount | memo | payment_id | txid |
| :---: | :---: | :---: | :---: | :---: | :---: |
| `user_uid` | apple-pie-1 | 3.14 | Reembolso para torta de maçã | NULL | NULL |

```python
user_uid = "GET-THIS-SECRET-DATA-FROMFRONTEND" #único para cada usuário

""" Crie seu pagamento """
payment_data = {
"amount": 3.14,
"memo": "Teste - Saudações do MyApp",
"metadata": {"product_id": "apple-pie-1"},
"uid": user_uid
}

"""
Criar um pagamento
É fundamental que você armazene o paymentId em seu banco de dados
para não pagar o mesmo usuário duas vezes, mantendo o controle do pagamento.

"""
payment_id = pi.create_payment(payment_data)
```

3. Armazene o `paymentId` em seu banco de dados

Após criar o pagamento, você receberá o `paymentId`, que deverá ser armazenado em seu banco de dados.

| uid | product_id | valor | memorando | ID_do_pagamento | txid |
| :---: | :---: | :---: | :---: | :---: |
| `uid_do_usuário` | torta-de-maçã-1 | 3.14 | Reembolso para torta de maçã | `ID_do_pagamento` | NULO |

4. Envie o pagamento para o Blockchain Pi
```python
"""É altamente recomendável que você armazene o txid junto com o ID_do_pagamento que você armazenou anteriormente para sua referência."""
txid = pi.submit_payment(id_do_pagamento, False)
```

5. Armazene o txid em seu banco de dados

Da mesma forma que você fez na etapa 3, mantenha o txid junto com outros dados.

| uid | id_do_produto | valor | memorando | ID_do_pagamento | txid |
| :---: | :---: | :---: | :---: | :---: | :---: |
| `user_uid` | apple-pie-1 | 3.14 | Reembolso para apple pie | `paymentId` | `txid` |

6. Conclua o pagamento
```python
payment = pi.complete_payment(payment_id, txid)
```

## Fluxo geral para pagamento A2U (Aplicativo para Usuário)

Para criar um pagamento A2U usando o SDK Python do Pi, aqui está um fluxo geral que você precisa seguir:

1. Inicializar o SDK
> Você inicializará o SDK com a Chave de API do Pi do seu aplicativo e a Seed Privada da sua carteira de aplicativos.

2. Criar um pagamento A2U
> Você pode criar um pagamento A2U usando o método `createPayment`. Este método retorna um identificador de pagamento (ID de pagamento).

3. Armazene o ID de pagamento em seu banco de dados
> É fundamental que você armazene o ID de pagamento, retornado pelo método `createPayment`, em seu banco de dados para que você não pague duas vezes ao mesmo usuário, mantendo o controle do pagamento.

4. Envie o pagamento para a Blockchain Pi
> Você pode enviar o pagamento para a Blockchain Pi usando o método `submitPayment`. Este método cria uma transação de pagamento e a envia para a Blockchain Pi para você. Uma vez enviada, o método retorna um identificador de transação (txid).

5. Armazene o txid em seu banco de dados
> É altamente recomendável que você armazene o txid junto com o ID de pagamento armazenado anteriormente para sua referência.

6. Conclua o pagamento
> Após verificar a transação com o txid obtido, você deve concluir o pagamento, o que pode ser feito com o método `completePayment`. Após a conclusão, o método retorna o objeto de pagamento. Verifique o campo `status` para garantir que tudo esteja correto.

## Referência do SDK

Esta seção mostra uma lista de métodos disponíveis.
### `createPayment`

Este método cria um pagamento A2U.

- Parâmetro obrigatório: `PaymentArgs`

Você precisa fornecer 4 dados diferentes e passá-los como um único objeto para este método.
```typescript
type PaymentArgs = {
amount: number // o valor em Pi que você está pagando ao seu usuário
memo: string // um breve memorando que descreve o pagamento
metadata: object // um objeto arbitrário que você pode anexar a este pagamento. Isto é para seu próprio uso. Você deve usar este objeto como uma forma de vincular este pagamento à sua lógica de negócios interna.
uid: string // um uid de usuário do seu aplicativo. Você deve ter acesso a este valor se um usuário tiver se autenticado no seu aplicativo.
}
```

- Valor de retorno: `um identificador de pagamento (paymentId: string)`

### `submitPayment`

Este método cria uma transação de pagamento e a envia para o Blockchain Pi.

- Parâmetro obrigatório: `paymentId, pending_payments`
- Valor de retorno: `um identificador de transação (txid: string)`

### `completePayment`

Este método conclui o pagamento no servidor Pi.

- Parâmetro obrigatório: `paymentId, txid`
- Valor de retorno: `um objeto de pagamento (payment: PaymentDTO)`

O método retorna um objeto de pagamento com os seguintes campos:

```typescript
payment: PaymentDTO = {
// Dados do pagamento:
identifier: string, // identificador do pagamento
user_uid: string,