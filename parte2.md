 Parte 2

Na Parte 1, fizemos uma primeira versão de uma página com conteúdo dinâmico, cujos dados estavam armazenados em Arrays.
As páginas dinâmicas foram geradas pelo PHP, que construiu o HTML e fez o envio dos conetúdos "prontos" para o cliente.

Agora, faremos alterações, no sentido de dotar nossas páginas com códigos JS que farão requisições e receberão respostas, e que farão a manipulação das mesmas, alterando as páginas.


## 1. Criando um "endpoint" PHP para cursos

Antes, fizemos com que a página cursos fosse construída completamente pelo PHP, que entregou o HTML pronto para o navegador.
Agora, vamos criar um script que será responsável por entregar uma lista de Cursos ou um único curso, em formato JSON.

JSON (JavaScript Object Notation) é um formato semi-estruturado de dados que possibilita a interoperabilidade entre sistemas, de tal forma que os dados sejam enviados contendo, também, alguma estrutura.

### 1.1 A função **json_encode()** do PHP

O PHP conta com uma fução chamada `json_encode()`, que é capaz de "serializar" dados, ou seja, transformar dados em estruturas próprias (Arrays ou Objetos) em texto, sem perder a sua estrutura.

Verifique o material da aula sobre JSON no SIGAA.

### 1.2 Criando o "endpoint" PHP que retornará dados de curso em JSON

Na pasta do seu projeto, crie uma pasta chamada **api**, e, dentro desta, um arquivo chamado `cursos.php`.
O script `cursos.php` será responsável por obter os dados de UM ou de TODOS OS CURSOS, e retornar os dados em formato JSON. Veja que, agora, não queremos retornar um HTML.

Conteúdo do arquivo `api/dadoscursos.php`
```php
<?php
//este script busca os dados de um curso ou de todos
//inclui o arquivo que contém os dados e as funções getCursos() e getCursoById()
  include_once('../dados.php');

//caso o Query String traga um parâmetro id
if(isset($_GET['id'])){
    //busca o Array com os dados de um curso específico, e codifica em JSON e escreve
    echo json_encode(getCursoPorId($_GET['id']));
}else{ // caso não traga id
    //busca o Array com os dados de todos os cursos, e codifica em JSON e escreve
    echo json_encode(getCursos());
  }
?>
```

Para testar a utilização do script, vá ao navegador, e teste o acesso direto a ele em **http://localhost/.../api/dadoscursos.php** (observe o caminho correto do seu projeto). Deverá ver algo como a imagem abaixo.

![Visualização do resultado do script api/dadoscursos.php](imgs/img11_roteiro.png)

Agora, para testar o retorno dos dados de um curso específico, vamos passar um parâmetro id no **Query String**. Para isso, acesse **http://localhost/.../api/dadoscursos.php?id=1** (observe o caminho correto do seu projeto). Deverá ver algo como a imagem abaixo (dados de apenas um curso).

![Visualização do resultado do script api/dadoscursos.php?id=1](imgs/img12_roteiro.png)

Com isso, conseguimos criar um script PHP, que retorna dados de cursos em formato JSON. E este script funciona tanto para retornar todos os cursos, como um curso específico. Este é um comportamento sobre o qual falaremos mais depois, mas que vai na direção daquilo que se deseja quando se desenvolve na forma de Web Services.

## 2. Alterando a página cursos.php, para que faça requisição Assíncrona

Vamos alterar a PÁGINA `cursos.php`. Trata-se daquela página que exibe os dados dos cursos. Originalmente, criamos esta página de forma que ela tem o seu HTML construído pelo PHP. Agora, desejamos fazer com que o JavaScript faça uma requisição assíncrona (AJAX), e receba os dados em JSON.

Remova o trecho PHP que fazia a montagem das linhas da tabela (dentro de tbody).

