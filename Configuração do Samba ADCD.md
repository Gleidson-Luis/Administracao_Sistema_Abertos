# Configuração do Samba ADCD

1. Na Máquina Virtual, nomeada como "SAMBA ADCD", acesssar as configurações, nas interfaçes de redes criar um terceiro adaptador para se comunicar com o hospedeiro (máquina real) habilitando como "Placa de rede exclusiva de hospedeiros (host-only)", e fechar as configurações dando "OK"
2. Para verificar se está habilitado é só clirar em Ferramenas > Rede, e na janela "Redes Exclusivas de Hospedeiro (Host-only), verificar na opção abaixo se está marcado "Configurar Adaptador Automaticamente". Se não tiver marcada você marca.
Feito isso, ligar a máquina virtual.

3. Para conseguir acessar via SSH, fazer a instalação
```
$ sudo apt-get update
$ sudo apt-get install openssh-server -y
```
4. Importante: Como adicionou um  novo adaptador e está configurando as interfaces através do Netplan, a gente precisa acessá-lo para fazer essas configurações
5. Identificar qual o nome da interface
```
$ ip -br a
```
6. Identificar a nova interface que geralmente é "enp0s9" ou a que não está recebendo o ip
7. Acessar o Netplan para configurar
```
$ cd /etc/netplan
```
8. Abrir o arquivo de configuração usando o editor "nano" ($ sudo nano /etc/netplan/......)
9. Configurar o arquivo conforme abaixo:
```
# let NetwoerkManager manage all devices on this system
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      addresses: [192.168.10.2/24]
    enp0s9:
      dhcp4: true
      nameservers:
        addresses: [8.8.8.8]
```
10. Salvar as configurações e na sequencia digitar o comando:
```
$ sudo netplan apply
```
Para aplicar as configurações.
11. Para verificar os endereços definidos para a nova interface de rede que servirar da comunicação da mãquina do sampa para o windows usando o powershell
```
$ ip -br a
```
12. Acessar o powershell no windows e dar o comando: ssh (usurário)@(ip definido) para acessar o samba através da máquina windows

13. Verificar o nome do host e mudar o nome desse host (após cada comando dar um enter)
```
$ hostname
$ sudo hostnamectl set-hostname dc
```
14. Alterar os arquivos de host para definir o DNS para que possa enxegar o "DC" como DNS central, editando o aquivo de hosts
```
$ sudo nano /etc/hosts
```
```
127.0.0.1    localhost
127.0.1.1    samba
(IP SERVIDOR LOCAL)  dc.samba.local dc
(IP SERVIDOR LOCAL) samba.local samba

# The following lines are desirable for IPv6 capable hosts
::1      ip6-localhost ip6-loopback
fe00::0  ip6-localnet
ff00::0  ip6-mcastprefix
ff02::1  ip6-allnodes
ff02::2  ip6-allrouters
```
Salvar o arquivo e verificar se tudo está com o nome completo do domínio
```
$ hostname -f
```
15. Verificar se esse domínio totalmente qualificado é capaz de resolver ip é só dar um ping para o endereço completo:
```
$ ping -c2 dc.samba.local
```
Retornando o ping está tudo ok

16. Desativar de forma imediata o serviço do system resolved e configurar o dns de forma manual criando um novo arquivo(após cada comando dar um "enter")
```
$ sudo systemctl disable --now systemd-resolved
$ sudo unlink /etc/resolv.conf
$ sudo nano /etc/resolv.conf
```
```
nameserver    192.168.10.2
nameserver    8.8.8.8
search        samba.local
```
Salvar o arquivo

17. Tornar esse arquivo inalterável para impedir que venha a ser modificado por algum programa
```
$ sudo chattr +i /etc/resolv.conf
```
18. Instalar o samba com todas as suas dependências
```
$ sudo apt-get update
$ sudo apt install -y acl attr samba samba-dsdb-modules samba-vfs-modules smbclient winbind libpam-winbind libnss-winbind libpam-krb5 krb5-config krb5-user dnsutils chrony net-tools
```
Durante a instalação aparecerá essa janela
<img width="958" height="512" alt="image" src="https://github.com/user-attachments/assets/9ce406c3-8e8c-4077-ac60-2ae8abf1af72" />
No campo deve conter o título: SAMBA.LOCAL, aperta "OK"
Na sequencia digitar o servidor Kerberos: dc.samba.local
<img width="962" height="506" alt="image" src="https://github.com/user-attachments/assets/9cc09126-5d52-41d2-b1d9-9bbe0b72e899" />
Repetir novamente na próxima janela: dc.samba.local
<img width="963" height="513" alt="image" src="https://github.com/user-attachments/assets/ba0f7ec8-3144-44c4-b0ca-72c4fd23caa5" />
Após isso irá concluir a instalação.

19. Desabilitar alguns serviços do samba que não seráo necessários nesse momento
```
$ sudo systemctl disable --now smbd nmbd winbind
```
20. Desbloquear a execução dos modulos removendo o bloqueio e habilitar para fazer com que o servidor se torne o controlador do domínio
```
$ sudo systemctl unmask samba-ad-dc
$ sudo systemctl enable samba-ad-dc
```
# Configuração do Active Direct
21. Fazer uma cópia de segurança do arquivo padrão para deixar reservado
```
$ sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.orig
```
22. Rodar o comando "samba-tools" para começar a provisionar o samba do Active Direct
```
$ sudo samba-tool domain provision
```
```
Realm [SAMBA.LOCAL]: (enter)
Domain [SAMBA]: (enter)
Server Role (dc, member, standalone) [dc]: (enter)
DNS backend (SAMBA-INTERNAL, BIND9_FLATFILE, BIND9_DLZ, NONE) [SAMBA_INTERNAL]: (enter)
DNS forwarder IP address (write 'none' do disable forwarding) [192.168.10.2]: 8.8.8.8 (enter)
Administrator password: (cria senha para o ad) - Recomendado a mesma senha do servidor
Retype password: (repetir a senha)
```

