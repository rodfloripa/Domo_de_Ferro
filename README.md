<p align="justify"><h1>Projeto de Interceptação com Otimização Convexa: Inspirado no Domo de Ferro</h1></p>

<p align="justify"><b>Objetivo do Projeto:</b> Este projeto visa desenvolver um sistema de controle para um interceptador capaz de neutralizar alvos em movimento no espaço 3D, utilizando princípios de otimização matemática. A principal inspiração para este modelo é o <b>Domo de Ferro (Iron Dome)</b>, um sistema de defesa aérea móvel desenvolvido por Israel. O Domo de Ferro foi projetado para interceptar e destruir foguetes de curto alcance e projéteis de artilharia disparados de distâncias próximas. Ele funciona identificando a ameaça via radar, calculando sua trajetória e lançando um míssil interceptador apenas se o alvo representar um risco a áreas povoadas. O desafio central aqui é reproduzir essa inteligência: calcular a trajetória que minimize o consumo de energia (esforço de aceleração) enquanto garante o impacto no ponto e tempo exatos.</p>

<p align="justify"><h3>1. Definição das Variáveis de Estado</h3></p>

<p align="justify">O primeiro passo é definir o que o otimizador pode controlar. No nosso caso, as variáveis de decisão são as acelerações em cada eixo (x, y, z) e as posições e velocidades resultantes ao longo do tempo.</p>

``` 
Python

#N é o número de passos temporais
ax = cp.Variable(N)  # Controle de aceleração no eixo X
ay = cp.Variable(N)  # Controle de aceleração no eixo Y
az = cp.Variable(N)  # Controle de aceleração no eixo Z
``` 

<p align="justify"><h3>2. Modelagem da Dinâmica (Restrições Afins)</h3></p>

<p align="justify">Para manter o problema convexo, a dinâmica de movimento é inserida como restrições lineares. Isso descreve como a posição e a velocidade evoluem a cada passo de tempo <i>dt</i> baseado na aceleração aplicada.</p>

```
Python

# Exemplo da dinâmica no eixo Z considerando a gravidade (g)
constraints = [
    vz[k+1] == vz[k] + (az[k] - g) * dt,
    z[k+1]  == z[k] + vz[k] * dt + 0.5 * (az[k] - g) * dt**2
]
```


<p align="justify"><h3>3. Condição de Interceptação e Altura Crítica</h3></p>

<p align="justify">O "objetivo" físico é traduzido como uma restrição de igualdade no instante do impacto (índice <i>k_hit</i>). Um ponto crucial é a <b>restrição de altura mínima no choque</b>: garantimos que a interceptação ocorra acima de um limite de segurança (<i>z_min</i>), impedindo que o sistema valide uma colisão que aconteça em uma altura muito baixa, oque seria um risco para a população.</p>

```
Python

# Restrição de impacto e altura mínima de interceptação
constraints += [
    x[k_hit] == alvo_x[k_hit],
    y[k_hit] == alvo_y[k_hit],
    z[k_hit] == alvo_z[k_hit],
    z[k_hit] >= h_min  # Garante que o choque ocorra na altura minima
]
```

<p align="justify"><h3>4. Função Objetivo: Minimização de Esforço</h3></p>

<p align="justify">Para que o interceptador seja eficiente, minimizamos a "energia" gasta, representada pela soma dos quadrados das acelerações. Como a soma de quadrados é uma função convexa, o CVXPY consegue resolvê-la rapidamente.</p>

```
Python

# Minimizando a norma quadrática da aceleração total
objetivo = cp.Minimize(cp.sum_squares(ax) + cp.sum_squares(ay) + cp.sum_squares(az))
problema = cp.Problem(objetivo, constraints)
problema.solve()
```

<p align="justify"><h3>5. Fundamentos da Convexidade</h3></p>

<p align="justify">O que garante a robustez matemática deste modelo é a <b>convexidade</b>. Nenhuma operação mistura variáveis de forma multiplicativa ou discreta. Tudo no modelo é construído rigorosamente usando combinações lineares, constantes e funções convexas conhecidas, como normas e quadrados.</p>

<p align="justify"><h3>6. O que quebraria a convexidade (Regras DCP)</h3></p>

<p align="justify">É fundamental evitar certos elementos que impediriam o solver de encontrar a solução ótima global. Se você alterar o código para incluir qualquer um dos itens abaixo, o problema deixa de ser <b>DCP (Disciplined Convex Programming)</b>:</p>

<p align="justify">
<ul>
<li>Termos como v² no modelo (como o arrasto do ar);</li>
<li>Produto entre variáveis (ex: x * v);</li>
<li>Otimização do tempo de interceptação (tornar o tempo uma variável);</li>
<li>Restrições do tipo “mínimo tempo”;</li>
<li>Lógica condicional (uso de <i>if</i> dentro do modelo de otimização).</li>
</ul>
</p>

<p align="justify"><h3>7. Conclusão Técnica e Video</h3></p>

<p align="justify">Ao estruturar o problema como um controle ótimo linear com custo quadrático, garantimos que o sistema não apenas funcione, mas seja o mais eficiente possível dentro dos limites físicos estabelecidos pelas restrições de aceleração máxima e altura mínima.</p>

![Video](https://github.com/rodfloripa/Domo_de_Ferro/blob/main/interceptacao(1).gif)
