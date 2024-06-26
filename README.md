# Desafio Luiza Labs

<p align="center">
  <img src="https://github.com/jnerydesigner/luzia-labs-logistics/blob/main/assets/luizalabs.png" width="700" alt="Luiza Labs Logo" />
</p>





## Ferramentas:

- NestJS Framework do NodeJS

## Comandos a serem amplamente utilizados:

Se for utilizar o npm:

- Apagar o arquivo package.lock.json
- Rode o comando abaixo

```jsx
npm install
```

Se for utilizar o yarn (supondo que não tenha o yarn)

```jsx
npm install --global yarn

yarn install
```

Vamos rodar os testes:

```jsx
yarn test
ou
npm run test
```

Gerando um relatório coverage:

```jsx
yarn test:cov

ou

npm test:cov
```


## Running the app

```bash

# watch mode
npm run start:dev

# build
npm run build

# development
npm run start

# production mode
npm run start:prod
```

## Criei uma view para envio do arquivo:

- Criação de uma arquitetura simples em MVC, para facilitar o envio do arquivo.

```jsx
http://localhost:3333
```

# Dados o cenário abaixo:

A escolha do NodeJS com o NestJS, se deu por ele ter uma implementação mais rapida, dando enfase apenas ao desenvolvimento e não preocupando-se muito com o start do projeto.

### O cliente tem um banco de dados não sanitizado no formato:

"0000000001                             John Doe00000000010000000001     100.0020210909"

1 - Primeira parte da solução é o envio do arquivo, pois o cliente necessita que o arquivo no formato acima, seja transformado em um json dessa forma:

```json

{
	"user_id": 1,
	"name": "John Doe",
	"orders": [
		{
			"order_id": 1,
			"total": "100",
			"date": "2024-05-27",
			"products": [
				{
					"product_id": 1,
					"value": "100"
				}
			]
		}
	]
}

```

2 - Parte da Solução é capturar as linhas onde ocorrem os erros, claro que não peguei todos os erros.

```json
{
  "errors": [
    {
      "line": 2,
      "message": "Error in line 2."
    },
    {
      "line": 20,
      "message": "Error in line 20."
    },
    {
      "line": 54,
      "message": "Error in line 54."
    }
  ]
}

```

| ID USUÁRIO | NOME | ID PEDIDO | ID PRODUTO | VALOR DO PRODUTO | DATA DA COMPRA |
| --- | --- | --- | --- | --- | --- |
| 0000000001 | John Doe | 0000000001 | 0000000001 | 100.00 | 20210909 |

A Ideia Inicial será enviar o arquivo, utilizando streams do nodejs, executar uma transformação linha a linha utilizando a biblioteca readline.

A solução inicial, será transformar todos as linhas em um json no formato abaixo.

```jsx
{
  user_id: 1,
  name: 'John Doe',
  order_id: 1,
  date: '2024-05-27',
  product_id: 1,
  value: 100
}
```

A Ideia é chegar nesse resultado, utilizando de funções com responsablidade clara.

```jsx
## Essa é a função principal
async getLogisticsMap(file: Express.Multer.File);

## Função responsavel por processar o resultado das streams linha a linha
processLine(line: string)

## Função responsável por mapear os dados e transformalos na resposta solicitada
## do cliente
updateUserMap(
  userMap: Map<string, ILogisticsMapResponse>,
  entry: ILogisticsMap,
)

## Função responsável por encontrar a ocorrência da string e encontrar o final
## da ocorrência
extractUntilFirstZero(str: string)

## Função extra para transformar a data de formato '20240527' para "2024-05-27"
formatDate(dateString: string)

# Salva so dados em um relatório para posterior verificação
saveJsonToFile(filename: string, data: any, error: boolean)
```

Como criou-se a solução:

- [x]  Pega-se a ocorrencia de userId : const userId = *line*.slice(0, 10).trim();
- [x]  Pega o restante: let rest = *line*.slice(10).trim();
- [x]  Pega a data: const date = rest.slice(-8).trim();
- [x]  Pega o restante: rest = *line*.slice(10).trim();
- [x]  const userName = *this*.extractUntilFirstZero(rest);
- [x]  Pega o restante: rest = *line*.slice(10).trim();
- [x]  Pega a orderId: const orderId = rest.slice(0, 10).trim();
- [x]  Pega o restante: rest = *line*.slice(10).trim();
- [x]  Pega o productId: const productId = rest.slice(0, 10).trim();
- [x]  E o que restar é o value: parseFloat(rest.slice(10).trim());

