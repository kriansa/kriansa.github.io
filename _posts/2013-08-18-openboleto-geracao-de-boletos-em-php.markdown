---
layout: post
title: OpenBoleto – Biblioteca de boletos bancários em PHP
date: 2013-03-19 20:44:34 BRT
tags: boleto, open-source, openboleto, php
categories: projects
permalink: /:categories/:title
redirect_from:
  - /blog/2013/03/openboleto-biblioteca-de-geracao-de-boletos-bancarios/
  - /blog/2013/08/openboleto-geracao-de-boleto-bancario/
---
![boleto]({{ site.path }}/img/boleto.png)

O <a title="OpenBoleto no GitHub" href="http://kriansa.github.io/openboleto/?utm_source=blog&amp;utm_medium=post&amp;utm_campaign=Blog" target="_blank">OpenBoleto</a> é uma biblioteca de código aberto para geração de boletos bancários, um meio de pagamento muito comum no Brasil. O foco é ser simples e ter uma arquitetura compatível com os recursos mais modernos do PHP, mais amigável com frameworks MVC. Criada em Janeiro de 2013, hoje finalmente lançamos a primeira versão estável.

<a href="https://github.com/kriansa/openboleto"><img style="border-radius: none; box-shadow: none; position: fixed; top: 0; left: 0; border: 0;" alt="Fork me on GitHub" src="https://s3.amazonaws.com/github/ribbons/forkme_left_darkblue_121621.png" /></a>

<h2>Como usar?</h2>
O OpenBoleto é simples e flexível, serve tanto para quem não tem tanta experiência em programação quanto para quem <del>manja dos paranauê</del> gosta de customizar tudo. Veja um exemplo abaixo de como gerar um boleto para o Banco do Brasil:

{% highlight php %}
<?php

use OpenBoleto\Banco\BancoDoBrasil;
use OpenBoleto\Agente;

$sacado = new Agente('Fernando Maia', '023.434.234-34', 'ABC 302 Bloco N', '72000-000', 'Brasília', 'DF');
$cedente = new Agente('Empresa de cosméticos LTDA', '02.123.123/0001-11', 'CLS 403 Lj 23', '71000-000', 'Brasília', 'DF');

$boleto = new BancoDoBrasil(array(
    // Parâmetros obrigatórios
    'dataVencimento' => new DateTime('2013-01-24'),
    'valor' => 23.00,
    'sequencial' => 1234567, // Para gerar o nosso número
    'sacado' => $sacado,
    'cedente' => $cedente,
    'agencia' => 1724, // Até 4 dígitos
    'carteira' => 18,
    'conta' => 10403005, // Até 8 dígitos
    'convenio' => 1234, // 4, 6 ou 7 dígitos
));

echo $boleto->getOutput();
{% endhighlight %}

Sim, é só isso! :) Claro que existem outras opções configuráveis, para saber as específicas de cada banco, consulte a pasta <strong>samples</strong>.

Note que o sufixo "Dv" se refere à dígito verificador. Todos os parâmetros para geração podem ser acessados via getters e setters, por exemplo:

{% highlight php startinline=true %}
$agencia = $boleto->getAgencia();
$boleto->setAgencia(1584);
{% endhighlight %}

<h2>Instalação</h2>
<h3>Composer</h3>
Se você já conhece o <a title="Composer" href="http://getcomposer.org/" target="_blank"><strong>Composer</strong></a> (o que é extremamente recomendado), simplesmente adicione a dependência abaixo à diretiva "<strong>require</strong>" seu composer.json:

{% highlight javascript %}
{
  "kriansa/openboleto": "v1.0"
}
{% endhighlight %}

<h3>Como library de algum framework</h3>
Primeiro baixe ou clone o repositório no <a title="OpenBoleto no GitHub" href="https://github.com/kriansa/openboleto" target="_blank">GitHub</a>. Se você usa algum framework com autoloader, basta apontar o namespace '<em><strong>OpenBoleto</strong></em>' para a pasta '<em>src</em>'. Ou simplesmente mova o conteúdo da pasta '<em>src</em>' para a library de seu framework.
<h3>Com PHP puro (biblioteca standalone)</h3>
Caso não use framework ou nenhum autoloader, ou você simplesmente quer dar "<em>um include e pronto</em>", você pode usar o autoloader incluso na biblioteca, para isso simplesmente dê um include no arquivo "_autoloader.php_" e tudo deverá funcionar normalmente!

## Bancos suportados
Atualmente estão suportados os seguintes bancos:

* Banco de Brasília (BRB)
* Banco do Brasil
* Bradesco
* Itaú
* Santander
* Unicred

Você também pode ajudar adaptando a outros bancos, o processo consiste basicamente em identificar o modo pelo qual o banco gera o seu <strong><em>campo livre</em></strong>, ou seja, o campo de 20 a 44 do código de barras. Desta forma, basta implementar em uma classe observando outros atributos como código do banco. Como exemplo, veja a classe <strong>Bradesco</strong>.

<h2>Algumas considerações</h2>
O método <strong>getNossoNumero</strong> é read-only, ou seja, não existe um <strong>setNossoNumero</strong>. Por que isso? Alguns bancos geram o nosso número, aquele que fica impresso no boleto, de forma diferente do código único que você define por boleto.

Para evitar confusões, definimos que o código único por boleto se chama <strong>sequencial</strong>. Nosso número é aquele que aparece no boleto, e geralmente é o que vem nos arquivos de retorno do banco.

## #TODO

* <del>Apesar do bootstrap do PHPUnit estar pronto, os testes unitários não estão. É necessário fazê-los.</del> <strong>DONE.</strong>
* Falta adaptar a biblioteca a todos os outros bancos existentes
* Outro objetivo é que a classe também trabalhe com arquivos de remessa e retorno dos bancos

Toda contribuição é muito bem vinda! Note que os padrões de codificação utilizados são os PSR-0, PSR-1 e PSR-2.

## Agradecimentos
Agradeço aos mantenedores das bibliotecas <a title="BoletoPHP no GitHub" href="https://github.com/BielSystems/BoletoPHP" target="_blank">BoletoPHP</a> e <a title="Boleto Library PHP" href="https://github.com/drupalista-br/Boleto" target="_blank">Boleto Library PHP</a>, principalmente à este último que explica muito bem como funciona a geração de boletos e foi de grande inspiração para a abstração do OpenBoleto. Agradeço também por toda comunidade que apoia e contribui para o projeto. E por último, mas não menos importante ao <a title="Cristiano Teles" href="http://www.cristianoteles.com.br/blog/" target="_blank">Cristiano Teles</a>, coordenador da comunidade PHP-DF e grande e inspirador mentor com o qual tive o privilégio de aprender muitas das coisas aqui postas em prática.

## Licença
Este projeto é de código aberto licenciado pela MIT License.
