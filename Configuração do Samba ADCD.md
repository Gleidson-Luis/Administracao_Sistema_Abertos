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

Fim da documentação
