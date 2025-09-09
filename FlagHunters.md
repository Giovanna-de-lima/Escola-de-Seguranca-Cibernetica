# Flag Hunters (Reverse Engenieering) 
###### Solved by @Giovanna-de-lima

> This is a CTF about Reverse Engenieering

## Desafio: Flag Hunters (Engenharia Reversa)
#### Introdução

Este é o segundo exercicio proposto na Escola de Cyber Segurança da plataforma [picoCTF]([https://cryptohack.org/](https://play.picoctf.org/practice)). A trilha é composta por uma série de 4 exercícios introdutórios voltados para a prática de resolução de problemas de [CTFs](https://ctf-br.org/sobre/), com o objetivo é desenvolver o raciocinio, habilidades de pesquisa e 
treinar as habilidades de segurança da informação por meio desses exercícios práticos. 

- [Página do exercicio](https://play.picoctf.org/practice/challenge/472)

Esta questão tem como objetivo análisar e caso necessário, modificar o código fornecido em [Python](https://aws.amazon.com/pt/what-is/python/), chamdao hamado lyric-reader.py que lê e exibe uma letra de música interativa, para que esse código mostre a flag escondida.

#### Análise Inicial

Ao entrar no exercicio temos o enunciado abaixo:
> *"Lyrics jump from verses to the refrain kind of like a subroutine call. There's a hidden refrain this program doesn't print by default. Can you get it to print it? There might be something in it for you."*  
> *(As letras saltam dos versos para o refrão, como uma chamada de subrotina. Há um refrão oculto que este programa não imprime por padrão. Você consegue fazer com que ele o imprima? Pode haver algo nele para você.)*

Além disso, há três dicas:
> *"Este programa pode facilmente entrar em estados indefinidos. Não tenha vergonha de usar Ctrl-C."*  
> *A entrada de dados do usuário não higienizada é sempre boa, certo?*
> *Existe alguma sintaxe que seja propícia à subversão?*

E para esse desafio temos o código a ser análisado:
```
import re
import time


# Read in flag from file
flag = open('flag.txt', 'r').read()

secret_intro = \
'''Pico warriors rising, puzzles laid bare,
Solving each challenge with precision and flair.
With unity and skill, flags we deliver,
The ether’s ours to conquer, '''\
+ flag + '\n'


song_flag_hunters = secret_intro +\
'''

[REFRAIN]
We’re flag hunters in the ether, lighting up the grid,
No puzzle too dark, no challenge too hid.
With every exploit we trigger, every byte we decrypt,
We’re chasing that victory, and we’ll never quit.
CROWD (Singalong here!);
RETURN

[VERSE1]
Command line wizards, we’re starting it right,
Spawning shells in the terminal, hacking all night.
Scripts and searches, grep through the void,
Every keystroke, we're a cypher's envoy.
Brute force the lock or craft that regex,
Flag on the horizon, what challenge is next?

REFRAIN;

Echoes in memory, packets in trace,
Digging through the remnants to uncover with haste.
Hex and headers, carving out clues,
Resurrect the hidden, it's forensics we choose.
Disk dumps and packet dumps, follow the trail,
Buried deep in the noise, but we will prevail.

REFRAIN;

Binary sorcerers, let’s tear it apart,
Disassemble the code to reveal the dark heart.
From opcode to logic, tracing each line,
Emulate and break it, this key will be mine.
Debugging the maze, and I see through the deceit,
Patch it up right, and watch the lock release.

REFRAIN;

Ciphertext tumbling, breaking the spin,
Feistel or AES, we’re destined to win.
Frequency, padding, primes on the run,
Vigenère, RSA, cracking them for fun.
Shift the letters, matrices fall,
Decrypt that flag and hear the ether call.

REFRAIN;

SQL injection, XSS flow,
Map the backend out, let the database show.
Inspecting each cookie, fiddler in the fight,
Capturing requests, push the payload just right.
HTML's secrets, backdoors unlocked,
In the world wide labyrinth, we’re never lost.

REFRAIN;

Stack's overflowing, breaking the chain,
ROP gadget wizardry, ride it to fame.
Heap spray in silence, memory's plight,
Race the condition, crash it just right.
Shellcode ready, smashing the frame,
Control the instruction, flags call my name.

REFRAIN;

END;
'''

MAX_LINES = 100

def reader(song, startLabel):
  lip = 0
  start = 0
  refrain = 0
  refrain_return = 0
  finished = False

  # Get list of lyric lines
  song_lines = song.splitlines()
  
  # Find startLabel, refrain and refrain return
  for i in range(0, len(song_lines)):
    if song_lines[i] == startLabel:
      start = i + 1
    elif song_lines[i] == '[REFRAIN]':
      refrain = i + 1
    elif song_lines[i] == 'RETURN':
      refrain_return = i

  # Print lyrics
  line_count = 0
  lip = start
  while not finished and line_count < MAX_LINES:
    line_count += 1
    for line in song_lines[lip].split(';'):
      if line == '' and song_lines[lip] != '':
        continue
      if line == 'REFRAIN':
        song_lines[refrain_return] = 'RETURN ' + str(lip + 1)
        lip = refrain
      elif re.match(r"CROWD.*", line):
        crowd = input('Crowd: ')
        song_lines[lip] = 'Crowd: ' + crowd
        lip += 1
      elif re.match(r"RETURN [0-9]+", line):
        lip = int(line.split()[1])
      elif line == 'END':
        finished = True
      else:
        print(line, flush=True)
        time.sleep(0.5)
        lip += 1

reader(song_flag_hunters, '[VERSE1]')
```

Print do enunciado:
[![FH.png](https://i.postimg.cc/gj2ZFZwn/FH.png)](https://postimg.cc/gX1J3rHW)


#### Interpretando a dica
A dica que nos direciona para a melhor resolução é sobre as entradas do usuario, nesse caso, como vimos que a entrada de dados não é tratada podemos manipular o fluxo do programa através dela. 


#### Solução
Inicialmente recebemos o código fonte (lyric-reader.py) em Python para ser analisado. Para resolver este exercicio, utilizei duas ferramentas, o VScode, para verificar o funcionamento das partes do código e depois o próprio terminal da plataforma picoCTF (o picoCTF Webshell) para fazer alterações e capturar a flag. Antes de usar o picoCTF Webshell , é necessário Iniciar a Instancia e utilizar o comando: nc verbal-sleep.picoctf.net [numero da porta fornecida], para vermos o código sendo executado. 

Há três partes principais no código:
- Leitura da Flag: o código que lê o conteúdo de um arquivo flag.txt e armazena na variável chamada flag.
- Letra da Música: a variável song_flag_hunters contém a letra da música, incluindo um trecho inicial chamado secret_intro, onde está a nossa flag.
- Função reader: responsável por exibir a letra da música interativamente e começa a exibição a partir de um marcador específico (startLabel), permitindo a interação do usuário em pontos definidos pela palavra chave CROWD.

Notamos que a música começa a ser exibida a partir do [VERSE1], mas precisamos que comece exibindo a partir do inicio, contemplando o trecho [secret_intro] onde está a nossa flag. Para isso, manipulamos a entrada do usuário, que nos da essa permissão, pois não é tratada. Para isso, utilizaremos o RETURN 0 para controlar o fluxo de exibição da letra. Entretanto para que sea intermpretado de forma correta, é necessário formatar a entrada como uma string seguida de ;RETURN 0, garantindo que o comando seja executado e não apenas exibido como texto.
Se usarmos só o RETURN 0, será tratado como texto e não será executado, já se colocarmos o ;RETURN 0 ele é executado e  altera o fluxo do programa.

Assim, usando o comando "string;RETURN 0" no picoCTF Webshell, em frente a palavra chave CROWD, assim o codigo irá para a primeira linha da letra, exibindo o trecho secret_intro onde conseguimos mostrar nossa flag. 

#### Conclusão

Flag:
>`picoCTF{70637h3r_f0r3v3r_c373964d}`



Este exercicio foi um tanto desafiador, pois foi necessário entender como todo código funciona e como a função que continha a flag poderia ser exibida no formato certo. Além de testar a leitura, interpretação e estrutura do código, ele  reforça sobre boas práticas no tratamento das entradas de dados feitas pelo usuário, fazer as validações e a "higienização" apropriada, pois sem essas validações o usuário pode manipular o fluxo do programa, exibindo informações sensíveis em contextos reais e que podem comprometer a segurança do sistema.