Seu arquivo `cursos.php` ficará assim:
```php
<?php
  include('cabecalho.html');
  include('dados.php');
?>
  <main class="container">
      <h2>Lista dos Cursos</h2>
      <table class="table table-striped">
          <thead>
              <th>Id</th>
              <th>Nome</th>
              <th>Semestres</th>
              <th>Coordenador</th>
          </thead>
          <tbody id="tabela-cursos">
              <!-- Aqui vão as linhas com os dados -->

          </tbody>
      </table>
  </main>
<?php
  include('rodape.html');
?>
```

Na pasta do seu projeto, crie uma pasta chamada **js**. Dentro dela, crie um arquivo chamado `cursos.js`. Este será o arquivo JavaScript onde serão desenvolvidas as funções para busca dos dados e manipulação dos resultados.

No arquivo `js/cursos.js`, começaremos criando uma função para o evento **onload** da página. Em outras palavras, estamos indicando o que deve ser executado quando a página terminar de ser carregada.

```javascript
window.onload = function(){
    console.log('terminou de carregar a página');
}
```

Para que a página utilize esse javascript, adicione em `cursos.php`, depois do `</main>`, a tag script, para que possa utilizar o cursos.js.

```php
...
    </table>
  </main>

<script src="js/cursos.js"></script>  
<?php
```

Isto faz com que, ao ser carregada, a página `cursos.php` chame o `js/cursos.js`. Ao completar o carregamento da página, o evento **onload** é executado, e a frase é escrita no console.

Para visualizar o console, acesse a página **cursos.php** do seu projeto no navegador, e tecle F12. Na aba console, você deverá ver uma mensagem assim:

![Visualização da mensagem no console](imgs/img13_roteiro.png)

Isto indica que o seu JavaScript está funcionando, ou seja, que ao terminar de carregar a página, o evento onload foi executado, e, consequentemente, uma mensagem foi escrita no console.

Uma vez que o JS está executando, vamos às requisições.

### 2.1. Criando uma função JS para requisição dos dados de Cursos

Já temos:
- Script php `api/dadoscursos.php` que retorna os dados dos cursos em JSON;
- Página `cursos.php`, com a tabela em branco;
- Arquivo `js/cursos.js` executando a função para onload.

Vamos criar uma função para fazer a requisição Assíncrona (AJAX) dos dados.

Em termos simples, queremos que a função JS faça:
1 - Requisição GET para `api/dadoscursos.php`;
2 - Como resposta a essa requisição, receberá dados de todos os cursos, em formato JSON;
3 - Os dados serão percorridos (como um Array);
4 - Para cada curso contido nos dados, criaremos uma linha na tabela HTML da página;

Para fazer a requisição, utilizaremos a fetch API.

No arquivo `js/cursos.js`:
```javascript
function buscaCursos(){
    //funcao que busca os dados em api/cursos.php e monta o HTML na página
    
    fetch('api/dadoscursos.php') //url sendo requisitada
        .then((resposta) => { //pega a resposta no formato json
            return resposta.json();
        })
        .then((dados) => {    //aquela resposta contem dados
            console.log(dados);  //exibe os dados no console
        }); 
    })
    .catch(err => console.error(err)); //caso dê erro, mostra no console
}
//as variaveis resposta e dados poderiam ter outros nomes, a seu gosto, mas assim fica bem explicativo
```

O que temos aqui, é uma requisição assíncrona, utilizando a fetch API. De maneira simplificada, ela é assíncrona, pois ela retorna uma PROMESSA **promise**. Isto quer dizer que não precisa esperar a resolução para fazer outras coisas na sequência. Tem-se ali a promessa de que retornará algo.

Altere o arquivo `js/cursos.js`, acrescentando uma chamada à função recem criada:

```javascript
window.onload = function(){
    console.log('terminou de carregar a página');
    buscaCursos();
}
```

Observe que, agora, além de exibir a mensagem de que a página foi carregada, o console exibirá os dados que retornaram da requisição, tal como se vê na figura abaixo:

![Visualização dos dados de curso no console](imgs/img14_roteiro.png)

Isto indica que houve uma requisição (verifique a aba Network - ou Rede do Navegador) para `api/dadoscursos.php`. Esta requisição retornou um conjunto de dados em JSON, e o seu JavaScript, na página, conseguiu capturá-lo.

