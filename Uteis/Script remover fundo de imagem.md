```python
from pathlib import Path
from rembg import remove
from PIL import Image
import io

# Pastas
PASTA_ORIGEM = r"C:\Users\estevam.nascimento\OneDrive - TELTEC SOLUTIONS LTDA\Documentos\Arquivos fundo\Origem"
PASTA_PROCESSADOS = r"C:\Users\estevam.nascimento\OneDrive - TELTEC SOLUTIONS LTDA\Documentos\Arquivos fundo\Processados"

origem = Path(PASTA_ORIGEM)
processados = Path(PASTA_PROCESSADOS)

# Cria pasta de destino caso não exista
processados.mkdir(parents=True, exist_ok=True)

# Extensões suportadas
extensoes = [".jpg", ".jpeg", ".png", ".webp", ".bmp", ".tiff"]

arquivos = [
    arquivo
    for arquivo in origem.iterdir()
    if arquivo.suffix.lower() in extensoes
]

if not arquivos:
    print("Nenhuma imagem encontrada.")
    exit()

for arquivo in arquivos:
    try:
        print(f"Processando: {arquivo.name}")

        # Lê a imagem
        with open(arquivo, "rb") as f:
            input_bytes = f.read()

        # Remove fundo
        output_bytes = remove(input_bytes)

        # Nome de saída sempre em PNG
        nome_saida = f"{arquivo.stem}.png"
        arquivo_saida = processados / nome_saida

        # Salva PNG transparente
        imagem = Image.open(io.BytesIO(output_bytes))
        imagem.save(arquivo_saida, "PNG")

        print(f"OK -> {arquivo_saida}")

    except Exception as e:
        print(f"ERRO em {arquivo.name}: {e}")

print("\nProcessamento concluído.")
```
