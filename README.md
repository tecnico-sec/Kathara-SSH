Instituto Superior Técnico, Universidade de Lisboa

**Segurança Informática em Redes e Sistemas**

# Guia de Laboratório - *Secure Shell*

## Objetivo

O objetivo deste trabalho é aprender a utilizar o protocolo SSH usando o pacote OpenSSH.
Vai ser usado o emulador de redes *Kathará*.

Os protocolos SSH servem para fazer *login* remoto, copiar ficheiros e executar comandos remotamente de modo seguro.
Os mecanismos criptográficos usados são semelhantes aos de protocolos como o SSL/TLS, como a criptografia assimétrica para garantir autenticação e a criptografia simétrica para garantir confidencialidade.

---

## Exercício 1 -- *Login* remoto

1. Crie uma rede virtual com apenas duas máquinas virtuais interligadas entre si.
Chame-lhes PC1 e PC2.

2. Para usar o protocolo SSH é preciso iniciar o serviço *ssh*.
Execute nas duas VMs o comando:  
`/etc/init.d/ssh start`

3. Crie um utilizador em cada máquina usando `adduser`, p.ex., `user1` e `user2`.
Faça *login* em cada um deles usando *su - username*.
Pode ser útil também usar o comando `whoami` para saber qual é o utilizador e o comando `pwd` para saber a pasta actual.

4. Estando o servidor *ssh* a correr numa máquina, é possível aceder-lhe remotamente usando o cliente *ssh*.
Aceda do PC1 ao outro executando:  
`ssh username@remote_host`  
e substituindo *username* pelo utilizador remoto (*user1* ou *user2*) e *remote_host* pelo endereço IP da VM remota (podia ser também o nome da máquina remota mas na rede em causa não há DNS para traduzir os nomes em endereços IP).
Note que sempre que acede a uma máquina remota pela primeira vez é apresentado um aviso:  
*"The authenticity of host (...) cannot be established."*  
Este aviso serve apenas para o utilizador confirmar a que máquina está a aceder, indicando que esta máquina ainda não é conhecida.
Após autorizar, a chave pública do servidor é armazenada e este aviso não voltará a surgir pois a máquina já foi acedida anteriormente e é reconhecida.

5. Repare que pode dar comandos na máquina remota.
A *shell* apresentada é da outra máquina.
Experimente p.ex. ver o conteúdo da pasta `home` do utilizador remoto.
Pode sair da máquina remota com `exit` ou *\^d (CTRL+d)*.

6. Vamos usar criptografia de chave pública para autenticar o utilizador, para evitar a necessidade de fornecer uma *password* quando se faz um acesso remoto.
Gere um par de chaves RSA para o utilizador do PC1 com o comando:  
`ssh-keygen -t rsa`  
Deixe o comando gravar as chaves na pasta por omissão (`.ssh`) e não indique uma *password* (que seria usada para proteger o ficheiro com a chave privada).
Veja o conteúdo da pasta e repare que há dois ficheiros: um com a chave privada e outro com a chave pública.

7. As chaves privadas não devem estar visíveis para outros utilizadores, caso contrário o SSH recusa-se a usá-las (e por isso pede a *password*).
Confirme as permissões da pasta `.ssh` e do ficheiro `id_rsa` e assegure-se que as permissões são  
`rwx --- ---` (permissão de leitura, escrita e execução para o utilizador e nenhuma para o grupo e todos os utilizadores) ou  
`rw- --- ---` (só leitura e escrita).

8. Copie as chaves públicas (nunca as privadas!) de um computador para o outro.
Poderia fazer essa cópia copiando os ficheiros entre as máquinas usando o *secure copy* (ver exercício 2.2).
Como ainda não vimos esse comando, uma alternativa prática é fazer *copy-paste* do conteúdo do ficheiro entre as duas janelas. Outra alternativa é usar a pasta `/shared` do Kathará (que existe em todas as máquinas virtuais e é partilhada por elas).

9. No PC2 crie a pasta `.ssh`, com as permissões `700`.
Pode alterar as permissões dando o comando:  
`chmod 700 /<caminho para ficheiro ou pasta>`  
Na pasta `.ssh` do utilizador para cuja conta quer fazer SSH sem ter de fornecer a *password*, crie um ficheiro chamado `authorized_keys` com a chave pública do utilizador remoto (aquela que copiou do outro computador no ponto 8). O formato desse ficheiro será:
` ssh-rsa ...chave pública... user1@pc1 `
Poderia colocar nesse ficheiro mais chaves de outros utilizadores, uma por linha, se fosse o caso.
O objetivo deste ficheiro consiste em indicar as entidades que estão autorizadas a entrar, ou  seja, indica que o computador deve aceitar comandos SSH de utilizadores que tenham acesso a uma chave privada correspondendo a uma das chaves públicas presentes no ficheiro `authorized_keys`.