O próximo passo é percorrer estes dados, e manipular o documento HTML, incluindo elementos e os dados na forma de linhas de uma tabela.

### 2.2. Manipulando os dados e exibindo no console com JS

Agora, nossa função de busca dos dados, será responsável por fazer a manipulação dos dados recebidos, e acrescentá-los à página HTML `cursos.php`.

Na página `cursos.php`, adicione um id ao elemento **tbody**. 
```html
<tbody id="tabelaCursos">
```

No arquivo `js/cursos.js`, faremos o parse (percorrer) dos dados JSON recebidos, cusando uma estrutura de repetição forEach.

```javascript
function buscaCursos(){
    //funcao que busca os dados em api/cursos.php e monta o HTML na página
    
    fetch('api/dadoscursos.php') //url sendo requisitada
        .then((resposta) => { //pega a resposta no formato json
            return resposta.json();
        })
        .then((dados) => {    //aquela resposta contem dados
            dados.forEach(curso => { //para cada curso contido em dados
                console.log(curso.nome); //exibe o nome do curso
            })
        }); 
    })
    .catch(err => console.error(err)); //caso dê erro, mostra no console
}
```

Estamos dizendo algo como: para cada item dos `dados`, chamando este item de `curso`, exiba no console o nome do curso (`curso.nome`).

Acessando o console em seu navegador, terá algo como:

![Visualização o nome de cada curso no console](imgs/img15_roteiro.png)

### 2.3. Exibindo os dados na Página com JS

Já conseguimos exibir os dados no console. Para exibir na página, precisaremos manipular os elementos HTML. Para isto, utilizaremos o DOM (Document Object Model). Com DOM, um documento HTML é visto como um conjunto de objetos estruturados de forma hierárquica, de tal forma que podemos recuperar ou incluir elementos, atributos, textos, em qualquer ponto de um documento.

Lembre-se que o elemento **tbody** de nossa tabela ganhou um ID. Utilizaremos desse ID para recuperar o elemento **tbody**, e, posteriomente, incluir conteúdos dentro dele.

Modifique a função buscaCursos() no arquivo `js/cursos.js`:

```javascript
function buscaCursos(){
    //funcao que busca os dados em api/cursos.php e monta o HTML na página

    //recupera o elemento com id #tabelaCursos, e guarda em uma variável com o mesmo nome
    let tabelaCursos = document.querySelector(#tabelaCursos);

    fetch('api/dadoscursos.php') //url sendo requisitada
        .then((resposta) => { //pega a resposta no formato json
            return resposta.json();
        })
        .then((dados) => {    //aquela resposta contem dados
            dados.forEach(curso => { //para cada curso contido em dados
                console.log(curso.nome); //exibe o nome do curso no console
                let textnome = document.createTextNode(curso.nome); //cria um nó de texto
                tabelaCursos.appendChild(textNome); //adiciona o nó de texto dentro do elemento #tabelaCursos
            })
        }) 
        .catch(err => console.error(err)); //caso dê erro, mostra no console
}
```

Acesse agora a página `cursos.php`, e verifique que os nomes dos cursos foram criados dentro do **tbody** da tabela.

A figura abaixo apresenta o resultado na página. À direita, no inspetor de códigos, é possível ver os conteúdos de texto gerados dentro do **tbody**.

![Inclusão dos nomes de curso na tabela](imgs/img16_roteiro.png)

No próximo passo, precisamos melhorar essa estrutura da tabela. Lembre-se que uma tabela HTML contém linhas (`<tr>`). As linhas contém células (`<td>`). E as células contém nós de texto (`textNode`).

Assim, precisamos fazer com que a função crie esta estrutura para cada curso. Iniciaremos criando uma linha com uma célula para cada nome de curso:

Portanto, o arquivo `js/cursos.js` ficará, agora:

