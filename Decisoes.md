# Decisões de projeto
Foi usado o Haskell Playground por familiaridade e histórico de uso baseado em aulas presenciais.

## Camada 1
Record em vez de tupla. Partida e Linha têm campos nomeados. Em uma tupla trocar os gols da casa pelos do visitante compila sem aviso.

Resultado é um tipo soma (Vitoria | Empate | Derrota), não String nem Int. Um "v" minúsculo não compila, e o -Wall avisa se um case esquecer um construtor.

golsPara (função auxiliar): devolve Maybe (golsPró, golsContra) do ponto de vista de um time. resultadoPara usa golsPara com <$>, então "de que lado o time jogou?" é decidido num lugar só.

times é sort . nub . concatMap ...: sem repetição e em ordem alfabética.

## Camada 2

Sem recursão explícita. linhaDoTime é um foldl sobre filter (participou t); classificacao é sortBy criterio (map (linhaDoTime ps) (times ps)); ateRodada é um filter com composição, ((<= n) . rodada).

Por que foldl e não foldr em linhaDoTime: a linha vai sendo acumulada da primeira partida para a última, e o resultado é o mesmo nos dois casos (soma). foldl deixa a leitura mais natural. Já em maiorGoleada e em separar usei foldr, porque ali é mais simples olhar a lista de trás para frente.

Critério de desempate (estilo Brasileirão): pontos, vitórias, saldo de gols, gols pró e, por fim, o nome do time. É uma única função de comparação: comparing (Down . pontosLinha) <> comparing (Down . vitorias) <> .... Ordering é um Monoid: o <> só consulta o critério seguinte quando o anterior dá EQ. Down inverte a ordem nos critérios em que o maior vem primeiro. 

Encadear vários sortBy não funcionaria, porque o último sobrescreve a ordem dos anteriores.

O nome como último critério não é regra do campeonato: serve para a tabela nunca depender da ordem em que as partidas foram digitadas.

Funções totais. maiorGoleada é um foldr que começa em Nothing; mediaGols casa [] com Nothing. Não há head, maximum nem foldr1.

aproveitamento usa fromIntegral e /. Devolve 0 se o time não jogou, evitando dividir por zero. Pontos disputados = 3 × jogos.

## Camada 3
readMaybe, nunca read. "dois", "2a", "1.5" e campo vazio viram Left (NumeroInvalido linha texto).

Either em do. lerPartida encadeia as três conversões e a validação; o primeiro Left interrompe tudo. lerCampeonato usa traverse para repetir isso em todas as linhas e parar no primeiro erro.

todosOsErros não para no primeiro: junta os Left de todas as linhas (lefts) com os times repetidos na rodada, calculados só sobre as partidas válidas (rights).

Todo erro diz onde está o problema: número da linha original (comentários e linhas em branco são ignorados, mas contam na numeração) ou número da rodada (TimeRepetidoNaRodada).

Construtores além do mínimo: TimeVazio (1;;2;1;Athletico) e RodadaInvalida (rodada 0 ou negativa).

Campeonato vazio (texto vazio ou só comentários) não é erro: dá Right [], e todas as funções respondem sem quebrar (classificacao [] é [], maiorGoleada e mediaGols dão Nothing).

Núcleo puro, casca de IO. O playground não tem entrada padrão, então os dados ficam em dadosCampeonato. O relatório é montado por funções puras (relatorio, relatorioErros) e o main só chama putStr/putStrLn.

## Testes
Testes.hs é o Main.hs com a seção de testes no lugar do main. A duplicação existe porque o playground não permite import entre arquivos.

Os valores esperados (tabela final, tabela após a rodada 3, goleada, média de 36 gols em 15 partidas) foram calculados à mão e conferidos com um cálculo independente fora do Haskell.

Casos de borda cobertos: campeonato vazio, time que não jogou, \r no fim da linha, campo vazio, decimal no lugar de inteiro, campo a mais, rodada 0, desempate por vitórias antes do saldo e desempate final por nome.

Cada construtor de Erro tem pelo menos um teste.

## Limitações
Não confere se o campeonato está completo. Um time pode jogar menos vezes que os outros, e o mesmo confronto pode aparecer duas vezes em rodadas diferentes. Só foi validado o que foi pedido na rúbrica.

Desempate simplificado. Não existe confronto direto nem critério de cartões. Dois times empatados em todos os critérios numéricos são ordenados pelo nome.

Nomes de time são comparados exatamente: "Paraná" e "parana" contam como times diferentes.

Placar sem limite superior. 99;...;1000;0 é aceito.

ateRodada filtra pelo número da rodada, não pela posição na lista, então a lista não precisa estar ordenada. Em compensação, se uma rodada faltar na entrada, ela simplesmente não aparece.

Erros no meio da leitura: lerCampeonato só mostra o primeiro. Para ver todos, deve ser usado todosOsErros.
