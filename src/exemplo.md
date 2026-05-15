# Algoritmo de Karatsuba

## As Limitações do Método Tradicional

Até agora, tratamos qualquer operação aritmética — soma, subtração, multiplicação — como tendo custo $O(1)$. Isso faz sentido para números que cabem em um registrador de 32 ou 64 bits. Mas e quando precisamos multiplicar números com **milhares ou milhões de dígitos**, como em criptografia ou simulações científicas?

Nesses casos, o computador opera **dígito por dígito**. Por isso, precisamos redefinir a operação básica.

!!! Aviso
A partir de agora, a **operação básica** é a manipulação de um único dígito. Isso muda tudo na nossa análise de complexidade.
!!!

Consequências imediatas:

- **Somar** dois números de $n$ dígitos: custo $O(n)$.
- **Multiplicar** pelo método escolar: custo $O(n^2)$.

Nosso objetivo: encontrar um algoritmo melhor do que $O(n^2)$.

## Dividir para Conquistar

Dado dois números $x$ e $y$ de $n$ dígitos, separe-os em metade alta e metade baixa:

$$x = A_1 \cdot 10^{n/2} + A_0$$
$$y = B_1 \cdot 10^{n/2} + B_0$$

**Exemplo:** $x = 1234$, $n=4$ → $A_1 = 12$, $A_0 = 34$, pois $1234 = 12 \cdot 10^2 + 34$.

Expandindo o produto:

$$x \cdot y = (A_1 \cdot 10^{n/2} + A_0)(B_1 \cdot 10^{n/2} + B_0)$$
$$= A_1 B_1 \cdot 10^{n} + (A_1 B_0 + A_0 B_1) \cdot 10^{n/2} + A_0 B_0$$

Multiplicar por $10^{n/2}$ ou $10^n$ é um simples deslocamento (*shift*), custo $O(n)$. O custo real está nas multiplicações entre blocos.

??? Checkpoint
Quantas multiplicações entre blocos de $n/2$ dígitos precisamos realizar?
::: Gabarito
São **4 multiplicações**: $A_1 B_1$, $A_1 B_0$, $A_0 B_1$ e $A_0 B_0$.
:::
???

## Por Que a Divisão Simples Não Basta

Vimos que, ao dividir os números ao meio, geramos **4 multiplicações** de subproblemas de tamanho $n/2$ e um custo adicional de $O(n)$ para as somas e os *shifts*. A relação de recorrência desse método é:

$$T(n) = 4T(n/2) + O(n)$$

Será que essa estratégia de "dividir para conquistar" já é suficiente para vencer o método escolar? Vamos analisar a **árvore de recursão** desse método ingênuo para entender o que está acontecendo.

![Árvore de recursão da divisão simples](divisao_conquista_arvore.png)

??? Checkpoint
No **Nível 0**, temos apenas 1 problema de tamanho $n$. Qual é o custo extra (trabalho de somar e deslocar) desse nível?
::: Gabarito
$$c \cdot n$$
O custo é proporcional a $n$ porque, como vimos, as operações realizadas (somas, subtrações e deslocamentos) percorrem todos os dígitos dos números.

Multiplicamos por uma constante $c$ porque estamos realizando **um número fixo de operações** desse tipo, independentemente de $n$.

Ou seja, o tempo total é algo como:

$$(\text{número de operações}) \times (\text{custo de cada operação}) = c \cdot n$$
:::
???

??? Checkpoint
No **Nível 1**, existem 4 problemas de tamanho $n/2$. Qual é o custo total deste nível?
::: Gabarito
$$4 \cdot (c \cdot \frac{n}{2}) = 2cn$$
:::
???

??? Checkpoint
No **Nível 2**, existem $4^2 = 16$ problemas de tamanho $n/4$. Qual é o custo total?
::: Gabarito
$$16 \cdot (c \cdot \frac{n}{4}) = 4cn$$
:::
???

??? Checkpoint
Consegue perceber o padrão? Escreva a fórmula do custo total no **Nível $i$**.
::: Gabarito
O custo **dobra** a cada nível da árvore. A fórmula para o custo total de um nível $i$ qualquer é:
$$c \cdot n \cdot 2^i$$
:::
???

??? Checkpoint
Se o tamanho do problema começa em $n$ e é **dividido por 2** a cada nível, qual será o tamanho do subproblema após $k$ níveis?
::: Gabarito
O tamanho será:
$$\frac{n}{2^k}$$
:::
???