```javascript
function buscaCursos(){
    //funcao que busca os dados em api/cursos.php e monta o HTML na página

    //recupera o elemento com id #tabelaCursos, e guarda em uma variável com o mesmo nome
    let tabelaCursos = document.queryString(#tabelaCursos);

    fetch('api/dadoscursos.php') //url sendo requisitada
        .then((resposta) => { //pega a resposta no formato json
            return resposta.json();
        })
        .then((dados) => {    //aquela resposta contem dados
            dados.forEach(curso => { //para cada curso contido em dados
                console.log(curso.nome); //exibe o nome do curso no console
                let tr = document.createElement("tr"); //cria um tr
                let tdnome = document.createElement("td"); //cria um td
                let textnome = document.createTextNode(curso.nome); //cria um nó de texto
                tdnome.appendChild(textnome); //adiciona o nó de texto dentro do td
                tr.appendChild(tdnome); //adiciona o td dentro do tr
                tabelacursos.appendChild(tr); //adiciona o tr dentro da tabela
            })
        }); 
    })
    .catch(err => console.error(err)); //caso dê erro, mostra no console
}
```

A imagem abaixo apresenta os nomes dentro da tabela, cada um em uma linha (observe que já tem cor de fundo para linhas alternadas). No inspetor de código é possível observar a estrutura de trs e tds criadas na tabela.

![Inclusão dos nomes de curso nas linhas da tabela](imgs/img17_roteiro.png)

Na sequência, é necessário criar mais células, uma para id, uma para semestres, e uma para o coordenador.

### 2.3.1. Adicionando as outras colunas (id, semestres e coordenador)

Da mesma forma como fizemos com o nome do curso, desejamos que cada linha da tabela contenha 4 colunas. Assim, precisamos incluir uma célula (`<td>`) antes daquela que criamos para o nome do curso. Considerando que os dados retornados em JSON trazem todos os dados de todos os cursos, devemos lembrar que:
* os dados retornados em json estão em um array;
* cada posição desse array contém um objeto, que representa cada curso;
* em um `foreach`, acessamos o nome do curso na propriedade `curso.nome`;

Portanto, temos disponíveis, ainda, `curso.id`, `curso.semestres` e `curso.coordenador`.

Vamos alterar a função `buscaCursos()` no arquivo `cursos.js`. Dentro do `foreach()` dessa função, teremos, então:

```javascript
...
                //DOM - Document Object Model
                let tr = document.createElement("tr");
                //cria uma célula para o id, dentro dela um nó de texto, e dentro do nó um valor - adiciona célula na linha
                let tdid = document.createElement("td");
                let textid = document.createTextNode(curso.id);
                tdid.appendChild(textid);
                tr.appendChild(tdid);

                //cria uma célula para o id, dentro dela um nó de texto, e dentro do nó um valor - adiciona célula na linha
                let tdnome = document.createElement("td");
                let textnome = document.createTextNode(curso.nome);
                tdnome.appendChild(textnome);
                tr.appendChild(tdnome);

                //cria uma célula para o id, dentro dela um nó de texto, e dentro do nó um valor - adiciona célula na linha
                let tdsemestres = document.createElement("td");
                let textsemestres = document.createTextNode(curso.semestres);
                tdsemestres.appendChild(textsemestres);
                tr.appendChild(tdsemestres);

                //cria uma célula para o id, dentro dela um nó de texto, e dentro do nó um valor - adiciona célula na linha
                let tdcoord = document.createElement("td");
                let textcoord = document.createTextNode(curso.coordenador);
                tdcoord.appendChild(textcoord);
                tr.appendChild(tdcoord);

                //adiciona a linha na tabela
                tabelacursos.appendChild(tr);
...
```

Com estas alterações, a página ficou conforme apresenta a figura:
![Inclusão dos nomes de curso nas linhas da tabela](imgs/img18_roteiro.png)

### 2.4. Melhorando a qualidade dos dados

Vimos que foi possível retornar, como JSON, os dados dos cursos. Todavia, percebemos que trouxemos apenas o **id** do coordenador. O nome deste professor está em outro conjunto de dados.

Podemos melhorar os dados a serem retornados em nosso JSON do `dadoscursos.php`, de tal forma que já tragam também o nome do professor coordenador.

Em outras palavras, queremos que o JSON a ser retornado traga elementos um pouco mais complexos.
Em vez de termos o JSON assim:

