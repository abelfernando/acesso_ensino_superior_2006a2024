# Análise Histórica de Matrículas no Ensino Superior Brasileiro (2006–2024)

Este projeto analisa a evolução das matrículas em cursos de graduação no Brasil entre 2006 e 2024, segmentando os resultados por:

- grau acadêmico: **Bacharelado**, **Licenciatura** e **Tecnológico**;
- dependência administrativa: **Pública** e **Privada**.

O notebook consolida os dados, calcula indicadores de crescimento e gera visualizações da série histórica.

## Notebook

O código está no arquivo [`Acesso_Ensino_Superior_2006_a_2024.ipynb`](./Acesso_Ensino_Superior_2006_a_2024.ipynb).

[![Abrir no Google Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/abelfernando/acesso_ensino_superior_2006a2024/blob/main/Acesso_Ensino_Superior_2006_a_2024.ipynb)

## Fontes dos dados

### 2006 a 2008 — microdados do INEP

Os microdados brutos do Censo da Educação Superior de 2006, 2007 e 2008 foram obtidos na página oficial do INEP:

<https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/microdados/censo-da-educacao-superior>

O notebook lê os arquivos de graduação presencial e a distância, identifica a rede e o grau acadêmico e soma as variáveis de matrícula do primeiro semestre.

### 2009 a 2024 — Base dos Dados / Google BigQuery

Os dados de 2009 a 2024 são consultados no Google BigQuery a partir da tabela harmonizada de cursos da Base dos Dados:

<https://basedosdados.org/dataset/a3b57cca-ff80-4bf2-8bac-c145109e06a7?table=03f7e043-9ea1-47f9-9e77-f55dfe449381>

O código consulta a tabela:

```text
basedosdados.br_inep_censo_educacao_superior.curso
```

## Arquivos de 2006 a 2008

Depois de baixar e extrair os microdados do INEP, renomeie os seis arquivos exatamente como abaixo, usando letras maiúsculas e a extensão `.CSV`:

```text
GRADUACAO_PRESENCIAL_2006.CSV
GRADUACAO_DISTANCIA_2006.CSV
GRADUACAO_PRESENCIAL_2007.CSV
GRADUACAO_DISTANCIA_2007.CSV
GRADUACAO_PRESENCIAL_2008.CSV
GRADUACAO_DISTANCIA_2008.CSV
```

No Google Colab, envie os arquivos para `/content/`, pois esse é o caminho configurado no notebook:

```text
/content/GRADUACAO_PRESENCIAL_2006.CSV
/content/GRADUACAO_DISTANCIA_2006.CSV
/content/GRADUACAO_PRESENCIAL_2007.CSV
/content/GRADUACAO_DISTANCIA_2007.CSV
/content/GRADUACAO_PRESENCIAL_2008.CSV
/content/GRADUACAO_DISTANCIA_2008.CSV
```

Em ambiente local, coloque os arquivos na pasta usada pelo notebook ou altere o prefixo `/content/` na lista `arquivos_config` para o caminho local, por exemplo:

```python
BASE_DADOS = "./dados"

arquivos_config = [
    (f"{BASE_DADOS}/GRADUACAO_PRESENCIAL_2006.CSV", 2006, colunas_matr_presencial),
    (f"{BASE_DADOS}/GRADUACAO_DISTANCIA_2006.CSV", 2006, colunas_matr_ead_2006),
    (f"{BASE_DADOS}/GRADUACAO_PRESENCIAL_2007.CSV", 2007, colunas_matr_presencial),
    (f"{BASE_DADOS}/GRADUACAO_DISTANCIA_2007.CSV", 2007, colunas_matr_ead_2007_2008),
    (f"{BASE_DADOS}/GRADUACAO_PRESENCIAL_2008.CSV", 2008, colunas_matr_presencial),
    (f"{BASE_DADOS}/GRADUACAO_DISTANCIA_2008.CSV", 2008, colunas_matr_ead_2007_2008),
]
```

Os arquivos podem ser separados por ano em subpastas, desde que os caminhos em `arquivos_config` sejam atualizados. O leitor detecta automaticamente os delimitadores `|`, `;` ou `,` e usa a codificação `latin1`.

## Chave e autenticação do Google Cloud para o BigQuery

É necessário um projeto Google Cloud para faturar as consultas do BigQuery. A tabela consultada é pública, mas a execução da consulta exige um projeto de cobrança.

### Criar um projeto e uma conta de serviço

1. Acesse o [Google Cloud Console](https://console.cloud.google.com/).
2. Crie um projeto ou selecione um projeto existente.
3. Ative a **BigQuery API** em **APIs e serviços → Biblioteca**.
4. Acesse **IAM e administrador → Contas de serviço**.
5. Crie uma conta de serviço.
6. Conceda à conta, no projeto, o papel **BigQuery Job User**.
7. Para consultar dados públicos, conceda também **BigQuery Data Viewer** quando necessário.
8. Na conta de serviço, abra **Chaves → Adicionar chave → Criar nova chave → JSON**.
9. Baixe o arquivo JSON e mantenha-o fora do repositório. Não publique esse arquivo nem o adicione ao Git.

### Google Colab

No Colab, a forma recomendada é autenticar com a conta Google:

```python
from google.colab import auth
auth.authenticate_user()
```

Depois, crie um segredo chamado `GCP_PROJECT_ID` em **Secrets** no painel lateral do Colab. O valor deve ser o ID do projeto Google Cloud usado para faturar a consulta, por exemplo:

```text
meu-projeto-123456
```

O notebook recupera esse valor com:

```python
from google.colab import userdata
PROJECT_ID = userdata.get("GCP_PROJECT_ID")
```

Se a autenticação de usuário não estiver disponível, é possível usar a chave JSON como segredo/arquivo temporário, configurando a variável `GOOGLE_APPLICATION_CREDENTIALS` para o caminho do arquivo. Nunca deixe a chave gravada no notebook ou no repositório.

## Executar no Google Colab

1. Abra o notebook pelo botão **Abrir no Google Colab** acima.
2. Execute a autenticação do Google Cloud.
3. Crie o segredo `GCP_PROJECT_ID`.
4. Baixe os seis arquivos do INEP, renomeie-os conforme a seção anterior e faça upload para `/content/`.
5. Execute as células em ordem.

As células instalam as dependências, consultam o BigQuery, processam os microdados anteriores a 2009, consolidam a série e exibem as tabelas e gráficos.

Se algum arquivo de 2006 a 2008 não for encontrado, o notebook informa o arquivo ausente e continua usando os dados disponíveis. Para uma série completa de 2006 a 2024, os seis arquivos devem estar presentes.

## Executar em ambiente local

Requisitos:

- Python 3.9 ou superior;
- uma conta Google Cloud com acesso ao BigQuery;
- os seis arquivos CSV do INEP, quando a série de 2006 a 2008 for necessária.

Crie um ambiente virtual e instale as dependências:

```bash
python -m venv .venv
```

Linux/macOS:

```bash
source .venv/bin/activate
```

Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Instale os pacotes:

```bash
python -m pip install --upgrade pip
pip install basedosdados pandas numpy matplotlib seaborn jupyter
```

Configure a autenticação local usando o Google Cloud CLI:

```bash
gcloud auth application-default login
```

Alternativamente, use a conta de serviço:

```bash
export GOOGLE_APPLICATION_CREDENTIALS="/caminho/seguro/da-chave.json"
```

No Windows PowerShell:

```powershell
$env:GOOGLE_APPLICATION_CREDENTIALS = "C:\caminho\seguro\da-chave.json"
```

Defina o ID do projeto de cobrança antes de executar a célula da consulta:

```python
PROJECT_ID = "meu-projeto-123456"
```

Por fim, abra e execute o notebook:

```bash
jupyter notebook Acesso_Ensino_Superior_2006_a_2024.ipynb
```

Ou:

```bash
jupyter lab Acesso_Ensino_Superior_2006_a_2024.ipynb
```

## Indicadores produzidos

O notebook gera:

- totais de matrículas por ano, rede e grau;
- subtotais por rede e total do Brasil;
- variação absoluta e crescimento acumulado;
- CAGR (taxa composta de crescimento anual);
- participação pública e privada por grau;
- crescimento anual (YoY);
- gráficos da evolução e do crescimento acumulado.

## Licença

Consulte o arquivo [`LICENSE`](./LICENSE) para conhecer os termos de uso do código.