??? Checkpoint
A recursão para quando o tamanho do problema chega a **1**. Use a fórmula anterior ($\frac{n}{2^k} = 1$) para isolar o $k$ e descobrir a **altura** da árvore.
::: Gabarito
$$\frac{n}{2^k} = 1 \implies n = 2^k \implies k = \log_2 n$$
A altura da árvore é $\log_2 n$.
:::
???

Agora, vamos somar o custo de todos os níveis para chegar no custo total $S$.

??? Checkpoint
Monte a soma dos custos de todos os níveis (do nível 0 até o nível $\log_2 n$).
::: Gabarito
$$S = cn + 2cn + 4cn + \dots + cn \cdot 2^{\log_2 n}$$
Fatorando $cn$:
$$S = cn \cdot [1 + 2 + 4 + \dots + 2^{\log_2 n}]$$
:::
???

??? Checkpoint
A soma dentro dos colchetes é uma PG. Identifique os termos para usar na fórmula da soma ($S_x = a_1 \cdot \frac{q^x - 1}{q - 1}$):
*   Primeiro termo ($a_1$):
*   Razão ($q$):
*   Quantidade de termos ($x$):
::: Gabarito
*   $a_1 = 1$
*   $q = 2$
*   $x = \log_2 n + 1$ (contando do nível 0 ao nível $\log_2 n$)
:::
???

??? Checkpoint
Calcule a soma total $S$ simplificando a expressão. 
*(Dica: lembre-se que $2^{\log_2 n} = n$)*.
::: Gabarito
$$S = cn \cdot \left( \frac{2^{\log_2 n + 1} - 1}{2 - 1} \right)$$
$$S = cn \cdot (2 \cdot 2^{\log_2 n} - 1)$$
$$S = cn \cdot (2n - 1) = 2cn^2 - cn$$
:::
???

**Veredito:** Como o termo de maior ordem é $n^2$, a complexidade é **$O(n^2)$**.

Dividir o número ao meio e gerar 4 subproblemas **não melhorou a eficiência**, continuamos empatados com o método escolar.

Será que existe alguma forma matemática de reduzir esse número de ramificações?

## O Problema Central

Observe novamente a expressão que precisamos computar:

$$x \cdot y = A_1B_1 \cdot 10^{n} + (A_1B_0 + A_0B_1) \cdot 10^{n/2} + A_0B_0$$

Os termos que realmente exigem multiplicação são:
- $A_1B_1$ (produto das partes altas)
- $A_1B_0$ (produto cruzado 1)
- $A_0B_1$ (produto cruzado 2)
- $A_0B_0$ (produto das partes baixas)

 **A Pergunta de Karatsuba**

 Será que realmente precisamos calcular $A_1B_0$ e $A_0B_1$ separadamente?

 Afinal, na expressão final, eles **sempre aparecem somados**: $(A_1B_0 + A_0B_1)$.

 Será que existe um jeito de obter essa **soma** diretamente, sem precisar calcular cada termo individualmente?

??? Checkpoint
Por que estamos interessados apenas na SOMA $A_1B_0 + A_0B_1$ e não nos termos separados?
::: Gabarito
Porque na fórmula final do produto, os dois termos cruzados aparecem APENAS como uma soma:
$$x \cdot y = A_1B_1 \cdot 10^{n} + \color{red}({A_1B_0 + A_0B_1}) \color{black} \cdot 10^{n/2} + A_0B_0$$

Se conseguirmos calcular essa soma diretamente, economizamos uma multiplicação inteira!
:::
???

## A Ideia de Karatsuba

Karatsuba percebeu que poderíamos usar um produto "auxiliar" para nos dar os termos cruzados. Observe o seguinte produto:

$$(\color{blue}A_1\color{black} + \color{green}A_0 \color{black}) \cdot (\color{purple}B_1\color{black} + \color{orange}B_0\color{black})$$

Vamos expandir essa expressão:

$$(\color{blue}A_1\color{black} + \color{green}A_0 \color{black}) \cdot (\color{purple}B_1\color{black} + \color{orange}B_0\color{black}) = \color{blue}A_1 \color{purple}B_1 \color{black}+ \color{blue}A_1\color{orange}B_0 \color{black} + \color{green} A_0 \color{purple}B_1 \color{black}+ \color{green}A_0 \color{orange}B_0$$

