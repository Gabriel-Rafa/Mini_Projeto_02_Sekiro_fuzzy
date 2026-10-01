# Mini-Projeto 02 — Conselheiro Shinobi Fuzzy

**Autor:** Gabriel Rafá Martins Freire  
**Matrícula:** 20230145310  
**Equipe:** individual  
**Caminho B:** reescrita do Mini-Projeto 01 no domínio de Sekiro  
**Tecnologia:** Python 3.12 e scikit-fuzzy 0.5.0; inferência Mamdani

## Problema e motivação

O projeto estima **quanto priorizar o recuo em Sekiro: Shadows Die Twice**, a partir da porcentagem restante de vida e do preenchimento da barra de postura do jogador. Em vez de classificar cada condição apenas como verdadeira ou falsa, usa graus de pertinência e produz um índice contínuo.

O MP1 escolhia uma ação imediata, como Mikiri, saltar ou afastar. O MP2 mantém o domínio de aconselhamento de combate, mas reformula a decisão para uma intensidade de recuo. Ele **não reproduz todas as funcionalidades do MP1**: ataque, Mikiri, curas e abertura segura não entram neste controlador. Uma saída numérica não determina a reação correta para cada golpe.

O controlador real está em Python com `ctrl.Antecedent`, `ctrl.Consequent`, `ctrl.Rule`, `ctrl.ControlSystem` e `ctrl.ControlSystemSimulation`. Não depende de uma simulação JavaScript nem do site do MP1.

## Arquivos

| Arquivo | Conteúdo |
|---|---|
| [Mini_Projeto_02_Sekiro.ipynb](Mini_Projeto_02_Sekiro.ipynb) | Notebook independente, explicações, implementação, testes e saídas salvas |
| [sekiro_fuzzy.py](sekiro_fuzzy.py) | Mesma lógica, executável pelo terminal |
| [requirements.txt](requirements.txt) | Versões das cinco dependências de cálculo |
| [requirements-notebook.txt](requirements-notebook.txt) | Dependências adicionais para executar no Jupyter |
| [resultados_testes.json](resultados_testes.json) | Resultados reais dos três casos e resumo da grade |
| [figuras/](figuras/) | Pertinências, agregação do caso misto e superfície de controle |
| [ROTEIRO_APRESENTACAO.md](ROTEIRO_APRESENTACAO.md) | Roteiro de até 15 minutos e perguntas para a discussão |
| [VIDEO_DEMONSTRACAO.mp4](VIDEO_DEMONSTRACAO.mp4) | Vídeo explicativo com texto na tela, sem narração |

## Execução no Google Colab