23. Criar uma cópia de segurança do Kerberos e substituir com um novo
```
$ sudo mv /etc/krb5.conf /etc/krb5.conf.orig
$ sudo  cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
```

24. Rodar o samba addc
```
$ sudo systemctl start samba-ad-dc
```
Verificar se o arquivo está rodando
```
$ sudo systemctl status samba-ad-dc
```
Apresentando no Active: active (running), significa que está rodando.

25. O protocolo Kerberos que faz com que os horarios do AD sejam sicronizados para comunicar as maquinas no tempo de maneira adequada. Para garantir a sincronização tem que configurar um servidor NTP (Network Time Protocols) e primeiro tem que alterar algumas permissões dos arquivos que já são criados juntos das bibliotecas instaladas
```
$ sudo apt-get update
$ sudo chown root:_chrony /var/lib/samba/ntp_signd/
# sudo chmod 750 /var/lib/samba/ntp_signd/
```
26. Alterar o arquivo de configuração para inserir os dados do servidor
```
$ sudo nano /etc/chrony/chrony.conf
```
No final do arquivo adicionar as seguintes linhas
```
bindcmdaddress 192.168.10.2
allow 192.168.10.0/24
ntpsigndsocket /var/lib/samba/ntp_signd
```
Salvar o arquivo

27. Reiniciar o serviço do Chrony
```
$ sudo systemctl restart chronyd
```
28. Verificar o status
```
$ sudo systemctl status chronyd
```
Active: running, está ok

29. Verificar o host do samba
```
$ host -t A samba.local
$ host -t A dc.samba.local
```
Os 3 IPs tem que está respondendo e tudo funcionando

30. Verificar se os registros Kerberos
```
$ host -t SRV _kerberos._udp.samba.local
$ host -t SRV _ldap._tcp.samba.local
```
31. Verificar os recursos padrões que estarão disponíveis no samba
```
$ smbcliente -L samba.local -N
```
Vai mostrar os usuários que estão no samba

32. Verificar a autenticidade do kerberos
```
$ kinit administrator@SAMBA.LOCAL
```
Digita a senha do servidor
```
$ klist
```
Lista todos os usuários

33. Fazer o login no servidor através do SMB
```
$ sudo smbclient //localhost/netlogon -U 'administrator'
Password for [SAMBA\administrator]:
```
smb:\>
Agora consegue acessar o smb
Podendo alterar a senha do administrador
```
$ sudo samba-tool user setpassword administrator
New Password:
Retype Password:
Changed password ok
```
34. Posso verificar a integridade desse arquivo de configuração do samba
```
$ testparm
```
35. Posso verificar o funcionamento do domínio
```
$ sudo samba-tool domain level show
```
36. Posso criar um usuário no samba
```
$ sudo samba-tool user create (nome do usuario)
New Password:
Retype Password:
```
37. Posso ver a lista de usuários
```
$ sudo samba-tool user list
```
38. Posso excluir usuários
```
$ sudo samba-tool user delete (nome do usuario)
```
39. Posso listar os computadores
```
$ sudo samba-tool computer list
```
40. Posso remorer computadores
```
$ sudo samba-tool computer delete (nome do computador)
```
41. Posso criar grupos
```
$ sudo samba-tools group add (nome do grupo)
```
42. Para listar os grupos
```
$ sudo samba-tool group list
```
43. Posso remover grupos, atribuir membros aos grupos e listar os membros desses grupos
```
$ samba-toll group limembers (nome do grupo)
```
Adicionar
```
$ sudo samba-tool group addmembers (nome do grupo) (nome do usuário)
```
Listar membros do grupo
```
$ sudo samba-tool group listmembers (nome do grupo)
```
Eliminar membros
```
$ sudo samba-tool troup removemembers (nome do grupo) (nome do usuário)
```
44. Atrelar um computador ao AD
44.1. Abrir a máquina windows e verificar primeiro se está enchergando o domínio abrindo o CMD
```
ping 192.168.10.2
```
Verificar também o samba.local
```
ping samba.local
```
Não havendo resposta vai nas configurações de rede e internet no windows > Ethernet> Alterar opões de adaptador.
Botão direito do mouse > Propriedades > Protocolo IP Versão 4 (TCP/IPv4) > Propriedades e inserir o IP [192.168.10.2] no servidor DNS Preferencial e no Gateway padrão.
Fechar tudo e voltar ao CMD e pingar
```
ping samba.local
```
Se pingar está resolvido.

# Para manter uma boa organização

45. Alterar o nome do computador (Na máquina windows), digitar na barra de pesquisa: "grupo de trabalho" e clicar em "Alterar nome do grupo de trabalho". Ja nela que abrir alterar primeiro o nome do computador para: "Cliente-Windows" e reinicia a máquina windows.

46. Após reiniciar voltar para "Alterar nome do grupo de trabalho" e selecionar "Domínio" para inserir o nosso domínio: samba.local. Vai abrir uma janela solicitando as credenciais Login: administrator e Senha: (do servidor samba), estando tudo ok, ele vai aparecer uma mensagem informando que entrou no domínio. Vai reiniciar a máquina windows

47. Após reiniciar será possível entar pc usando o usuário cadastrado no servidor samba, onde na tela de login clica em "Entrar com outro domínio" > Usuário: samba.loca\(usuário) e digita a senha criada para esse usuário.

Estando tudo ok ele vai acessar o usuário no samba ingressando essa máquina windows em um servidor samba.

Fim da documentação.