```javascript
[
...
    {
        "id":1,
        "nome":"Tecnologia em Sistemas para Internet",
        "semestres":6,
        "coordenador":5
    },
    {
        "id":2,
        "nome":"Bacharelado em Sistemas de Informação",
        "semestres":8,
        "coordenador":3
    },
    {
        "id":3,
        "nome":"Técnico em Informática Integrado",
        "semestres":6,
        "coordenador":2
    }
...
]
```

Queremos um JSON assim, onde a propriedade `coordenador` de cada curso contenha um **objeto** com os dados do professor.

```javascript
[
...
    {
        "id":1,
        "nome":"Tecnologia em Sistemas para Internet",
        "semestres":6,
        "coordenador":{
            "id":5,
            "nome":"Rafael Speroni"
        }
    },
    {
        "id":2,
        "nome":"Bacharelado em Sistemas de Informação",
        "semestres":8,
        "coordenador":{
            "id":3,
            "nome":"Daniel Anderle"
        }
    },
    {
        "id":3,
        "nome":"Técnico em Informática Integrado",
        "semestres":6,
        "coordenador":{
            "id":2,
            "nome":"Ângelo Frozza"
        }
    }
...
]
```
Desta forma, vamos alterar a função `getCursos()` em `api/dadoscursos.php`, de forma que ela retorne um Array modificado, mais completo, com os dados dos professores.

#### 2.4.1. Alterando a funçao getCursos()

Vamos alterar essa função, fazendo com que o Array de cursos a ser retornado passe a conter, em vez de apenas o **id** do coordenador, um **objeto** contendo os dados do professor coordenador.

No arquivo `dados.php`, vamos percorrer o Array `$cursos`, e alterar o valor da propriedade `coordenador` de cada curso. Vamos guardar nessa posição o Array do professor correspondente:

```php
function getCursos(){
    global $cursos; //a funcao passa a "conhecer" a variavel definida fora
    //percorre cada posição do array
    for($i=0; $i<sizeof($cursos); $i++){
        //busca o professor correspondente, recebe o Array
        $coordenador = getProfessor($cursos[$i]['coordenador']);
        //substitui o valor da posição coordenador pelo Array
        $cursos[$i]['coordenador'] = $coordenador;
    }    
    return $cursos;
}
```
Observe agora, no inspector, a requisição que é feita para `api/dadoscursos.php`, e veja que o JSON retornado na requisição está modificado, sendo que `coordenador` não tem apenas um **id**, e sim um **Objeto** que contem **id** e **nome** do professor.
![json alterado](imgs/img19_roteiro.png)

Podemos, então, alterar a função `buscaCursos()` em `cursos.js`, de forma a alterar o que é exibido na última coluna da tabela.

Antes, exibiamos `curso.coordenador`. Agora, essa propriedade contém um **Objeto**, que tem propriedade **id** e **nome**. Desejamos mostrar o nome, que está em `curso.coordenador.nome`:
```javascript
// cursos.js
// função buscaCursos()
...
                let tdcoord = document.createElement("td");
                let textcoord = document.createTextNode(curso.coordenador.nome);
                tdcoord.appendChild(textcoord);
                tr.appendChild(tdcoord);
...
```

E, então, a página `cursos.php` será apresentada conforme segue:
![Página com professores](imgs/img20_roteiro.png)

## 3. A página de professor.php

Tal qual fizemos com a página que lista os cursos, vamos alterar a página `professor.php`, de forma que o PHP a página seja carregada em sua estrutura básica, e os dados sejam carregados por meio de uma requisição AJAX, usando o `fetch()`.

### 3.1. Criação do link no nome do professor

Queremos fazer com que os nomes dos professores, na página `cursos.php`, sejam transformados em links. Todavia, temos que observar que, nesta versão da página os conteúdos da tabela foram gerados via Javascript, pela manipulação do DOM, ou seja, a criação dos elementos e seu posicionamento no documento foi feito na função Javascript `buscaCursos()` no `cursos.js`.

Queremos alterar o conteúdo da célula (`<td>`) correspondente ao nome do Coordenador do curso, acrescentando um link (`<a href>`).

