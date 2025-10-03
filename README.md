# AWS Step Functions — Workflow Automatizado 🚀  

## 📌 O que é  
O **AWS Step Functions** é um serviço de **orquestração de workflows serverless** da AWS.  
Ele permite integrar e coordenar serviços como **AWS Lambda, DynamoDB, SQS, SNS, API Gateway e muitos outros**, de forma visual e declarativa.  

Com Step Functions, é possível:  
- Criar fluxos robustos com controle de falhas;  
- Reduzir o acoplamento entre serviços;  
- Obter **observabilidade** completa com histórico de execuções, métricas e logs;  
- Modelar fluxos complexos com **validações, paralelismo e condições lógicas**.  

---

## 🎯 Objetivo do Desafio  
Este repositório consolida os aprendizados sobre **AWS Step Functions**, aplicados na construção de um fluxo automatizado.  

O laboratório teve como foco:  
- Criar e executar workflows de exemplo;  
- Documentar os processos técnicos de forma clara;  
- Compartilhar insights e boas práticas de uso.  

---

## 🛠️ Blocos Básicos do Step Functions  
- **Task** → executa ações (Lambda, DynamoDB, SQS, HTTP...).  
- **Choice** → desvio condicional (if/else).  
- **Pass** → transforma dados sem executar nada.  
- **Wait** → espera por tempo definido ou até data.  
- **Parallel** → executa ramos simultâneos.  
- **Map** → itera sobre listas.  
- **Succeed / Fail** → termina o fluxo com sucesso ou erro.  

---

## 🖼️ Exemplo de Fluxo  
Fluxo construído no laboratório:  

1. **Início** → Start.  
2. **Validação** → Lambda valida os dados recebidos.  
3. **Decisão** → Choice avalia se os dados são válidos.  
4. **Se válido** → Grava no DynamoDB.  
5. **Se inválido** → Retorna erro (Fail).  

📌 **Visão geral:**  

 ## **Workflow Studio — Tela Inicial**

<img width="1851" height="878" alt="inicio" src="https://github.com/user-attachments/assets/60725584-92bf-460a-9aaa-cecb459cead1" />


Interface inicial do Workflow Studio.


O fluxo começa em Start e termina em End.
Podemos arrastar serviços da AWS (Lambda, SNS, DynamoDB, HTTP APIs etc.) para montar o workflow.
À direita, é possível configurar:
Comentário (descrição da state machine),
TimeoutSeconds (tempo máximo da execução),
E a linguagem de query recomendada (JSONata).
````

Start → Lambda (Validate Account) → Choice (Valid?)
├── Válido → DynamoDB PutItem → End
└── Inválido → Fail


````
## Adicionando uma Função Lambda
<img width="1834" height="869" alt="criação" src="https://github.com/user-attachments/assets/1326012d-ffef-403f-8e32-7cb4559345a4" />

Aqui eu conectei o estado Lambda Invoke ao fluxo.

Essa task é responsável por chamar uma função Lambda dentro do fluxo.
O Step Functions já adiciona automaticamente as conexões entre Start → Lambda → End.
Essa função pode ser configurada para receber parâmetros de entrada, validar dados e devolver um resultado que será usado nos próximos estados.

---

## 📑 Exemplo em JSON (State Machine Definition)  
```json
{
  "Comment": "Validação de conta e gravação no DynamoDB",
  "StartAt": "ValidateAccount",
  "States": {
    "ValidateAccount": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Parameters": {
        "FunctionName": "arn:aws:lambda:REGIAO:CONTA:function:validate-account",
        "Payload.$": "$"
      },
      "ResultPath": "$.validation",
      "Retry": [
        {
          "ErrorEquals": ["Lambda.ServiceException", "Lambda.AWSLambdaException", "Lambda.SdkClientException"],
          "IntervalSeconds": 2,
          "MaxAttempts": 3,
          "BackoffRate": 2.0
        }
      ],
      "Catch": [
        {
          "ErrorEquals": ["States.ALL"],
          "ResultPath": "$.error",
          "Next": "ValidationFailed"
        }
      ],
      "Next": "AccountValid?"
    },
    "AccountValid?": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.validation.Payload.isValid",
          "BooleanEquals": true,
          "Next": "PutItemDynamoDB"
        }
      ],
      "Default": "ValidationFailed"
    },
    "PutItemDynamoDB": {
      "Type": "Task",
      "Resource": "arn:aws:states:::dynamodb:putItem",
      "Parameters": {
        "TableName": "Accounts",
        "Item": {
          "accountId": { "S.$": "$.validation.Payload.accountId" },
          "status": { "S": "VALID" }
        }
      },
      "End": true
    },
    "ValidationFailed": {
      "Type": "Fail",
      "Error": "AccountInvalid",
      "Cause": "Conta inválida ou erro na validação"
    }
  }
}

````
---
🖥️ Workflow Studio

O Workflow Studio fornece uma interface drag & drop para montar os
fluxos sem precisar escrever o JSON manualmente.


---
➡️ Recursos úteis do Workflow Studio:

Visualização clara do fluxo;

Teste de payloads mock;

Integração com AWS Application Composer e VS Code Toolkit;

Novos recursos: HTTP Endpoints e TestState (testar estados isolados).

---
📸 Capturas de Tela

Diagrama do fluxo

Configuração da função Lambda

Execução bem-sucedida


---
📚 Recursos Úteis

Documentação Oficial AWS Step Functions

AWS Lambda

AWS DynamoDB

---
✅ Conclusão

Esse laboratório mostrou como o AWS Step Functions pode:

Reduzir complexidade em arquiteturas distribuídas;

Garantir resiliência com Retry/Catch;

Fornecer observabilidade e rastreabilidade de ponta a ponta.

Esse repositório será usado como material de apoio para futuras implementações e estudos.
