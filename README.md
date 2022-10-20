# dio-dynamodb
Replicando [repositório](https://github.com/cassianobrexbit/dio-live-dynamodb) para testes.

Meu primeiro contato com este Serviço de banco de dados NoSQL.

### Serviço e distribuição utilizados
  - Linux [Ubuntu 20.04](https://learn.microsoft.com/pt-br/windows/wsl/install) no Windows 11

  - [Amazon CLI](https://aws.amazon.com/pt/cli/) para execução em linha de comando

  - [Amazon DynamoDB](https://aws.amazon.com/pt/pm/dynamodb/?trk=9f13c7e6-248e-4519-8a1f-a041f2e877c4&sc_channel=ps&s_kwcid=AL!4422!3!626321541353!e!!g!!amazon%20dynamodb&ef_id=Cj0KCQjw48OaBhDWARIsAMd966CKJ0SixizRoT01B8CtYnv8JYsrWfrzXqL349nICCDA-AW-IM-5rxsaAqHgEALw_wcB:G:s&s_kwcid=AL!4422!3!626321541353!e!!g!!amazon%20dynamodb)

    ![](https://uploaddeimagens.com.br/images/004/070/208/thumb/dynamodb_imagem.png)

### Comandos para execução do experimento:


- Criar uma tabela

```
aws dynamodb create-table \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema \
        AttributeName=Artist,KeyType=HASH \
        AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5
```

- Inserir um item

```
aws dynamodb put-item \
    --table-name Music \
    --item file://itemmusic.json \
```

- Inserir múltiplos itens

```
aws dynamodb batch-write-item \
    --request-items file://batchmusic.json
```

- Criar um index global secundário baseado no título do álbum

```
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions AttributeName=AlbumTitle,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"AlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Criar um index global secundário baseado no nome do artista e no título do álbum

```
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions\
        AttributeName=Artist,AttributeType=S \
        AttributeName=AlbumTitle,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"ArtistAlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"Artist\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Criar um index global secundário baseado no título da música e no ano

```
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions\
        AttributeName=SongTitle,AttributeType=S \
        AttributeName=SongYear,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"SongTitleYear-index\",\"KeySchema\":[{\"AttributeName\":\"SongTitle\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"SongYear\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Pesquisar item por artista

```
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :artist" \
    --expression-attribute-values  '{":artist":{"S":"Capital Inicial"}}'
```
- Pesquisar item por artista e título da música

```
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :artist and SongTitle = :title" \
    --expression-attribute-values file://keyconditions.json
```

- Pesquisa pelo index secundário baseado no título do álbum

```
aws dynamodb query \
    --table-name Music \
    --index-name AlbumTitle-index \
    --key-condition-expression "AlbumTitle = :name" \
    --expression-attribute-values  '{":name":{"S":"Acustico MTV"}}'
```

- Pesquisa pelo index secundário baseado no nome do artista e no título do álbum

```
aws dynamodb query \
    --table-name Music \
    --index-name ArtistAlbumTitle-index \
    --key-condition-expression "Artist = :v_artist and AlbumTitle = :v_title" \
    --expression-attribute-values  '{":v_artist":{"S":"Engenheiros do Hawaii"},":v_title":{"S":"O Papa e Pop"} }'
```

- Pesquisa pelo index secundário baseado no título da música e no ano

```
aws dynamodb query \
    --table-name Music \
    --index-name SongTitleYear-index \
    --key-condition-expression "SongTitle = :v_song and SongYear = :v_year" \
    --expression-attribute-values  '{":v_song":{"S":"Natasha"},":v_year":{"S":"2000"} }'
```

**Obs.:**

*Inicialmente utilizei  `Anaconda Prompt` e `PowerShell` no Windows 11, houve problemas com caracteres [aspas("" e '')](https://docs.aws.amazon.com/pt_br/cli/latest/userguide/cli-usage-parameters-quoting-strings.html) durante as `querys` e a barra invertida referindo-se a quebra de linha.*

*Por `default` a saída foi definida `'json'` mas pode ser alterada para `'text'` ou `'table'` com comando ` --output text'` ou ` --output table`* 