Agora, vamos reorganizar os termos:

$$(\color{blue}A_1\color{black} + \color{green}A_0\color{black})(\color{purple}B_1\color{black} +\color{orange} B_0)\color{black} = \underbrace{\color{blue}A_1\color{purple}B_1\color{black} + \color{green}A_0\color{orange}B_0}_{\text{já conhecemos}}\color{black} + \underbrace{(\color{blue}A_1\color{orange}B_0\color{black} + \color{green}A_0\color{purple}B_1\color{black})}_{\text{o que queremos!}}$$

Isolando o termo que nos interessa:

$$\color{blue}A_1\color{orange}B_0 \color{black}+ \color{green}A_0\color{purple}B_1\color{black} = (\color{blue}A_1 \color{black}+ \color{green}A_0\color{black})(\color{purple}B_1 \color{black}+\color{orange} B_0 \color{black}) \color{black}- \color{blue}A_1\color{purple}B_1 \color{black}- \color{green}A_0\color{orange}B_0$$

??? Checkpoint
O que torna essa descoberta tão especial? Que tipo de operação estamos "trocando"?
::: Gabarito
Estamos trocando **DUAS multiplicações** ($\color{blue}A_1\color{orange}B_0$ e $\color{green}A_0\color{purple}B_1$) por:
- **UMA multiplicação** extra: $(\color{blue}A_1 \color{black}+ \color{green}A_0\color{black})(\color{purple}B_1\color{black} + \color{orange}B_0\color{black})$
- **DUAS somas**: $\color{blue}A_1\color{black} + \color{green}A_0$ e $\color{purple}B_1\color{black} + \color{orange}B_0$
- **DUAS subtrações**: $(\color{blue}A_1\color{black} + \color{green}A_0\color{black})(\color{purple}B_1 +\color{orange} B_0\color{black}) - \color{blue}A_1\color{purple}B_1 \color{black}- \color{green}A_0\color{orange}B_0$

Como somas e subtrações custam $O(n)$ enquanto multiplicações custam mais caro ($O(n^2)$),a substituição colabora para a radução da complexidade
:::
???

## Estrutura do Algoritmo

Agora, em vez de 4 multiplicações, fazemos apenas **3 multiplicações recursivas**:

**Passo 1: Calcule três produtos especiais**

- **$Z_1 = A_1 \cdot B_1$** (produto das partes altas)
- **$Z_2 = A_0 \cdot B_0$** (produto das partes baixas)
- **$Z_3 = (A_1 + A_0) \cdot (B_1 + B_0)$** (produto das somas)

**Passo 2: Combine os resultados**

Substituindo na fórmula original:

$$x \cdot y = Z_1 \cdot 10^{n} + (Z_3 - Z_1 - Z_2) \cdot 10^{n/2} + Z_2$$

??? Checkpoint
Quantas multiplicações de números de tamanho $n/2$ são necessárias agora? Compare com o método ingênuo.
::: Gabarito
**Apenas 3 multiplicações!** $Z_1$, $Z_2$ e $Z_3$.

O método ingênuo precisava de 4 multiplicações. Karatsuba economizou **25% das multiplicações** em cada nível da recursão!

A relação de recorrência agora é:
$$T(n) = 3T(n/2) + O(n)$$

Isso é muito melhor que 
$T(n) = 4T(n/2) + O(n)$!
:::
???

Agora que entendemos a ideia matemática, vamos ver como isso aparece em formato de algoritmo.
```c
long karatsuba(long x, long y) {
    if (x < 10 || y < 10) {
        return x * y;
    }

    int n = max(num_digitos(x), num_digitos(y));

   // Garante divisão exata no meio
    if (n % 2 != 0) {
        n++;
    }

    int m = n / 2;

    long potencia = potencia10(m);

    long a1 = x / potencia;
    long a0 = x % potencia;

    long b1 = y / potencia;
    long b0 = y % potencia;

    long ac = karatsuba(a1, b1);
    long bd = karatsuba(a0, b0);
    long abcd = karatsuba(a1 + a0, b1 + b0);

    long meio = abcd - a1c - bd;

    return ac * potencia10(n) + meio * potencia + bd;
}
```
!!! Aviso: Visão de Hardware e Otimização
O código acima utiliza a base 10 (potências de 10) para facilitar o entendimento didático da matemática. No entanto, em implementações reais de baixo nível, as operações de divisão ```txt (/)``` e módulo ```txt (%)``` são bastante custosas para o processador. 

