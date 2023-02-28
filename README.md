1. Crie uma função Lambda no console do AWS Lambda
    
    - Acesse o console do AWS Lambda e clique no botão "Criar função"
    - Escolha "Autor do zero/Author from scratch" e dê um nome à sua função
    - Escolha a linguagem Node.js
    - Clique em "Criar função" para criar a sua função
2. Adicione a política de acesso à função Lambda para gerenciar as instâncias EC2 (Configuration > Execution role)
    
    - Clique na guia "Permissões" na página da sua função
    - Clique no botão "Adicionar política"
    - Selecione a política "AmazonEC2FullAccess" e clique em "Adicionar política"
3. Adicione o código para ligar e desligar as instâncias com tag Desliga com valor true
    
    - Substitua o código da função Lambda pelo seguinte:
    
```
const AWS = require('aws-sdk');
const ec2 = new AWS.EC2();

exports.handler = async (event) => {
  const instances = await getInstances();
  for (const instance of instances) {
    if (instance.Tags && instance.Tags.find(tag => tag.Key === 'Desliga' && tag.Value === 'true')) {
      if (instance.State.Name === 'running') {
        console.log(`Desligando a instância ${instance.InstanceId}`);
        await ec2.stopInstances({ InstanceIds: [instance.InstanceId] }).promise();
      } else if (instance.State.Name === 'stopped') {
        console.log(`Ligando a instância ${instance.InstanceId}`);
        await ec2.startInstances({ InstanceIds: [instance.InstanceId] }).promise();
      }
    }
  }
};

async function getInstances() {
  const instances = [];
  let result;
  do {
    result = await ec2.describeInstances().promise();
    result.Reservations.forEach(reservation => {
      reservation.Instances.forEach(instance => {
        instances.push(instance);
      });
    });
  } while (result.NextToken);
  return instances;
}
```

4. Adicione o trigger do EventBridge
    
    - Clique na guia "EventBridge" na página da sua função
    - Clique no botão "Adicionar trigger"
    - Selecione o tipo de evento que você quer usar como trigger (por exemplo, "CloudWatch Events - Schedule")
    - Configure o evento de acordo com suas necessidades (por exemplo, para disparar a função a cada 5 minutos, use a expressão cron "0/5 \* \* \* ? \*")
5. Salve e teste sua função Lambda
    
    - Clique no botão "Salvar" para salvar a sua função Lambda
    - Clique no botão "Testar" para testar sua função Lambda com um evento de exemplo
    - Verifique se as instâncias com a tag "Desliga" e valor "true" são ligadas ou desligadas corretamente.
---
Este código em Node.js é uma função Lambda que é executada pelo serviço AWS Lambda e é projetada para ser acionada por um evento do Amazon EventBridge.

O objetivo da função é verificar o estado de todas as instâncias EC2 em uma conta da AWS e ligar ou desligar aquelas que possuem uma tag "Desliga" com valor "true".

Aqui está uma explicação do que cada parte do código faz:

```
const AWS = require('aws-sdk');
const ec2 = new AWS.EC2();
```

Essas linhas importam o AWS SDK para Node.js e criam um objeto EC2 que a função usará para se comunicar com o serviço EC2 da AWS.

```
exports.handler = async (event) => {
```

Essa linha exporta a função Lambda como um módulo do Node.js e define a função de tratamento de eventos.

```
const instances = await getInstances();
```

Essa linha chama a função "getInstances" para obter informações sobre todas as instâncias EC2 na conta da AWS.

```
for (const instance of instances) {
```

Essa linha inicia um loop que percorre todas as instâncias EC2 retornadas pela função "getInstances".

```
if (instance.Tags && instance.Tags.find(tag => tag.Key === 'Desliga' && tag.Value === 'true')) {
```
Essa linha verifica se a instância atual tem uma tag chamada "Desliga" com valor "true".

```
if (instance.State.Name === 'running') {
```
Se a instância estiver em execução, a função chama a API "stopInstances" para desligá-la. Caso contrário, se a instância estiver parada, a função chama a API "startInstances" para iniciá-la.
```
async function getInstances() {
```
Essa linha define a função "getInstances", que usa a API "describeInstances" para obter informações sobre todas as instâncias EC2 na conta da AWS.
```
const instances = [];
let result;
do {
  result = await ec2.describeInstances().promise();
  result.Reservations.forEach(reservation => {
    reservation.Instances.forEach(instance => {
      instances.push(instance);
    });
  });
} while (result.NextToken);
return instances;
```

Essa parte do código é a implementação da função "getInstances". A função usa a API "describeInstances" para obter informações sobre todas as instâncias EC2 na conta da AWS. O resultado é paginado, então a função usa um loop "do-while" para processar todas as páginas de resultados. Para cada instância, a função adiciona o objeto de instância ao array "instances" e, quando não houver mais páginas, retorna o array completo.
