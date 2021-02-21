LETI 2020/21 | Segurança Informática em Redes e Sistemas

---

# Guia de Laboratório 7 - NetKit-SSH

## Objectivo

O objectivo do trabalho consiste em aprender a utilizar o protocolo SSH
usando o pacote OpenSSH. Vai ser usado o emulador de redes Netkit.

Os protocolos SSH servem para fazer login remoto, copiar ficheiros e
executar comandos remotamente de modo seguro. Os mecanismos usados são
semelhantes aos de protocolos como o SSL/TLS. Leia uma introdução a
esses protocolos na referência indicada abaixo.

## Exercício 1 -- Login remoto

1.  Crie uma rede virtual com apenas duas máquinas virtuais interligadas
    entre si. Chame-lhes PC1 e PC2

2.  Para usar o protocolo SSH é preciso iniciar o serviço *ssh.* Execute
    nas duas VMs o comando:

```bash
/etc/init.d/ssh start
```

3.  Crie um utilizador em cada máquina usando *adduser*, p.ex., *user1*
    e *user2.* Faça login em cada um deles usando *su - username*. Pode
    ser útil também usar o comando *whoami* para saber qual é o
    utilizador e o comando *pwd* para saber a pasta actual.

4.  Estando o servidor *ssh* a correr numa máquina, é possível
    aceder-lhe remotamente usando o cliente *ssh.* Aceda do PC1 ao outro
    executando:

```bash
ssh username@remote_host
```

substituindo *username* pelo utilizador remoto (*user1* ou *user2*) e
*remote_host* pelo endereço IP da VM remota (podia ser também o nome da
máquina remota mas na rede em causa não há DNS para traduzir os nomes em
endereços IP). Note que sempre que acede a uma máquina remota pela
primeira vez é apresentado um aviso: *"The authenticity of host (...)
can\'t be established."* Este serve apenas para o utilizador estar
atento a que máquina está a aceder, indicando que esta máquina ainda não
é conhecida. Pode autorizar. Após autorizar, este aviso não voltará a
surgir pois a máquina já foi acedida anteriormente e é reconhecida.

5.  Repare que pode dar comandos na máquina remota. A "shell"
    apresentada é da outra máquina. Experimente p.ex. ver o conteúdo da
    pasta *home* do utilizador remoto. Pode sair da máquina remota com
    *exit* ou *\^d (CRTL+d)*.

6.  Vamos usar criptografia de chave pública para evitar a necessidade
    de fornecer uma *password* quando se faz um acesso remoto. Gere
    chaves RSA para o utilizador do PC1 com o comando:

```bash
ssh-keygen -t rsa
```

Deixe o comando gravar as chaves na pasta por omissão (.ssh) e não
indique uma *password* (que seria usada para proteger o ficheiro com a
chave privada). Veja o conteúdo da pasta e repare que há dois ficheiros:
um com a chave privada e outro com a chave pública.

7.  As chaves privadas não devem estar visíveis para outros
    utilizadores, caso contrário o SSH recusa-se a usá-las (e por isso
    pede a *password*). Observe as permissões da pasta *.ssh* e do
    ficheiro *id_rsa* e repare que as permissões são rwx\-\-\-\-\-\--
    (permissão de leitura, escrita e execução para o utilizador e
    nenhuma para o grupo e todos os utilizadores) ou rw\-\-\-\-\-\-- (só
    leitura e escrita).

8.  Copie as chaves públicas (nunca as privadas!) de um computador para
    o outro. Pode fazer essa cópia copiando os ficheiros entre as
    máquinas usando o secure copy (ver exercício 2.2). Uma alternativa
    mais prática é fazer copy e paste do conteúdo do ficheiro entre as
    duas janelas.

9.  No PC2 crie a pasta .ssh, com as permissões 700. Pode alterar as
    permissões dando o comando *chmod 700 \<caminho para ficheiro ou
    pasta\>.* Na pasta *.ssh* do utilizador para cuja conta quer fazer
    ssh sem ter de fornecer a password, crie um ficheiro chamado
    *authorized_keys* com a chave pública do utilizador remoto (podia lá
    colocar mais chaves de outros utilizadores, uma por linha, se fosse
    o caso). O objectivo deste ficheiro consiste em indicar as entidades
    que estão autorizadas a entrar. Ou seja, indica que o computador
    deve aceitar comandos SSH de utilizadores que tenham acesso a uma
    chave privada correspondendo a uma das chaves públicas presentes no
    ficheiro *authorized_keys.*

