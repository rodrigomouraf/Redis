# Redis

* [Motivação](#Motivacao)
* [Instalação no Windows](#install)
* [Manipulando Valores](#manipulando-valores)
* [Inserindo Dados](#inserindo-dados)
* [Otimizando Buscas](#otimizando-buscas)
* [Trabalhando com Hashes](#trabalahndo-com-hashes)
* [Sessões](#sessoes)
* [Manipulações de Maneira Atômica](#manipulacoes-de-maneira-atomica)
* [Inserindo e Retirando Valores](#inserindo-retirando-valores)
* [Utilizando Coleção de Boolean](#utilizando-colecoes-boolean)
* [Listas](#listas)
* [Filas](#filas)
* [Conclusao](#conclusao)

## <a name='Motivacao'></a> Motivação

Redis é um banco de dados que armazena os dados na forma de chave-valor.

Quando utilizamos um banco sql precisamos varrer todo o banco para trazer o resultado de algumas respostas. Lembrando que consultas em um banco sql podem ser lentas e quando o banco é indexado os inserts passam a ser lentos.

Quando utilizamos o Redis uma chave fica associada a um valor, dessa forma podemos obter um valor de uma forma muito mais rápida.

#

## <a name='install'></a> Instalação no Windows

### Link para instalação

<https://github.com/MSOpenTech/redis/releases/download/win-3.0.500/Redis-x64-3.0.500.msi>

#

## <a name='manipulando-valores'></a> Manipulando valores

Criando valores

```!/bin/bash
    set total_de_cursos 105 
```

Recuperando valores

```!/bin/bash
    get total_de_cursos
```

Deletando valores

```!/bin/bash
    Del total_de_cursos
```

#

## <a name='inserindo-dados'></a> Inserindo dados

Criando estrutura de busca para sorteios megasena:

```!/bin/bash
    SET resultado:17-05-2015:megasena "2, 15, 18, 28, 32"
```

```!/bin/bash
    SET resultado:10-05-2015:megasena "4, 16, 19, 23, 28, 43"
```

Armazenando múltiplos valores:

```!/bin/bash
    MSET resultado:03-05-2015:megasena "1, 3, 17, 19, 24, 26" resultado:22-04-2015:megasena "15, 18, 20, 32, 37, 41" resultado:15-04-2015:megasena "10, 15, 18, 22, 35, 43"
```

## <a name='otimizando-buscas'></a> Otimizando buscas

Buscando todas as chaves

```!/bin/bash
    KEYS *
```

Buscando todas as consultas que contenham resultado:

```!/bin/bash
    KEYS "resultado:*"
```

Buscando todas as keys do mês 05:

```!/bin/bash
    KEYS "resultado:*-05-2015*"
```

outra forma de fazer essa busca seria:

```!/bin/bash
    KEYS "resultado:??-05-????*"
```

Garantindo o retorno de um valor para a consulta.

Vamos analisar os registros abaixo:

```text
"resultado:10-05-2015:megasena"
"resultado:22-04-2015:megasena"
"resultado:15-04-2015:megasena"
"resultado:17-05-2015:megasena"
"resultado:03-05-2015:megasena"
```

Digamos que queremos garantir a existência do zero na consulta do dia do resultado, então a busca pode ficar:

```!/bin/bash
KEYS resultado:0?-05-2015:megasena
```

Agora, vamos analisar outra demanda, e se quisermos trazer um dia que tenha 7 e tenha 3, então nossa consulta pode ser montada dessa forma:

```!/bin/bash
KEYS resultado:?[37]-05-2015:megasena
```

Outro exemplo bacana seria se nós quiséssemos trazer apenas os dias 15 e 17, dos resultados da megasena, independente dos meses e anos:

```!/bin/bash
KEYS resultado:1[57]-??-????
```

## <a name='trabalahndo-com-hashes'></a> Trabalhando com Hashes

Até o momento verificamos como tratar os valores como strings soltas, mas e se quiséssemos inserir valores com mais algumas informações? E se nós quisermos adicionar a quantidade de ganhadores da megasena para o sorteio que saiu em determinado dia?

A partir de agora vamos começar a trabalhar com essa forma de estrutura de dicionário de informações.

### Inserindo dados com hashes

Para inserir dados em formato de hashe teremos que inserir um H antes do set, e adicionar um parâmetro para o campo de busca do resultado, ficando assim:

```!/bin/bash
HSET resultado:24-05-2015:megasena "numeros"  "13, 17, 19, 25, 28, 32"
```

```!/bin/bash
HSET resultado:24-05-2015:megasena "ganhadores"  "23"
```

### Inserindo múltiplos valores em um hashe

Assim como temos o MSET temos também o HMSET, esse comando permite que seja inserido vários valores quando utilizamos o hashe

```!/bin/bash
HMSET resultado:24-05-2015:megasena numeros "13, 17, 19, 25, 28, 32" ganhadores 23
```

### Buscando dados com hashes

Para buscar dados em formato de hashe teremos que inserir um H antes do get, e precisamos também chamar o parâmetro do item requirido, ficando assim:

```!/bin/bash
HGET resultado:24-05-2015:megasena "numeros"
```

```!/bin/bash
HGET resultado:24-05-2015:megasena "ganhadores"
```

### Recuperando todos os valores dentro de um hashe

Podemos utilizar o comando HGETALL para retornar todos os valores dentro de uma chave.

```!/bin/bash
HGETALL resultado:24-05-2015:megasena
```

### Deletando valores dentro de hashes

Para deletar valores dentro de uma hashe devemos usar o comando HDEL e passar o parâmetro do identificador do valor.

Caso desejado remover a hashe inteira, então devemos utilizar o comando DEL normalmente passando a chave principal.

```!/bin/bash
HDEL resultado:24-05-2015:megasena "numeros"
```

Removendo o registro:

```!/bin/bash
DEL resultado:24-05-2015:megasena
```

## <a name='sessoes'></a> Sessões

### Motivação

Para não precisarmos compartilhar informações de sessão entre servidores podemos compartilhar as informações no banco de dados.

Outra vantagem é que por ser um banco chave-valor o acesso as informações acabam sendo bastante rápidas.

Criando sessão

```!/bin/bash
HMSET "sessao:usuario:1675" "nome" "guilherme" "total_de_produtos" 3
```

Definindo a expiração de sessão em 30 minutos:

30 minutos = 60segundos * 30 = 1800segundos

```!/bin/bash
EXPIRE "sesseao:usuario:1675" 1800
```

Verificar quanto tempo falta para expirar a sessão:

```!/bin/bash
TTL "sessao:usuario:1675"
```

## <a name='manipulacoes-de-maneira-atomica'></a> Manipulações de maneira atômica

### Motivação

Digamos que o analista de dados da empresa queira ver como está a aceitação de uma página web.

Como poderíamos fazer para que cada usuário que acessasse essa página fosse registrado um incremento no número de registro?

Lembrando que enquanto estamos com o site online podem existir várias pessoas acessando a página que desejamos estudar e chamar um GET key e inserir um incremento no valor de retorno trazido pela consulta não garante que outras pessoas já tenham acessado a página enquanto estamos manipulando a informação.

Para garantir a atomicidade das informações seguem os comandos a serem utilizados, tanto para incremento, quanto para decremento:

Criando um registro:

```!/bin/bash
SET pagina:/contato:25-05-2015 1 
```

Incrementando um registro

```!/bin/bash
INCR pagina:/contato:25-05-2015
```

Decrementando um registro

```!/bin/bash
DECR pagina:/contato:25-05-2015
```

## <a name='inserindo-retirando-valores'></a> Inserindo e retirando valores

### Motivação

Vamos analisar como podemos somar e subtrair valores utilizando o Redis, sem precisar manipular um set dos seus dados

Inserindo e atribuindo um valor inteiro a uma compra:

```!/bin/bash
INCRBY compras:25-05-2015:valor 12
```

Retirando um valor inteiro de uma compra:

```!/bin/bash
DECRBY compras:25-05-2015:valor 12
```

Inserindo ou retirando um valor decimal na compra:

Para inserir:

```!/bin/bash
INCRBYFLOAT compras:25-05-2015:valor 12.5
```

Para retirar:

```!/bin/bash
INCRBYFLOAT compras:25-05-2015:valor -12.5
```

## <a name='utilizando-colecoes-boolean'></a> Utilizando coleção de boolean

### Motivação

Digamos que queiramos saber quem acessou nosso site em determinado dia, como fazer para ter acesso a informação por id de usuário?

Digamos que desejamos saber quantas pessoas acessaram meu site em determinado dia, como fazer para ter essa resposta rapidamente?

Ou talvez ainda mais interessante, Quantas pessoas acessaram meu site em mais de um dia? Quem são essas pessoas?

O Redis já contém uma estrutura otimizada para nós, essa estrutura nos permite setarmos diversos bits. Essa estrutura se chama BITSET.

#

Criando registros com BITSET:

No exemplo abaixo iremos criar um registro com os parâmetros:

SETBIT dia_de_acesso id_usuario true_or_falseo

```!/bin/bash
SETBIT acesso:25-05-2015 15 1
SETBIT acesso:25-05-2015 32 1
SETBIT acesso:25-05-2015 46 1
SETBIT acesso:25-05-2015 11 1
```

Verificando as respostas dos id_usuari

```!/bin/bash
GETBIT acesso:25-05-2015 46 = 1
GETBIT acesso:25-05-2015 2 = 0
```

Agora vamos inserir mais alguns registros de acesso com outras datas, vamos fazer isso para verificarmos didaticamente como utilizarmos o BITCOUNT e BITOP que nos ajudará a responder a uma pergunta como: Quantas pessoas acessaram meu site no dia 25-06-2015, ou quantas pessoas acessaram meu site no dia 26-06-2015 e no dia 25-06-2015?

```!/bin/bash
SETBIT acesso:26-06-2015 1 1
SETBIT acesso:26-06-2015 2 1
SETBIT acesso:26-06-2015 3 1
```

```!/bin/bash
SETBIT acesso:27-06-2015 1 1
SETBIT acesso:27-06-2015 3 1
```

Verificando quantas pessoas acessaram meu site no dia 26-06-2015

```!/bin/bash
BITCOUNT acesso:26-06-2015
```

Verificando quantas pessoas acessaram meu site nos dias 25 e 26

```!/bin/bash
BITOP AND acesso:25-e-26-05-2015 acesso:26-06-2015 acesso:27-06-2015 
```

Verificando quantas pessoas acessaram meu site em pelo menos um dos dias 26 ou 27.

```!/bin/bash
BITOP OR acesso:26-ou-27-06-2015 acesso:26-06-2015 acesso:27-06-2015
```

## <a name='listas'></a> Listas

### Motivação

Já entrou em algum site e viu algo como as 5 notícias mais recentes, ou como no Twitter as últimas mensagens que aparecem por ordem cronológicas.

Vamos entender agora como manipular listas e limita-las.

No exemplo a seguir, vamos criar uma lista que deve armazenar as últimas 3 notícias mais recentes de um site, ou seja, a última notícia inserida deverá aparecer por primeiro:

Para inserir um item:

```!/bin/bash
LPUSH ultimas_noticias "teste para exclusão"
LPUSH ultimas_noticias "Rede Record sobre ataque cibernético e muda programação às pressas"
LPUSH ultimas_noticias "Michael Myers: veja a ordem cronológica dos filmes da saga Halloween"
LPUSH ultimas_noticias "Fique longe do WhatsApp', alerta fundador do Telegram"
```

Também podemos inserir os valores acima de uma forma mais fácil caso queiramos inserir vários valores basta passar a ordem que desejamos inserir os valores LPUSH ou RPUSH chave e os valores entre espaços, ex:

```!/bin/bash
LPUSH ultimas_noticias "noticia_4" "noticia_3" "noticia_2" "noticia_1"
```

Para manter apenas as 3 posições requeridas

```!/bin/bash
LTRIM ultimas_noticias 0 2
```

Obtendo as notícias de acordo com os índices:

```!/bin/bash
LINDEX ultimas_noticias 0
LINDEX ultimas_noticias 1
LINDEX ultimas_noticias 2
```

Obtendo o tamanho da lista:

```!/bin/bash
LLEN ultimas_noticias
```

Obtendo um range da lista:

```!/bin/bash
LRANGE ultimas_noticias 1 2
```

## <a name='filas'></a>  Filas

### Motivação

Nosso site alterou a forma de login, agora o usuário criar uma conta precisa receber um e-mail para confirmação.

Como faremos para tratar essa fila em que os usuários que entrarem primeiro deverão ser atendidos primeiro?

Um outro ponto muito importante que trataremos na hora de consumir filas é a recursividade. Precisamos sempre estar perguntando se existe algum usuário esperando um e-mail? É necessário ficarmos consumindo processamento de máquina perguntando todo o momento se existe alguém na fila? O Redis já nos trás um solução para esse problema, vamos analisa-lo melhor [aqui](#filas-recursividade).

Vamos inserir alguns usuários para demonstração:

```!/bin/bash
RPUSH fila:confirma-email "pessoa_1" "pessoa_2" "pessoa_3" "pessoa_4"
```

Vamos confirmar os usuários inseridos:

```!/bin/bash
LRANGE fila:confirma-email 0 3
```

Para atendermos e retirarmos o usuário do começo da lista podemos efetuar:

```!/bin/bash
LINDEX fila:confirma-email 0
LPOP fila:confirma-email
```

<a name="filas-recursividade"></a>Utilizando Recursividade:

Para deixarmos o Redis esperando por algum tempo alguém cair na fila, podemos usar  o comando:

```!/bin/bash
BLPOP fila:confirma-email 30
```

No exemplo acima, o Redis vai ficar esperando por 30 segundos alguém cair na fila. Mas e se quisermos esperar indeterminadamente por um usuário? basta alterarmos os parâmetros de 30 para 0, assim a aplicação ira esperar por tempo indeterminado um usuário cair na fila para ser processado.

``` !/bin/bash
BLPOP fila:confirma-email 0
```

## <a name='conclusao'></a> Conclusão

Verificamos neste documento alguns comandos básicos do Redis e manipulações de dados.

Lembre-se que todos os valores armazenados no Redis são armazenados por padrão na memória do computador. Por isso você vai encontrar diversos lugares recomendado o uso de outro banco de dados para armazenar a origem dos seus dados.