1. Abra o [Google Colab](https://colab.research.google.com/).
2. Use **Arquivo → Fazer upload de notebook** e escolha `Mini_Projeto_02_Sekiro.ipynb`.
3. Use um ambiente Python padrão; CPU é suficiente.
4. Execute todas as células, de cima para baixo. A primeira instala as dependências.
5. Confira `3 CASOS PRINCIPAIS: APROVADOS` e o resumo de 441 consultas na seção 10.
6. Na seção 12, altere `minha_vida` e `minha_postura` para consultar outro cenário.

Não é necessário subir o `.py`, baixar dados, montar o Drive ou configurar chaves. O notebook contém o código completo. É necessária internet para instalar as dependências. Se o runtime já tiver importado versões diferentes, reinicie a sessão após instalar e execute novamente desde a primeira célula.

A validação entregue foi feita executando **as 12 células de código na ordem com IPython local**, em Python 3.12.14, incluindo a instalação. Os gráficos estão incorporados ao notebook. Não foi usada uma sessão autenticada do Colab para a validação.

## Execução local

Não há etapa de build. Extraia o pacote ou baixe os arquivos do repositório e abra um terminal na pasta que contém este README. Use **Python 3.12**.

```bash
python -m venv .venv
```

Ative o ambiente no Linux/macOS:

```bash
source .venv/bin/activate
```

Ou no Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Instale e execute:

```bash
python -m pip install -r requirements.txt
python sekiro_fuzzy.py --vida 45 --postura 62.5
python sekiro_fuzzy.py --testes
```

A primeira consulta retorna aproximadamente **65.0000/100**, com F2, F3, F5 e F6 em grau 0,5. O comando `--testes` termina sem erro e imprime as verificações aprovadas.

Para abrir o notebook localmente, instale os pacotes adicionais e selecione o kernel:

```bash
python -m pip install -r requirements-notebook.txt
python -m ipykernel install --user --name sekiro-fuzzy --display-name "Python (Sekiro Fuzzy)"
python -m notebook Mini_Projeto_02_Sekiro.ipynb
```

Escolha `Python (Sekiro Fuzzy)` no Jupyter e execute todas as células. As versões de cálculo utilizadas nos testes são numpy 2.3.5, scipy 1.17.0, matplotlib 3.10.8, networkx 3.7 e scikit-fuzzy 0.5.0. A interface gráfica Jupyter não foi validada em navegador nesta entrega.

## Entradas, saída e premissas

| Variável | Tipo | Universo | Como obter / interpretar |
|---|---|---|---|
| `vida` | Entrada | 0 a 100% | Parte restante da barra de vitalidade do jogador |
| `postura` | Entrada | 0 a 100% | Parte preenchida da barra de postura **do jogador**, não do inimigo |
| `recuo` | Saída | Índice de 0 a 100 | Prioridade de buscar afastamento seguro; maior = mais recuo |

As entradas são fornecidas manualmente. A porcentagem pode ser estimada como comprimento preenchido dividido pelo comprimento total da barra, vezes 100. Não há integração com o jogo. A política supõe uma janela de decisão em que seja possível planejar um reposicionamento; não recomenda abandonar uma defesa durante um golpe.

Vida zero pertence ao domínio matemático para os testes de fronteira; não corresponde a uma recomendação executável para um jogador morto. Postura em 100 representa o limite da margem defensiva, sem modelar o estado de atordoamento. O índice de saída não é probabilidade de sobrevivência, distância, tempo de recuo ou comando de controle.

Quanto menor a vida, menor a margem para erros. Quanto maior o preenchimento da postura, maior a cautela. Isso é uma política conservadora de aconselhamento: não representa uma estratégia ótima comprovada para todos os inimigos.

## Funções de pertinência e justificativas

Cada variável possui **três termos linguísticos**. `trapmf` é trapezoidal; `trimf` é triangular. Parâmetros repetidos nas extremidades criam ombros que cobrem 0 e 100.

| Variável | Termo | Função | Parâmetros |
|---|---|---|---|
| Vida | baixa | trapmf | [0, 0, 30, 60] |
| Vida | media | trimf | [30, 60, 85] |
| Vida | alta | trapmf | [60, 85, 100, 100] |
| Postura | estavel | trapmf | [0, 0, 25, 50] |
| Postura | pressionada | trimf | [25, 50, 75] |
| Postura | critica | trapmf | [50, 75, 100, 100] |
| Recuo | baixo | trimf | [0, 20, 40] |
| Recuo | moderado | trimf | [30, 50, 70] |
| Recuo | alto | trimf | [60, 80, 100] |

**Vida:** 30% conserva a referência de pouca vida do MP1, mas a pertinência baixa passa a cair gradualmente até 60%. O centro intermediário é 60%; a partir de 85%, a vida é considerada plenamente alta. A sobreposição evita uma mudança brusca logo acima de 30%.

**Postura:** até um quarto da barra, o modelo assume boa margem; metade é a condição intermediária; a partir de três quartos, a política trata a margem como crítica. As transições são graduais. Esses limites não são pontos oficiais de quebra do jogo.

**Recuo:** centros em 20, 50 e 80 representam baixo, intermediário e alto sem prescrever “nunca recuar” ou “sempre fugir”. Triângulos simétricos com mesma largura evitam favorecer uma consequência apenas por sua área. A saída calculada neste modelo permanece entre 20 e 80; o universo 0–100 não implica que o centroide deva chegar a 0 ou 100.

Os limites são escolhas didáticas justificadas qualitativamente, **não calibração empírica**. Devem ser revisados com observações de combate e avaliação de jogadores. Pertinência 0,5 não significa probabilidade 50%.

![Funções de pertinência](figuras/pertinencias.png)

## Nove regras em linguagem natural

| Regra | SE a vida for... | E a postura for... | ENTÃO o recuo será... |
|---|---|---|---|
| F1 | baixa | estável | moderado |
| F2 | baixa | pressionada | alto |
| F3 | baixa | crítica | alto |
| F4 | média | estável | baixo |
| F5 | média | pressionada | moderado |
| F6 | média | crítica | alto |
| F7 | alta | estável | baixo |
| F8 | alta | pressionada | baixo |
| F9 | alta | crítica | moderado |

A base cobre as nove combinações, acima das seis regras mínimas e atendendo ao patamar de nove regras indicado na rubrica. Com pouca vida, mesmo postura estável justifica cautela moderada; pouca vida com postura pressionada ou crítica justifica alto recuo. Vida alta tolera mais pressão, mas postura crítica ainda produz recuo moderado.

A matriz respeita a ordem dos termos: aumentar a postura não diminui o termo de recuo; aumentar a vida não o aumenta. A ordenação linguística não é, por si só, prova de monotonicidade numérica global do centroide.

## Mamdani: como a saída é calculada

1. Fuzzificação das duas entradas por interpolação das funções de pertinência.
2. Operador **E = mínimo** para obter o grau de ativação de cada regra.
3. Implicação de Mamdani: cortar cada consequente na altura da ativação.
4. Agregação pelo **máximo** das funções cortadas.
5. Defuzzificação pelo **centroide** da região agregada.

Todas as regras têm peso 1. A biblioteca efetua os cálculos; `avaliar()` lê os graus efetivos de `aggregate_firing` e mostra a saída de `sim.output['recuo']`. O gráfico reconstitui os cortes para visualização, sem substituir o resultado do motor.

O centroide é a integral de `z × pertinência_agregada(z)` dividida pela integral de `pertinência_agregada(z)`. Não é votação entre regras nem uma média simples dos graus de ativação.

O resumo textual usa os pontos médios entre os centros: abaixo de 35 = baixo; de 35 a 65 = moderado; acima de 65 = alto. O resumo ocorre **após** a inferência, sem alterar a saída contínua. Um arredondamento a oito casas apenas nessa classificação evita rótulos incoerentes por ruído de ponto flutuante.

## Três casos principais e resultados

| Caso | Vida | Postura | Saída defuzzificada | Interpretação |
|---|---:|---:|---:|---|
| C1 | 20 | 90 | 80,0000 | Recuo alto: pouca margem de vida e postura |
| C2 | 90 | 10 | 20,0000 | Recuo baixo: boas condições para manter engajamento |
| C3 | 45 | 62,5 | 65,0000 | Limite superior do recuo moderado; mistura de moderado e alto |

Os três casos têm entradas, saídas esperadas, interpretação e `assert` no código. C1 ativa apenas F3 em grau 1; C2 ativa apenas F7 em grau 1.

**C3, passo a passo:**

1. Vida 45: baixa 0,5; média 0,5; alta 0.
2. Postura 62,5: estável 0; pressionada 0,5; crítica 0,5.
3. F2, F3, F5 e F6 ativam em grau `min(0,5; 0,5) = 0,5`.
4. F2, F3 e F6 contribuem para alto; o máximo dessas contribuições continua 0,5. F5 contribui para moderado com 0,5.
5. Os conjuntos moderado e alto são cortados em 0,5. A união é simétrica em torno de 65; o centroide é 65.

![Agregação do caso C3](figuras/inferencia_caso3.png)

## Cobertura, testes complementares e limites da validação

**Justificativa analítica de ausência de lacunas:** cada entrada possui pelo menos um termo com grau positivo em qualquer ponto de [0,100]. Isso decorre dos ombros nos extremos e da sobreposição das funções lineares adjacentes. Para qualquer par de entradas há, portanto, ao menos uma combinação de termos positivos. Essa combinação está na base completa 3×3 e produz ativação positiva. Todo consequente tem área positiva após um corte positivo; a união também, permitindo o centroide.

**Verificações executadas:**

- Três cenários principais, incluindo quatro regras simultâneas no caso misto.
- Nove regras isoladas nos centros dos antecedentes.
- 10.001 amostras por entrada para verificar a partição e ausência de lacunas.
- Grade de 21×21: **441 consultas reais ao scikit-fuzzy**, todas com saída finita e válida.
- Quatro extremos: (0,0)→50; (0,100)→80; (100,0)→20; (100,100)→50.
- Dezesseis entradas inválidas rejeitadas: negativos, acima de 100, NaN, infinitos, booleanos, texto e nulo em ambas as entradas.
- Vida 29,9 versus 30,1, com postura 50: saída **80,0000 → 79,8245**, sem salto no antigo limiar de 30%.

A grade não testa todos os números reais; ela complementa o argumento de cobertura. Esses testes verificam implementação e coerência com a política escolhida, sem demonstrar eficácia em partidas reais. Não há avaliação quantitativa de vitória, dano ou sobrevivência.

![Superfície de controle](figuras/superficie.png)

## Comparação com o MP1: complexidade e expressividade

| Aspecto | MP1 — Experta | MP2 — scikit-fuzzy |
|---|---|---|
| Recorte | Ação imediata de combate | Intensidade de recuo |
| Entradas | Vida, ataque, Mikiri, curas e abertura segura | Vida e postura do jogador |
| Verdade | Booleana: vida ≤30 ou não | Gradual: vida baixa em grau 0,5 |
| Regras | 14, com três níveis de encadeamento | Nove, combinadas em um controlador |
| Resolução | `salience` + `NOT` escolhem uma recomendação | Mínimo + máximo + centroide combinam contribuições |
| Explicação | Trace e cadeia causal da ação vencedora | Pertinências, ativações, agregação e centroide |
| Expressividade | Condições categóricas e dependências lógicas | Intensidades e transições graduais |
| Complexidade | Fatos intermediários e conflitos entre ações | Desenho de conjuntos, calibração e superfície |
| Crescimento | Depende das interações e do encadeamento | Base completa pode exigir 3^k regras para k entradas com três termos |

Nove regras não provam menor complexidade que 14: os recortes e as variáveis são diferentes. Não foi feito benchmark de tempo entre os motores. Ambos representam conhecimento explícito; nenhum foi treinado com dados. Não há regra final F12 equivalente à R12 antiga: a saída resulta da área agregada.

## Desenvolvimento e limitações

O desenvolvimento seguiu: recorte contínuo da decisão → entradas observáveis → funções justificadas e sobrepostas → matriz completa de regras → inferência Mamdani → testes e gráficos → comparação com o MP1.

O modelo ignora tipo do ataque, distância, postura do inimigo, cura disponível, ferramentas, chefes, timing, atordoamento e ressurreição. Vida e postura precisam ser informadas corretamente. Recuar demais pode prejudicar a pressão ofensiva. Uma extensão relevante seria incluir distância ou postura do inimigo e recalibrar o sistema com partidas, mantendo a cobertura das novas entradas.

O site público anterior continua sendo a demonstração do MP1. A implementação exigida pelo MP2 é o notebook/script deste pacote; o site não foi atualizado nesta entrega.

## Onde estão as entradas, funções e regras no notebook?

Contando **somente células de código**:

| Célula | Conteúdo |
|---:|---|
| 1 | Instalação das dependências |
| 2 | Importações e versão |
| 3 | Variáveis linguísticas e funções de pertinência |
| 4 | Base com F1–F9 e criação do controlador |
| 5 | Validação, execução e explicação da inferência |
| 6–7 | Funções gráficas e gráfico de pertinências |
| 8 | Três casos de teste comentados |
| 9 | Gráfico da agregação do caso misto |
| 10–11 | Testes complementares e superfície |
| 12 | Valores de entrada do seu cenário personalizado |

Agora não há classes `Fact`: as entradas são atribuídas a `sim.input['vida']` e `sim.input['postura']`, na célula 5. Os valores de teste estão na célula 8; os editáveis, na 12.

## Correspondência com o roteiro e a rubrica

| Critério | Peso | Evidência |
|---|---:|---|
| Modelagem fuzzy | 20% | Duas entradas, uma saída, três termos cada, justificativas e premissas |
| Base de regras | 25% | Nove regras, todas as combinações, argumento de cobertura e extremos |
| Casos de teste | 15% | Três casos com entradas, saída defuzzificada e interpretação comentadas no código |
| README e reprodutibilidade | 10% | Instruções Colab/local, versões fixadas e notebook executado |
| Entrega e discussão | 30% | Motivação, arquitetura, desenvolvimento, comparação MP1×MP2, roteiro e vídeo de apoio |

Os materiais para apresentação estão preparados. A publicação no GitHub, o envio ao Classroom e a apresentação/discussão pelo autor ainda precisam ser realizados; não são substituídos pelos testes do código.

## Vídeo, publicação e entrega

O arquivo `VIDEO_DEMONSTRACAO.mp4` apresenta o problema, variáveis, nove regras, inferência, resultados, comparação e limitações. É um vídeo com texto e gráficos, **sem narração**, para acompanhamento e estudo. O PDF não define formato de narração; caso o professor exija fala do aluno, use `ROTEIRO_APRESENTACAO.md` para gravar sua própria explicação.

1. Crie um repositório GitHub, por exemplo `mini-projeto-02-sekiro-fuzzy`.
2. Extraia o ZIP e envie os arquivos à raiz, incluindo este README, o notebook, o `.py`, os arquivos de dependências, os resultados, a pasta `figuras` e o vídeo. Não envie somente o ZIP.
3. Confira o notebook, as imagens do README e o vídeo no repositório. Garanta acesso ao professor.
4. Envie o **link real do repositório** no Classroom até **01/10/2026, às 12h59**, conforme o roteiro fornecido.
5. Prepare a apresentação e discussão de até 15 minutos com o roteiro anexo.

O repositório e o envio ao Classroom não foram criados automaticamente. Não há URL de GitHub inventada no material.

## Referências

- `Mini_Projeto_02_-_Roteiro.pdf`, fornecido pela disciplina: requisitos, rubrica e prazo.
- Mini-Projeto 01 — Conselheiro Shinobi: notebook e README anteriores.
- [scikit-fuzzy — exemplo oficial da API de controle](https://scikit-fuzzy.github.io/scikit-fuzzy/auto_examples/plot_tipping_problem_newapi.html).
- [scikit-fuzzy — implementação de ControlSystemSimulation](https://github.com/scikit-fuzzy/scikit-fuzzy/blob/master/skfuzzy/control/controlsystem.py).
- [scikit-fuzzy — exemplo de superfície de controle](https://scikit-fuzzy.readthedocs.io/en/latest/auto_examples/plot_control_system_advanced.html).
- [Activision — mecânicas de combate e postura](https://support.activision.com/sekiro/articles/sekiro-shadows-die-twice-game-mechanics).

Consultadas em 30/09/2026. As fontes de domínio fundamentam o significado de vida/postura; os limiares e a política de recuo são escolhas didáticas deste projeto.
