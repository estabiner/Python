# Script Remover Fundo de Imagem

## Descrição

Script Python que automatiza a remoção de fundo de imagens usando a biblioteca `rembg`. Processa múltiplas imagens em uma pasta e salva o resultado com fundo transparente em formato PNG.

## Requisitos

- Python 3.7+
- Bibliotecas necessárias:

```
pip install rembg pillow
```

## Como Usar

1. Atualize as variáveis `PASTA_ORIGEM` e `PASTA_PROCESSADOS` com os caminhos das suas pastas
2. Execute o script: `python script.py`
3. As imagens processadas serão salvas em `PASTA_PROCESSADOS`

## Formatos Suportados

O script aceita as seguintes extensões: `.jpg`, `.jpeg`, `.png`, `.webp`, `.bmp`, `.tiff`

As imagens serão convertidas para **PNG transparente** na saída.

## Script Completo

```
from pathlib import Path
from rembg import remove
from PIL import Image
import io

# Configuração de Pastas
PASTA_ORIGEM = r"C:\Users\estevam.nascimento\OneDrive - TELTEC SOLUTIONS LTDA\Documentos\Arquivos fundo\Origem"
PASTA_PROCESSADOS = r"C:\Users\estevam.nascimento\OneDrive - TELTEC SOLUTIONS LTDA\Documentos\Arquivos fundo\Processados"

origem = Path(PASTA_ORIGEM)
processados = Path(PASTA_PROCESSADOS)

# Cria pasta de destino caso não exista
processados.mkdir(parents=True, exist_ok=True)

# Lista de extensões de imagem suportadas
extensoes = [".jpg", ".jpeg", ".png", ".webp", ".bmp", ".tiff"]

# Cria uma lista de arquivos que correspondem às extensões suportadas
arquivos = [
    arquivo
    for arquivo in origem.iterdir()
    if arquivo.suffix.lower() in extensoes
]

# Verifica se há imagens para processar
if not arquivos:
    print("Nenhuma imagem encontrada.")
    exit()

# Loop principal: processa cada imagem encontrada
for arquivo in arquivos:
    try:
        print(f"Processando: {arquivo.name}")

        # Lê o arquivo de imagem em modo binário
        with open(arquivo, "rb") as f:
            input_bytes = f.read()

        # Remove fundo da imagem usando rembg
        output_bytes = remove(input_bytes)

        # Define nome de saída sempre em PNG
        nome_saida = f"{arquivo.stem}.png"
        arquivo_saida = processados / nome_saida

        # Salva a imagem como PNG com transparência
        imagem = Image.open(io.BytesIO(output_bytes))
        imagem.save(arquivo_saida, "PNG")

        print(f"OK -> {arquivo_saida}")

    except Exception as e:
        # Captura e exibe qualquer erro durante o processamento
        print(f"ERRO em {arquivo.name}: {e}")

print("\nProcessamento concluído.")
```

## Explicação Detalhada do Código

### 1. Importações

| Biblioteca | Função |
|-----------|--------|
| `Path` (pathlib) | Manipulação de caminhos de arquivo de forma multiplataforma |
| `remove` (rembg) | Função principal para remover fundo de imagens |
| `Image` (PIL) | Manipulação e salvamento de imagens |
| `io` | Trabalhar com dados em memória |

### 2. Configuração de Pastas

```
PASTA_ORIGEM = r"..."
PASTA_PROCESSADOS = r"..."
```

Define os caminhos absolutos:
- **PASTA_ORIGEM**: Onde estão as imagens a processar
- **PASTA_PROCESSADOS**: Onde as imagens processadas serão salvas

O `r` antes da string indica "raw string" (não interpreta caracteres de escape).

```
origem = Path(PASTA_ORIGEM)
processados = Path(PASTA_PROCESSADOS)
```

Converte strings em objetos `Path` para facilitar operações com diretórios.

### 3. Criação da Pasta de Destino

