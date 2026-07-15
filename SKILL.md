---
name: formatacao-tabelas-quantitativas
description: >
  Use esta skill quando precisar formatar tabelas quantitativas e econométricas a partir de saídas brutas de software analítico.
  Ativa quando o usuário pede para formatar tabelas de regressão, estatísticas descritivas, ANOVA, matrizes de correlação ou qualquer resultado quantitativo de Stata, R, Python, SPSS ou SAS.
  Entrega tabelas acadêmicas prontas para inserção em manuscritos, com alinhamento perfeito, asteriscos de significância, erros padrão entre parênteses e notas de rodapé.
---
# Skill: Formatação de Tabelas Quantitativas e Econométricas

## Propósito
Converter saídas brutas de software analítico (Stata, R, Python, SPSS, SAS) em tabelas acadêmicas formatadas com rigor tipográfico e matemático, seguindo padrões internacionais de publicação. A skill identifica automaticamente tipos de resultados, extrai métricas-chave, aplica formatação consistente e renderiza no formato nativo do processador de texto.

## O que esta skill NÃO faz
- Não executa análises estatísticas — apenas formata saídas já geradas
- Não interpreta resultados — apenas os organiza e apresenta
- Não cria gráficos — apenas tabelas
- Não substitui software estatístico — complementa a formatação final

## Fontes
- Referência: `Formatação de Tabelas Quantitativas.pdf` (skill externa EscreveAI)
- Internet: Documentação oficial do `pandas` para formatação de DataFrames
- Internet: Guia de formatação de tabelas do `stargazer` (R) e `stargazer` (Python)
- Internet: Padrões APA 7ª edição para tabelas acadêmicas

## Pré-requisitos
- Python 3.10+
- pandas (pip install pandas)
- numpy (pip install numpy)
- re (biblioteca padrão)
- Formato de saída: Markdown, LaTeX ou HTML (configurável)

## Fluxo de Execução

### Passo 1: Ingestão de Dados Brutos
A skill aceita múltiplos formatos de entrada:

```python
import pandas as pd
import re
import numpy as np
from pathlib import Path

def ingerir_dados(caminho_arquivo: str) -> dict:
    """
    Inger saída bruta de qualquer software analítico.
    Retorna dicionário com dados estruturados.
    """
    extensao = Path(caminho_arquivo).suffix.lower()
    
    if extensao == '.log':
        return _parse_log_stata(caminho_arquivo)
    elif extensao == '.csv':
        return _parse_csv(caminho_arquivo)
    elif extensao == '.html':
        return _parse_html(caminho_arquivo)
    elif extensao in ['.xls', '.xlsx']:
        return _parse_excel(caminho_arquivo)
    elif extensao == '.txt':
        return _parse_texto_generico(caminho_arquivo)
    else:
        raise ValueError(f"Formato não suportado: {extensao}")

def _parse_log_stata(caminho: str) -> dict:
    """Parse de logs do Stata com regex avançado."""
    with open(caminho, 'r', encoding='utf-8') as f:
        conteudo = f.read()
    
    # Padrões regex para diferentes tipos de saída
    padroes = {
        'regressao': r'(\w+)\s+[\|]\s+Coef\.\s+Std\.\s+Err\.\s+t\s+P>\|t\|\s+\[(\d+\.?\d*)%\s+IC\]',
        'anova': r'Source\s+SS\s+df\s+MS\s+F\s+Prob>F',
        'correlacao': r'Correlation matrix',
        'descritiva': r'Variable\s+Obs\s+Mean\s+Std\.\s+Dev\.\s+Min\s+Max'
    }
    
    dados = {'tipo': 'desconhecido', 'conteudo_raw': conteudo}
    
    for tipo, padrao in padroes.items():
        if re.search(padrao, conteudo):
            dados['tipo'] = tipo
            break
    
    # Extração de coeficientes para regressões
    if dados['tipo'] == 'regressao':
        dados['resultados'] = _extrair_coeficientes_stata(conteudo)
    
    return dados

def _extrair_coeficientes_stata(conteudo: str) -> list:
    """Extrai coeficientes, erros padrão e valores-p do Stata."""
    resultados = []
    linhas = conteudo.split('\n')
    
    padrao_coef = r'^(\w+)\s+([\d\.\-]+)\s+([\d\.\-]+)\s+([\d\.\-]+)\s+([\d\.\-]+)'
    
    for linha in linhas:
        match = re.match(padrao_coef, linha.strip())
        if match:
            resultados.append({
                'variavel': match.group(1),
                'coeficiente': float(match.group(2)),
                'erro_padrao': float(match.group(3)),
                't': float(match.group(4)),
                'p': float(match.group(5))
            })
    
    return resultados
```

