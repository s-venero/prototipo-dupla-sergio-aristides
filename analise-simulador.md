# Análise do simulador de investimentos da ArenaCash

Documento sobre o arquivo `simulador.js`, do repositório público
[Product-Arena/arena-cash](https://github.com/Product-Arena/arena-cash/blob/main/simulador.js).
Escrito para quem não programa. Todos os exemplos foram conferidos rodando a lógica real do arquivo.

---

## 1. O que o simulador faz

O simulador pede três informações ao cliente: **idade**, **salário mensal** e **perfil de investidor** (conservador, moderado ou arrojado). Com elas, ele escolhe **um** entre cinco produtos da ArenaCash, sugere quanto a pessoa deveria investir por mês (sempre **15% do salário**) e mostra os motivos da escolha. A decisão segue uma sequência fixa: primeiro verifica se a pessoa já consegue guardar o mínimo; depois transforma idade e perfil em uma pontuação; por fim, a pontuação aponta o produto. Há duas regras que podem mudar o resultado: uma que vale antes de tudo e outra que só vale para o produto mais arriscado.

---

## 2. Os produtos

| Produto | Rendimento | Prazo | Risco |
|---|---|---|---|
| **CDB Reserva** | 102% do CDI | Dinheiro disponível todo dia | Muito baixo |
| **LCI Isenta** | 95% do CDI, sem Imposto de Renda | 1 ano preso | Baixo |
| **CDB Progressivo** | 112% do CDI | 2 anos | Baixo |
| **CDB Longo Prazo** | 124% do CDI | 5 anos | Médio |
| **Arena Quant** | Variável, sem garantia | Resgate em 30 dias | Alto |

*O CDI é a taxa de juros de referência do mercado. "102% do CDI" quer dizer que o dinheiro rende um pouco mais do que essa taxa.*

*"Prazo preso" significa que tirar o dinheiro antes do fim tem custo. No CDB Longo Prazo, por exemplo, o resgate antecipado tem desconto.*

---

## 3. Como a pontuação é calculada, passo a passo

Se a pessoa passar pela regra de reserva (seção 4), o simulador faz esta conta.

**Passo 1. Idade vira pontos de "tempo pela frente".** Quanto mais nova a pessoa, mais tempo ela tem e mais risco pode correr.

| Idade | Pontos |
|---|---|
| Menos de 30 anos | 3 |
| De 30 a 49 anos | 2 |
| 50 anos ou mais | 1 |

**Passo 2. O perfil vira pontos de "tolerância a risco".** É o que a própria pessoa declara.

| Perfil | Pontos |
|---|---|
| Arrojado | 3 |
| Moderado | 2 |
| Conservador | 1 |

**Passo 3. Somam-se os dois.** O resultado vai de 2 (mais cauteloso possível) a 6 (mais arrojado possível).

**Passo 4. A soma aponta o produto.**

| Pontuação | Produto indicado |
|---|---|
| 2 | CDB Reserva |
| 3 | LCI Isenta |
| 4 | CDB Progressivo |
| 5 | CDB Longo Prazo |
| 6 | Arena Quant (com uma condição, veja a seção 4) |

**Detalhe importante:** idade e perfil têm o **mesmo peso**. Por isso, pessoas muito diferentes podem ter a mesma pontuação e receber o mesmo produto.

---

## 4. As regras que têm precedência sobre a pontuação

### Regra 1: reserva antes de tudo (vale primeiro, sobre todo o resto)

O simulador calcula o aporte (15% do salário, arredondado). Se der **menos de R$ 100 por mês**, a resposta é sempre o **CDB Reserva**. Idade e perfil **são ignorados**.

- O salário mínimo para passar dessa regra é **R$ 664** (aporte de R$ 100). Com R$ 663, o aporte é R$ 99 e a regra vale.
- A justificativa no próprio código: quem ainda não consegue guardar R$ 100 por mês não deveria prender dinheiro em prazo longo.
- O código admite que é a regra que mais gera reclamação de cliente jovem e arrojado, mas que também evita resgate com prejuízo.

### Regra 2: o Arena Quant exige aporte de R$ 500 ou mais

Quem chega a 6 pontos só recebe o Arena Quant se o aporte for de **R$ 500 por mês ou mais**. Caso contrário, "desce um degrau" e recebe o **CDB Longo Prazo**.

- O salário mínimo para isso é **R$ 3.330** (aporte de R$ 500). Com R$ 3.329, o aporte é R$ 499 e a pessoa fica no CDB Longo Prazo.
- Como 6 pontos só existem para quem tem **menos de 30 anos e se declara arrojado**, **só esse grupo pode receber o Arena Quant**.

---

## 5. Casos-limite

Todos os resultados abaixo foram obtidos rodando o código do simulador.

### 5.1 Regra 1 ignora o perfil arrojado

| Idade | Salário | Perfil | Aporte | Resultado |
|---|---|---|---|---|
| 24 | R$ 660 | arrojado | R$ 99 | **CDB Reserva** |
| 24 | R$ 667 | arrojado | R$ 100 | **CDB Longo Prazo** |

Com R$ 7 a mais de salário, a pessoa pula do produto mais conservador para um de 5 anos com dinheiro preso. Na tela do primeiro caso, aparece "pontuação de risco: **6 de 6**" ao lado do CDB Reserva, porque o programa mostra a pontuação mesmo quando não a usou.

### 5.2 Um degrau de R$ 40 separa o CDB do Arena Quant

| Idade | Salário | Perfil | Aporte | Resultado |
|---|---|---|---|---|
| 25 | R$ 3.300 | arrojado | R$ 495 | **CDB Longo Prazo** |
| 25 | R$ 3.340 | arrojado | R$ 501 | **Arena Quant** |

Uma diferença pequena de salário leva de um produto de risco médio a um de risco alto, que "pode fechar o ano no negativo".

### 5.3 O aniversário pode piorar a indicação

| Idade | Salário | Perfil | Resultado |
|---|---|---|---|
| 29 | R$ 5.000 | conservador | **CDB Progressivo** (112% do CDI, 2 anos) |
| 30 | R$ 5.000 | conservador | **LCI Isenta** (95% do CDI, 1 ano) |

Ao completar 30 anos, o rendimento indicado cai. O mesmo acontece aos 49 e 50 anos para o perfil moderado (de CDB Progressivo para LCI Isenta).

### 5.4 Pessoas diferentes, mesma indicação

| Idade | Perfil | Pontos | Resultado |
|---|---|---|---|
| 25 | moderado | 3 + 2 = 5 | CDB Longo Prazo |
| 40 | arrojado | 2 + 3 = 5 | CDB Longo Prazo |
| 22 | conservador | 3 + 1 = 4 | CDB Progressivo |
| 49 | moderado | 2 + 2 = 4 | CDB Progressivo |

Um conservador de 22 anos, que "prioriza não perder dinheiro", recebe um produto de 2 anos, mais preso do que o recebido por um conservador de 30 anos (1 ano).

### 5.5 Salário alto não compensa a idade

| Idade | Salário | Perfil | Aporte | Resultado |
|---|---|---|---|---|
| 40 | R$ 20.000 | arrojado | R$ 3.000 | **CDB Longo Prazo** |

Mesmo investindo seis vezes o mínimo exigido, essa pessoa nunca chega ao Arena Quant.

### 5.6 Os limites de entrada estão só na página

A página do simulador só aceita **idade de 18 a 100 anos** e **salário a partir de R$ 0** (em passos de R$ 100). Esses limites ficam no formulário, não na parte que decide a indicação. Por isso:

- Um **salário de R$ 0** é barrado pelo programa (a conta não roda).
- Se a regra de decisão for reaproveitada em outro lugar (outro site, aplicativo), ela **aceita qualquer valor**. Por exemplo, idade 150 e salário R$ 5.000 com perfil arrojado resultam em **CDB Progressivo**, sem nenhum aviso, porque 150 é tratado como "50 ou mais".

Dentro dos limites da página, o risco é pequeno. A questão é de manutenção: a proteção depende de ninguém mexer no formulário.

---

## 6. Perguntas que eu levaria ao time de produto

**Sobre a Regra 1**
1. Quando a Regra 1 manda um cliente arrojado para o CDB Reserva, **a tela deveria dizer isso de forma explícita** (por exemplo, "seu perfil foi desconsiderado por enquanto")? Hoje ela mostra "6 de 6" ao lado de um produto conservador.
2. O corte em R$ 100 de aporte foi **definido com base em quê**? Existe algum dado de resgate antecipado que o justifique?
3. Faz sentido a recomendação **pular de CDB Reserva direto para CDB Longo Prazo** com R$ 1 a mais de aporte, sem passar por um produto intermediário?

**Sobre a Regra 2**
4. Por que o Arena Quant é **exclusivo de menores de 30 anos**? É uma decisão de risco ou um efeito colateral da conta?
5. Um cliente de 40 anos, arrojado e com aporte de R$ 3.000, **pode querer o Arena Quant**. Ele deveria poder escolher mesmo assim, com um aviso?
6. O corte de R$ 500 tem **degrau brusco**. Vale uma faixa de transição?

**Sobre a pontuação**
7. **Idade e perfil devem pesar igual?** Um arrojado de 40 e um moderado de 25 receberem o mesmo produto é a intenção?
8. A idade corta **em degraus** (aos 30 e aos 50). Existe um motivo para esses números, ou um corte mais suave seria melhor?
9. O perfil é **declarado pelo cliente**. Existe alguma checagem, como um questionário, ou a pessoa simplesmente escolhe?
10. Um **conservador de 22 anos** receber um produto de 2 anos preso está de acordo com o que a empresa promete a esse perfil?

**Sobre a tela e os dados**
11. Os limites de idade (18 a 100) e salário ficam **só no formulário**. Se essa regra de decisão for usada em outro canal, ela passa a aceitar qualquer valor. Vale colocar a validação também dentro da regra?
12. O cliente sabe que **15% do salário** é só uma sugestão padrão? Faz sentido ele poder ajustar esse valor?
