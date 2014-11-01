# Sobre este livro

## Licença
_The Little Redis Book_ está licenciado sobre _Attribution-NonCommercial 3.0 Unported license_. Você não deve pagar por este livro.

Você está livre para copiar, distribuir, modificar ou distribuir este livro. No entanto, eu peço que você sempre atribua o livro a mim, Karl Seguin, e não o use para propósitos comerciais.

Você pode ver o **texto completo** sobre **a licença** em <http://creativecommons.org/licenses/by-nc/3.0/legalcode>

## Sobre o Autor
Karl Seguin é um desenvolvedor com experiência em diversas áreas e tecnologias. Ele é um contribuinte ativo de projetos de __software open-source__ (código aberto), um escritor técnico e, ocasionalmente, um palestrante. Ele escreveu vários artigos, bem como algumas ferramentas, sobre o Redis. O Redis lidera o ranking e estatística de seu serviço gratuito para jogadores casuais: [mogade.com](http://mogade.com/).

Karl escreveu [The Little MongoDB Book (http://openmymind.net/2011/3/28/The-Little-MongoDB-Book/), o gratuito e popular livro sobre MongoDB.

Seu blog pode ser visto em <http://openmymind.net> e seus tweets em
[@karlseguin](http://twitter.com/karlseguin).

## Agradecimentos
Um agradecimento especial a [Perry Neal](https://twitter.com/perryneal) por me emprestar seus olhos, mente e paixão. Você me proveu inestimável auxílio. Obrigado.

## Última versão
A última versão deste livro é disponível em: <http://github.com/karlseguin/the-little-redis-book>

# Introdução
Ao longo dos últimos anos, as técnicas e ferramentas utilizadas para persistir e consultar dados têm crescido a um ritmo incrível. Embora seja seguro dizer que bancos de dados relacionais não vão a lugar algum, também podemos dizer que o ecossistema em torno dos dados nunca vai ser o mesmo.

De todas as novas ferramentas e soluções, para mim, Redis tem sido a mais emocionante. Por quê? Primeiro porque é inacreditavelmente fácil de aprender. Hora é a unidade certa para usar quando se fala sobre a quantidade de tempo que se leva para se acostumar com o Redis. Em segundo lugar, ele resolve um conjunto específico de problemas ao mesmo tempo que é bastante genérico. O que exatamente isso quer dizer? O Redis não tenta ser todas as coisas para todos os dados. À medida que você começa a conhecer o Redis, se tornará incrivelmente evidente o que ele faz e o que não pertence a ele. E quando isso acontecer, como um desenvolvedor, será uma grande experiência.

Enquanto você pode criar um sistema completo utilizando apenas o Redis, eu acho que a maioria das pessoas vão achar que ele suplementa sua solução de dados mais genérica - quer seja um banco de dados relacional tradicional, um sistema orientado a documentos, ou outra coisa. Esse é o tipo de solução que você usa para implementar funcionalidades específicas. Desse modo, ele é similar a um motor de indexação. Você não iria construir sua aplicação completa no Lucene. Mas quanto você precisa de uma boa busca, é uma experiência muito melhor - para ambos você e seu usuário. Claro, as similaridades entre Redis e motores de indexação terminam ai.

O objetivo deste livro é construir a base que você precisará para dominar o Redis. Iremos nos concentrar nas cinco estruturas de dados do Redis e olhar vários abordagens de moedagens de dados. Iremos também tocar em alguns detalhes administrativos e técnicas de _debugging_.

# Começando
Vamos aprender de forma diferente: alguns gostam de sujar suas mãos, alguns gostam de assistir videos, e alguns gostam de ler.

Nada o ajudará a entender o Redis mais do que realmente experimenta-lo. Redis é simples de instalar e vem com um _shell_ simples que nos dará tudo o que é necessário. Vamos gastar alguns minutos e colocá-lo para rodar em nossas máquinas.

## No Windows
O próprio Redis não suporta oficialmente o Windows, mas existem opções disponíveis. Você não as utilizaria em produção, mas eu nunca vivi nenhuma limitação durante o desenvolvimento.

Uma versão da Microsoft Open Technologies, Inc. pode ser encontrar em <https://github.com/MSOpenTech/redis>.Esta solução não está pronta para uso em sistema de produção.

Um outra solução, que está disponível há algum tempo, pode ser encontrada em <https://github.com/dmajkic/redis/downloads>. Você pode baixar a versão mais atual (que deve estar no inicio da lista). Extraia o arquivo _zip_ e, baseado na sua arquitetura, abra a pasta `64bit` ou `32bit.`

## No \*nix e MacOSX
Para usuários \*nix e Mac, compilá-lo a partir do código fonte é sua melhor opção. As instruções, juntamente com o número da versão mais recente, estão disponíveis em <http://redis.io/download>. No momento da escrita deste livro a última versão é 2.6.2; para instalar essa versão nós executaríamos:

    wget http://redis.googlecode.com/files/redis-2.6.2.tar.gz
    tar xzf redis-2.6.2.tar.gz
    cd redis-2.6.2
    make

(De forma alternativa, o Redis está disponível através de vários gerenciadores de pacotes. Por exemplo, usuários de MacOSX com Homebrew instalados pode simplesmente digitar `brew install redis`.)

Se você compilá-lo a partir do código fonte, as saídas binárias foram colocadas na pasta `src`. Navegue para a pastas `src` executando `cd src`.

## Executando e conectando no Redis
Se tudo funcionou, o executável do Redis deve estar disponível ao seu alcance. O Redis tem um punhado de arquivos executáveis. Nos concentraremos no servidor Redis e na interface de linha de comando do Redis (um cliente DOS-like). Vamos iniciar o servidor. No Windows, duplo clique em `redis-server`. No \*nix/MacOSX execute `./redis-server`.

Se você ler a mensagem de inicialização, você verá uma advertência de que o arquivo `redis.conf` não pode ser encontrado. Ao invés disso o Redis utilizará padrão internos, o que é suficiente para o que estamos fazendo.

Depois inicie o console do Redis por duplo clique no `redis-cli` (Windows) ou executando `./redis-cli` (\*nix/MacOSX). Esse irá conectar ao servidor local na porta padrão (6379).

Você pode testar se tudo está funcionando pelo digitando `info` na interface de linha de comando. Você verá (espero) um monte de pares chave-valor que fornecem um grande quantidade de informações sobre o estado interno do servidor.

Caso você esteja tendo problemas com o _setup_ acima eu sugiro que você procure ajuda no [grupo oficial de suporte do Redis](https://groups.google.com/forum/#!forum/redis-db).

# Redis Drivers
Como você vai aprender em breve, a API do Redis é melhor descrita como um conjunto explícito de funções. Isso significa que se você está usando a ferramenta de linha de comando, ou um _driver_ para a sua linguagem favorita, as coisas são muito similares. Então, você não deve ter qualquer problema acompanhando se você preferir trabalhar com uma linguagem de programação. Se você quiser, siga para a [página dos __clients__](http://redis.io/clients) e baixe o _driver_ apropriado.

# Capítulo 1 - O Básico

O que faz o Redis especial? Quais tipos de problemas ele resolve? O que os desenvolvedores devem estar atentos quando usá-lo? Antes de respondermos qualquer dessas questões, precisamos entender o que é o Redis.

Redis é frequentemente descrito como um armazenamento de chave-valor persistente em memória. Eu não acho que essa seja uma descrição exata. Redis mantém todos os dados em memória (mais sobre isso à frente), e os grava no disco para persistência, isso é muitos mais do um simples armazenamento chave-valor. É importante dar um passo além desses equívoco, caso contrário a sua perspectiva do Redis e dos problemas que ele resolve será muito restrita.

A realidade é que o Redis expõem cinco estruturas de dados diferentes, apenas uma destas é uma estrutura chave-valor. Entendendo estas cinco estruturas de dados, como funcionam, quais métodos expõem e o que se pode modelar é a chave para entender o Redis. Primeiramente, vamos pensar sobre o que significa expor estruturas de dados.

Se fôssemos aplicar este conceito de estruturas de dados para o mundo relacional, poderíamos dizer que bases de dados expõem uma única estrutura de dados - tabelas. Tabelas são complexas e flexíveis. Há muito que se possa modelar, armazenar ou manipular com tabelas. No entanto, sua natureza genérica não está livre de desvantagens. Especialmente porque nem tudo é tão simples, ou tão rápido, como deveria ser. E se, em vez de termos uma estrutura _one-size-fits-all_, usássemos estruturas mais especializadas? Pode haver algumas coisas que não possamos fazer (ou pelo menos, não possamos fazer muito bem), mas com certeza teríamos ganho em simplicidade e velocidade?

Usar um estrutura de dados específica para problemas específicos? Não é assim que nós codificamos? Você não usa uma _hashtable_ para cada pedaço de dado e nem para cada variável escalar. Para mim, isso define a abordagem do Redis. Se você está manipulando escalares, listas, _hashes_, ou _sets_, por que não armazená-los como escalares, listas, _hashes_ e _sets_? Por que verificar a existência de um valor deve ser mais complexo do que chamar `exists(key)` ou mais lento do que O(1) (busca de tempo constante que não diminuirá, independentemente de quantos itens existirem)?

# Os blocos de construção (the building blocks)

## Bases de dados

O Redis tem o mesmo conceito básico de uma base de dados que você já deve estar familiarizado. Uma base de dados contém um conjunto de dados. O caso de uso típico para um base de dados é agrupar todos os dados de uma aplicação e mantê-los separados de outras aplicações.

No Redis, bases de dados são simplesmente identificadas por um número, sendo a base de dados padrão iniciando no número `0`. Se você desejar selecionar uma base de dados diferente, você pode fazê-lo via o comando `select`. Na interface de linha de comando, digite `select 1`. O Redis deve responder uma mensagem `OK` e o seu _prompt_ deve alterar para algo como `redis 127.0.0.1:6379[1]>`. Se você quiser voltar para a base de dados padrão, entre com o comando `select 0` na interface de linha de comando.

## Comando, Chaves e Valores

Enquanto o Redis é mais do que apenas um banco de dados chave-valor, em seu núcleo, cada uma de suas cinco estruturas de dados tem pelo menos um chave e um valor. É importante entendermos chaves e valores antes de continuarmos.

Chaves são como você identifica pedaços de informações. Nós vamos lidar muito com chaves, mas agora, é o suficiente sabermos que um chave pode parecer com `user:leto`. Nesse caso, esperasse que essa seja uma chave que contenha informações sobre um usuário chamado `leto`. Os dois pontos não tem significado especial, no que diz respeito aos interesses do Redis, mas usar um separador é uma abordagem comum para organizar chaves.

Valores representam o data atual associado com a chave. Eles podem ser qualquer coisa. As vezes você armazenará _strings_, as vezes inteiros, as vezes você armazenará objetos (em JSON, XML ou algum outro formato). Na maioria das vezes, o Redis trata valores como um _array_ de _bytes_ e não se importa com o que eles são. Note que diferentes _drivers_ serializam objetos de formas diferentes (alguns deixam isso para você), então, neste livro só falaremos sobre _strings_, inteiros, e JSON.

Vamos sujar um pouco nossas mão. Entre com o comando a seguir:

	set users:leto "{name: leto, planet: dune, likes: [spice]}"

Essa é a anatomia básica de um comando do Redis. Primeiro nós temos o comando atual, neste caso `set`. Depois temos seus parâmetros. O comando `set` recebe dois parâmetros: a chave que estamos aplicando e o valor. Muitos comando, mas não todos, recebem uma chave (e quando eles o fazem, a chave é frequentemente o primeiro parâmetro). Você pode imaginar como recuperar o valor? Espero que você diga (mas não se preocupe se não você não tinha certeza!)

	get users:leto

Vá em frente e teste algumas outras combinações. Chaves e valores são conceitos fundamentais, e os comandos `get` e `set` são as formas mais simples de testá-los. Crie mais usuários, tente tipos diferentes de chaves, tente valores diferentes.

## Consultando

À medida que avançamos, duas coisas vão se tonando claras. Quando se fala sobre o Redis, as chaves são tudo e os valores não são nada. Ou, colocado de outra forma, o Redis não permite a você consultar um valor do objeto. Diante disso, não podemos encontrar o usuário(s) que vive no planeta `dune`.

Para muitos, isso causará alguma preocupação. Nós vivemos em um mundo onde a consulta de dados é tão flexível e poderosa que a abordagem do Redis parece primitiva e não pragmática. Não deixe que isso o perturbe muito. Lembre-se, o Redis não é uma solução _one-size-fits-all_. Haverá coisas que apenas não pertencem ao seu domínio (por causa das limitações de busca). Além disso, considere que em alguns casos você procurará novas formas para modelar seus dados.

Encontraremos mais exemplos concretos à medida que avançamos, mas é importante que entendamos esta funcionalidade básica do Redis. Isso nos ajuda a entender porque valores podem ser qualquer coisa - o Redis nunca precisa ler ou entender esses valores. Além disso, esse fato nos ajuda a fazer nossas mentes começar a pensar como modelar nesse novo mundo.


## Memória e persistência

Mencionamos anteriormente que o Redis é um banco de dados com persistência em memória. A respeito de persistência, por padrão, o Redis faz cópias instantâneas (_snapshots_) do banco de dados no disco baseado na quantidade de chaves que são alteradas. Você o configura de modo que se X número de chaves mudar, então salva o banco de dados a cada Y segundos. Por padrão, o Redis vai salvar o banco de dados a cada 60 segundos se 1000 ou mais chaves forem alteradas, ou no máximo em 15 minutos se 9 chaves ou menos forem alteradas.

Alternativamente (ou em adição as cópias instantâneas), o Redis pode rodar em _append mode_. Toda vez que um chave for alterada, um arquivo _append-only_ é alterado no disco. Em alguns casos é aceitável perder 60 segundos de informação útil, em troca de desempenho, quando houver alguma falha no _hardware_ ou no _software_. em alguns casos essa perda de dados não é aceitável. O Redis lhe dá a opção. No capítulo 6 veremos a terceira opção, que é o descarremento de persistência para um escravo.

No que diz respeito à memória, o Redis mantém todos os seus dados em memória. A implicação óbvia disso é o custo para executar o Redis: RAM ainda é a parte mais cara do _hardware_ para servidores.

Eu realmente sinto que alguns desenvolvedores perderam o tato em relação ao pequeno espaço que o dado ocupa. Todos os trabalhos completos de William Shakespeare ocupam grosseiramente 5.5MB de armazenamento. Quanto ao escalonamento, outras soluções tendem a ser IO- ou CPU-bound. A limitação (RAM ou IO) que vai exigir que você redimensione para mais máquinas realmente depende do tipo de dado e como você o armazena e consulta. A menos que você esteja armazenando arquivos de multimídia grandes no Redis, o aspecto _in-memory_ está provavelmente fora de questão. Para aplicações onde isso é um problema você provavelmente vai estar negociando entre limitações de IO (IO-bound) e limitações de memória (_memory bound_).


O Redis adicionou suporte para memória virtual. Contudo, essa funcionalidade foi vista como um falha (pelos próprios desenvolvedores do Redis) e o seu uso foi depreciado.

(Em um nota lateral, o arquivo de 5.5MB dos trabalhos completos de Shakespeare pode ser comprimidos para grosseiramente 2MB. O Redis não faz auto-compressão, mas, desde que trata valores como bytes, não há motivo para você gastar tempo de processamento para RAM através de compressão/descompressão dos seus próprios dados.)

## Colocando tudo junto

Tocamos em uma série de tópicos de alto nível. A última coisa que quero fazer antes de mergulhar de cabeça no Redis é colocar alguns desses tópicos juntos. Específicamente, limitações em busca, estruturas de dados e o jeito do Redis para armazenar os dados em memória.

Quando você coloca essas três coisas juntas, você acaba com algo maravilho: velocidade. Algumas pessoas pensam "Claro que o redis é rápido, tudo está na memória". Mas isso é apenas parte do que o faz rápido. O motivo real do Redis brilhar contra outras soluções são suas estruturas de dados especializadas.

Quão rápido? Isso depende da quantidade de coisas - quais comando você está usando, o tipo de dado, e assim por diante. Mas a performance do Redis tende a ser medida em dezenas de milhares, ou centenas de milhares de operações **por segundo**. Você pode rodar o comando `redis-benchmark` (presente na mesma pastas do `redis-server` e `redis-cli`) para testá-lo você mesmo.

Uma vez eu mudei um código que utilizava um modelo tradicionar para funcionar no Redis. Em um teste de carga eu levei 5 minutos para carregar utilizando o modelo relaciona. O mesmo exemplo demorou 150ms para carregar no Redis.

É importante entender estes aspectos do Redis porque isso impacta em como você interage com ele. Desenvolvedores com um _background_ em SQL frequentemente trabalham para minimizar o número de _round trips_ (idas e vindas) ao banco de dados. Esse é um bom conselho para qualquer sistema, incluíndo o Redis. Contudo, dado que estamos lidando com estrutura de dados simples, algumas vezes precisaremos ir ao servidor do Redis multiplas vezes para alcançar nosso objetivo. Alguns padrões de acesso a dados podem se sentir pouco naturais a primeira vista, mas na realidade estes tendem a ser um custo insignificante comparado ao desempenho bruto de performance que ganhamos.

## Neste capítulo

Apesar de quase não usar o Redis, nós cobrimos uma extensa gama de tópicos. Não se preocupe se alguma coisa não está clara - como por exemplo, busca de dados. No próximo capítulo vamos por a mão na massa e quaisquer dúvidas que você tiver serão respondidas.

Os tópicos importantes desse capítulo são:

* Chaves são _strings_ que identificam pedaços de dados (valores);

* Valores são arrays de bytes arbitrários que o Redis não se preocupa;

* O Redis expõe (e é implementado como) cinco estruturas de dados especializadas;

* Combinados, os tópicos acima, faz o Redis rápido e fácil de usar, mas não é adequado para todos os cenários.

# Capítulo 2 - Estruturas de Dados

É hora de dar uma olhada nas cinco estruturas de dados do Redis. Explicaremos o que é cada estrutura de dados, quais métodos estão disponíveis e quais tipos de características/dados você utilizaria.

As únicas construções do Redis que vimos até o momento são comandos, chaves e valores. Até agora, nada sobre estruturas de dados tem sido concreto. Quando utilizamos o comando `set`, como o Redis sabia qual estrutura de dados utilizar? Acontece que todo comando é específico para uma estrutura de dados. Por exemplo quando você utiliza o comando `set` você está armazenando o valor em uma estrutura de dados `string`. Quando você usa `hset` você está armazenando o dado em um `hash`. Dada a pequena dimensão do vocabulário do Redis, ele é bastante manuseável.

**[O site do Redis](http://redis.io/commands) tem uma ótima documentação de referência. Não há motivos para repetir o trabalho que eles já fizeram. Cobriremos apenas os comando mais importantes e necessários para entender o propósito das estruturas de dados.**

Não há nada mais importante do que se divertir e experimentar as coisas. Você sempre pode apagar todos os dados em sua base de dados informando o comando `flushdb`, então não seja tímido e tente fazer _doideiras_!

## Strings

Strings é a estrutura de dados mais básica disponível no Redis. Quando você pensa em um par chave-valor, você está pensando em strings. Não misture os nomes, como sempre, seu valor pode qualquer coisa. Eu prefiro chamá-los de "escalares", mas talvez apenas eu os chame assim.

Já vimos um caso de uso comum para strings, armazenando instâncias de objetos por chave. Isto é algo que você utilizará muito:

	set users:leto "{name: leto, planet: dune, likes: [spice]}"

Além disso, o Redis deixar você fazer algumas operações comuns. Por exemplo `strlen <key>` pode ser usada para obter o tamanho de um valor de uma chave; `getrange <key> <start> <end>` retorna o _range_ especificado de um valor; `append <key> <value>` junta o valor a um valor existente (ou o cria se ele não existir ainda). Vá em frente e tente os comandos a seguir. Isto foi o que eu obtive:

	> strlen users:leto
	(integer) 42

	> getrange users:leto 27 40
	"likes: [spice]"

	> append users:leto " OVER 9000!!"
	(integer) 54

Agora você deve estar pensando, isto é ótimo, mas não faz sentido. Você não pode pegar, de forma significativa, um _range_ de um JSON ou juntar um valor. Você está certo, a lição aqui é que alguns dos comandos, especialmente com a estrutura de dados string, só faz sentido quando dado um tipo específico de dado.

Anteriormente aprendemos que o Redis não se importa com seus valores. Na maioria das vezes isso é verdade. Contudo, alguns comandos de string são específicos para alguns tipos ou estruturas de valores. Como um exemplo vago, poderia ver o exemplo acima, os comandos `append` e `getrange`, sendo usados em alguma forma de serialização eficiente para armazenamento customizada. Para um exemplo mais concreto execute os comandos `incr`, `incrby`, `decr` e `decrby`. Esses incrementam ou decrementam os valores de uma string:

	> incr stats:page:about
	(integer) 1
	> incr stats:page:about
	(integer) 2

	> incrby ratings:video:12333 5
	(integer) 5
	> incrby ratings:video:12333 3
	(integer) 8


Como você pode imaginar, as strings do Redis são ótimas para _analytics_. Tente incrementar `users:leto` (um valor não inteiro) e veja o que acontece (você de obter um erro).

Um exemplo mais avançado são os comandos `setbit` e o `getbit`. Existe um [ótimo post](http://blog.getspool.com/2011/11/29/fast-easy-realtime-metrics-using-redis-bitmaps/) de como os usuários do Spool usam esses dois comandos para responder de forma eficiente a questão "quantos visitantes únicos tivemos hoje". Para 128 milhões de usuários um laptop gerou a resposta em menos de 50ms e ocupou apenas 16MB de memória.

Não é importante que você entenda como _bitmaps_ funcionam, ou como os usuários do Spool os utilizam, mas sim entender que as strings do Redis são mais poderosas do que elas inicialmente parecem ser. Ainda, os casos mais comum são aqueles que mostramos acima: armazenar objetos (complexos ou não) e contadores. Além disso, obter um valor pela chave é tão rápido que string são frequentemente utilizadas para armazenar dados de cache.

## Hashes

Hashes são um bom exemplo do porque chamar o Redis um banco de dados chave-valor não é muito preciso. Você sabe, em uma séries de formas, hashes são como strings. A diferença importante é que eles provem um nível extra de indireção: um campo. Portanto, os equivalentes de `set` e `get` para o hash são:

	hset users:goku powerlevel 9000
	hget users:goku powerlevel

Nós também podemos aplicar valores para múltiplos campos de uma só vez, obter múltiplos campos, obtém todos os campos e valores, listar todos os campos ou apagar um campo específico:

	hmset users:goku race saiyan age 737
	hmget users:goku race powerlevel
	hgetall users:goku
	hkeys users:goku
	hdel users:goku age

Como você pode perceber, hashes nos dão um pouco mais de controle sobre strings simples. Ao invés de armazenarmos um usuário como uma único valor serializado, nós podemos usar um hash para obter uma representação mais precisa. O benefício seria a habilidade de enviar tudo para o Redis e atualizar/apagar pedaços de dados específicos, sem ter que obter ou atualizar todos os valores.

Observar hashes a partir de uma perspectiva de objeto bem definido, como um usuário, é a chave para entender como eles funcionam. E é verdade que, por motivos de performance, controle mais granular pode ser útil. No entanto, no próximo capítulo vamos ver como hashes podem ser utilizados para organizar seus dados e fazer consultas mais práticas. Na minha opinião, isso é o que faz os hashes realmente úteis.

## Listas

Listas permitem que você armazene e manipule um array de valores para uma chave dada. Você pode adicionar valores a lista, obter o primeiro ou o último valor e manipular valores para um índice. Listas mantém sua ordem e utilizam operações eficientes baseadas em índice. Nós podemos ter uma lista de  `newusers` na qual registra os usuários mais novos para nosso site:

	lpush newusers goku
	ltrim newusers 0 49

Primeiro empurramos na lista, pela frente, um novo usuário, então nós aparamos a lista, assim ela contém apenas os últimos 50 usuários. Esse é um padrão comum. `ltrim` é uma operação O(N), onde N é o número de valores que estamos removendo. Neste caso, nós sempre aparamos após cada inserção, sendo assim, este terá sempre uma performance constante O(1) (porque N sempre será igual a 1). 

Esta é a primeira vez que estamos vendo um valor em uma chave referenciando um outro valor. Se quiséssemos obter os detalhes dos últimos 10 usuários, executaríamos a cominação a seguir:

	ids = redis.lrange('newusers', 0, 9)
	redis.mget(*ids.map {|u| "users:#{u}"})

No código acima temos um pouco de Ruby, que nos mostra os múltiplos _roundtrips_ que falamos anteriormente.

Claro, listas não são apenas boas para armazenar referências para outras chaves. Os valores podem ser qualquer coisa. Você poderia usar listas para armazenar _logs_ ou os passos de um usuário enquanto ele navega pelo site. Se você estiver construindo um jogo, você poderia usar uma lista para acompanhar as ações do usuário em uma fila. 

## Conjuntos (Sets)

Conjuntos (ou em inglês Sets) são usados para armazenar valores únicos. Estes provem um número de operações baseadas em conjuntos matemáticos, como _união_ (_unions_). Conjuntos não são ordenados, mas eles provem operações baseadas em valores de forma eficiente. Um lista de amigos é um exemplo clássico do uso de conjuntos:

	sadd friends:leto ghanima paul chani jessica
	sadd friends:duncan paul jessica alia

Independentemente da quantidade de amigos que um usuário tenha, podemos dizer de forma eficiente (O(1)) se o usuárioX é amigo do usuárioY or não:

	sismember friends:leto jessica
	sismember friends:leto vladimir

Além disso podemos ver se duas ou mais pessoas compartilham as mesmas amizades:

	sinter friends:leto friends:duncan

e ainda armazenar o resultado em uma nova chave:

	sinterstore friends:leto_duncan friends:leto friends:duncan

Conjuntos são ótimos para etiquetar (_tagging_) ou rastrear qualquer outra propriedade de um valor quando não faz nenhum sentido criar duplicidades (ou onde quisermos aplicar um conjunto de operações como uma intersecção e união).

## Conjunto ordenados (Sorted Sets)

A última e mais poderosa estrutura de dados são os conjuntos ordenados (sorted sets). Se _hashes_ são como strings com campos, então conjuntos ordenados são como conjuntos com um contador. O contador prove ordenação e ranqueamento dos conjuntos. Se nós quisermos uma lista ranqueada de amigos, nós podemos utilizar o comando a seguir:

	zadd friends:duncan 70 ghanima 95 paul 95 chani 75 jessica 1 vladimir

Para procurar quantos amigos `ducan` tem com uma pontuação 90 ou superior? 

	zcount friends:duncan 90 100

Que tal descobrir a pontuação de `chani`?

	zrevrank friends:duncan chani

Usamos `zrevrank` ao invés de `zrank` porque a ordenação padrão do Redis é do menor para o maior (mas neste caso estamos ordenando do maior para o menor). O caso de uso mais óbvio para conjuntos ordenados é um sistema de tabela de classificação. Na realidade, porém, qualquer coisa que você quiser ordenar por um valor inteiro, e ser capaz de manipular eficientemente baseado nessa ordenação, pode ser um bom candidato para um conjunto ordenado.

## Neste capítulo

Esta foi uma visão geral de alto nível das cinco estruturas de dados do Redis. Uma das coisa legais sobre o Redis é que você pode fazer mais do que você imagina inicialmente. Existem, provavelmente, jeitos de usar _strings_ e conjuntos ordenados que ninguém pensou ainda. Contanto que você entenda o caso de uso normal, você vai achar o Redis ideal para todos os tipos de problemas. Além disso, só porque o Redis expõe cinco estruturas de dados e vários métodos, não ache que você precisa usar todos eles. Não é incomum construir uma funcionalidade utilizando um vários comandos.

# Capítulo 3 - Aproveitando as estruturas de dados

No capítulo anterior falamos sobre as cinco estruturas de dados e demos alguns exemplos de quais problemas elas podem resolver. Agora é hora de darmos uma olhada em tópicos um pouco mais avançados, ainda comums, e também em alguns padrões de design.

## Notação Grande-O (Big O)

Ao longo deste livro fizemos referência a notação Grande-O (Big O), na forma O(n) ou O(1). A notação Grande-O é usada para explicar como alguma coisa se comporta dando um certo numero de elementos. No Redis, é usado para dizer-nos o quão rápido é um comando baseado no número de itens que estamos lidando.

A documentação do Redis nos diz a notação Grande-O para cada um dos comandos. Também nos diz quais fatores influenciam na performance. Vamos dar uma olhada em alguns exemplos.

O mais rápido que qualquer coisa pode ser é O(1), que é constante. Se nós estamos lidando com 5 itens ou 5 milhões de itens, você obterá a mesma performance. O comando `sismember`, que nos diz se um valor pertence a um conjunto, é O(1). `sismember` é um comando poderoso, e suas características de performance são a grande razão para isso. O número de comandos do Redis é O(1).

Logarítmico, ou O(log(N)), é o próximo mais rápido possível, porque ele precisa percorrer partições menores e menores. Usando esse tipo de abordagem de dividir e conquistar, um número grande de itens rapidamente é dividido em poucas iterações. `zadd` é um comando O(log(N)), onde N é o número de elementos presentes em um conjunto ordenado.

A segui temos os comandos lineares, ou O(N). A busca em uma coluna não indexada em uma tabela é uma operação O(N). Isso é feito utilizando o comando `ltrim`. Contudo, no caso do `ltrim`, N não é o número de elementos em uma lista, mas sim os elementos que estão sendo removidos. Usar `ltrim` para remover 1 item de uma lista de milhões será mais rápido do que usar `ltrim` para remover 10 itens de uma lista de milhares de itens. (Embora ambos os comandos serão provavelmente tão rápidos que você não seria capaz de cronometra-los).

`zremrangebyscore` que remove elementos de um conjunto ordenado com uma pontuação entre um valor mínimo e máximo tem uma complexidade de O(log(N)+M). Esse faz uma mistura. Lendo a documentação podemos ver que N é o número total de elementos no conjunto e M é o número de elementos a serem removidos. Em outras palavras, o número de elementos que serão removidos vai provavelmente ser mais significante, em termos de performance, do que o número total de elementos no conjunto.

O comando `sort`, que discutiremos em grande detalhe no próximo capítulo tem uma complexidade de O(N+M*log(M)). De sua característica de performance, você pode provavelmente dizer que este é um dos comandos mais complexos do Redis. 

Existem outros números de complexidades, os dois restante mais comuns são O(N^2) e O(C^N). Quanto maior o N, pior é sua performance relativa ao menor N. Nenhum dos comandos do Redis tem este tipo de complexidade.

É importante ressaltar que a notação Grande-O lida com o pior caso. Quando dizemos que alguma coisa leva O(N) para executar, nós na verdade podemos encontrá-lo imediatamente ou ele pode ser o último elemento possível.

## Pseudo Multi Key Queries

Uma situação comum que você vai enfrentar, é buscar o mesmo valor por chaves diferentes. Por exemplo, você pode querer obter um usuário por e-mail (para quando ele logar a primeira vez) e também por id (após ele ter logado). Uma solução horrível é duplicar o objeto de usuário em dois valores:

	set users:leto@dune.gov "{id: 9001, email: 'leto@dune.gov', ...}"
	set users:9001 "{id: 9001, email: 'leto@dune.gov', ...}"
	
Isso é ruim porque é um pesadelo gerenciar e ocupa o dobro de memória.

Seria legal se o Redis permitisse ligar uma chave a outra, mas isso não é possível (e provavelmente nunca será). Um dos principais motores do desenvilvimento do Redis é manter o código limpo e simples. A implementação interna de ligação de chaves (existem muitas coisas que podemos fazer com chaves que ainda não falamos) não vale a pena quando consideramos que o Redis já provê uma solução: hashes.

Usando um hash, podemos remover a necessidade de duplicacão:

	set users:9001 "{id: 9001, email: leto@dune.gov, ...}"
	hset users:lookup:email leto@dune.gov 9001


O que estamos fazendo é utilizar o campo como um pseudo índice secundário e referenciando um único objeto de usuário.

	get users:9001

Para obter um usuário por e-mail, executamos um `hget` seguido de um `get` (em Ruby):

	id = redis.hget('users:lookup:email', 'leto@dune.gov')
	user = redis.get("users:#{id}")

Isso é algo que você provavelmente vai acabar fazendo muitas vezes. Para mim, isso é onde os hashes realmente brilhão, mas não é um caso de uso óbvio até que você o veja.

## Referências e Índices

Já vimos alguns exemplos de um valor referenciando o outro. Vimos isso quando demos uma olhada no nosso exemplo de lista, e também vimos acima quando usamos hashes para fazer consultas um pouco mais fáceis. Isso nos resume que é essencial ter que gerenciar manualmente seus índices e referências entre valores. Para ser honesto, eu acho que podemos dizer que isso é meio chato, especialmente quando você considerar que terá que gerenciar/atualizar/deletar essas referências manualmente. Não há solução mágica para resolver esse problema no Redis.

Nós já vimos como os conjuntos (sets) são usados com frequência para implementar esse tipo de índice manual:

	sadd friends:leto ghanima paul chani jessica

Cada membro desse conjunto é uma referência para um valor string do Redis contendo detalhes do usuário atual. E se `chani` mudar o nome dela, ou deletar sua conta? Talvez faça sentido também manter a trilha inversa do relacionamento:

	sadd friends_of:chani leto paul

A manutenção é um custo à parte, se você é como eu, você pode se assustar com o custo do processamento e memória por ter esses valores a mais indexados. Nesta secão falaremos sobre as formas de reduzir o custo de performance quando temos round trips a mais (nós falamos brevemente sobre isso no primeiro capítulo).

Se você verdadeiramente está pensando sobre isso, saiba que, bancos de dados relacionais têm a mesma sobrecarga. Índices usam memória. Eles devem ser escaneados, ou de preferência procurados, e então o registro que corresponde a busca deve ser encontrado. A sobrecarga é nitidamente abstraída (e ele faz um monte de otimizações em termos de processamento para deixá-lo muito eficiente).

Novamente, ter que lidar manualmente com referências no Redis é desagradável. Mas qualquer preocupações iniciar que você tenha sobre performance ou implicações de memória devem ser testadas. Acho que você não vai considerar isso um problema.

## Round Trips and Pipelining

Já mencionamos que fazer viagens (trips) frequentes ao servidor é um padrão comum no Redis. Dado que é uma coisa que você vai fazer frequentemente, vale a pena dar uma olhada mais de perto nos recursos que temos para aproveitar ao máximo desse padrão.

Muitos comandos aceitam um ou mais conjuntos de parâmetros ou tem um _comando-irmão_ que leva vários parâmetro. Vimos o `mget` anteriormente, que aceita múltiplas chaves e retorna os valores:

	keys = redis.lrange('newusers', 0, 10)
	redis.mget(*keys.map {|u| "users:#{u}"})

Ou o comando  `sadd` que adiciona 1 ou mais membros a um conjunto:

	sadd friends:vladimir piter
	sadd friends:paul jessica leto "leto II" chani

O Redis também suporta _pipelining_. Normalmente quanto um cliente envia uma requisição ao Redis, ele aguarda pela resposta antes de enviar a próxima requisição. Com _pipelining_ você pode enviar um número de requisições sem aguardar pelas respostas. Isso reduz a sobrecarga de rede e pode resultar em um significante ganho de memória.

É importante notar que o Redis usará memória para enfileirar os comandos, então é uma boa idéia criar enviá-los em lote. O tamanho do lote que você vai usar dependerá de qual comando você escolher, e mais especificamente, do tamanho dos parâmetros. Mas, se você está executando comandos de aproximadamente 50 caracteres, você pode provavelmente criar lotes deles em milhares ou dezenas de milhares.

Exatamente como você executa comando dentro de um _pipeline_ vai variar de driver para driver. No Ruby você passa um bloco de métodos `pipelined`:

	redis.pipelined do
	  9001.times do
		redis.incr('powerlevel')
	  end
	end

Como você provavelmente deve imaginar, _pipeling_ pode realmente aumentar a velocidade na importação de um lote!

## Transações

Todo comando no Redis é atômico, incluindo aqueles que fazem múltiplas coisas. Adicionalmente, o Redis tem suporte para transações quando usamos múltiplos comandos.

Você pode não saber disso, mas o Redis é atualmente _single-threaded_, o que permite que todo comando seja garantidamente atômico. Enquanto um comando está em execução, nenhum outro comando será executado. (Falaremos brevemente sobre escalabilidade em um capítulo a frente.) Isso é particularmente útil quanto você considerar que alguns comandos fazem múltiplas coisas. Por exemplo:

`incr` é essencialmente um `get` seguido de um `set`.

`getset` aplica um novo valor e retorna o valor original.

`setnx` primeiro verifica se a chave existe, e apenas aplica o valor caso ela não exista.

Apesar desses comandos serem úteis, você inevitavelmente precisará executar múltiplos comandos como um grupo atômico. É possível fazer isso primeiro executando o comando `multi`, seguido por todos os comandos que você deseja executar como parte de uma transação, e finalmente informar `exec` para executar os comandos informados anteriormente ou `discard` para jogá-los fora. Quais garantias o Redis faz sobre transações?

* Os comandos serão executados em ordem;

* Os Comandos serão executados como uma única operação atômica (sem outros comandos de clientes serem executados pela metade);

* Que todos ou nenhum dos comandos serão executados.

Você pode, e deve, testar isso na interface de linha de comando. Note também que não existe razão para não combinar _pipelining_ e transações. 

	multi
	hincrby groups:1percent balance -9000000000
	hincrby groups:99percent balance 9000000000
	exec

Finalmente, o Redis permite especificar uma chave (ou chaves) para monitorar e condicionalmente aplicar uma transação se a chave(s) mudar. Isso é usado quanto você precisa obter valores e executar um código com base nesses valores, tudo em uma transação. Com o código acima, nós não seriamos capazes de implementar nosso próprio comando `incr` desde que eles são todos executados juntos uma vez que `exec` é chamado. Do código, nós não podemos fazer:

	redis.multi()
	current = redis.get('powerlevel')
	redis.set('powerlevel', current + 1)
	redis.exec()

Isso não é como as transações do Redis funcionam. Mas Se adicionássemos um `watch` para `powerlevel`,  poderíamos fazer:

	redis.watch('powerlevel')
	current = redis.get('powerlevel')
	redis.multi()
	redis.set('powerlevel', current + 1)
	redis.exec()

Se outro cliente alterar o valor de `powerlevel` após chamarmos o `watch` para ele, nossa transação falhará. Se o cliente não alterar o valor, o nosso `set` funcionará. Podemos executar esse código em _loop_ até que ele funcione.

## Keys Anti-Pattern

In the next chapter we'll talk about commands that aren't specifically related to data structures. Some of these are administrative or debugging tools. But there's one I'd like to talk about in particular: the `keys` command. This command takes a pattern and finds all the matching keys. This command seems like it's well suited for a number of tasks, but it should never be used in production code. Why? Because it does a linear scan through all the keys looking for matches. Or, put simply, it's slow.

How do people try and use it? Say you are building a hosted bug tracking service. Each account will have an `id` and you might decide to store each bug into a string value with a key that looks like `bug:account_id:bug_id`. If you ever need to find all of an account's bugs (to display them, or maybe delete them if they delete their account), you might be tempted (as I was!) to use the `keys` command:

	keys bug:1233:*

The better solution is to use a hash. Much like we can use hashes to provide a way to expose secondary indexes, so too can we use them to organize our data:

	hset bugs:1233 1 "{id:1, account: 1233, subject: '...'}"
	hset bugs:1233 2 "{id:2, account: 1233, subject: '...'}"

To get all the bug ids for an account we simply call `hkeys bugs:1233`. To delete a specific bug we can do `hdel bugs:1233 2` and to delete an account we can delete the key via `del bugs:1233`.


## In This Chapter

This chapter, combined with the previous one, has hopefully given you some insight on how to use Redis to power real features. There are a number of other patterns you can use to build all types of things, but the real key is to understand the fundamental data structures and to get a sense for how they can be used to achieve things beyond your initial perspective.

# Chapter 4 - Beyond The Data Structures

While the five data structures form the foundation of Redis, there are other commands which aren't data structure specific. We've already seen a handful of these: `info`, `select`, `flushdb`, `multi`, `exec`, `discard`, `watch` and `keys`. This chapter will look at some of the other important ones.

## Expiration

Redis allows you to mark a key for expiration. You can give it an absolute time in the form of a Unix timestamp (seconds since January 1, 1970) or a time to live in seconds. This is a key-based command, so it doesn't matter what type of data structure the key represents.

	expire pages:about 30
	expireat pages:about 1356933600

The first command will delete the key (and associated value) after 30 seconds. The second will do the same at 12:00 a.m. December 31st, 2012.

This makes Redis an ideal caching engine. You can find out how long an item has to live until via the `ttl` command and you can remove the expiration on a key via the `persist` command:

	ttl pages:about
	persist pages:about

Finally, there's a special string command, `setex` which lets you set a string and specify a time to live in a single atomic command (this is more for convenience than anything else):

	setex pages:about 30 '<h1>about us</h1>....'

## Publication and Subscriptions

Redis lists have an `blpop` and `brpop` command which returns and removes the first (or last) element from the list or blocks until one is available. These can be used to power a simple queue.

Beyond this, Redis has first-class support for publishing messages and subscribing to channels. You can try this out yourself by opening a second `redis-cli` window. In the first window subscribe to a channel (we'll call it `warnings`):

	subscribe warnings

The reply is the information of your subscription. Now, in the other window, publish a message to the `warnings` channel:

	publish warnings "it's over 9000!"

If you go back to your first window you should have received the message to the `warnings` channel.

You can subscribe to multiple channels (`subscribe channel1 channel2 ...`), subscribe to a pattern of channels (`psubscribe warnings:*`) and use the `unsubscribe` and `punsubscribe` commands to stop listening to one or more channels, or a channel pattern.

Finally, notice that the `publish` command returned the value 1. This indicates the number of clients that received the message.


## Monitor and Slow Log

The `monitor` command lets you see what Redis is up to. It's a great debugging tool that gives you insight into how your application is interacting with Redis. In one of your two redis-cli windows (if one is still subscribed, you can either use the `unsubscribe` command or close the window down and re-open a new one) enter the `monitor` command. In the other, execute any other type of command (like `get` or `set`). You should see those commands, along with their parameters, in the first window.

You should be wary of running monitor in production, it really is a debugging and development tool. Aside from that, there isn't much more to say about it. It's just a really useful tool.

Along with `monitor`, Redis has a `slowlog` which acts as a great profiling tool. It logs any command which takes longer than a specified number of **micro**seconds. In the next section we'll briefly cover how to configure Redis, for now you can configure Redis to log all commands by entering:

	config set slowlog-log-slower-than 0

Next, issue a few commands. Finally you can retrieve all of the logs, or the most recent logs via:

	slowlog get
	slowlog get 10

You can also get the number of items in the slow log by entering `slowlog len`

For each command you entered you should see four parameters:

* An auto-incrementing id

* A Unix timestamp for when the command happened

* The time, in microseconds, it took to run the command

* The command and its parameters

The slow log is maintained in memory, so running it in production, even with a low threshold, shouldn't be a problem. By default it will track the last 1024 logs.

## Sort

One of Redis' most powerful commands is `sort`. It lets you sort the values within a list, set or sorted set (sorted sets are ordered by score, not the members within the set). In its simplest form, it allows us to do:

	rpush users:leto:guesses 5 9 10 2 4 10 19 2
	sort users:leto:guesses

Which will return the values sorted from lowest to highest. Here's a more advanced example:

	sadd friends:ghanima leto paul chani jessica alia duncan
	sort friends:ghanima limit 0 3 desc alpha

The above command shows us how to page the sorted records (via `limit`), how to return the results in descending order (via `desc`) and how to sort lexicographically instead of numerically (via `alpha`).

The real power of `sort` is its ability to sort based on a referenced object. Earlier we showed how lists, sets and sorted sets are often used to reference other Redis objects. The `sort` command can dereference those relations and sort by the underlying value. For example, say we have a bug tracker which lets users watch issues. We might use a set to track the issues being watched:

	sadd watch:leto 12339 1382 338 9338

It might make perfect sense to sort these by id (which the default sort will do), but we'd also like to have these sorted by severity. To do so, we tell Redis what pattern to sort by. First, let's add some more data so we can actually see a meaningful result:

	set severity:12339 3
	set severity:1382 2
	set severity:338 5
	set severity:9338 4

To sort the bugs by severity, from highest to lowest, you'd do:

	sort watch:leto by severity:* desc

Redis will substitute the `*` in our pattern (identified via `by`) with the values in our list/set/sorted set. This will create the key name that Redis will query for the actual values to sort by.

Although you can have millions of keys within Redis, I think the above can get a little messy. Thankfully `sort` can also work on hashes and their fields. Instead of having a bunch of top-level keys you can leverage hashes:

	hset bug:12339 severity 3
	hset bug:12339 priority 1
	hset bug:12339 details "{id: 12339, ....}"

	hset bug:1382 severity 2
	hset bug:1382 priority 2
	hset bug:1382 details "{id: 1382, ....}"

	hset bug:338 severity 5
	hset bug:338 priority 3
	hset bug:338 details "{id: 338, ....}"

	hset bug:9338 severity 4
	hset bug:9338 priority 2
	hset bug:9338 details "{id: 9338, ....}"

Not only is everything better organized, and we can sort by `severity` or `priority`, but we can also tell `sort` what field to retrieve:

	sort watch:leto by bug:*->priority get bug:*->details

The same value substitution occurs, but Redis also recognizes the `->` sequence and uses it to look into the specified field of our hash. We've also included the `get` parameter, which also does the substitution and field lookup, to retrieve bug details.

Over large sets, `sort` can be slow. The good news is that the output of a `sort` can be stored:

	sort watch:leto by bug:*->priority get bug:*->details store watch_by_priority:leto

Combining the `store` capabilities of `sort` with the expiration commands we've already seen makes for a nice combo.

## Scan

In the previous chapter, we saw how the `keys` command, while useful, shouldn't be used in production. Redis 2.8 introduces the `scan` command which is production-safe. Although `scan` fulfills a similar purpose to `keys` there are a number of important difference. To be honest, most of the *differences* will seem like *idiosyncrasies*, but this is the cost of having a usable command.

First amongst these differences is that a single call to `scan` doesn't necessarily return all matching results. Nothing strange about paged results; however, `scan` returns a variable number of results which cannot be precisely controlled. You can provide a `count` hint, which defaults to 10, but it's entirely possible to get more or less than the specified `count`.

Rather than implementing paging through a `limit` and `offset`, `scan` uses a `cursor`. The first time you call `scan` you supply `0` as the cursor. Below we see an initial call to `scan` with an pattern match (optional) and a count hint (optional):

    scan 0 match bugs:* count 20

As part of its reply, `scan` returns the next cursor to use. Alternatively, scan returns `0` to signify the end of results. Note that the next cursor value doesn't correspond to the result number or anything else which clients might consider useful.

A typical flow might look like this:

    scan 0 match bugs:* count 2
    > 1) "3"
    > 2) 1) "bugs:125"
    scan 3 match bugs:* count 2
    > 1) "0"
    > 2) 1) "bugs:124"
    >    2) "bugs:123"

Our first call returned a next cursor (3) and one result. Our subsequent call, using the next cursor, returned the end cursor (0) and the final two results. The above is a *typical* flow. Since the `count` is merely a hint, it's possible for `scan` to return a next `cursor` (not 0) with no actual results. In other words, an empty result set doesn't signify that no additional results exist. Only a 0 cursor means that there are no additional results.

On the positive side, `scan` is completely stateless from Redis' point of view. So there's no need to close a cursor and there's no harm in not fully reading a result. If you want to, you can stop iterating through results, even if Redis returned a valid next cursor.

There are two other things to keep in mind. First, `scan` can return the same key multiple times. It's up to you to deal with this (likely by keeping a set of already seen values). Secondly, `scan` only guarantees that values which were present during the entire duration of iteration will be returned. If values get added or removed while you're iterating, they may or may not be returned. Again, this comes from `scan`'s statelessness; it doesn't take a snapshot of the existing values (like you'd see with many databases which provide strong consistency guarantees), but rather iterates over the same memory space which may or may not get modified.

In addition to `scan`, `hscan`, `sscan` and `zscan` commands were also added. These let you iterate through hashes, sets and sorted sets. Why are these needed? Well, just like `keys` blocks all other callers, so does the hash command `hgetall` and the set command `smembers`. If you want to iterate over a very large hash or set, you might consider making use of these commands. `zscan` might seem less useful since paging through a sorted set via `zrangebyscore` or `zrangebyrank` is already possible. However, if you want to fully iterate through a large sorted set, `zscan` isn't without value.

## In This Chapter

This chapter focused on non-data structure-specific commands. Like everything else, their use is situational. It isn't uncommon to build an app or feature that won't make use of expiration, publication/subscription and/or sorting. But it's good to know that they are there. Also, we only touched on some of the commands. There are more, and once you've digested the material in this book it's worth going through the [full list](http://redis.io/commands).

# Chapter 5 - Lua Scripting

Redis 2.6 includes a built-in Lua interpreter which developers can leverage to write more advanced queries to be executed within Redis. It wouldn't be wrong of you to think of this capability much like you might view stored procedures available in most relational databases.

The most difficult aspect of mastering this feature is learning Lua. Thankfully, Lua is similar to most general purpose languages, is well documented, has an active community and is useful to know beyond Redis scripting. This chapter won't cover Lua in any detail; but the few examples we look at should hopefully serve as a simple introduction.

## Why?

Before looking at how to use Lua scripting, you might be wondering why you'd want to use it. Many developers dislike traditional stored procedures, is this any different? The short answer is no. Improperly used, Redis' Lua scripting can result in harder to test code, business logic tightly coupled with data access or even duplicated logic.

Properly used however, it's a feature that can simplify code and improve performance. Both of these benefits are largely achieved by grouping multiple commands, along with some simple logic, into a custom-build cohesive function. Code is made simpler because each invocation of a Lua script is run without interruption and thus provides a clean way to create your own atomic commands (essentially eliminating the need to use the cumbersome `watch` command). It can improve performance by removing the need to return intermediary results - the final output can be calculated within the script.

The examples in the following sections will better illustrate these points.

## Eval

The `eval` command takes a Lua script (as a string), the keys we'll be operating against, and an optional set of arbitrary arguments. Let's look at a simple example (executed from Ruby, since running multi-line Redis commands from its command-line tool isn't fun):

    script = <<-eos
      local friend_names = redis.call('smembers', KEYS[1])
      local friends = {}
      for i = 1, #friend_names do
        local friend_key = 'user:' .. friend_names[i]
        local gender = redis.call('hget', friend_key, 'gender')
        if gender == ARGV[1] then
          table.insert(friends, redis.call('hget', friend_key, 'details'))
        end
      end
      return friends
    eos
    Redis.new.eval(script, ['friends:leto'], ['m'])

The above code gets the details for all of Leto's male friends. Notice that to call Redis commands within our script we use the `redis.call("command", ARG1, ARG2, ...)` method.

If you are new to Lua, you should go over each line carefully. It might be useful to know that `{}` creates an empty `table` (which can act as either an array or a dictionary), `#TABLE` gets the number of elements in the TABLE, and `..` is used to concatenate strings.

`eval` actually take 4 parameters. The second parameter should actually be the number of keys; however the Ruby driver automatically creates this for us. Why is this needed? Consider how the above looks like when executed from the CLI:

    eval "....." "friends:leto" "m"
    vs
    eval "....." 1 "friends:leto" "m"

In the first (incorrect) case, how does Redis know which of the parameters are keys and which are simply arbitrary arguments? In the second case, there is no ambiguity.

This brings up a second question: why must keys be explicitly listed? Every command in Redis knows, at execution time, which keys are going to needed. This will allow future tools, like Redis Cluster, to  distribute requests amongst multiple Redis servers. You might have spotted that our above example actually reads from keys dynamically (without having them passed to `eval`). An `hget` is issued on all of Leto's male friends. That's because the need to list keys ahead of time is more of a suggestion than a hard rule. The above code will run fine in a single-instance setup, or even with replication, but won't in the yet-released Redis Cluster.

## Script Management

Even though scripts executed via `eval` are cached by Redis, sending the body every time you want to execute something isn't ideal. Instead, you can register the script with Redis and execute it's key. To do this you use the `script load` command, which returns the SHA1 digest of the script:

    redis = Redis.new
    script_key = redis.script(:load, "THE_SCRIPT")

Once we've loaded the script, we can use `evalsha` to execute it:

    redis.evalsha(script_key, ['friends:leto'], ['m'])

`script kill`, `script flush` and `script exists` are the other commands that you can use to manage Lua scripts. They are used to kill a running script, removing all scripts from the internal cache and seeing if a script already exists within the cache.

## Libraries

Redis' Lua implementation ships with a handful of useful libraries. While `table.lib`, `string.lib` and `math.lib` are quite useful, for me, `cjson.lib` is worth singling out. First, if you find yourself having to pass multiple arguments to a script, it might be cleaner to pass it as JSON:

    redis.evalsha ".....", [KEY1], [JSON.fast_generate({gender: 'm', ghola: true})]

Which you could then deserialize within the Lua script as:

    local arguments = cjson.decode(ARGV[1])

Of course, the JSON library can also be used to parse values stored in Redis itself. Our above example could potentially be rewritten as such:

      local friend_names = redis.call('smembers', KEYS[1])
      local friends = {}
      for i = 1, #friend_names do
        local friend_raw = redis.call('get', 'user:' .. friend_names[i])
        local friend_parsed = cjson.decode(friend_raw)
        if friend_parsed.gender == ARGV[1] then
          table.insert(friends, friend_raw)
        end
      end
      return friends

Instead of getting the gender from specific hash field, we could get it from the stored friend data itself. (This is a much slower solution, and I personally prefer the original, but it does show what's possible).

## Atomic

Since Redis is single-threaded, you don't have to worry about your Lua script being interrupted by another Redis command. One of the most obvious benefits of this is that keys with a TTL won't expire half-way through execution. If a key is present at the start of the script, it'll be present at any point thereafter - unless you delete it.

## Administration

The next chapter will talk about Redis administration and configuration in more detail. For now, simply know that the `lua-time-limit` defines how long a Lua script is allowed to execute before being terminated. The default is generous 5 seconds. Consider lowering it.

## In This Chapter

This chapter introduced Redis' Lua scripting capabilities. Like anything, this feature can be abused. However, used prudently in order to implement your own custom and focused commands, it won't only simplify your code, but will likely improve performance. Lua scripting is like almost every other Redis feature/command: you make limited, if any, use of it at first only to find yourself using it more and more every day.

# Chapter 6 - Administration

Our last chapter is dedicated to some of the administrative aspects of running Redis. In no way is this a comprehensive guide on Redis administration. At best we'll answer some of the more basic questions new users to Redis are most likely to have.

## Configuration

When you first launched the Redis server, it warned you that the `redis.conf` file could not be found. This file can be used to configure various aspects of Redis. A well-documented `redis.conf` file is available for each release of Redis. The sample file contains the default configuration options, so it's useful to both understand what the settings do and what their defaults are. You can find it at <http://download.redis.io/redis-stable/redis.conf>.

Since the file is well-documented, we won't be going over the settings.

In addition to configuring Redis via the `redis.conf` file, the `config set` command can be used to set individual values. In fact, we already used it when setting the `slowlog-log-slower-than` setting to 0.

There's also the `config get` command which displays the value of a setting. This command supports pattern matching. So if we want to display everything related to logging, we can do:

	config get *log*

## Authentication

Redis can be configured to require a password. This is done via the `requirepass` setting (set through either the `redis.conf` file or the `config set` command). When `requirepass` is set to a value (which is the password to use), clients will need to issue an `auth password` command.

Once a client is authenticated, they can issue any command against any database. This includes the `flushall` command which erases every key from every database. Through the configuration, you can rename commands to achieve some security through obfuscation:

	rename-command CONFIG 5ec4db169f9d4dddacbfb0c26ea7e5ef
	rename-command FLUSHALL 1041285018a942a4922cbf76623b741e

Or you can disable a command by setting the new name to an empty string.

## Size Limitations

As you start using Redis, you might wonder "how many keys can I have?" You might also wonder how many fields can a hash have (especially when you use it to organize your data), or how many elements can lists and sets have? Per instance, the practical limits for all of these is in the hundreds of millions.


## Replication

Redis supports replication, which means that as you write to one Redis instance (the master), one or more other instances (the slaves) are kept up-to-date by the master. To configure a slave you use either the `slaveof` configuration setting or the `slaveof` command (instances running without this configuration are or can be masters).

Replication helps protect your data by copying to different servers. Replication can also be used to improve performance since reads can be sent to slaves. They might respond with slightly out of date data, but for most apps that's a worthwhile tradeoff.

Unfortunately, Redis replication doesn't yet provide automated failover. If the master dies, a slave needs to be manually promoted. Traditional high-availability tools that use heartbeat monitoring and scripts to automate the switch are currently a necessary headache if you want to achieve some sort of high availability with Redis.

## Backups

Backing up Redis is simply a matter of copying Redis' snapshot to whatever location you want (S3, FTP, ...). By default Redis saves its snapshot to a file named `dump.rdb`. At any point in time, you can simply `scp`, `ftp` or `cp` (or anything else) this file.

It isn't uncommon to disable both snapshotting and the append-only file (aof) on the master and let a slave take care of this. This helps reduce the load on the master and lets you set more aggressive saving parameters on the slave without hurting overall system responsiveness.

## Scaling and Redis Cluster

Replication is the first tool a growing site can leverage. Some commands are more expensive than others (`sort` for example) and offloading their execution to a slave can keep the overall system responsive to incoming queries.

Beyond this, truly scaling Redis comes down to distributing your keys across multiple Redis instances (which could be running on the same box, remember, Redis is single-threaded). For the time being, this is something you'll need to take care of (although a number of Redis drivers do provide consistent-hashing algorithms). Thinking about your data in terms of horizontal distribution isn't something we can cover in this book. It's also something you probably won't have to worry about for a while, but it's something you'll need to be aware of regardless of what solution you use.

The good news is that work is under way on Redis Cluster. Not only will this offer horizontal scaling, including rebalancing, but it'll also provide automated failover for high availability.

High availability and scaling is something that can be achieved today, as long as you are willing to put the time and effort into it. Moving forward, Redis Cluster should make things much easier.

## In This Chapter

Given the number of projects and sites using Redis already, there can be no doubt that Redis is production-ready, and has been for a while. However, some of the tooling, especially around security and availability is still young. Redis Cluster, which we'll hopefully see soon, should help address some of the current management challenges.

# Conclusion

In a lot of ways, Redis represents a simplification in the way we deal with data. It peels away much of the complexity and abstraction available in other systems. In many cases this makes Redis the wrong choice. In others it can feel like Redis was custom-built for your data.

Ultimately it comes back to something I said at the very start: Redis is easy to learn. There are many new technologies and it can be hard to figure out what's worth investing time into learning. When you consider the real benefits Redis has to offer with its simplicity, I sincerely believe that it's one of the best investments, in terms of learning, that you and your team can make.