Algumas resalvas, conforme a quantidade dados for aumentando, é recomendado uma solução de fila, exemplo é o redis com uso de uma ferramenta do NesJS chamada Bull para uso de fila para criar um arquivo json, já com o formato desejado, apesar de trabalhar com streams, nada mais justo que se precaver.

Pontos a serem abordados após a primeira implementação:

1. O Processo não pode ser terminado com um throw
2. Precisava salvar a linha que desse erro por algum motivo
3. Mapeei apenas um erro, podendo haver mais, a ideia é que passar o tempo a implementação torn-se mais robusta, a ponto de identificar o erro e criar uma reesposta em log (arquivo ou observabilidade).

Os teste conseguiram cobrir 100%, conforme imagem abaixo e arquivo coverage do projeto:

<p align="center">
  <img src="https://github.com/jnerydesigner/luiza-labs-logistics/blob/main/assets/test.png" width="700" alt="Relatório de Testes" />
</p>


## Envio via Insomnia

### Supondo que não tenha o insomnia instalado, o link para instalação está abaixo:

<a href="https://insomnia.rest/download" target="_blank" style="font-size:2rem">Insomnia</a>

<p align="center">
  <img src="https://github.com/jnerydesigner/luiza-labs-logistics/blob/main/assets/tela-insomnia.png" width="700" alt="Tela Principal Insomnia" />
</p>

<p align="center">
  <img src="https://github.com/jnerydesigner/luiza-labs-logistics/blob/main/assets/coluna_de_recursos.png" width="700" alt="Tela de Request" />
</p>

<p align="center">
  <img src="https://github.com/jnerydesigner/luiza-labs-logistics/blob/main/assets/coluna_de_manipulacao_da_request.png" width="700" alt="Tela de Manipulação das Requests" />
</p>

<p align="center">
  <img src="https://github.com/jnerydesigner/luiza-labs-logistics/blob/main/assets/coluna_de_resposta_da_requisicao.png" width="700" alt="Tela de Resposta da Requisição" />
</p>

## Arquitetura da Aplicação

### Diagrama de Sequência

<p align="center">
  <img src="https://github.com/jnerydesigner/luiza-labs-logistics/blob/main/out/01_controller/Diagrama%20de%20Sequ%C3%AAncia.png" width="700" alt="Diagrama de Sequencia" />
</p>

### Diagrama de Classe

<p align="center">
  <img src="https://github.com/jnerydesigner/luiza-labs-logistics/blob/main/out/02_arquetetura-service/Arquiterura%20de%20Classes.png" width="700" alt="Diagrama de Classe" />
</p>


### Diagrama de Json da Resposta

<p align="center">
  <img src="https://github.com/jnerydesigner/luiza-labs-logistics/blob/main/out/03_response_json/Resposta%20em%20Json.png" width="700" alt="Diagrama de Json da Resposta" />
</p>

### Diagrama de Json do Erro

<p align="center">
  <img src="https://github.com/jnerydesigner/luiza-labs-logistics/blob/main/out/04_error_json/Resposta%20em%20Json.png" width="700" alt="Diagrama de Json do Erro" />
</p>



## Developer

- Author - <a href="https://jandernery.seligadev.com.br/portfolio" target="blank">Jander Nery</a>
- Linkedin - <a href="https://www.linkedin.com/in/jander-nery" target="blank">Linkedin de Jander Nery</a>
- Youtube - <a href="https://www.youtube.com/channel/UCr5fGhSnZV7eqfDYcV09XOQ" target="blank">Youtube de Jander Nery</a>


## Stay in touch

- Author - [Kamil Myśliwiec](https://kamilmysliwiec.com)
- Website - [https://nestjs.com](https://nestjs.com/)
- Twitter - [@nestframework](https://twitter.com/nestframework)

## License

Nest is [MIT licensed](LICENSE).