# desafio-aws-lambda-run-as-cloud
Neste laboratório, vamos criar uma função AWS Lambda que interrompe uma instância EC2 a cada minuto, utilizando recursos elegíveis ao Free Tier da AWS. Esta atividade faz parte do grupo de estudos Run as Cloud e tem como objetivo te guiar na prática sobre automações com Lambda.

## ⚠️ Atenção antes de começar

Embora este laboratório utilize recursos dentro da camada gratuita da AWS, **é fundamental acompanhar o uso**:

- Monitore o uso no **AWS Billing Dashboard** e no **AWS Cost Explorer**
- **Evite publicar prints com dados sensíveis** (como IDs de conta ou URLs privadas)

## O AWS Lambda é um serviço de computação serverless da Amazon Web Services que permite executar código sem precisar provisionar ou gerenciar servidores.
Em outras palavras:
Você só escreve o código da função, define quando e como ele deve ser executado (por exemplo, em resposta a eventos do S3, DynamoDB, API Gateway, etc.), e o Lambda cuida de todo o resto: escala, execução, infraestrutura, disponibilidade e monitoramento.

---

## 🔐 Etapa Inicial – Criar Role IAM para Lambda

Antes de criar a função Lambda, é necessário criar uma **role IAM** com permissões apropriadas.

### 📌 Nome da role
`lambda-ec2-role`

### 📄 Descrição
Permite que a função Lambda interaja com o serviço EC2 em sua conta AWS.

### 🔑 Políticas anexadas
| Política                         | Tipo           | Descrição                                 |
|----------------------------------|----------------|--------------------------------------------|
| AmazonEC2FullAccess              | AWS gerenciada | Acesso total ao EC2                        |
| AWSLambdaBasicExecutionRole      | AWS gerenciada | Permissões básicas de execução do Lambda   |

### 📸 Print obrigatório:
- Console IAM mostrando a role criada: `lambda-ec2-role`
- Aba Policies da role com as permissões anexadas

![image](https://github.com/user-attachments/assets/aa504c82-08d0-4a11-86f6-66f8aebdbca7)

![image (97)](https://github.com/user-attachments/assets/217ce9ac-f44f-4446-bd5d-998d7a9f6b6c)

### Criar uma Instância EC2 para ser interrompida

- Acesse o Console EC2
- AWS Console > EC2 > Instâncias > Executar instância

Configurações básicas:

- Nome da instância: `instance1`

- AMI: Amazon Linux 2023 (ou similar, Free Tier elegível)

- Tipo de instância: t2.micro (dentro do Free Tier)

- Par de chaves: Pode deixar sem, se não for acessar via SSH

  ### 📸 Print obrigatório:
  
- Instância EC2 para ser interrompida

![image](https://github.com/user-attachments/assets/a218e1d5-8593-42f4-83b1-63db4edbada2)


  

---

## 🟢 Etapa 1 – Criar a função Lambda

1. Acesse **Lambda > Functions > Create function**
2. Configure:
   - **Function name**: `myStopinator`
   - **Runtime**: `Python 3.13`
   - **Execution role**: Usar uma role existente → `lambda-ec2-role`
3. Clique em **Create function**

### 📸 Print obrigatório:
- Tela de criação da função `myStopinator`
  
![image (98)](https://github.com/user-attachments/assets/d071ffc1-988d-4e06-b710-27399861b349)

![image](https://github.com/user-attachments/assets/08767788-8395-4d31-87f2-539a5108e69a)

---

## 🟡 Etapa 2 – Adicionar o gatilho (trigger)

1. Na função criada, clique em **Add trigger**
2. Selecione: **EventBridge (CloudWatch Events)**
3. Crie uma nova regra:
   - **Rule name**: `everyMinute`
   - **Rule description** (opcional): `Runs the Lambda function every minute to scan and stop EC2 instance`
   - **Schedule expression**: `rate(1 minute)`
4. Clique em **Add**

### 📸 Print obrigatório:
- Tela de configuração do trigger com a regra `everyMinute`

![image](https://github.com/user-attachments/assets/6d8fa520-7ab1-4acc-8649-987cf08eeba7) 

![image](https://github.com/user-attachments/assets/7033014c-0e96-4c82-b980-3c1473bdd96a)

---

## 🛠️ Etapa 3 – Configurar o código da função Lambda

1. Na aba **Code**, substitua o conteúdo do arquivo `lambda_function.py` pelo seguinte código:

```python
import boto3

region = 'us-east-1'
instance_id = 'i-xxxxxxxxxxxxxxxxx'
ec2 = boto3.client('ec2', region_name=region)

def lambda_handler(event, context):
    ec2.stop_instances(InstanceIds=[instance_id])
    print(f'Stopped instance: {instance_id}')
```
2. Altere:

`'us-east-1'` para sua região real (por exemplo: `'us-east-1'`)

`'i-xxxxxxxxxxxxxxxxx'` para o ID da sua instância EC2

Clique em Deploy

📸 Print obrigatório:

- Código da função com os valores da região e ID da instância preenchidos

![image](https://github.com/user-attachments/assets/6b33668d-af1d-4a09-abb5-94c175cf1de0)

---
## 🔸 Etapa 4 – Verificar a execução

1. Acesse o console EC2

2. Confirme se a instância foi interrompida automaticamente

3. Verifique os logs da função na aba Monitor > Logs

📸 Print obrigatório:

- Tela do EC2 com a instância parada

- Tela dos logs do Lambda com a execução registrada

![image](https://github.com/user-attachments/assets/e0a0e77f-2a86-4678-85d4-1dc4c5afb984)

![image](https://github.com/user-attachments/assets/73ff1612-a69c-44c1-ad8c-ecadfab71ab4)

---

## ✅ Conclusão
Neste desafio prático do grupo de estudos Run as Cloud, você aprendeu a automatizar tarefas na AWS com o serviço AWS Lambda, criando uma função que interrompe uma instância EC2 automaticamente a cada minuto.

Durante esta atividade, você:

- Criou uma role IAM com permissões adequadas para o Lambda interagir com o EC2.

- Lançou uma instância EC2 elegível ao Free Tier para ser gerenciada automaticamente.

- Configurou uma função Lambda usando Python e associou uma política para execução segura.

- Criou um gatilho com o Amazon EventBridge, acionando a função a cada minuto.

- Editou e implantou o código Lambda para parar a instância EC2 programaticamente.

- Verificou a execução e os logs no console AWS, validando o funcionamento da automação.
 
## 🧹 Limpeza dos recursos

1. Delete a função Lambda myStopinator

2. Delete instância EC2 usada

3. (Opcional) Delete a rule do EventBridge

4. Verifique o console de Billing para garantir que não há recursos ativos

---
## 📢 Compartilhe seu progresso!

Marque a comunidade **Run as Cloud** no LinkedIn:  
🔗 [Run as Cloud no LinkedIn](https://www.linkedin.com/company/run-as-cloud/?viewAsMember=true)

**Organizadores:**

- [Heberton Geovane](https://www.linkedin.com/in/heberton-geovane/)
- [Maik Biazi](https://www.linkedin.com/in/maik-biazi-47b9291a5/)
- [Michel Ernesto](https://www.linkedin.com/in/mernesto/)