```javascript
...
                let tdcoord = document.createElement("td");
                let linkcoord = document.createElement("a");
                let textcoord = document.createTextNode(curso.coordenador.nome);
                linkcoord.appendChild(textcoord);
                linkcoord.setAttribute("href", "#");
                linkcoord.setAttribute("onclick", "pagProfessor(this)");
                linkcoord.setAttribute("data-coord", curso.coordenador.id);
                tdcoord.appendChild(linkcoord);
                tr.appendChild(tdcoord);
...                
```
Descrevendo o código, temos:
* criação do elemento célula (`<td>`), que chamaremos `tdcoord`;
* criação do elemento link (`<a>`), que chamaremos `linkcoord`;
* criação do "nó de texto", que chamaremos `textcoord`, contendo o valor do nome a exibir;
* adiciona o `textcoord` dentro do `linkcoord`, fazendo com que crie algo como `<a>Rafael Speroni</a>`;
* adiciona um atributo `href="#"` ao `linkcood`, fazendo com que crie algo como `<a href="#">Rafael Speroni</a>`;
* adiciona um atributo `data-coord` com o valor do id do professor ao `linkcoord`, fazendo com que se crie algo como `<a href="#" data-coord="1">Rafael Speroni</a>`; 
* adiciona um atributo `onclick="buscaProfessor(this)"`, fazendo com que se crie algo como `<a href="#" data-coord="1" onclick="buscaProfessor(this)">Rafael Speroni</a>`;
* adiciona o link `linkcoord` à celula `tdcoord`;
* adiciona a célula `tdcoord` à linha da tabela `<tr>`;

Isto fará com que o nome do professor seja exibido na forma de uma **célula da tabela (td)**, que contém um link **<a>**, que aponta para a própria página **href="#"**, que ao ser clicado chama uma função **buscaProfessor(this)**, e que tem o id do professor armazenado no atributo **data-coord**;

Visualize sua página no navegador, e verifique se os links foram criados. Inspecione o elemento do nome, e verifique se o HTML foi gerado corretamente, conforme exemplo da figura abaixo:
![Página com professores com links](imgs/img21_roteiro.png)

Uma vez que tenha criado os links, eles ainda não funcionam. Observe no console javascript que o erro indica que a função `buscaProfessor()` não está definida. De fato, ainda não foi criada.
![Erro JS - função não definida](imgs/img22_roteiro.png)

### 3.2. Criação do script api/dadosprofessores.php

Este é um arquivo PHP que faz papel de backend. Ao receber uma requisição GET, com um id de professor, retornará um JSON com os dados do referido professor.

Na pasta `api`, crie o `dadosprofessores.php`
```php
<?php
include('../dados.php'); //para que acesse as funções e os dados

if(isset($_GET['id'])){
    echo json_encode(getProfessor($_GET['id']));
}else{
    echo "id não definido";
}
```
Neste primeiro momento, o script somente funcionará para os casos em que seja enviado um **id** de professor no **queryString**. Por exemplo, teríamos uma requisição para algo como `http://localhost/..../api/dadosprofessores.php?id=1`;

Portanto, o que fazemos é testar se há um valor de **id** enviado na **queryString**. Caso afirmativo, chamamos a função `getProfessor()`, que está definida em `dados.php`, e exibimos o seu retorno, codificando em JSON.

Desta forma, se testarmos o acesso direto ao link `http://localhost/..../api/dadosprofessores.php?id=1`, teremos como resposta o JSON com os dados de um professor, tal qual exemplificado na imagem que segue:
![dados de professor json](imgs/img23_roteiro.png)

Queremos, portanto, criar uma função javascript que faça uma requisição AJAX, usando fetch(), a este `api/dadosprofessores.php`, e que seja capaz de tratar sua resposta, apresentando os dados na página.

### 3.3. Criação do script js/professores.js

Agora, criaremos um arquivo javacript para tratar dos dados de professores.

Queremos criar uma função que:
* receba um id de professor;
* faça uma requisição HTTP para um PHP que responda com um JSON