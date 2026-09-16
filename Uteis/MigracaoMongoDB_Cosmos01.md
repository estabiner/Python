# Script usado para migração entre containers do Azure Cosmos DB for MongoDB account (RU)

## 1 - Script de configuração

Você só precisa ajustar os parametros nesse script, nos restantes é só executar.

- Mude o SOURCE_CONN_STR e DEST_CONN_STR, DATABASE_NAME, COLLECTION_ORIGEM e COLLECTION_DESTINO, e por fim, OUTPUT_DIR

Salve o script de configuração como: a00_config

```python
"""
Arquivo de configuração global - Versão MongoDB API.
"""
import os
from pymongo import MongoClient

# ===============================
# CONFIGURAÇÕES DE CONEXÃO
# ===============================
SOURCE_CONN_STR = "mongodb://mongotestecosmopadrao:GGOGbvvGHKHTLZsaeGRnAmFiJfPR4n22RU57Y2OrSE5YFAzPbaoR9fl8Ar37MAC1vyygzuWA3zdOACDb8pg8cg==@mongotestecosmopadrao.mongo.cosmos.azure.com:10255/?ssl=true&replicaSet=globaldb&retrywrites=false&maxIdleTimeMS=120000&appName=@mongotestecosmopadrao@"

DEST_CONN_STR = "mongodb://mongotestecosmopadrao:GGOGbvvGHKHTLZsaeGRnAmFiJfPR4n22RU57Y2OrSE5YFAzPbaoR9fl8Ar37MAC1vyygzuWA3zdOACDb8pg8cg==@mongotestecosmopadrao.mongo.cosmos.azure.com:10255/?ssl=true&replicaSet=globaldb&retrywrites=false&maxIdleTimeMS=120000&appName=@mongotestecosmopadrao@"

# Nome do database 
DATABASE_NAME = "NewDatabase01Teste"


# ===============================
# ESPAÇO PARA DEFINIR COLLECTIONS
# ===============================
COLLECTION_ORIGEM = "TesteColection"  # Nome da collection que você está exportando agora
COLLECTION_DESTINO = "TesteColection_Old"  # Nome da collection onde os dados serão inseridos


# Diretório de saída 
OUTPUT_DIR = r"C:\Users\estevam.nascimento\OneDrive - TELTEC SOLUTIONS LTDA\Documentos\Teste_migracao_mongo_cosmos\Exportado"

# Configurações de performance separadas
BATCH_EXPORT = 1000  # Leitura rápida da origem
BATCH_IMPORT = 100   # Escrita lenta e segura para o Serverless


# ===============================
# CLIENTES MONGO
# ===============================
source_client = MongoClient(SOURCE_CONN_STR)
dest_client = MongoClient(DEST_CONN_STR)
```


----------------


## 2 - Exportação

