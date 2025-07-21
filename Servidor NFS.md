# Servidor NFS
Permite que monte uma pasta em um servidor para usar como um disco local do sistema.

# Instalação e Configuração
```
$ sudo apt-get update
$ sudo apt-get instal nfs-kernel-server -y
```
1. Criar e modificar o diretorio que vai ser utilizado para compartilhamento e alterar o dono do grupo e dar permissão total:
```
$ sudo mkdir -p /srv/nfs/publico
$ sudo chown nobody:nogroup /srv/nfs/publico
$ sudo chmod 777 /srv/nfs/publico
```
2. Permitir a disponiblidade do servidor
```
$ sudo nano exports
```
Criar uma nova linha
```
/srv/nfs/publico (ip de rede do servidor/mascara)(rw, sync, no_subtree_check)
```
Salvar
3. Exportar e reiniciar o serviço
```
$ sudo exportfs -a
$ sudo systemctl restart nfs-kernel-server
```
5. Instalar o cockpit
```
$ sudo apt-get install cockpit -y
```
6. Habilitar o serviço
```
$ sudo systemctl enable --now cockpit.socket
```
7. Abrir o navegador linux para verificar se está funcionando
```
hocalhost:9090
```
8. Colocar usuario e senha cadastrado no linux e vai abrir a interface gráfica do cockpit.

9. Em Armazenamento montar a NFS
Endereço do servidor = (IP da máquina) /
Caminho do servidor = /srv/nfs/publico /
Ponto de montagem local = /mnt/nfs_publico

10. Acessar o windows e acessar por era
11. Acessar o painel de controle > Programas e Recursos > Ativar e desativar recursos do Windows e habilitar em Serviços de NFS e marcar "Client for NFS", dá OK e esperar o windows instalar o serviço.
12. Acessar o PowerShell e digitar o comando:
```
cmd /c "mount \\(ip da máquina)\srv\nfs\público Z: -o anon"
```
13. Abrir o Windows Explorer e conferir se a unidade Z está compartilhada em rede
14. Fim da prática