### Passo 2: Identificação Automática de Resultados
```python
def identificar_tipo_resultado(dados: dict) -> str:
    """Identifica automaticamente o tipo de resultado estatístico."""
    conteudo = dados.get('conteudo_raw', '')
    
    # Padrões de identificação
    padroes = {
        'regressao_linear': [
            r'Linear regression',
            r'Number of obs\s*=\s*\d+',
            r'F\(\d+,\s*\d+\)\s*='
        ],
        'regressao_logistica': [
            r'Logistic regression',
            r'Log likelihood\s*=',
            r'Prob > chi2\s*='
        ],
        'anova': [
            r'Analysis of Variance',
            r'Source\s+SS\s+df\s+MS',
            r'F\s+Prob\s*>\s*F'
        ],
        'correlacao': [
            r'Correlation matrix',
            r'Pairwise correlations'
        ],
        'descritiva': [
            r'Summary statistics',
            r'Variable\s+Obs\s+Mean\s+Std\.\s+Dev'
        ]
    }
    
    for tipo, lista_padroes in padroes.items():
        for padrao in lista_padroes:
            if re.search(padrao, conteudo, re.IGNORECASE):
                return tipo
    
    return 'desconhecido'
```

### Passo 3: Extração Estruturada de Métricas
```python
def extrair_metricas(dados: dict) -> dict:
    """Extrai métricas-chave do resultado identificado."""
    tipo = dados.get('tipo', 'desconhecido')
    
    if tipo == 'regressao':
        return {
            'coeficientes': dados.get('resultados', []),
            'estatisticas_modelo': _extrair_estatisticas_modelo(dados['conteudo_raw']),
            'observacoes': _extrair_observacoes(dados['conteudo_raw'])
        }
    elif tipo == 'anova':
        return _extrair_anova(dados['conteudo_raw'])
    elif tipo == 'correlacao':
        return _extrair_matriz_correlacao(dados['conteudo_raw'])
    elif tipo == 'descritiva':
        return _extrair_estatisticas_descritivas(dados['conteudo_raw'])
    else:
        return {'erro': 'Tipo de resultado não identificado'}

def _extrair_estatisticas_modelo(conteudo: str) -> dict:
    """Extrai R², R² ajustado, F, etc."""
    estatisticas = {}
    
    # R²
    match_r2 = re.search(r'R-squared\s*=\s*([\d\.\-]+)', conteudo)
    if match_r2:
        estatisticas['r2'] = float(match_r2.group(1))
    
    # R² ajustado
    match_r2_adj = re.search(r'Adj R-squared\s*=\s*([\d\.\-]+)', conteudo)
    if match_r2_adj:
        estatisticas['r2_ajustado'] = float(match_r2_adj.group(1))
    
    # Estatística F
    match_f = re.search(r'F\((\d+),\s*(\d+)\)\s*=\s*([\d\.\-]+)', conteudo)
    if match_f:
        estatisticas['f'] = {
            'gl_numerador': int(match_f.group(1)),
            'gl_denominador': int(match_f.group(2)),
            'valor': float(match_f.group(3))
        }
    
    # Prob > F
    match_pf = re.search(r'Prob\s*>\s*F\s*=\s*([\d\.\-]+)', conteudo)
    if match_pf:
        estatisticas['prob_f'] = float(match_pf.group(1))
    
    return estatisticas

def _extrair_observacoes(conteudo: str) -> int:
    """Extrai número de observações."""
    match = re.search(r'Number of obs\s*=\s*(\d+)', conteudo)
    return int(match.group(1)) if match else None
```

### Passo 4: Transformação e Formatação
```python
def formatar_tabela(dados_extraidos: dict, formato: str = 'markdown') -> str:
    """Aplica formatação acadêmica rigorosa aos dados extraídos."""
    
    if formato == 'markdown':
        return _formatar_markdown(dados_extraidos)
    elif formato == 'latex':
        return _formatar_latex(dados_extraidos)
    elif formato == 'html':
        return _formatar_html(dados_extraidos)
    else:
        raise ValueError(f"Formato de saída não suportado: {formato}")

def _formatar_markdown(dados: dict) -> str:
    """Gera tabela Markdown com formatação acadêmica."""
    if 'coeficientes' not in dados:
        return "Dados não formatáveis como tabela de regressão"
    
    linhas = []
    
    # Cabeçalho
    linhas.append("| Variável | Coeficiente | Erro Padrão | t | p-valor |")
    linhas.append("|:---|:---:|:---:|:---:|:---:|")
    
    # Coeficientes
    for coef in dados['coeficientes']:
        # Formatação de significância
        p = coef['p']
        if p < 0.001:
            sig = "***"
        elif p < 0.01:
            sig = "**"
        elif p < 0.05:
            sig = "*"
        elif p < 0.1:
            sig = "^"
        else:
            sig = ""
        
        # Formatação numérica
        coef_str = f"{coef['coeficiente']:.4f}{sig}"
        ep_str = f"({coef['erro_padrao']:.4f})"
        t_str = f"{coef['t']:.3f}"
        p_str = f"{coef['p']:.4f}"
        
        linhas.append(f"| {coef['variavel']} | {coef_str} | {ep_str} | {t_str} | {p_str} |")
    
    # Estatísticas do modelo
    if 'estatisticas_modelo' in dados:
        est = dados['estatisticas_modelo']
        linhas.append("")
        linhas.append("**Notas:**")
        if 'r2' in est:
            linhas.append(f"- R² = {est['r2']:.4f}")
        if 'r2_ajustado' in est:
            linhas.append(f"- R² ajustado = {est['r2_ajustado']:.4f}")
        if 'f' in est:
            linhas.append(f"- F({est['f']['gl_numerador']}, {est['f']['gl_denominador']}) = {est['f']['valor']:.2f}")
        if 'prob_f' in est:
            linhas.append(f"- p-valor = {est['prob_f']:.4f}")
        if 'observacoes' in dados and dados['observacoes']:
            linhas.append(f"- Observações: {dados['observacoes']:,}")
    
    # Notas de significância
    linhas.append("")
    linhas.append("* p < 0.1, ** p < 0.05, *** p < 0.01, **** p < 0.001")
    
    return "\n".join(linhas)
```

