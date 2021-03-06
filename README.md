
<h1 align="center">
<br>
Lendo Valores medidores Veeder-Root
</h1>

Os medidores Gilbarco Veeder-Root modelo [TLS4](https://www.gilbarco.com/br/produtos/medicao-e-monitoramento/tls4) possui interface web para gerenciamento.
O objetivo seria integrar, de forma automática, o sistema veeder-root com um banco de dados, no nosso caso seria o historiador PIMS - OSIsoft para ter histórico e também poder gerar relatórios de forma automática.

<p align="center">
  <a href="https://www.apache.org/licenses/LICENSE-2.0">
    <img src="https://img.shields.io/badge/apache-2.0-blue" alt="License Apache">
  </a>
</p>


<div align="center">
  <img src="https://github.com/dedynobre/lendo-valores-medidores-veeder-root/blob/master/imagens/lub.gif" alt="conexoes-clientes" height="250" width="400">
</div>

<hr />

## Desenvolvimento

O sistema da Gilbarco possui um protocolo aberto de comunicação.
Conforme informações obtidas o equipamento responde uma requisição tcp na porta **10001**:

<img src="https://github.com/dedynobre/lendo-valores-medidores-veeder-root/blob/master/imagens/img-01.png"/>

Que após fazer a consulta os dados são retornados da seguinte forma:

<img src="https://github.com/dedynobre/lendo-valores-medidores-veeder-root/blob/master/imagens/img-02.png"/>

O manual, que se encontra no repositório, traz diversos códigos que podem ser digitados para poder obter dados específicos.
No nosso caso queremos coletar os dados de volume atual, consumido, restante, temperatura, nível de água e altura.
Para este caso, conforme manual na página 51, o código a ser digitado seria **<SOH>I201TT**.
Este código quando é passado para a requisição TCP deve ser convertido para hexadecimal, com isso temos:
+ SOH: atalho ctrl + A, que equivale a 0x01
+ I: equivale a 0x49
+ 2: equivale a 0x32
+ 0: equivale a 0x30
+ 1: equivale a 0x31
+ 0: equivale a 0x30
+ 1: equivale a 0x31

A seguencia será sempre assim, envia o comando ctrl+A para 'abrir a comunicação' e entra com o código desejado, como mostrado acima.

O site interessante que ajuda a trabalhar com a conversão é este [aqui](http://www.insecuritynet.com.br/ferramentas-online/converter-texto-para-hexadecimal).

Bom, depois de termos estas definições temos que programar para poder coletar os dados de forma automática pois desta forma continua sendo manual igual foi feito acima através do comando telnet pelo powershell ou pelo próprio cmd.

Existem várias formas de buscar os dados, para o meu caso foi utilizado o [Node-Red](https://nodered.org/), pois considerar um ferramente poderosíssima para conexão de 'coisas' e também por estsar alinhado com estratégia de IOT.

Um fluxo básico de extração dos dados está abaixo:
<img src="https://github.com/dedynobre/lendo-valores-medidores-veeder-root/blob/master/imagens/img-03.png"/>
 
No bloco de função temos o seguinte código, sendo que os dois últimos equivale ao número do tanque, por exemplo, 00 - todos os tanques, 01 - tanque 01, 02 - tanque 02, etc:
```javascript
var a = new Buffer([

	0x01,
	0x49,
	0x32,
	0x30,
	0x31,
	0x30,
	0x31
	
])


msg.payload = a
return msg;
```

O comando de envio para a conexão TCP deve ser no formato de buffer, por isso existe a função new Buffer() na conversão do valor em hexadecimal.

O node TCP Request basicamente precisa do IP do dispostivo e a porta de conexão.

o Formato de saída do node TCP é buffer. Sendo que temos que converter o mesmo para uma string. A função para conversar é `.toString()`.

Depois desta conversão a string fica no formato da segunda imagem mostrada acima, sendo que na imagem mostra os dados de todos os tanques. Temos opção de busca por tanque individualmente.

O código da segunda função que faz a extração dos dados desejados é:

```javascript
var st
var resultado
var j
var dados = {}

st = msg.payload.toString()

resultado = st.substring(205);

resultado = resultado.split(/\s+/)

dados['volume1'] = Number(resultado[4])
dados['tc_volume'] = Number(resultado[5]) 
dados['volume2'] = Number(resultado[6])
dados['altura'] = Number(resultado[7])
dados['agua'] = Number(resultado[8])
dados['temperatura'] = Number(resultado[9])
dados['produto'] = resultado[3]
dados['capacidade'] = dados['volume1'] + dados['volume2']
dados['capacidade_pct'] = parseFloat((dados['volume1'] / dados['capacidade']) * 100).toFixed(2)

msg.payload = dados
msg.topic = resultado

return msg;
```

A imagem acima mostra mostra o retorno da requisição do tanque 01. Essa string possui um cabeçalho e no final retornar as informações requerida.
Explicando as variáveis do código acima:
+ st = recebe o retorno da requisição TCP convertida para string, usando o .toString().
+ resultado = posição da string que começa as informações do tanque, sem considerar o cabeçalho da mensagem.
+ resultado segunda linha = através da função `.split(/\s+/)` é criado um array com os dados retirando os `" " espaços`. Onde existe um espaço ele criar um objeto dentro do array.
+ dados = é um array com as informações desejadas.

O retorno da função é um array com as informações desejadas.

Após extraído as informações desejadas iremos enviar estes dados para o PIMS - OSIsoft através do PI Web API.

A configuração do envio dos dados para o PIMS é conforme abaixo:
<img src="https://github.com/dedynobre/lendo-valores-medidores-veeder-root/blob/master/imagens/img-04.png"/>

O código da função:

```javascript

var servidor = "localhost:444/piwebapi/";

var funcao1 = "streams/";
var funcao2 = "/value";

var webid = "A0EGyLsNdWwy0acdR72SBNFTgYD5QUgs26hGS1wBQVoJmcQxALmSClpw0ihAmHczVhQvwUFRVLUFVVEFGLTAxXEtJTlJPU1MgUEFSQUNBVFVcQVZBTsOHQURPXFBPU1RPIERFIENPTUJVU1RJVkVJU1xTSVNURU1BIFZFRURFUiBST09UfFRRNTA1fFZPTFVNRQ";

msg.url = servidor + funcao1 + webid + funcao2;
msg.method = "POST";

msg.payload = {
    
    "Value": msg.payload.volume1
}

return msg;

```

O envio de dados para o PI Web API é feito conforme descrição do manual e feita acima:
<img src="https://github.com/dedynobre/lendo-valores-medidores-veeder-root/blob/master/imagens/img-06.png"/>

Como vamos enviar dados o método deve ser o `post`. 
A url padrão já está definida que é `nome_do_servidor:porta/piwebapi`. 
A conexão é `stream/webid/value'.
O `webid` é o identificador do atributo do AF que irá receber os dados.
O `valor` vem da função anterior onde foi extraido os dados.

A configuração do node `http request` ficou da seguinte forma:</br>
<img src="https://github.com/dedynobre/lendo-valores-medidores-veeder-root/blob/master/imagens/img-05.png"/>

Sendo que a maioria das cofiguração ficaram em branco porque foram passadas pela função através do `msg`.

Desta forma os dados são enviados para o PIMS escrito no atributo do AF:
<img src="https://github.com/dedynobre/lendo-valores-medidores-veeder-root/blob/master/imagens/img-07.png"/>


> As informações são consultadas e enviadas ao PIMS a cada 20 segundos.

Uma vez que as informações estão sendo historizadas podemos visualizar as informações em tempo real:
<img src="https://github.com/dedynobre/lendo-valores-medidores-veeder-root/blob/master/imagens/img-08.png"/>


Ou também fazer consultas por período:

<div align="center">
  <img src="https://github.com/dedynobre/lendo-valores-medidores-veeder-root/blob/master/imagens/graf.gif" alt="conexoes-clientes">
</div>

## Conclusão

Este é um exemplo claro de aplicação de IIOT, onde podemos integrar um dispositivo que possui comunicação TCP e integrar com um historiador sendo possível enviar também para um banco de dados relacional e até mesmo um controlador PLC.
Desta forma podemos ter histórico de consumo de lubrificantes e/ou combustíveis.

Foi configurado envio automático´destas informações tanto para quem recebe quanto para quem fornece o produtos. Com isso houve ganho na dinamica do processo de compra pois os dados são enviados automaticamente e a coleta das informações são feitas em tempo real sem a necessidade de intervença de operadores. 

## Help

Caso precisem te ajuda ou tenham alguma sugestão, deixe seu comentário [Aqui](https://github.com/dedynobre/lendo-valores-medidores-veeder-root/issues).
