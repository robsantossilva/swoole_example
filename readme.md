### Exemplo com Swoole

O <strong>Swoole</strong> é um framework de programação assíncrona para PHP e já se posiciona como uma opção estável e confiável para se desenvolver de forma concorrente com alta performance e uso equilibrado de recursos. 

Ele implementa <strong>I/O assíncrono</strong> orientado a eventos baseado no padrão Reactor (o mesmo utilizado por ReactPHP, NodeJS, Netty do Java, Twisted do Python entre outros). O Swoole é escrito em C e disponibilizado como uma extensão para PHP.

### TCP, UDP, Unix Socket, HTTP e WebSockets
O Swoole permite que escrevamos aplicações altamente performáticas e concorrentes usando TCP, UDP, Unix Socket, HTTP e WebSockets sem que precisemos ter grandes conhecimentos de baixo nível sobre assincronismo e/ou sobre o kernel do Linux. Ele fornece uma completa API de alto nível com foco na produtividade. O Swoole é usado em grandes projetos enterprises, principalmente na China sendo Baidu (maior portal de busca da China) e Tencent (um dos maiores portais de serviços da China) os principais casos de uso.

É muito importante e recomendado que antes de continuar aqui, você leia o artigo sobre Introdução à programação assíncrona em PHP usando o ReactPHP, pois ele vai te introduzir alguns importantes conceitos de como é o modelo tradicional síncrono de se desenvolver em PHP e como funciona o modelo assíncrono usando o padrão Reactor.

### Para o que veio e para o que não veio o Swoole

<strong>O Swoole não veio para substituir a abordagem tradicional e síncrona</strong> de como a maioria do projetos em PHP são escritos. Ao contrário disso, ele veio pra suprir os outros casos de uso como, por exemplo, a criação de servidores TCP, RPC, Websockets etc. Ele veio para resolver o problema de servidores que precisem de alta concorrência. Para se ter ideia, um único servidor escrito usando o Swoole consegue lidar com 1 milhão de conexões (C1000K). Também podemos dizer que o Swoole veio para que você não precise mudar toda sua stack e linguagem para resolver as necessidades acima listadas.

### Exemplo Prático

#### Estrutura de pastas:

```bash
/Dockerfile
/docker-compose.yml
/app/index.php
```

```bash
#Content of index.php

<?php
$http = new swoole_http_server("0.0.0.0", 8101);

$http->on("start", function ($server) {
    echo "Swoole http server is started at http://127.0.0.1:8101\n";
});

$http->on("request", function ($request, $response) {
    $response->header("Content-Type", "text/plain");
    $response->end("Hello World\n");
});

$http->start();
```

```bash
#Content of Swoole Dockerfile:
FROM php:7.2-fpm

RUN apt-get update

RUN apt-get install vim -y && \
    apt-get install openssl -y && \
    apt-get install libssl-dev -y && \
    apt-get install wget -y 

RUN cd /tmp && wget https://pecl.php.net/get/swoole-4.2.9.tgz && \
    tar zxvf swoole-4.2.9.tgz && \
    cd swoole-4.2.9  && \
    phpize  && \
    ./configure  --enable-openssl && \
    make && make install

RUN touch /usr/local/etc/php/conf.d/swoole.ini && \
    echo 'extension=swoole.so' > /usr/local/etc/php/conf.d/swoole.ini

RUN mkdir -p /app/data

WORKDIR /app

COPY ./app /app

EXPOSE 8101
CMD ["/usr/local/bin/php", "/app/index.php"]
```

```bash
#docker-compose.yml
version: '3.7'

services:
    swoole:
        build:
            context: ./
        ports: 
            - 8101:8101
```