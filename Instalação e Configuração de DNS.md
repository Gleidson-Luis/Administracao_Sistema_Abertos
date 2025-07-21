# Instalacao-e-configuracao-de-DNS

## ⌛ Intalação do Servidor DNS

1. Abrir o terminal e digitar
```
$ sudo nano /etc/hosts
```
e apagar as entradas deixando o arquivo padrão: 127.0.0.1  site1 (alterar para localhost), 127.0.0.1  site2, 127.0.0.1  aluno e
127.0.0.1  aluno.
e Salva.

2. Atualiza os repositórios:
```
$ sudo apt-get update
```
3. Instala o servidor DNS:
```
$ sudo apt-get install bind9 -y
```
4. Verificar se está rodando:
```
$ sudo /etc/init.d/named status
```
5. Antes de configurar o DNS, fixar o ip na máquina em configurações de conexão cabeada
```
IPv4: 192.168.0.10
Máscara: 255.255.255.0
Gatwey: 192.168.0.1
DNS: 192.168.0.10
```
e reinicia a conexão

## 🚀 Configuração do Servidor DNS

6. No terminal precisa acessar os arquivos do Bind, digite:
```
$ cd /etc/bind
```
7. Antes de prosseguir com a configuração, fazer backup alterando o arquivo: "named.conf.local" para evitar problemas futuros que possa voltar caso aconteça algo que não deu certo.
```
$ sudo cp named.conf.local named.conf.localOLD
```
Feito isso agora pode editar o arquivo "named.conf.local" no terminal:
```
$ sudo nano named.conf.local
```
8. Vai abrir o arquivo de configuração.
9. Primeira coisa a fazer é configurar uma "Zona Direta" e logo após a "Zona Reversa"
```
//Zona Direta
zone "ifrn.com" {
    type master;
    file "/etc/bind/db.ifrn.com";
}:

//Zona Reversa
zone "192.168.0.-in.addr.arpa" {
    type master;
    file "/etc/bind/db.reverso10
}:
```
Salvar

10. Editar os arquivos DB (db.local e db.127) fazendo uma cópia para facilitar a configuranção
```
$ sudo cp db.local db.ifrn.com
$ sudo cp db.127 db.reverso10
```
11. Agora editar os arquivos DB criados
```
$ sudo nano db.ifrn.com
```
12. Com o arquivo aberto, trocar onde tem nome "localhost" por "ifrn.com" e substituir as linhas:
```
0    IN    NS    ifrn.com
0    IN    A     192.168.0.10
```
Salvar

13. Agora edita o "Reverso10"
```
$ sudo nano db.reverso10
```
14. Com o arquivo aberto, onde tem "localhost",, colocar "ifrn.com"
```
;BIND reverse data file for local loopback interface
;
;
$TTL     604800
@        IN        SOA        ifrn.com. root.ifrn.com. (
                                    1        ; Serial
                               604800        ; Refresh
                                86400        ; Retry
                              2419200        ; Expire
                               604800 )      ; Negative Cache TTL
;
@        IN        NS        ifrn.com.
10       IN        PTR       ifrn.com.
```
Salvar

15. Agora ir no "resolv.conf"
```
$ sudo nano /etc/resolv.conf
```
16. Adiciona o Servidor
```
nameserver 192.168.10
nameserver 127.0.0.53
options edns0 trust-ad
search .
```
Salvar e reiniciar
```
$ sudo  /etc/init.d/named restart
```
17. Pode verificar
```
$ nslookup ifrn.com
```
Irá retornar o IP que está responsável

18. Abrir o navegador e colocar o site: ifrn.com.  Abrindo o site, esta tudo ok

19. Para que seja possível acessar os outros sites criados. Dentro da zona direta
```
$ sudo nano db.ifrn.com
```
E nas linhas abaixo adiciona os site
```
site1        IN        A        192.168.0.10
site2        IN        A        192.168.0.10
```
Salvar

OBSERVAÇÃO IMPORTANTE: Tem que editar o arquivo .conf para indicar o domínio dele lá do Apache
```
$ cd /etc/apache2/sites-available/
$ sudo nano site1.conf
```
Editar o final do arquivo
```
ServerAdmin admin@site1.ifrn.com
ServerName site1.ifrn.com
ServerAlias site1.ifrn.com
```
Salvar e fazer o mesmo para o "Site2"

20. Dar um restar no Apache
```
$ sudo /etc/init.d/apache2 restart
```
21. Dar um restart no DNS
```
$ sudo /etc/init.d/named restart
```
22. Abrir o navegador e testar o "http://site1.ifrn.com" e o "http://site2.ifrn.com". Abrindo os sites a tarefa está concluida.