Na prática, o Karatsuba é implementado em **base 2 (binário)**. Isso permite substituir as divisões e multiplicações por **deslocamentos de bits** ```txt (>> e <<)``` e **máscaras bit a bit** ```txt (&)```, que geralmente custam apenas 1 ciclo de *clock*, maximizando a eficiência da execução.
!!!

## Por Que Isso é Revolucionário?

| Abordagem | Multiplicações por nível | Complexidade final |
|-----------|:------------------------:|:------------------:|
| Método escolar | 1 (mas dígito a dígito) | $O(n^2)$ |
| Divisão ingênua | 4 | $O(n^2)$ |
| **Karatsuba** | **3** | $\mathbf{O(n^{1.585})}$ |

Reduzir de 4 para 3 multiplicações pode parecer pouco, mas veja o impacto na árvore de recursão:
Cada nível tem **3 vezes mais problemas** que o anterior
Mas os problemas têm **metade do tamanho**
O número total de operações cresce como $n^{\log_2 3} \approx n^{1.585}$

 Para $n = 1.000.000$ dígitos, isso é **centenas de vezes mais rápido** que $O(n^2)$!

## Exemplo Passo a Passo

Vamos aplicar Karatsuba a um exemplo concreto para fixar o entendimento.

**Problema:** Multiplicar $A = 12$ e $B = 34$ (base 10, $n=2$)

### Etapa 1: Dividir os números

- $A = 12 \rightarrow A_1 = 1$, $A_0 = 2$
- $B = 34 \rightarrow B_1 = 3$, $B_0 = 4$

### Etapa 2: Calcular os três produtos

- $Z_1 = A_1 \cdot B_1 = 1 \cdot 3 = 3$
- $Z_2 = A_0 \cdot B_0 = 2 \cdot 4 = 8$
- $Z_3 = (A_1 + A_0) \cdot (B_1 + B_0) = (1+2) \cdot (3+4) = 3 \cdot 7 = 21$

??? Checkpoint
Por que $Z_3$ usa as somas $A_1+A_0$ e $B_1+B_0$? O que isso representa?
::: Gabarito
$Z_3$ é o produto auxiliar que nos dará os termos cruzados. Quando expandimos:

$$(A_1+A_0)(B_1+B_0) = A_1B_1 + A_1B_0 + A_0B_1 + A_0B_0 = Z_1 + (A_1B_0 + A_0B_1) + Z_2$$

É como se $Z_3$ contivesse "tudo de uma vez" — daí podemos isolar a soma que precisamos.
:::
???

### Etapa 3: Calcular o termo do meio

- $Z_3 - Z_1 - Z_2 = 21 - 3 - 8 = 10$

Isto é exatamente $(A_1B_0 + A_0B_1)$!

### Etapa 4: Montar o resultado final

Lembre-se: $10^{n} = 10^2 = 100$ e $10^{n/2} = 10^1 = 10$

$$x \cdot y = Z_1 \cdot 100 + (Z_3 - Z_1 - Z_2) \cdot 10 + Z_2$$
$$= 3 \cdot 100 + 10 \cdot 10 + 8$$
$$= 300 + 100 + 8 = 408$$

Conferindo: $12 \times 34 = 408$ ✓

??? Checkpoint
Neste exemplo, usamos multiplicações diretas porque $n=2$ (caso base). Em um caso maior, como $n=8$, o que aconteceria?
::: Gabarito
Para $n=8$, cada $Z_1$, $Z_2$ e $Z_3$ seria calculado **recursivamente** usando o MESMO algoritmo!

Ou seja:
- $Z_1 = \text{Karatsuba}(A_1, B_1)$ (subproblemas de 4 dígitos)
- $Z_2 = \text{Karatsuba}(A_0, B_0)$ (subproblemas de 4 dígitos)
- $Z_3 = \text{Karatsuba}(A_1+A_0, B_1+B_0)$ (subproblemas de 4 dígitos)

E cada um desses, por sua vez, faria mais 3 chamadas recursivas, e assim por diante, até chegar em números de 1 dígito.
:::
???

## A Recursão

Agora que entendemos como o algoritmo reduz o número de multiplicações para 3, precisamos analisar como isso impacta o tempo de execução.

