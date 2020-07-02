
# Postman Tricks

- Conceitos e Testes de API
- Tipos de Requisições
- Pre-Request´s
- Variáveis de Ambiente
- Testes de Contrato
- Testes de Conteúdo 
- Automatizando os Testes com Newman
- Rodando testes Newman + Docker
- Macetes


## Conceitos e Testes de API
-   API é um conjunto de rotinas e padrões de programação para acesso a um aplicativo de software ou plataforma baseado na Web;
-   A sigla API refere-se ao termo em inglês “Application Programming Interface”, que significa em tradução para o português “Interface de Programação de Aplicativos”.
-   Estão entre a camada de testes de UI e Unitários;
-   Podem ser automatizados em paralelo com o desenvolvimento da API;
-   Facilitam a validação de múltiplos cenários;
-   Garantem que a estrutura do JSON de retorno esteja correta.

## Tipos de Requisições

- POST
- GET
- PUT
- DELETE

## Pre-Request´s

São requisições que vão ser executadas antes de qualquer outra, comparação direta com "OneTimeSetup" e "Setup" do NUnit. É possível configurar de dois modos:

- Pre-request da Collection (OneTimeSetup)
- Pre-request da Requisição (Setup)

Muito utilizada em testes que exigem token de autenticação. Exemplo de utilização:

    var teste ;
    var responseJSON;
    const echoPostRequest = {
      url: 'https://XXXXXX/core/connect/token',
      method: 'POST',
       header: {
            'Accept': 'application/json',
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization':'Basic XXXXXXX'
          },
      body: {
        mode: 'urlencoded',
        urlencoded:[
            {key: "grant_type", value: "XXXXXXXXX"},
            {key: "scope",      value: "XXXXXXXXX"}
            ]
        }
    };
    
     pm.sendRequest(echoPostRequest, function (err, res) {
        console.log(err ? err : res.json());
            if (err === null) {
                var responseJson = res.json();
                pm.environment.set('access_token', responseJson.access_token)
            }
     });
Este pre-request faz um post em uma rota que retorna o token de autenticação e posteriormente é setado como variável de ambiente na linha:  
>pm.environment.set('access_token', responseJson.access_token)

Também temos a opção de setar qual será a próxima rota desta collection que deve executar após a execução da atual:
>postman.setNextRequest('Postman Echo PUT')

## Variáveis de Ambiente
Pegando a variável de Ambiente
>pm.environment.get("variable_key");

Pegando a variável global
>pm.globals.get("variable_key");

Pegando a variável local
>pm.variables.get("variable_key");

Setando a variável de ambiente
>pm.environment.set("variable_key", "variable_value");

Setando a variável global
>pm.globals.set("variable_key", "variable_value");

Limpando a variável de ambiente
>pm.environment.unset("variable_key");

Limpando a variável global
>pm.globals.unset("variable_key");
## Testes de Contrato

Tem como objetivo garantir que o conteúdo fornecido não tenha sido modificado, poderá mostrar que tem permissão para validar se o contrato acordado foi ou não foi quebrado; deve validar se o esquema permanecerá o mesmo, assim como a integridade dos dados na comunicação entre cliente / servidor;
É possível validar se os dados continuam iguais, como os valores limites, uma restrição de valores recebidos, se uma estrutura foi ou não modificada etc.

Em poucas palavras, validação do tipo primitivo dos atributos.
Ex:
**RESPONSE**

    [
    {
    "codigoInstituicao":  1,
    "nomeInstituicao":  "Centro Universitário de Belo Horizonte",
    "siglaInstituicao":  "UniBH",
    "codigoEstabelecimentoLio":  "123",
    "caminhoImagemLogLio":  "x",
    "caminhoImagemIntLio":  "x"
    },
    {
    "codigoInstituicao":  3,
    "nomeInstituicao":  "Centro Universitário Una",
    "siglaInstituicao":  "Una",
    "codigoEstabelecimentoLio":  "123",
    "caminhoImagemLogLio":  "x",
    "caminhoImagemIntLio":  "x"
    }
    ]

   


**TESTES**
 
    var responseSchema = {
                  "type": "array",
                    "items": {
                    "type": "object",
                    "properties": {
                        "codigoInstituicao": {
                            "type": "number"
                        },
                        "nomeInstituicao": {
                            "type": "string"
                        },
                        "siglaInstituicao": {
                            "type": "string"
                        },
                        "codigoEstabelecimentoLio": {
                            "type": "string"
                        },
                        "caminhoImagemLogLio": {
                            "type": "string"
                        },
                        "caminhoImagemIntLio": {
                            "type": "string"
                        }
                    }
                    }       
    }
    
    // Get response data as JSON
    var jsonData = pm.response.json();
    
    // Test for response data structure
    pm.test('Validação da estrutura do JSON', function () {
        var validation = tv4.validate(jsonData, responseSchema);
        pm.expect(validation).to.be.true;
    });
Neste teste acima é validado o tipo de cada atributo, lembrando que é possível ter mais de um tipo de retorno. Neste caso utilize:
  >"type": "string|number"

## Testes de Conteúdo 
**RESPONSE**

    [
    {
    "codigoInstituicao":  1,
    "nomeInstituicao":  "Centro Universitário de Belo Horizonte",
    "siglaInstituicao":  "UniBH",
    "codigoEstabelecimentoLio":  "123",
    "caminhoImagemLogLio":  "x",
    "caminhoImagemIntLio":  "x"
    },
    {
    "codigoInstituicao":  3,
    "nomeInstituicao":  "Centro Universitário Una",
    "siglaInstituicao":  "Una",
    "codigoEstabelecimentoLio":  "123",
    "caminhoImagemLogLio":  "x",
    "caminhoImagemIntLio":  "x"
    }
    ]

   


**TESTES**
 
   

    responseJSON = JSON.parse(responseBody);
    
    tests["UNIBH - CodigoIES"] = responseJSON[0].codigoInstituicao === 1
    tests["UNIBH - CodEstabelecimento IES"] = responseJSON[0].codigoEstabelecimentoLio === "123"
    
    tests["UNA - CodigoIES"] = responseJSON[1].codigoInstituicao === 3
    tests["UNA - CodEstabelecimento IES"] = responseJSON[1].codigoEstabelecimentoLio === "123"
    
## Automatizando os Testes com Newman
Necessário instalar o NodeJs e abrir o cmd, executar o seguinte comando para instalar o newman.
>npm install -g newman

Rodando testes com newman:
>newman run examples/sample-collection.json

### Relatório HTML
Instalar a dependência 
>npm install -g newman-reporter-html 

Rodando os testes
>newman run /path/to/collection.json -r cli,html

Dentro da pasta da sua Collection será criada uma pasta com o nome "newman" e dentro dela vai existir um arquivo HTML em cada execução dos testes.

### Relatório XML (aba de testes no AzureDevOps)
Instalar a dependência 
>npm install -g newman-reporter-html 

Rodando os testes
>newman run /path/to/collection.json -r cli,junit

Dentro da pasta da sua Collection será criada uma pasta com o nome "newman" e dentro dela vai existir um arquivo XML em cada execução dos testes.

## Rodando testes Newman + Docker
Instale o Docker Desktop e abra o PowerShell, executar o comando abaixo para baixar a imagem que contém as dependências do NPM.
>docker pull postman/newman

Execute os testes da mesma forma do tópico anterior 
>docker run -t postman/newman run https://www.getpostman.com/collections/8a0c9bc08f062d12dcda


## Macetes
Sem nenhum no momento, volte mais tarde.


https://medium.com/better-practices/consumer-driven-contract-testing-using-postman-f3580dba5370
