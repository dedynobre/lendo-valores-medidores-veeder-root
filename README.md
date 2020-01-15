# Lendo valores de medição dos medidores Gilbarco Veeder-Root


## Objetivo

Os medidores Gilbarco Veeder-Root modelo [TLS4](https://www.gilbarco.com/br/produtos/medicao-e-monitoramento/tls4) possui interface web para gerenciamento.
O objetivo seria integrar, de forma automática, o sistema veeder-root com um banco de dados, no nosso caso seria o historiador PIMS - OSIsoft para ter histórico e também poder gerar relatórios de forma automática.

## Desenvolvimento

O sistema da Gilbarco possui um protocolo aberto de comunicação.
Conforme informações obtidas o equipamento responde uma requisição tcp na porta **10001**.
<img src="https://github.com/dedynobre/lendo-valores-medidores-veeder-root/blob/master/imagens/img-01.png"/>

 
