---
title: Download de páginas Web com Python
layout: lesson
slug: download-de-paginas-web-com-python
date: 2012-07-17
translation_date: 2020-03-26 
authors:
- William J. Turkel
- Adam Crymble
reviewers:
- Jim Clifford
- Frederik Elwert
editors:
- Miriam Posner
translator:
- Bruno Gasparotto Ponne
translation-editor:
- 
translation-reviewer:
- Josir Cardoso Gomes
difficulty: 2
review-ticket: 360
activity:
topics: [python]
abstract: "Esta lição apresenta o conceito de *Localizador Uniforme de Recursos* (URL em inglês) e explica como usar o Python para fazer o download de uma página *Web* no seu disco local."
redirect_from:
original: working-with-web-pages
avatar_alt: A tall man next to a short woman
---


{% include toc.html %}



### Objetivos da Lição

Esta lição apresenta o conceito de *Localizador Uniforme de Recursos* (URL em inglês) e explica como usar o Python para fazer o download de uma página *Web* no seu disco local.

### Sobre URLs

Uma página *Web* é um arquivo hospedado em outro computador, conhecido como *servidor*. Quando um *site* é acessado, na realidade, o seu computador (o *cliente*) envia um pedido ao *servidor de hospedagem* por meio da rede e o servidor responde enviando uma cópia da página ao seu computador. Uma forma de acessar uma página por meio do seu navegador é seguir um link. É possível também colar ou digitar uma URL (localizador uniforme de recursos) diretamente no seu navegador. A URL informa ao seu navegador onde encontrar um recurso online, especificando o servidor, diretório e nome do arquivo a ser recuperado, bem como o tipo de *protocolo* que o servidor e o seu navegador utilizarão para troca de informações (como o HTTP, *protocolo de transferência de hipertexto*). A estrutura básica de uma URL é

```
protocol://host:port/path?query
```

Vejamos alguns exemplos:

``` xml
http://oldbaileyonline.org
```