A resposta está em uma mudança de perspectiva fundamental: **essas três multiplicações não são resolvidas diretamente** como uma conta de escola. Em vez disso, cada uma delas é uma **chamada recursiva** à própria função de Karatsuba.

Essas multiplicações são resolvidas de forma recursiva: o algoritmo chama a si mesmo para resolver subproblemas menores.

!!! Aviso
A base de todo processo recursivo é saber a hora de parar. Quando um número é tão pequeno que tem apenas **1 dígito**, a multiplicação se torna a nossa operação básica e tem custo $O(1)$. Essa é a condição de parada do nosso algoritmo.
!!!

O algoritmo realiza três chamadas recursivas sobre subproblemas de tamanho $n/2$.

Agora que temos clareza sobre **quais** são e **como** essas três multiplicações se conectam, vamos focar no que realmente importa para o nosso objetivo: **provar que essa estratégia é, de fato, mais rápida.**

Antes de montar a árvore, precisamos separar bem duas coisas que acontecem dentro do algoritmo:

1.  **O custo das chamadas recursivas:** As três multiplicações que o algoritmo chama para resolver os subproblemas menores. Esse custo é representado por $T(n/2)$.
2.  **O custo do trabalho extra:** Tudo que o algoritmo faz *além* das chamadas recursivas. Estamos falando das somas ($A_1 + A_0$ e $B_1 + B_0$), das subtrações para recompor o resultado e dos deslocamentos (multiplicar por $10^n$ ou $10^{n/2}$).

Como as somas e os deslocamentos não envolvem nenhum laço aninhado ou operação mais cara, podemos analisar seu custo pensando na quantidade de dígitos.

Se os números têm $n$ dígitos, operações como soma, subtração e deslocamento percorrem todos os dígitos do número. Como lidamos com $n$ dígitos, cada uma dessas operações custa $O(n)$.

Como realizamos apenas um número constante dessas operações em cada nível, o custo total desse trabalho extra cresce proporcionalmente a $n$.

Por isso, representamos esse custo como $O(n)$.

!!! Aviso
Esse termo $O(n)$ **não representa uma nova chamada recursiva**.

Ele corresponde ao trabalho feito **dentro de cada nível** do algoritmo (somas, subtrações e deslocamentos).
!!!


## Montando a Recorrência

Com isso, o custo do algoritmo em um único nível é: $3$ chamadas recursivas (cada uma com metade do tamanho) mais esse trabalho extra de $O(n)$.

??? Checkpoint
Com base no que acabamos de ver sobre a estrutura recursiva, monte a relação de recorrência para o algoritmo de Karatsuba.
Escreva a equação $T(n) = \dots$
::: Gabarito
A relação de recorrência que descreve o tempo de execução do algoritmo é:

$$T(n) = 3T\left(\frac{n}{2}\right) + O(n)$$

Onde $3$ representa o número de chamadas, $\frac{n}{2}$ é o tamanho da entrada que cada chamada resolve, e $O(n)$ é o custo de "remontar" a solução com somas e deslocamentos.
:::
???

## Analisando a Complexidade

Agora que entendemos a estrutura recursiva do algoritmo, queremos responder à pergunta central:

**qual é o custo total do Karatsuba?**

Vamos usar a **árvore de recursão**, pois ela nos permite visualizar o custo em cada nível da execução.

Observe a árvore abaixo. Cada nó representa uma chamada recursiva do algoritmo, e cada nível mostra como o problema é dividido em subproblemas menores.

![Árvore de recursão do algoritmo de Karatsuba](karatsuba_arvore.png)

??? Checkpoint
Observe a árvore acima. Em cada chamada, quantos subproblemas são gerados? E qual é o tamanho de cada um?
::: Gabarito
O algoritmo gera 3 subproblemas, cada um com tamanho $n/2$.
???
Agora vamos analisar o custo por nível da árvore.

Como o trabalho extra para combinar os resultados (somas e subtrações) continua sendo proporcional ao número de dígitos, o custo do nível 0 ainda é $cn$.

??? Checkpoint
No nível 0 existe apenas 1 problema de tamanho $n$.
Qual é o custo extra desse nível?
::: Gabarito
$$c\cdot n$$

Como o trabalho extra para combinar os resultados (somas e subtrações) continua sendo proporcional ao número de dígitos, o custo do nível 0 ainda é $cn$.
:::
???

