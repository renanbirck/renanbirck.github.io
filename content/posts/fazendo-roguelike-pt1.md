---
title: "Fazendo um roguelike simples: parte 1"
date: 2021-07-13T10:40:32-03:00
draft: false 
tags: 
  - desenvolvimento
  - roguelike
  - python
enableMathJax: true
---


Como jogador de [nethack](https://nethack.org) que sou, paixão que descobri durante o trabalho em casa forçado pela covid, decidi que era hora de eu desenvolver meu próprio roguelike... mas o que é um roguelike, você caro leitor pode estar se perguntando? 

A definição de _roguelike_ é um jogo com níveis gerados de forma procedural (isto é, por algoritmos e não de forma manual) e com morte permanente (morreu? que Deus o tenha); a maioria deles tem narrativa épica, mas isso não é um requisito do estilo em si e nem é preciso ser leitor desse gênero para apreciar os jogos. (eu mesmo comecei a ler _O Hobbit_, mas dormi no meio).

Dessa forma surgiram _roguelikes_ com temática pós-apocalipse ([Cataclysm: Dark Days Ahead](https://cataclysmdda.org/)), espaço ([FTL: Faster than Light](https://store.steampowered.com/app/212680/FTL_Faster_Than_Light/)) e simulação ([Dwarf Fortress](https://www.bay12games.com/dwarves/)) entre outros; mas o mais clássico é o [nethack](https://nethack.org).

Neste projetinho de tutorial, vamos criar um _roguelike_ simples: você precisa descer alguns níveis, matar monstros e recuperar a Chave que permiti-lo-á sair da masmorra. O conteúdo desse tutorial foi adaptado [daqui](http://rogueliketutorials.com/tutorials/tcod/v2/part-1/).

A primeira parte irá colocar um personagem na tela e posteriormente irá movê-lo.

### 1. Instalando as bibliotecas

Para desenvolver nosso _roguelike_, vamos utilizar Python e a `libtcod`. Esse tutorial presume que você usa um sistema operacional de verdade (leia-se: Linux, \*BSD, macOS etc...), mas talvez seja possível repeti-lo no Windows.

Não queremos mexer nas bibliotecas do sistema, então vamos usar [venv](https://docs.python.org/3/library/venv.html):

    python3 -m venv ~/src/minirogue   # Adaptar conforme a organização de diretórios do seu projeto.

Após, vamos criar um arquivo `requirements.txt` para coordenar a instalação dos pacotes necessários. Crie um arquivo com o nome anteriormente referido e com as linhas:

    tcod>=11.13
    numpy>=1.18

e, para instalar as dependências, bastará digitar `pip install -r requirements.txt`. Se apareceu `successfully installed tcod-12.7.2`, pronto. A partir daqui, iremos trabalhar dentro do `venv` que criamos acima; portanto, ative ele digitando `source bin/activate` dentro do diretório criado acima.

Para nos certificarmos que a `libtcod` instalou:

    $ python3
    Python 3.9.6 (default, Jun 29 2021, 00:00:00) 
    [GCC 11.1.1 20210531 (Red Hat 11.1.1-3)] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import tcod
    >>>  # não deverá acontecer nenhum erro.

### 1.5. Preparação: baixar um arquivo de fontes
Baixe o [arquivo](http://rogueliketutorials.com/images/dejavu10x10_gs_tc.png) e coloca na mesma pasta do projeto. Será necessário depois.

### 2. Colocando nosso personagem na tela

Vamos criar um arquivo `main.py`, com o conteúdo abaixo:


{{< highlight python >}}

    #!/usr/bin/env python3
    #
    # minirogue: a simple roguelike game for testing libtcod.
    #
    # (c) 2021- Renan Birck Pinheiro [renan.birck.pinheiro at Gmail's "Google" mail service]
    #
    #

    import tcod

    def main():
        screen_width, screen_height = 80, 50  # Altura da tela
        
        # Carregar a fonte que anteriormente foi baixada no diretório atual.
        tileset = tcod.tileset.load_tilesheet(
            "dejavu10x10_gs_tc.png", 32, 8, tcod.tileset.CHARMAP_TCOD
        )
        
        # Criar a interface...
        with tcod.context.new_terminal(screen_width, screen_height, tileset=tileset,
                                    title="MiniRogue", vsync=True) as context:
            root_console = tcod.Console(screen_width, screen_height, order="F")
            while True:
                # Desenhar a tela, até a hora em que PEDIR PRA SAIR
                root_console.print(x=1, y=1, string="@")
                context.present(root_console)
                for event in tcod.event.wait():
                    if event.type == "QUIT": # PEDIU PRA SAIR
                        raise SystemExit()

    if __name__ == "__main__":
        main()
{{< / highlight >}}

Vamos explicar, em linhas gerais:

* `screen_width, screen_height` são as dimensões da tela (no caso, 80x50).
* `tileset` recebe o arquivo de ícones (ou, no nosso caso, a fonte a ser usada).
* Após, é inicializado um terminal e um console nas dimensões acima definidas.
* O _loop_ principal lê o teclado, toma a ação e atualiza a tela.

### 3. Movendo nosso personagem

