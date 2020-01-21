# Lendo valores de medição dos medidores Gilbarco Veeder-Root


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




 