??? Checkpoint
No nível 1 existem 3 problemas de tamanho $n/2$.
Qual é o custo total desse nível?
::: Gabarito
$$3\cdot c\cdot \frac{n}{2}=c\cdot n\cdot \frac{3}{2}$$
:::
???

??? Checkpoint
No nível 2 existem $3^2=9$ problemas de tamanho $n/4$.
Qual é o custo total?
::: Gabarito
$$9\cdot c\cdot \frac{n}{4}=c\cdot n\cdot \left(\frac{3}{2}\right)^2$$
:::
???

Consegue perceber o padrão?

??? Checkpoint
Escreva a fórmula do custo total no nível $i$.
::: Gabarito
$$c\cdot n\cdot \left(\frac{3}{2}\right)^i$$
:::
???

## Somando os Níveis da Árvore

A altura da árvore continua sendo $k = \log_2 n$, pois o tamanho do problema ainda é dividido por 2 a cada passo. Vamos montar a soma total $S$:

??? Checkpoint
Monte a soma dos custos dos níveis da árvore.
::: Gabarito
$$S=c\cdot n\left[1+\frac{3}{2}+\left(\frac{3}{2}\right)^2+\dots+\left(\frac{3}{2}\right)^{\log_2 n}\right]$$
:::
???

??? Checkpoint
A soma obtida é uma progressão geométrica.

$$1+\frac{3}{2}+\left(\frac{3}{2}\right)^2+\dots+\left(\frac{3}{2}\right)^{\log_2 n}$$

Identifique:

*   o primeiro termo $a_1$
*   a razão $q$
*   a quantidade de termos $x$
::: Gabarito
*   $a_1=1$
*   $q=\frac{3}{2}$
*   $x=\log_2 n+1$
:::
???

??? Checkpoint
Agora calcule a soma total dos custos. Para isso, use a fórmula da progressão geométrica.

$$S_n=a_1\cdot \frac{q^x-1}{q-1}$$
::: Gabarito
$$S=c\cdot n\cdot \left(\frac{\left(\frac{3}{2}\right)^{\log_2 n+1}-1}{\frac{3}{2}-1}\right)$$

Simplificando:

$$S=2c\cdot n\left[\left(\frac{3}{2}\right)^{\log_2 n+1}-1\right]$$
:::
???

??? Checkpoint
Reescreva a potência separando o expoente $\log_2 n+1$.
Partimos de:

$$S=2c\cdot n\left[\left(\frac{3}{2}\right)^{\log_2 n+1}-1\right]$$
::: Gabarito
$$\left(\frac{3}{2}\right)^{\log_2 n+1} = \left(\frac{3}{2}\right)\left(\frac{3}{2}\right)^{\log_2 n}$$

Portanto:

$$S=2c\cdot n\left[\frac{3}{2}\left(\frac{3}{2}\right)^{\log_2 n}-1\right]$$
:::
???

??? Checkpoint
Simplifique a potência:

$$\left(\frac{3}{2}\right)^{\log_2 n}$$
::: Gabarito
$$\left(\frac{3}{2}\right)^{\log_2 n} = \frac{3^{\log_2 n}}{2^{\log_2 n}}$$

Como:

$$2^{\log_2 n}=n$$
e
$$3^{\log_2 n}=n^{\log_2 3}$$

temos:

$$\left(\frac{3}{2}\right)^{\log_2 n} = \frac{n^{\log_2 3}}{n}$$
:::
???

Substituindo esse resultado na expressão do custo total $S$, vemos que o termo principal da soma cresce proporcionalmente a $n^{\log_2 3}$.

Como estamos interessados na ordem de crescimento, ignoramos constantes e termos menores.

Portanto, a complexidade do algoritmo de Karatsuba é:

$$T(n)=O(n^{\log_2 3})\approx O(n^{1.58})$$

## Resumindo 

Karatsuba percebeu que:

1. **O problema real** são as 4 multiplicações do método ingênuo
2. **A solução** é calcular um produto extra $(A_1 + A_0)(B_1 + B_0)$
3. **A economia** vem de obter $A_1B_0 + A_0B_1$ por diferença, evitando duas multiplicações

 Trocamos **multiplicações caras** por **adições e subtrações baratas**.

Isso exige apenas um pequeno trabalho extra de 
$O(n)$ por nível. O benefício é reduzir o fator de ramificação da árvore de 4 para 3.