---
title: Migração de SVN para GIT
layout: post
date: '2017-07-20 22:16:06 +0100'
excerpt_separator: <!--more-->
categories:
- git
- svn
- dicas
---

Recentemente foi-me lançado o desafio de migrar um servidor de svn para git , e comecei logo a procura de importações directas,
sendo que encontrei sugestões tais como o  [svn2git](https://github.com/nirvdrum/svn2git/ "svn2git") e [subGit](http://subgit.com/,"subit").

No entanto não se adequavam ao que se pretendia fazer, então após mais alguma pesquisa, encontrei várias referências a um comando do git que permitia
clonar um repositório de SVN para Git e conseguir **manter o histórico** do mesmo sem configuraçoes extra ou programa externos. O comando em causa é o

<!--more-->

{% highlight shell %}
git svn clone http://svn/
{% endhighlight %}

Atráves deste comando é efectuado um pedido ao servidor de svn e será criada uma nova directória para o repositório em causa, sendo que este comando tem a possibilidade de aceitar os parametros que podem ser passados para os **init** e **fetch** da utilização de git.
No entanto de nada nos serve um clone de um
repositório sem os logs e o mapeamento dos autores dos commits para o sistema que está a ser utilizado. Para tal basta utilizar os seguintes comandos.

{% highlight shell %}
svn log --xml | grep "<author>" | sort -u | perl -pe 's/.*>(.*?)<.*/$1 = /' | tee users.txt
{% endhighlight %}

Este primeiro comando deverá ser executado no servidor de svn actual , com este comando será possivel gerar um ficheiro com as informações relativas a todos
os autores de commits do svn em causa. Este ficheiro será utilizado como ficheiro de mapeamento de migração de commits e logs para o Git.

{% highlight shell %}
git svn clone --stdlayout --no-metadata --authors-file=users.txt svn://hostname/path dest_dir-tmp
{% endhighlight %}

Este segundo comando irá fazer clone do repositório no servidor de SVN e irá criar um repositório de Git local, com a informação de todos os commits e logs deste novo repositório.
Será necessário ainda especificar o **remote origin** que iremos utilizar para todas as ações futuras , para tal basta executar os seguintes comandos:

{% highlight shell %}
git remote add origin git@server:user/project-name.git
git push origin --all
{% endhighlight %}


A partir deste momento será possivel sincronizar o repositório migrado com o servidor de Git.