### Passo 5: Regras de Bordas Acadêmicas
```python
def aplicar_regras_bordas(tabela: str, formato: str = 'markdown') -> str:
    """Aplica regras estritas de bordas acadêmicas."""
    
    if formato == 'markdown':
        # Remove linhas verticais extras, mantém apenas separadores de cabeçalho
        linhas = tabela.split('\n')
        linhas_formatadas = []
        
        for i, linha in enumerate(linhas):
            if i == 1:  # Linha separadora do cabeçalho
                # Mantém apenas uma linha de separação
                linhas_formatadas.append("|:---|:---:|:---:|:---:|:---:|")
            else:
                linhas_formatadas.append(linha)
        
        return "\n".join(linhas_formatadas)
    
    elif formato == 'latex':
        # Aplica regras LaTeX: \hline apenas no topo e rodapé
        return _aplicar_hlines_latex(tabela)
    
    return tabela
```

### Passo 6: Motor Espacial e Ajuste de Layout
```python
def calcular_dimensoes_colunas(dados: dict, largura_maxima: float = 16.0) -> dict:
    """
    Calcula dimensões ótimas das colunas para não ultrapassar margens.
    Largura máxima em cm (padrão A4: 16cm de largura útil).
    """
    if 'coeficientes' not in dados:
        return {}
    
    # Calcula largura necessária para cada coluna
    larguras = {
        'variavel': max(len(c['variavel']) for c in dados['coeficientes']),
        'coeficiente': 10,  # Largura fixa para coeficientes
        'erro_padrao': 10,
        't': 8,
        'p': 8
    }
    
    # Ajusta largura total
    largura_total = sum(larguras.values()) + 4 * 3  # Espaçamento entre colunas
    
    if largura_total > largura_maxima:
        # Reduz proporcionalmente
        fator = largura_maxima / largura_total
        for col in larguras:
            larguras[col] = int(larguras[col] * fator)
    
    return larguras

def quebra_texto_inteligente(texto: str, largura_maxima: int = 30) -> str:
    """Quebra texto longo em rótulos de variáveis."""
    if len(texto) <= largura_maxima:
        return texto
    
    # Tenta quebrar em espaços
    palavras = texto.split()
    linhas = []
    linha_atual = ""
    
    for palavra in palavras:
        if len(linha_atual) + len(palavra) + 1 <= largura_maxima:
            linha_atual += (" " if linha_atual else "") + palavra
        else:
            if linha_atual:
                linhas.append(linha_atual)
            linha_atual = palavra
    
    if linha_atual:
        linhas.append(linha_atual)
    
    return " ".join(linhas)
```

### Passo 7: Notas de Rodapé Padronizadas
```python
def gerar_notas_rodape(dados: dict, controles: list = None, fontes: list = None) -> str:
    """Gera notas de rodapé padronizadas para a tabela."""
    notas = []
    
    # Nota de significância
    notas.append("* p < 0.1, ** p < 0.05, *** p < 0.01, **** p < 0.001")
    
    # Notas do modelo
    if 'estatisticas_modelo' in dados:
        est = dados['estatisticas_modelo']
        notas.append(f"Modelo: Regressão linear de mínimos quadrados ordinários")
        if 'observacoes' in dados and dados['observacoes']:
            notas.append(f"N = {dados['observacoes']:,}")
    
    # Controles
    if controles:
        notas.append(f"Controles incluídos: {', '.join(controles)}")
    
    # Fontes
    if fontes:
        notas.append(f"Fonte(s): {', '.join(fontes)}")
    
    return "\n\n".join(notas)
```