10. Aceda da conta que tem a chave privada à outra correndo o comando
    *ssh:*\
```bash
ssh username@remote_host
```

11. Repare que não precisou de dar a *password* do utilizador remoto
    para entrar na sua área. Esse é precisamente o papel da fase de
    troca das chaves públicas. Se foi feita a troca de chaves e mesmo
    assim foi pedida a *password* é porque algo se passa de errado
    (p.ex., o servidor não está a correr ou as permissões não estão
    correctas).

12. Execute o comando *ssh* novamente mas dando a opção *-v*. Observe a
    informação mostrada. Note que essa informação pode ser útil para
    fazer *debug.*

## Exercício 2 -- Outras operações remotas

1.  O mesmo comando *ssh* pode ser usado para dar comandos remotamente.
    Para o efeito basta acrescentar o comando no fim da instrução. Por
    exemplo, pode criar um ficheiro vazio chamado aaa na pasta remota
    usando o comando:

```bash
ssh username@remote_host touch aaa
```

(se não sabe o que faz o comando *touch,* execute *man touch*).

2.  O comando *scp* pode ser usado para transferir ficheiros entre
    máquinas usando um protocolo SSH. Experimente copiar um ficheiro
    entre as duas VMs. A sintaxe é:

```bash
scp ficheiro_local username@remote_host:destination
```

onde *destination* é a pasta de destino.

## Exercício 3 -- Operações avançadas

Para estes exercícios não vamos usar o netkit, mas os servidores
sigma.tecnico.ulisboa.pt. Deverá executar os comandos num terminal no
seu PC. Cada exercício é um exercício separado.

1.  O SSH permite a execução remota de aplicações gráficas, para tal
    basta usar a flag -X. Execute o seguinte comando para se ligar a uma
    das máquinas sigma:

```bash
ssh -X istxxx@sigma.tecnico.ulisboa.pt
```

Após introduzir a sua password, fica ligado à máquina sigma, onde pode
correr aplicações. Vamos correr uma aplicação gráfica, emitindo o
comando "xpdf". Se tiver um ficheiro PDF na sua área pode indica-lo como
argumento. Vai aparecer uma janela com o programa gráfico xpdf no seu
PC, embora este esteja a correr no sigma. O SSH está a encaminhar os
gráficos através de um túnel, por onde é transmitido o protocolo X11.

2.  O SSH permite a criação de túneis por onde uma ligação TCP pode ser
    encaminhada de forma cifrada. Execute o seguinte comando num
    terminal no seu PC

```bash
ssh -L 8080:sigma02.tecnico.ulisboa.pt:22 istxxx@sigma01.tecnico.ulisboa.pt
```

Deixe esta ligação aberta e noutro terminal emita o comando:

```bash
ssh istxxxx@localhost -p 8080
```

O segundo comando efectua uma ligação ssh para o seu PC (localhost) no
entanto aparece ligado ao sigma02. Explique o que está a acontecer.

3.  O comando rsync permite efectuar a sincronização de ficheiros ou
    pastas entre máquinas. Efectue o seguinte comando:

```bash
rsync -e ssh -azvP <pasta local com ficheiro> istxxx@sigma.tecnico.ulisboa.pt:teste
```

O conteúdo da pasta local foi copiado para a pasta teste no sigma.
Altere um ficheiro na pasta local e volte a executar o comando.

O que aconteceu de diferente na segunda execução?

Faça "man rsync" e veja o significado das várias flags utilizadas.

Referências

-   Netkit, [http://wiki.netkit.org/][3]

-   Secure Shell, [http://en.wikipedia.org/wiki/Secure_Shell][4]

-   ssh-keygen manual, [http://linux.die.net/man/1/ssh-keygen][5]

  [3]: http://wiki.netkit.org/
  [4]: http://en.wikipedia.org/wiki/Secure_Shell
  [5]: http://linux.die.net/man/1/ssh-keygen
