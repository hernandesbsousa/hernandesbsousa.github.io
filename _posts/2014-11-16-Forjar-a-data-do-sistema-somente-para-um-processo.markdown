---
layout: post
title:  "Forjando a data do sistema somente para um processo"
date:   2014-11-16 21:00:00
categories: linux date
---

Às vezes, durante o teste de certas funcionalidades de software, é necessário "avançar/voltar no tempo", fazer isso no sistema inteiro pode causar diversos problemas com ferramentas de log e monitoramento, por exemplo.

Será se não existe uma maneira de fazer com que somente o Apache, por exemplo, trabalhe com a data modificada?

## Conheça o `libfaketime`

O [libfaketime][libfaketime] é um projeto que tem como objetivo interceptar diversas chamadas de sistema que obtém data e hora. Ele trabalha utilizando o `LD_PRELOAD`, recurso disponível no carregador dinâmico de bibliotecas do Linux (explicá-lo em detalhes tomará um post :stuck_out_tongue_winking_eye:), mas basicamente basta o programa que você utiliza aceitar o carregamento dinâmico.


### Instalação

Este exemplo foi feito num Ubuntu Trusty, mas pode ser feito em qualquer distro _Debian-like_. Caso não exista pacote disponível para seu sistema, a compilação dela é bem simples também e sua utilização não exige permissões de root.

Instalação (para o Debian o pacote é somente faketime)

    apt-get install faketime libfaketime

Localizando a biblioteca (normalmente `libfaketime.so.1`)


    $dpkg -L libfaketime
    /.
    /usr
    /usr/lib
    /usr/lib/x86_64-linux-gnu
    /usr/lib/x86_64-linux-gnu/faketime
    /usr/lib/x86_64-linux-gnu/faketime/libfaketime.so.1
    /usr/lib/x86_64-linux-gnu/faketime/libfaketimeMT.so.1
    /usr/share
    /usr/share/doc
    /usr/share/doc/libfaketime
    /usr/share/doc/libfaketime/changelog.Debian.gz
    /usr/share/doc/libfaketime/copyright
    /usr/share/doc/libfaketime/README.gz


Carregando a biblioteca (mas sem definir a data ainda)

    $export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/faketime/libfaketime.so.1

Agora podemos definir a variável `FAKETIME` para a data que desejamos. É possível utilizar pontos específicos no tempo com o formato `YYY-MM-DD hh:mm:ss`.

### Brincando com a data


    $date
    Mon Nov 17 00:17:49 UTC 2014
    $FAKETIME="2000-01-01 10:05:07" date
    Sat Jan  1 10:05:07 UTC 2000


O problema dessa abordagem é que o tempo ficará para sempre definido como `2000-01-01 10:05:07`, o que pode causar problemas caso você queira testar um evento que ocorre certo tempo após a execução de um primeiro, como uma tarefa de `cron`. Para evitar isso, é possível definir o tempo baseado no atual com um determinado desvio, que pode ser positivo ou negativo:

- `-10m`: 10 minutos atrás
- `+1y`: daqui a 1 ano

É possível utilizar os valores `m`, `h`, `d` e `y`.

## Utilizando na prática

A data base é 17 de Novembro de 2014:

    $date
    Mon Nov 17 00:24:26 UTC 2014

### PHP

    $FAKETIME="-2d" php -r 'echo date("Y-m-d") . PHP_EOL;'
    2014-11-15

### Python

    $FAKETIME="-2d" python3

    >>> import datetime
    >>> print(datetime.date.today())
    2014-11-15

### Ruby

    $FAKETIME='-2d' irb
    2.1.4 :001 > Time.new.strftime("%Y-%m-%d")
     => "2014-11-15"

### Apache

Basta adicionar as variáveis `LD_PRELOAD` e `FAKETIME` como nos exemplos acima, porém no arquivo de variáveis de ambiente do Apache.


````
$ echo "export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/faketime/libfaketime.so.1" >> /etc/apache2/envvars
$ echo "FAKETIME='-7d'" >> >> /etc/apache2/envvars
service apache2 restart
````

## Velocidade

Outra funcionalidade é a de manipular a velocidade do relógio, onde com o a string `-7d 0.5x` a data estaria uma semana atrás e o tempo deverá correr 50% mais devagar, ou seja, para cada 2 segundos reais transcorridos, apenas 1 será contabilizado.


[libfaketime]: https://github.com/wolfcw/libfaketime