### Passo 8: Pipeline Completo
```python
def pipeline_formatacao_tabelas(
    caminho_entrada: str,
    formato_saida: str = 'markdown',
    controles: list = None,
    fontes: list = None,
    largura_maxima: float = 16.0
) -> str:
    """
    Pipeline completo de formatação de tabelas quantitativas.
    
    Args:
        caminho_entrada: Caminho para arquivo de saída bruta
        formato_saida: 'markdown', 'latex' ou 'html'
        controles: Lista de variáveis de controle para notas
        fontes: Lista de fontes para notas
        largura_maxima: Largura máxima da tabela em cm
    
    Returns:
        String com tabela formatada
    """
    # 1. Ingestão
    dados_brutos = ingerir_dados(caminho_entrada)
    
    # 2. Identificação
    tipo_resultado = identificar_tipo_resultado(dados_brutos)
    dados_brutos['tipo'] = tipo_resultado
    
    # 3. Extração
    dados_extraidos = extrair_metricas(dados_brutos)
    
    # 4. Formatação
    tabela_formatada = formatar_tabela(dados_extraidos, formato_saida)
    
    # 5. Regras de bordas
    tabela_com_bordas = aplicar_regras_bordas(tabela_formatada, formato_saida)
    
    # 6. Notas de rodapé
    notas = gerar_notas_rodape(dados_extraidos, controles, fontes)
    
    # 7. Composição final
    tabela_final = f"{tabela_com_bordas}\n\n{notas}"
    
    return tabela_final
```

## Tratamento de Erros
- **Arquivo não encontrado**: Verificar caminho e permissões de acesso
- **Formato não suportado**: Usar extensões: .log, .csv, .html, .xls, .xlsx, .txt
- **Tipo de resultado não identificado**: A skill tenta parsing genérico, mas pode falhar em formatos muito customizados
- **Dados inconsistentes**: Verificar se o arquivo de entrada está completo e não corrompido
- **Largura de colunas**: Se tabela ultrapassar margens, reduzir `largura_maxima` ou simplificar rótulos

## Regras
### O que SEMPRE fazer
- Preservar precisão decimal original dos dados
- Inserir asteriscos de significância automaticamente (p < 0.1, 0.05, 0.01, 0.001)
- Colocar erros padrão entre parênteses abaixo dos coeficientes
- Alinhar colunas numéricas por ponto decimal
- Incluir notas de rodapé com níveis de significância e fontes
- Remover linhas verticais em tabelas acadêmicas

### O que NUNCA fazer
- Alterar valores numéricos originais
- Adicionar interpretações ou conclusões
- Criar gráficos ou figuras
- Modificar estrutura dos dados brutos
- Omitir estatísticas de significância
- Usar bordas horizontais em todas as linhas (apenas topo e rodapé)

### Quando algo falhar
1. **Formato não reconhecido**: Tentar `_parse_texto_generico()` com regex mais flexível
2. **Dados faltantes**: Informar quais colunas não puderam ser extraídas
3. **Largura excedida**: Sugerir redução de casas decimais ou abreviação de rótulos
4. **Encoding problemático**: Tentar UTF-8, Latin-1, CP1252 automaticamente

## Exemplos de Uso

### Exemplo 1: Regressão do Stata
```python
# Entrada: arquivo de log do Stata
tabela = pipeline_formatacao_tabelas(
    caminho_entrada='regressao.log',
    formato_saida='markdown',
    controles=['idade', 'escolaridade', 'sexo'],
    fontes=['PNAD 2023', 'IBGE']
)
print(tabela)
```

### Exemplo 2: Tabela LaTeX para artigo
```python
tabela_latex = pipeline_formatacao_tabelas(
    caminho_entrada='resultados_r.csv',
    formato_saida='latex',
    largura_maxima=14.0
)
# Salvar em arquivo .tex
with open('tabela.tex', 'w') as f:
    f.write(tabela_latex)
```

### Exemplo 3: Múltiplas tabelas
```python
arquivos = ['reg1.log', 'reg2.log', 'reg3.log']
tabelas_comparativas = []
for arquivo in arquivos:
    tabela = pipeline_formatacao_tabelas(arquivo, formato_saida='markdown')
    tabelas_comparativas.append(tabela)

# Comparar modelos lado a lado
tabela_comparacao = combinar_tabelas(tabelas_comparativas)
```

## Extensões e Personalização
- **Novos formatos de saída**: Adicionar função `_formatar_[formato]()` 
- **Novos softwares**: Criar função `_parse_[software]()` com regex específicos
- **Personalização de significância**: Modificar limites no `formatar_tabela()`
- **Integração com LaTeX**: Usar `pandas.to_latex()` com configurações personalizadas
