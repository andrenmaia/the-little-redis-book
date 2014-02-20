>  A versão em pt-br ainda está **EM CONSTRUÇÃO**

## Sobre ##
*The Little Redis Book* é um livro gratuito e introdutório sobre o Redis.

O livro foi escrito por [Karl Seguin](http://openmymind.net), com assistência de [Perry Neal](http://twitter.com/perryneal). A tradução foi realizada por [André Maia](http://anmaia.wordpress.com).

Se você gostou deste livro, talvez você também gostará de [The Little MongoDB Book](http://openmymind.net/2011/3/28/The-Little-MongoDB-Book/).

## Licença ##
O livro é distribuído gratuitamente por meio da licença [Attribution-NonCommercial 3.0 Unported license](<http://creativecommons.org/licenses/by-nc/3.0/legalcode>).


## Outras traduções ##

* [Inglês](https://github.com/karlseguin/the-little-redis-book)
* [Russo](https://github.com/kondratovich/the-little-redis-book)
* [Italiano](https://github.com/sandroconforto/the-little-redis-book) - [pdf](https://github.com/sandroconforto/the-little-redis-book/raw/master/book/redisIt.pdf)
* [Chinês](https://github.com/JasonLai256/the-little-redis-book)
* [Japonês](https://github.com/craftgear/the-little-redis-book/)
* [Espanhol](https://github.com/raulexposito/the-little-redis-book)

## Formatos ##
O livro foi escrito em  [Markdown](http://daringfireball.net/projects/markdown/)  e convertido para PDF usando [Pandoc](http://johnmacfarlane.net/pandoc/).

O template TeX faz uso do [Lena Herrmann's JavaScript highlighter](http://lenaherrmann.net/2010/05/20/javascript-syntax-highlighting-in-the-latex-listings-package).

Os formatos Kindle e ePub são provido utilizando o [Pandoc](http://johnmacfarlane.net/pandoc/).

## Geração dos livros ##
Os pacotes listados abaixo são para Ubuntu. Se você utiliza outro Sistema Operacional (SO) os nomes de distribuição devem ser similares.

### PDF

#### Dependências

Pacotes:

* `pandoc`
* `texlive-xetex`
* `texlive-latex-extra`
* `texlive-latex-recommended`

Você deve ter [algumas fontes](https://github.com/karlseguin/the-little-redis-book/blob/master/common/pdf-template.tex#L11) instaladas também.

Ou você pode trocá-las para outras se quiser. Considere que as fontes podem causar [problemas de geração/construção](https://github.com/karlseguin/the-little-redis-book/issues/26).

#### Construção

Execute `make pt-br/redis.pdf`.

### ePub

#### Dependências

Pacotes:

* `pandoc`

#### Construção

Execute `make pt-br/redis.pdf`.

### Mobi

#### Dependências

Pacotes:

* `pandoc`

Você também deve ter o [KindleGen](http://www.amazon.com/gp/feature.html?ie=UTF8&docId=1000765211) instalado.

#### Construção

Execute `make pt-br/redis.mobi`.

## Imagem de título ##
Um PSD da imagem de título está inclusa. A fonte utilizado é [Comfortaa](http://www.dafont.com/comfortaa.font).