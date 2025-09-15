## 📌 Resumo do Script

Este script em Python gera uma **base de vendas simulada** para o ano de 2025.  
As principais etapas são:

1. 📅 **Definir período**  
   - Criação de todas as datas de 01/01/2025 até 31/12/2025.  

2. 🎯 **Criar metas mensais**  
   - Cada mês recebe um valor de meta específico.  

3. 💰 **Gerar dados de receita**  
   - Receita aleatória entre 500 e 3000 por dia.  
   - Combinação de Data, MetaMensal e Receita.  

4. 🗂️ **Montar DataFrame**  
   - Dados organizados em uma tabela `pandas.DataFrame`.  

5. 📂 **Exportar CSV**  
   - Arquivo salvo como `base_vendas.csv` no padrão brasileiro:  
     - Separador `;`  
     - Codificação `utf-8-sig`  

6. ✅ **Mensagem final**  
   - Exibe no console: `"Base criada com sucesso!"`.  


```py
import pandas as pd
import numpy as np

# Definir período
datas = pd.date_range(start="2025-01-01", end="2025-12-31", freq="D")

# Criar meta mensal (pode variar mês a mês)
metas = {
    1: 30000, 2: 28000, 3: 35000, 4: 32000, 5: 31000, 6: 36000,
    7: 33000, 8: 34000, 9: 32000, 10: 37000, 11: 38000, 12: 40000
}

# Gerar dados
dados = []
for data in datas:
    meta = metas[data.month]
    receita = np.random.randint(500, 3000)  # receita simulada
    dados.append([data.strftime("%d/%m/%Y"), meta, receita])

# Criar DataFrame
df = pd.DataFrame(dados, columns=["Data", "MetaMensal", "Receita"])

# Salvar em CSV (padrão brasileiro ; como separador)
df.to_csv("base_vendas.csv", sep=";", index=False, encoding="utf-8-sig")

print("Base criada com sucesso!")
```

### Resultado esperado:

| Data       | MetaMensal | Receita |
|------------|------------|---------|
| 01/01/2025 | 30000      | 1342    |
| 02/01/2025 | 30000      | 2058    |
| 03/01/2025 | 30000      | 2047    |
| 04/01/2025 | 30000      | 896     |
| 05/01/2025 | 30000      | 2062    |
| 06/01/2025 | 30000      | 955     |
| 07/01/2025 | 30000      | 2737    |
| 08/01/2025 | 30000      | 2300    |
| 09/01/2025 | 30000      | 1198    |
| ...        | ...        | ...     |
| 31/12/2025 | 40000      | 2500    | 