O tipo mais básico de URL especifica apenas o protocolo e o domínio. Quando inserido em seu navegador, esse URL retornará a página principal do *site* [Old Bailey Online](https://www.oldbaileyonline.org). O pressuposto convencional é que a página principal em um determinado diretório se chamará *index*, geralmente `index.html`.

A URL pode incluir também um *número de porta* opcional. Sem entrar em muitos detalhes, o protocolo de rede em que se baseia a troca de informações na Internet permite que computadores se conectem de diferentes maneiras. Números de portas são utilizados para distinguir esses diferentes tipos de conexão. Uma vez que a porta padrão para HTTP é a 80, a seguinte URL é equivalente à anterior. 

``` xml
http://oldbaileyonline.org:80
```

Como você deve saber, geralmente há muitas páginas *Web* em um determinado *site*. Essas páginas são armazenadas em diretórios no servidor, e você pode especificar o caminho para uma página em particular. A página "About" para o *site The Old Bailey Online* tem a seguinte URL:

``` xml
http://oldbaileyonline.org/static/Project.jsp
```

Por fim, algumas páginas permitem inserir *queries*, termo em inglês que significa pedido, solicitação. O *site The Old Bailey Online*, por exemplo, foi desenvolvido de forma que é possível requisitar uma de suas páginas utilizando uma *query string* (conjunto de caracteres que contém uma solicitação). A seguinte URL acessará uma página de resultado de buscas por registros de julgamentos criminais contendo a palavra "arsenic".

``` xml
https://www.oldbaileyonline.org/search.jsp?form=searchHomePage&_divs_fulltext=arsenic&kwparse=and&_persNames_surname=&_persNames_given=&_persNames_alias=&_offences_offenceCategory_offenceSubcategory=&_verdicts_verdictCategory_verdictSubcategory=&_punishments_punishmentCategory_punishmentSubcategory=&_divs_div0Type_div1Type=&fromMonth=&fromYear=&toMonth=&toYear=&ref=&submit.x=0&submit.y=0
```

O fragmento sequinte ao "?" reresenta a *query*. Você pode aprender mais sobre como criar *queries* na lição [Downloading Multiple Records Using Query Strings][] (em inglês).

### Acessando URLs com Python

Como um historiador da era digital, você frenquentemente desejará utilizar dados mantidos em *sites* acadêmicos. Para acessar esses dados, você poderia abrir as URLs uma por uma e copiar e colar os conteúdos em um arquivo de texto. Alternativamente, você pode utilizar Python para, automaticamente, coletar e processar os dados. Para isso, é preciso aprender como abrir uma URL por meio do seu próprio código. A linguagem Python inclui uma série de padrões para fazer isso.

Como exemplo, vamos trabalhar com o tipo de arquivo que provavelmente você vai encontrar ao realizar uma pesquisa na área de História. Suponhamos que você esteja interessado nas relações raciais na Inglaterra do século XVIII . O *site* *The Old Bailey Online* é uma fonte rica de informações históricas e disponibiliza transcrições de julgamentos que ocorreram entre 1674 e 1913.

{% include figure.html filename="old-bailey.png" caption=" A homepage do site Old Bailey Online" %}

Para esse exemplo, utilizaremos a transcrição do julgamento de Benjamin Bowsey, um negro condenado por perturbar a paz durante os protestos de Gordon em 1780. A URL para o registro é

``` xml
http://www.oldbaileyonline.org/browse.jsp?id=t17800628-33&div=t17800628-33
```

Estudando a URL, podemos verificar algumas coisas. Primeiro, o *site* é programado em JSP (*JavaServer Pages*, uma linguagem de programação para *web* cujo resultado é um arquivo HTML), e é possível acessar registros de julgamentos individuais fazendo uso de *query strings*. Cada registro recebe um número único (*id=t* na URL), formado a partir da data da sessão de julgamento no formato (*AAAAMMDD*) e o número do julgamento naquela sessão do tribunal. Neste caso, *33*. Caso você troque as duas ocorrências de `33` por `34` no link acima, seu navegador o ecaminhará ao próximo julgamento. Infelizmente, nem todos os *sites* possuem URLs tão acessíveis e confiáveis quanto essa.

{% include figure.html filename="bowsey-trial-page.png" caption=" Transcrição do Julgamento de Trial Benjamin Bowsey, 1780" %}

Observe a página do julgamento de Benjamin Bowsey. Mais importante do que o conteúdo são os elementos presentes na página. Note o link [View as XML](http://www.oldbaileyonline.org/browse.jsp?foo=bar&path=sessionsPapers/17800628.xml&div=t17800628-33&xml=yes) na parte inferior. Esse link apresenta uma versão repleta de marcações no texto que podem ser úteis para certos tipos de pesquisa. O [documento original digitalizado](http://www.oldbaileyonline.org/images.jsp?doc=178006280084) do julgamento também pode ser acessado.

Agora vamos tentar abrir a página utilizando Python. Copie o seguinte programa no *Komodo Edit* e salve o arquivo como `abrir-webpage.py`. Quando você executar o programa, a página do julgamento será acessada, seus conteúdos serão lidos e copiados em uma string chamada webContent. Na sequência, os primeiros 300 caracteres serão exibidos no *painel de saída de comandos*. Utilize `Ferramentas -> Ferramentas de Programação -> Ver Código Fonte` no navegador Firefox para verificar que o código HTML da página é o mesmo que o seu programa acessou. Outros navegadores podem ter caminhos distintos para acessar o código fonte. Caso não consiga encontrar o caminho no seu navegador, tente utilizar um mecanismo de busca para encontrá-lo. (Consulte a biblioteca de referência do Python para aprender mais sobre [urllib](https://docs.python.org/3/library/urllib.html?highlight=urllib).)

``` python
# abrir-webpage.py

import urllib.request, urllib.error, urllib.parse

url = 'http://www.oldbaileyonline.org/browse.jsp?id=t17800628-33&div=t17800628-33'

response = urllib.request.urlopen(url)
webContent = response.read()

print(webContent[0:300])
```

Utilizando apenas essas cinco linhas de código, é possível obter resultados substanciais. Agora, vamos nos assegurar de que cada linha de código está clara e que você consegue distinguir os blocos que permitem ao programa realizar a tarefa que desejamos.

*url*, *response* e *webContent* são todas variáveis nomeadas por nós.

*url* contém a URL da página que queremos baixar. Neste exemplo, trata-se do julgamento de Benjamin Bowsey.

Na linha seguinte, chamamos a função `urlopen`, contida no módulo do Python chamado `urllib.py`, e solicitamos que ela acesse o *site* especificado na variável *url*. Em seguida, salvamos o resultado desse processo em uma variável chamada *response*. Essa variável contém agora uma versão aberta do *site* solicitado.

No próximo passo, utilizamos o método `read`, que já utilizamos anteriormente, para copiar os conteúdos do *site* em uma nova variável chamada *webContent*.

Assegure-se de que você consegue identificar as variáveis (3), o módulo (1), os métodos (2), e os parâmetros (1) antes de prosseguir.

No resultado do código acima, você perceberá alguns marcadores da linguagem HTML:

``` xml
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
<head>
	<title>Browse - Central Criminal Court</title>
	<meta http-equiv="content-type" content=
```

O conteúdo do teste fica na parte inferior da página. O que vemos aqui é o código HTML no início do documento. Isso não é exatamente o que precisamos para pesquisa histórica, mas não se preocupe: você vai aprender em breve a eliminar o excesso de marcadores e obter o conteúdo que procura.

### Salvando uma cópia local da página

Considerando o que você já sabe sobre atribuir conteúdo a um arquivo, é bem fácil modificar o programa acima para salvar o conteúdo da variável *webContent* em um arquivo local no seu computador. Copie o seguinte programa no *Komodo Edit*, salve-o como `salvar-webpage.py` e o execute. Utilizando o comando `File -> Open File` no Firefox , abra o arquivo criado no seu disco local  (`obo-t17800628-33.html`) para confirmar que a cópia salva é a mesma que a online.

``` python
# salvar-webpage.py

import urllib.request, urllib.error, urllib.parse

url = 'http://www.oldbaileyonline.org/browse.jsp?id=t17800628-33&div=t17800628-33'

response = urllib.request.urlopen(url)
webContent = response.read()

f = open('obo-t17800628-33.html', 'wb')
f.write(webContent)
f.close
```

Se é possível salvar um único arquivo dessa maneira, seria possível escrever um programa para baixar um conjunto de arquivos? Por exemplo, seria possível percorrer os identificadores de um conjunto de páginas e copiá-las para o seu computador? Sim. Aprenda como na lição [Downloading Multiple Files using Query Strings](https://programminghistorian.org/en/lessons/downloading-multiple-records-using-query-strings) (em inglês), que recomendamos depois que você tenha terminado as lições introdutórias dessa série.

### Leituras Sugeridas


-   Lutz, Mark. “Ch. 4: Introducing Python Object Types”, *Learning
    Python* (O’Reilly, 1999).
    
### Sincronização do Código

Para acompanhar futuras lições, é importante que você tenha os arquivos e programas corretos no seu diretório “programming-historian”. Ao final de cada lição, é possível baixar o arquivo zip “programming-historian” para ter certeza que você está utilizando o código correto.

-   programming-historian-1 ([zip](https://programminghistorian.org/assets/python-lessons1.zip))

### Sobre os autores

William J. Turkel é professor de História na University of Western Ontario.

Adam Crymble, University College London.

### Citação sugerida

William J. Turkel and Adam Crymble, "Downloading Web Pages with Python," The Programming Historian 1 (2012), https://doi.org/10.46430/phen0021.


