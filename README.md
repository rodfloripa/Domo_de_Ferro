Aqui está o relatório técnico detalhado sobre a estrutura de otimização do projeto, formatado em Markdown com o alinhamento justificado conforme solicitado:

\<p align="justify"\>\<h1\>Análise de Convexidade e Estrutura DCP no Projeto de Interceptação\</h1\>\</p\>

\<p align="justify"\>O sucesso de um modelo de otimização desenvolvido em \<b\>CVXPY\</b\> reside na sua capacidade de ser classificado como \<b\>DCP (Disciplined Convex Programming)\</b\>. No projeto de interceptação em 3D, a garantia de que o algoritmo encontrará a solução ótima global de forma eficiente não é acidental; ela é o resultado de uma estruturação matemática rigorosa que transforma leis físicas em restrições convexas. Abaixo, detalhamos as simplificações e definições que permitem essa classificação.\</p\>

\<p align="justify"\>\<h3\>1. Dinâmica Linear (Afim nas Variáveis)\</h3\>\</p\>

\<p align="justify"\>As equações de movimento do interceptador foram modeladas de forma que a posição dependa da velocidade e da aceleração, e a velocidade dependa da aceleração de forma linear. Mesmo em equações como \<code\>x[k+1] = x[k] + v[k]·dt + 0.5·a[k]·dt²\</code\>, a estrutura é mantida como uma soma de variáveis multiplicadas por constantes. Não existe produto entre variáveis de decisão nem o uso de funções não lineares complexas (como exponenciais ou logaritmos), o que torna todas as equações de transição de estado afins.\</p\>

\<p align="justify"\>\<h3\>2. Tempo de Interceptação Fixo\</h3\>\</p\>

\<p align="justify"\>A interceptação ocorre em um índice temporal (k\_hit) já definido no problema. Ao fixar o momento do impacto, removemos a necessidade de tratar o tempo como uma variável de otimização ou incluir decisões discretas. Isso evita problemas combinatórios e garante que não existam variáveis inteiras ou lógica condicional "if/else" dentro do modelo, mantendo a convexidade estrita.\</p\>

\<p align="justify"\>\<h3\>3. Trajetória do Alvo como Parâmetro Fixo\</h3\>\</p\>

\<p align="justify"\>Os dados da trajetória do inimigo entram no modelo como constantes (parâmetros) e não como variáveis de decisão. Isso é fundamental para evitar termos bilineares (como o produto entre a posição do interceptador e a posição do alvo). Como a posição do alvo é um dado de entrada, a restrição de interceptação permanece uma igualdade linear simples.\</p\>

\<p align="justify"\>\<h3\>4. Função Objetivo Quadrática Convexa\</h3\>\</p\>

\<p align="justify"\>A função objetivo é definida para minimizar a soma dos quadrados das acelerações: \<code\>cp.Minimize(cp.sum\_squares(ax) + cp.sum\_squares(ay) + cp.sum\_squares(az))\</code\>. Por ser uma soma de quadrados, a função resultante é sempre não negativa e possui uma curvatura convexa (Hessiana definida positiva). Isso caracteriza o modelo como um Problema Quadrático Convexo (QP), que é extremamente estável para os solvers.\</p\>

\<p align="justify"\>\<h3\>5. Restrições de Valor Absoluto\</h3\>\</p\>

\<p align="justify"\>O modelo utiliza restrições de desigualdade para limitar a aceleração, como \<code\>cp.abs(ax[k]) \<= a\_max\</code\>. Embora a função valor absoluto possua uma "quina", ela é uma função convexa. O CVXPY a decompõe internamente em duas desigualdades lineares, mantendo o conjunto de restrições viáveis convexo e fácil de navegar pelo algoritmo.\</p\>

\<p align="justify"\>\<h3\>6. Gravidade e Ausência de Não Linearidades Físicas\</h3\>\</p\>

\<p align="justify"\>A aceleração da gravidade é tratada como uma constante externa na dinâmica vertical, mantendo a estrutura afim. Além disso, o modelo deliberadamente ignora o arrasto aerodinâmico (que dependeria do quadrado da velocidade) e outras forças não lineares. Essa simplificação essencial permite que o problema seja resolvido em milissegundos, garantindo uma solução global ótima que não seria possível em modelos de controle não linear tradicionais.\</p\>

\<p align="justify"\>\<h3\>7. Conclusão Técnica\</h3\>\</p\>

\<p align="justify"\>O problema é estritamente convexo porque foi estruturado como um controle ótimo linear com custo quadrático (LQR discreto simplificado). Ao utilizar apenas combinações lineares, constantes e funções convexas conhecidas (normas e quadrados), o modelo garante estabilidade numérica e rapidez de convergência, eliminando o risco de o sistema ficar "preso" em mínimos locais irrelevantes.\</p\>
