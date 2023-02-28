1. Crie uma função Lambda no console do AWS Lambda
    
    - Acesse o console do AWS Lambda e clique no botão "Criar função"
    - Escolha "Autor do zero/Author from scratch" e dê um nome à sua função
    - Escolha a linguagem Python 3.9
    - Clique em "Criar função" para criar a sua função
2. Adicione a política de acesso à função Lambda para gerenciar as instâncias EC2 (Configuration > Execution role)
    
    - Clique na guia "Permissões" na página da sua função
    - Clique no botão "Adicionar política"
    - Selecione a política "AmazonEC2FullAccess" e clique em "Adicionar política"
    - Em "Configuration" > "General Configuration", altere o tempo de execução para 5 minutos.
3. Adicione o código para ligar e desligar as instâncias com tag Desliga com valor true
    
    - Substitua o código da função Lambda pelo seguinte: (Clique em Deploy para salvar)
    
```
import boto3

ec2 = boto3.client('ec2')

def lambda_handler(event, context):
    instances = []
    filters = [
        {
            'Name': 'tag:Desliga',
            'Values': ['true']
        },
        {
            'Name': 'instance-state-name', 
            'Values': ['running', 'stopped']
        }
    ]
    response = ec2.describe_instances(Filters=filters)
    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            instances.append(instance['InstanceId'])

    # Ligando as instâncias
    ec2.start_instances(InstanceIds=instances)
    print('Instâncias ligadas: ' + str(instances))
    
    # Desligando as instâncias
    ec2.stop_instances(InstanceIds=instances)
    print('Instâncias desligadas: ' + str(instances))
    
    return 'Done!'
```

4. Adicione o trigger do EventBridge
    
    - Clique na guia "Add Trigger" na página da sua função lambda
    - Procure por Event Bridge e selecione-o
    - Selecione "Create new rule"
    - Em "Rule name", defina um nome para sua regra. (Ex.: start_stop_instances)
    - Em "Rule description", defina o que a regra faz. (Ex.: Liga e Desliga instancias EC2 às 23 horas de segunda a sexta)
    - Em "Schedule expression" coloque a seguinte sintaxe: cron(0 23 ? * MON-FRI *)
    - Clique em Add para salvar a regra.

5. Teste sua função Lambda
    
    - Clique no botão "Test" para testar sua função Lambda com um evento de exemplo
    - Verifique se as instâncias com a tag "Desliga" e valor "true" são ligadas ou desligadas corretamente.
---
Este código em Python 3.9 é uma função Lambda que é executada pelo serviço AWS Lambda e é projetada para ser acionada por um evento do Amazon EventBridge.

O objetivo da função é verificar o estado de todas as instâncias EC2 em uma conta da AWS e ligar ou desligar aquelas que possuem uma tag "Desliga" com valor "true".

---
---
Explicação:
Explicação:

- O código acima primeiro define o cliente EC2 com o Boto3.
- Em seguida, ele utiliza a função `describe_instances()` para buscar todas as instâncias com a tag "Desliga" com valor "true" e que estejam nos estados "running" ou "stopped".
- Depois, o código adiciona as IDs das instâncias encontradas em uma lista chamada "instances".
- Por fim, ele utiliza as funções `start_instances()` e `stop_instances()` para ligar e desligar as instâncias, respectivamente, e imprime uma mensagem no console.