```
processados.mkdir(parents=True, exist_ok=True)
```

- **parents=True**: Cria pastas pai se necessário
- **exist_ok=True**: Não gera erro se a pasta já existir

### 4. Definição de Extensões Suportadas

```
extensoes = [".jpg", ".jpeg", ".png", ".webp", ".bmp", ".tiff"]
```

Lista todas as extensões de imagem que o script irá processar.

### 5. Coleta de Arquivos

```
arquivos = [
    arquivo
    for arquivo in origem.iterdir()
    if arquivo.suffix.lower() in extensoes
]
```

- **origem.iterdir()**: Itera sobre todos os arquivos da pasta
- **arquivo.suffix.lower()**: Obtém a extensão do arquivo em minúsculas
- Filtra apenas arquivos com extensões suportadas

### 6. Validação de Imagens

```
if not arquivos:
    print("Nenhuma imagem encontrada.")
    exit()
```

Se não houver imagens, exibe mensagem e encerra o programa.

### 7. Loop de Processamento

```
for arquivo in arquivos:
    try:
        # ... código de processamento
    except Exception as e:
        print(f"ERRO em {arquivo.name}: {e}")
```

Para cada imagem encontrada:

#### 7.1 Leitura da Imagem

```
with open(arquivo, "rb") as f:
    input_bytes = f.read()
```

- Abre o arquivo em modo binário (`"rb"` = read binary)
- Lê todo o conteúdo em bytes
- Fecha automaticamente o arquivo ao sair do bloco `with`

#### 7.2 Remoção de Fundo

```
output_bytes = remove(input_bytes)
```

A função `remove()` do rembg:
- Recebe os bytes da imagem
- Processa usando inteligência artificial (U2-Net)
- Retorna os bytes da imagem com fundo transparente removido

#### 7.3 Definição do Nome de Saída

```
nome_saida = f"{arquivo.stem}.png"
arquivo_saida = processados / nome_saida
```

- **arquivo.stem**: Nome do arquivo sem extensão
- Sempre salva como `.png` para preservar transparência
- Cria o caminho completo combinando pasta + arquivo

#### 7.4 Salvamento da Imagem

```
imagem = Image.open(io.BytesIO(output_bytes))
imagem.save(arquivo_saida, "PNG")
```

- **io.BytesIO()**: Cria um arquivo em memória a partir dos bytes
- **Image.open()**: Abre a imagem
- **imagem.save()**: Salva em formato PNG (preserva transparência)

#### 7.5 Mensagem de Sucesso

```
print(f"OK -> {arquivo_saida}")
```

Exibe o caminho completo do arquivo salvo com sucesso.

### 8. Tratamento de Erros

```
except Exception as e:
    print(f"ERRO em {arquivo.name}: {e}")
```

Se algo der errado durante o processamento:
- Captura o erro
- Exibe a mensagem de erro
- **Continua** processando os demais arquivos (não interrompe)

### 9. Mensagem Final

```
print("\nProcessamento concluído.")
```

Exibida após todos os arquivos terem sido processados.

## Dicas de Uso

- ⏱️ O processamento pode ser lento dependendo do tamanho e quantidade de imagens
- 💾 Use PNG para manter a transparência (não use JPEG)
- 📁 Verifique se os caminhos das pastas existem antes de executar
- 🔍 Verificar os logs de erro se alguma imagem não for processada
- 🎯 Testar com uma imagem antes de processar um lote grande

## Troubleshooting

| Problema | Solução |
|----------|---------|
| "Nenhuma imagem encontrada" | Verifique se `PASTA_ORIGEM` existe e contém imagens |
| Erro ao instalar rembg | Instale dependências: `pip install rembg pillow torch` |
| Imagens com qualidade ruim | É normal, rembg tenta remover o fundo mesmo que não seja perfeito |
| Processamento muito lento | Redimensione as imagens antes ou processe em lotes menores |
