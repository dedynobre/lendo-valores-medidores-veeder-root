
## Objetivo

Os medidores Gilbarco Veeder-Root modelo [TLS4](https://www.gilbarco.com/br/produtos/medicao-e-monitoramento/tls4) possui interface web para gerenciamento.
O objetivo seria integrar, de forma automática, o sistema veeder-root com um banco de dados, no nosso caso seria o historiador PIMS - OSIsoft para ter histórico e também poder gerar relatórios de forma automática.

## Desenvolvimento

O sistema da Gilbarco possui um protocolo aberto de comunicação.
Conforme informações obtidas o equipamento responde uma requisição tcp na porta **10001**.
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
 
No bloco de função temos o seguinte código:
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

Depois desta conversão a string fica no formato da segunda imagem mostrada acima.
