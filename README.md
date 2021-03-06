Issue Topic 설정: KeyVault
=====
KeyVault 개념과 사용 예시
----
+ Key Vault

Azure Key Vault
-----
+ 비밀 관리 - 토큰, 암호, 인증서, API 키 및 기타 비밀에 대한 액세스를 안전하게 저장하고 엄격히 제어. ('비밀': 접근을 강하게 제어하고자 하는 모든 것)

+ 키 관리 - 키 관리 솔루션으로 사용.암호화 키를 쉽게 만들고 제어할 수 있음.

+ 인증서 관리 -  내부 연결 리소스에 사용할 퍼블릭 및 프라이빗 SSL/TLS(전송 계층 보안/Secure Sockets Layer) 인증서를 쉽게 등록, 관리, 배포


Key Valult 관련 용어 
-----

Storage Backend

+ Vault의 암호화된 Secret을 보관하는 곳이다. 

Barrier

+ Barrier는 Vault의 일부 구성요소를 감싸고 있다. 때문에 Vault와 Storage Backend 간의 통신은 Barrier를 거쳐야 하며, Barrier가 프록시 역할(대문 역할)을 한다. Barrier의 구성요소들은 서로 Trust 한 관계를 유지/성립한다.

+ Vault는 암호화된 데이터만 밖으로 나오게 한다.

+ Vault는 Barrier의 상태가 “unsealed”가 되어야 접근할 수 있다.

Secret Engine

+ Secret의 관리를 책임진다.

+ Secret 관련 작업은 Secret Engine으로 전달하고 Engine의 구현체마다 상이한 방식으로 저장한다. 

+ Secret Engine의 인터페이스를 활용하여 DB, File System 또는 유저가 정의한 방식으로 저장한다.

+ Audit Device

+ 모든 Vault의 Request/Respones는 Audit Device에 의해 감사 로깅이 된다. 

Auth Method 

+ Vault에 접근하는 클라이언트를 인증한다.

+ 인증된 클라이언트의 토큰을 반환한다.

Client Token

+ HTTP에서의 세션 ID와 같은 토큰을 반환한다.

+ Vault의 REST API를 사용하는 경우 HTTP 헤더에 토큰을 적재한다.

Secret

+ Vault에서 관리하는 비밀 객체이다.

+ Secret은 일정 주기를 가지며, 해당 주기가 만료되면 폐기해야 한다.


사용 예시 
-----

+ Key Valult 만들기
자격 증명 모음을 만들려면 Azure Cloud Shell에서 다음 명령을 실행합니다. 
--name 매개 변수에 고유한 자격 증명 모음 이름을 입력해야 합니다.

```
az keyvault create \
    --resource-group [sandbox resource group name] \
    --location centralus \
    --name <your-unique-vault-name>
```
완료되면 새 자격 증명 모음을 설명하는 JSON 출력이 표시됩니다.

+ 비밀 추가

```
az keyvault secret set \
    --name SecretPassword \
    --value reindeer_flotilla \
    --vault-name <your-unique-vault-name>
```

여기서 사용할 비밀의 이름은 SecretPassword 이고, 값은 reindeer_flotilla 입니다. 
--vault-name 매개 변수에 고유한 자격 증명 모음 이름을 입력해야 합니다.

+ Key Valult 사용하기

```
export KEY_VAULT_NAME=<your-key-vault-name>
```
Environment 변수 설정한다.


Keyvault 사용 샘플 파이썬코드
```
import os
import cmd
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

keyVaultName = os.environ["KEY_VAULT_NAME"]
KVUri = f"https://{keyVaultName}.vault.azure.net"

credential = DefaultAzureCredential()
client = SecretClient(vault_url=KVUri, credential=credential)

secretName = input("Input a name for your secret > ")
secretValue = input("Input a value for your secret > ")

print(f"Creating a secret in {keyVaultName} called '{secretName}' with the value '{secretValue}' ...")

client.set_secret(secretName, secretValue)

print(" done.")

print(f"Retrieving your secret from {keyVaultName}.")

retrieved_secret = client.get_secret(secretName)

print(f"Your secret is '{retrieved_secret.value}'.")
print(f"Deleting your secret from {keyVaultName} ...")

poller = client.begin_delete_secret(secretName)
deleted_secret = poller.result()

print(" done.")

```
