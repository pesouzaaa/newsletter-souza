# automationEconomy (V2)

Função em Deluge (`boletimDiario`) que monta e envia por e-mail um boletim diário com cotações de moedas, uma notícia sorteada e, às segundas-feiras, um resumo semanal de índices/ETFs.

Evolução da V1 (`../V1/automationDolar.deluge`): além do dólar, agora traz outras moedas, gráficos, notícia do dia e resumo semanal.

## O que o boletim contém

| Seção | Frequência | Fonte |
|---|---|---|
| Cotações (USD, EUR, CNY em BRL) com variação do dia | Seg–Sex | [AwesomeAPI](https://economia.awesomeapi.com.br) |
| Gráfico de linha dos últimos 30 dias de cada moeda | Seg–Sex | AwesomeAPI (histórico) + [QuickChart](https://quickchart.io) |
| Notícia do dia (categoria e item sorteados) | Seg–Sex | RSS do G1 (Tecnologia, Economia, Política) |
| Resumo da semana: DIVO11, S&P 500 e STOXX 600 (variação em 5 dias) | Apenas segunda-feira | Yahoo Finance |

Nos fins de semana (sábado e domingo) a função não envia nada e retorna um mapa vazio.

## Como funciona

1. **Verifica o dia**: `getDayOfWeek()` retorna 1 (domingo) a 7 (sábado). Domingo e sábado encerram a execução.
2. **Cotações**: busca `USD-BRL`, `EUR-BRL` e `CNY-BRL` em uma única chamada. Valores abaixo de R$ 1 são arredondados com 4 casas; os demais, com 2. A variação é colorida (verde = alta, vermelho = queda, cinza = estável).
3. **Gráficos**: para cada par, busca os últimos 30 dias, inverte a ordem (a API retorna do mais recente ao mais antigo) e gera uma imagem via QuickChart, embutida no e-mail por URL.
4. **Notícia**: sorteia uma categoria e uma das 20 primeiras notícias do feed RSS, extraídas com XPath.
5. **Índices (segunda-feira)**: consulta o Yahoo Finance (`range=5d&interval=1d`) e calcula a variação entre o fechamento anterior e o preço atual.
6. **Envio**: monta o HTML e envia com `sendmail`.

## Tratamento de erros

- Falha na notícia ou nos índices **não derruba o boletim**: a seção mostra uma mensagem de erro e o restante segue normalmente.
- Falha nas cotações (bloco principal) dispara um e-mail de **"Falha no boletim diário"** com a linha e a mensagem do erro, e o mapa de retorno traz a chave `erro`.

## Retorno

`Map` com:

- `tabela`: HTML da tabela de cotações (quando tudo ocorre bem);
- `erro`: mensagem do erro (quando o bloco principal falha);
- vazio: quando é fim de semana.

## Configuração

- **Destinatário**: definido diretamente no código (`to :"pedroteste38000@gmail.com"`, em dois pontos: e-mail do boletim e e-mail de falha). Altere conforme necessário.
- **Remetente**: `zoho.adminuserid`.
- **Agendamento**: configure a função para rodar diariamente (ex.: via Schedule no Zoho Creator).
- **Moedas**: edite a URL de `.../json/last/USD-BRL,EUR-BRL,CNY-BRL`.
- **Feeds de notícias**: edite o mapa `feeds`.
- **Ativos semanais**: edite o mapa `ativos` (ticker do Yahoo Finance → nome exibido).

## Observações

- Os `info` no código (URL do gráfico e número sorteado) são apenas para depuração e podem ser removidos.
- O Yahoo Finance é chamado com o header `User-Agent: Mozilla/5.0`; sem ele a requisição pode ser recusada.
- As APIs usadas são gratuitas e sem chave, mas podem ter limites de uso ou ficar indisponíveis.