10. Aceda da conta que tem a chave privada à outra correndo o comando:  
`ssh username@remote_host`

11. Repare que não precisou de dar a *password* do utilizador remoto para entrar na sua área.
É precisamente este o objetivo do procedimento de troca das chaves públicas.
Se foi feita a troca de chaves e mesmo assim foi pedida a *password* é porque está errado (p.ex., o servidor não está a correr ou as permissões não estão corretas).

12. Execute o comando `ssh` novamente mas dando a opção `-v`.
Observe a informação mostrada.
Note que essa informação pode ser útil para fazer *debug.*

13. No ponto 6 acima dissemos para não indicar uma *password* que seria usada para proteger o ficheiro que contém a chave privada. No entanto, fornecer essa *password* é uma boa prática (essencial mesmo!) para evitar que um intruso roube esta chave. Em concreto, a partir dessa *password*, o comando gera uma chave secreta (simétrica) que usa para cifrar o conteúdo do ficheiro. Repita o passo 6. mas desta ver fornecendo essa chave. Volte a inspecionar o ficheiro onde está a chave privada. O que é que mudou? De agora em diante, quando pretender usar esse ficheiro vai ter de fornecer essa *password*.

---

## Exercício 2 -- Outras operações remotas

1. O mesmo comando `ssh` pode ser usado para dar comandos remotamente.
Para o efeito basta acrescentar o comando no fim da instrução.
Por exemplo, pode criar um ficheiro vazio chamado `aaa` na pasta remota usando o comando:  
`ssh username@remote_host touch aaa`  
(se não sabe o que faz o comando `touch`, execute `man touch`).

2. O comando `scp` pode ser usado para transferir ficheiros entre máquinas usando o protocolo SSH.
Experimente copiar um ficheiro entre as duas VMs.
A sintaxe é:  
`scp ficheiro_local username@remote_host:destination`  
onde *destination* é a pasta de destino.

---

## Exercício 3 -- Operações avançadas

Para estes exercícios vamos usar os servidores `sigma.tecnico.ulisboa.pt`, em vez do *Kathará* que não fornece uma GUI às suas máquinas virtuais.
Deverá executar os comandos num terminal no seu PC.
Cada exercício é um exercício separado.

1. O SSH permite a execução remota de aplicações gráficas.
Para tal basta usar a flag `-X`.
Execute o seguinte comando para se ligar a uma das máquinas sigma:  
`ssh -X istxxx@sigma.tecnico.ulisboa.pt`  
Após introduzir a sua *password*, fica ligado à máquina sigma, onde pode correr aplicações. 
Vamos correr uma aplicação gráfica, emitindo o comando `xpdf`.
Se tiver um ficheiro PDF na sua área, pode indicá-lo como argumento.
Vai aparecer uma janela com o programa gráfico `xpdf` no seu PC, embora este esteja a correr no sigma.
O SSH está a encaminhar os gráficos através de um túnel, por onde é transmitido o protocolo de terminal gráfico X11.

2. O SSH permite a criação de túneis por onde uma ligação TCP pode ser encaminhada de forma cifrada.
Execute o seguinte comando num terminal no seu PC:  
`ssh -L 8080:sigma02.tecnico.ulisboa.pt:22 istxxx@sigma01.tecnico.ulisboa.pt`  
deixe esta ligação aberta e noutro terminal emita o comando:  
`ssh istxxxx@localhost -p 8080`  
O segundo comando efetua uma ligação SSH para o seu PC (localhost).
No entanto aparece ligado ao `sigma02`.
Consegue explicar o que está a acontecer?

3. O comando `rsync` permite efectuar a sincronização de ficheiros ou pastas entre máquinas. 
Efetue o seguinte comando:  
`rsync -e ssh -azvP <pasta local com ficheiro> istxxx@sigma.tecnico.ulisboa.pt:teste`  
O conteúdo da pasta local foi copiado para a pasta teste no sigma.
Altere um ficheiro na pasta local e volte a executar o comando.
O que aconteceu de diferente na segunda execução?  
Faça `man rsync` e veja o significado das várias *flags* utilizadas.

---

Referências

- *Kathará*, [https://github.com/KatharaFramework/Kathara/wiki][3]

- *Secure Shell*, [http://en.wikipedia.org/wiki/Secure_Shell][4]

- ssh-keygen manual, [http://linux.die.net/man/1/ssh-keygen][5]

  [3]: https://github.com/KatharaFramework/Kathara/wiki
  [4]: http://en.wikipedia.org/wiki/Secure_Shell
  [5]: http://linux.die.net/man/1/ssh-keygen
