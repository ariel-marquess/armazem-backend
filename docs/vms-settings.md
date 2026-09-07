# Configurando Máquinas Virtuais do Backend

Este projeto foi desenvolvido para uso em ambientes virtuais para simular o funcionamento de um servidor que utiliza separação de responsabilidades entre os dispositivos que compõem seu ecosistema. Dessa maneira, sendo executada a presente API (Application Programming Interface) em determida máquina, devemos objetivamente fornecer seu banco de dados em uma máquina diferente.

## Requisitos para configuração

- Possuir o programa VirtualBox instalado em sua máquina física;
- Ter duas máquinas virtuais devidamente configuradas no VirtualBox;
- Utilizar o sistema operacional [Debian](https://drive.google.com/drive/folders/18OHcenvD4wDcidbX7M_UCOB8A9IsVbcN?usp=sharing) nas máquinas virtuais.

> **Observação sobre a rede interna:**  
> Para o funcionamento esperado na rede interna, considere a seguinte configuração de IP:
>
> - **Máquina da API:** `192.168.0.3`
> - **Máquina do servidor de banco de dados:** `192.168.0.4`
>
> Certifique-se de que a configuração de rede das máquinas virtuais permite a comunicação entre ambas.

## Configuração do servidor de banco de dados

No terminal do Debian, siga os próximos passos para que possamos configurar a máquina virtual que servirá como servidor de banco de dados para a API.

### Instalando programas necessários

A configuração da máquina virtual inicia exclusivamente pela instalação do SGBD (Sistema de Gerenciamento de Banco de Dados). Vamos iniciar esta série de passos verificando e baixando novas atualizações nos pacotes do sistema operacional:

```bash
sudo apt update && sudo apt upgrade
```

Caso alguma atualização seja necessária, o terminal solicitará sua confirmação para realização das instalações; confirme e espere que o processo termine.

Vamos agora instalar o SGBD. Execute o seguinte comando: 

```bash
sudo apt install mariadb-server -y
```

Utilizamos aqui o sistema MariaDB pois o gerenciador de pacotes do Debian não fornece nativamente o sistema do MySQL. Todavia, não há nenhum prejuízo na utilização dessa alternativa para os objetivos deste projeto; o MariaDB foi construido utilizando as mesmas tecnologias do MySQL, dessa maneira o uso e as configurações que podem ser realizadas são análogas entre os sistemas.

### Verificando se o MariaDB está em execução

Para verificar se o SGBD está realmente em execução, realize o seguinte comando: 

```bash
systemctl status mariadb
```

Caso seja constatada a não execução do sistema, execute os seguintes comandos:

```bash
systemctl start mariadb && systemctl enable mariadb
```

Isso ativa o servidor para ocasião e habilita seu início para sempre que o programa for executado.

### Configurando permissões de conexão

Por padrão o serviço do MariaDB é executado sempre e exclusivamente no endereço IP **127.0.0.1**. Como precizaremos que a máquina do servidor de banco de dados se comunique com a máquina da API, devemos modificar tal paradigma.

Abra o seguinte arquivo:

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Este arquivo possuirá uma variável chamada "bind-address" definida com o valor "127.0.0.1". Troque seu valor por "0.0.0.0":

```text
bind-address = 0.0.0.0
```

Isso permitirá conexões com qualquer endereço IP. Agora, como última fase desta etapa, reinicie o serviço:

```bash
sudo systemctl restart mariadb
```

### Criando usuário com permissão remota

Precisamos acessar a interface de gerenciamento do MariaDB; para isso, execute o seguinte comando:

```bash
mariadb -u root -p
```

O programa solicitará a senha do usuário para permitir seu acesso; no usuário root normalmente essa senha é **"123"** ou **"1234"** (Obs.: estamos utilizando o usuário root, mas objetivamente poderia ser utilizado qualquer usuário cadastrado no programa).

Vamos dar permissão ao usuário remoto de um banco de dados em específico, então precisamos criá-lo, antes de mais nada. Já na interface do MariaDB, execute o seguinte comando:

```sql
CREATE DATABASE nome_do_banco;
```

Agora sim, vamos criar o usuário de permissão remota. Execute os seguintes comandos na interface do MariaDB:

```sql
CREATE USER 'nome_do_usuario'@'192.168.0.3' IDENTIFIED BY 'senha_a_ser_criada';
```

```sql
GRANT ALL PRIVILEGES ON nome_do_banco.* TO 'nome_do_usuario'@'192.168.0.3';
```

```sql
FLUSH PRIVILEGES;
```

Veja que utilizamos o endereço IP **192.168.0.3**, que prevenimos anteriormente ser o endereço da máquina virtual que abrigará a API. Você poderia utilizar qualquer outro endereço IP, porém, caso seu objetivo seja o mesmo que o nosso, sendo o endereço da máquina virtual da API na rede interna do ecosistema do VirtualBox.

## Configuração do servidor da API

Da mesma maneira que iniciamos a configuração do servidor de banco de dados, devemos aqui também iniciar verificando e instalando novas atualizações: 

```bash
sudo apt update && sudo apt upgrade
```

Para baixar a interface do cliente (aquele que recebe as informações) do MariaDB, devemos executar o seguinte comando:

```bash
sudo apt install mariadb-client -y
```

Agora, para testar a conexão entre servidor e cliente, execute o seguinte comando:

```bash
mariadb -h 192.168.0.4 -P 3306 -u nome_do_usuario -p
```

Observe que utilizamos o endereço IP **192.168.0.4** porque havíamos o definido como endereço do servidor do banco de dados. Agora você já pode utilizar a conexão com o banco de dados tendo a devida separação de responsabilidades.

**Aviso**: tanto o usuário quanto o banco de dados utilizados pela API devem ser exatamente aqueles cuja permessão foi dada pelo servidor.