Salve o script de exportação como: mongo_export_02 (Não precisa mudar nada nesse script, ele está puxando as configurações do script 

```python
import os
import time
import random
from bson import json_util
from pymongo.errors import PyMongoError
from a00_config import source_client, DATABASE_NAME, OUTPUT_DIR, COLLECTION_ORIGEM

def export_data_with_retry(max_retries=5):
    if not os.path.exists(OUTPUT_DIR):
        os.makedirs(OUTPUT_DIR)

    db = source_client[DATABASE_NAME]
    col_name = COLLECTION_ORIGEM
    collection = db[col_name]
    data_file = os.path.join(OUTPUT_DIR, f"{col_name}.json")

    print(f"[EXPORTANDO] Iniciando extração de: {col_name}")

    for attempt in range(max_retries):
        try:
            total_docs = collection.count_documents({})
            print(f"Total de documentos encontrados: {total_docs}")

            with open(data_file, "w", encoding="utf-8") as f:
                # O cursor do find() também pode sofrer timeout; o driver trata parte disso,
                # mas o loop garante a abertura segura.
                for doc in collection.find({}):
                    f.write(json_util.dumps(doc) + "\n")
            
            print(f"Sucesso! Exportação concluída: {data_file}")
            return # Sai da função se tudo der certo

        except PyMongoError as e:
            wait = (2 ** attempt) + random.random()
            print(f"Erro na exportação: {e}. Retentando em {wait:.2f}s... (Tentativa {attempt+1}/{max_retries})")
            time.sleep(wait)
            if attempt == max_retries - 1:
                print("Falha crítica: Máximo de retentativas atingido na exportação.")

if __name__ == "__main__":
    export_data_with_retry()
```

Para executar o script de exportação, vá ao diretório onde estão os scripts no windows explorer, na barra de pesquisa digite cmd, em seguida com a tela do cmd aberta no caminho do diretorio onde estão os scripts, execute:

python "mongo_export_02"

---------------


## 3 - Script importação em uma nova collection



```python
import os
import time
import random
from bson import json_util
from pymongo.errors import PyMongoError
# Importamos agora o BATCH_IMPORT
from a00_config import dest_client, DATABASE_NAME, OUTPUT_DIR, COLLECTION_ORIGEM, COLLECTION_DESTINO, BATCH_IMPORT

def import_data_with_retry(max_retries=10):
    db = dest_client[DATABASE_NAME]
    input_file = os.path.join(OUTPUT_DIR, f"{COLLECTION_ORIGEM}.json")
    collection = db[COLLECTION_DESTINO]

    if not os.path.exists(input_file):
        print(f"Erro: Arquivo {input_file} não encontrado!")
        return

    # Usamos o BATCH_IMPORT aqui
    print(f"[IMPORTANDO] Gravando em: {COLLECTION_DESTINO} (Lotes de {BATCH_IMPORT})")

    batch = []
    total = 0

    with open(input_file, "r", encoding="utf-8") as f:
        for line in f:
            batch.append(json_util.loads(line))
            
            if len(batch) >= BATCH_IMPORT:
                process_batch_with_backoff(collection, batch, max_retries)
                total += len(batch)
                print(f"Progresso: {total} documentos importados...")
                
                # PAUSA DE SEGURANÇA PARA SERVERLESS (Dá fôlego ao banco)
                time.sleep(0.2) 
                
                batch = []

        if batch:
            process_batch_with_backoff(collection, batch, max_retries)
            total += len(batch)

    print(f"\nMigração concluída! Total de {total} documentos inseridos.")

def process_batch_with_backoff(collection, batch, max_retries):
    for i in range(max_retries):
        try:
            collection.insert_many(batch, ordered=False)
            return
        except PyMongoError as e:
            if i == max_retries - 1:
                print(f"Erro fatal após {max_retries} tentativas: {e}")
                raise
            # Se der erro 429, o tempo de espera aumenta
            wait = (2 ** i) + random.random()
            print(f"Serverless ocupado. Aguardando {wait:.2f}s...")
            time.sleep(wait)

if __name__ == "__main__":
    import_data_with_retry()
```


Para executar o script de exportação, vá ao diretório onde estão os scripts no windows explorer, na barra de pesquisa digite cmd, em seguida com a tela do cmd aberta no caminho do diretorio onde estão os scripts, execute:

python "mongo_import_03"



Comando para renomear arquivo caso tenha ficado como txt

Digite no cmd "dir" para verificar os arquivos do diretorio

Se realmente o script estiver como txt, faça o seguinte para renomear o formato corretamente:

```ren mongo_import_03.py.txt mongo_import_03.py.py```




---------------------



# Verificar informações de containers no Mongo

Migração de dados Cosmos DB na Acert
 
Premissas:
 
O procedimento é feito na VM Migracao Cosmos (172.173.212.127) que possuí:
 
- o Python instalado;
 
- a biblioteca do cosmos DB;
 
- acesso liberado aos Cosmos DB na Azure.
 
 
 
Etapas
 
- Alinhar com o time de aplicações a pausa da escrita no container (Possivelmente irão parar a aplicação durante o procedimento);
 
- Validar espaço em disco na VM;
 
- Se possível, realizar contagem dos itens antes da exportação;
 
- Realizar exportação dos itens do container;
 
- Salvar arquivo preferencialmente no caminho: C:\tmp\python\arquivos\cosmosdb\
 
- Criar 2 novos containers iguais ao original, um para histórico e outro vazio para os novos dados;
 
- realizar importação dos itens no container de histórico;
 
- Se possível, realizar contagem dos itens depois da importação;
 
- Excluir container antigo.
 
 
--------------
 
Como realizar a contagem de itens no portal Azure:
 
 
Azure Cosmos DB for MongoDB account:
 
- Clicar em Open Mongo Shell;

```sql 
- use echoid-homolog (Use o nome do banco desejado);
 
- db.card_request.countDocuments({});
```

ou 
```sql 
db.runCommand({ collStats: "card_request" })
```
 
Esse segundo traz mais detalhes, como tamanho do banco, índice e etc:
 
No caso, o container abaixo tem apenas 20 itens:
 
 
Essa colection/container aparentemente não tem partition key (chamada de Shard Key no MongoDB), apareceria exatamente abaixo da seção de "Time-to-Live" então é uma Unsharded Collection, também conhecida como Fixed Collection.
 
 
 
----
 
 
Azure Cosmos DB account
 
- Clicar com botão direito no container e em "New Query";
 
- SELECT VALUE COUNT(1) FROM c;

