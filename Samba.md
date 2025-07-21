# Protocolo SMB
O SMB é o protocolo que atua no nível de aplicação que os clientes conectam ao servidor e, a partir deste, podem ler e escrever arquivos e fazer o uso de recursos compartilhados.

# Servidor SAMBA
Permite que máquinas Linux e Windows se comuniquem entre si compartilhando serviços (arquivos, diretório, impressão) através do protocolo SMB/CIFS. E uma das soluções em ambiente UNIX capax de interligar redes heterogêneas.

# Instalar o SAMBA
```
$ sudo apt-get update
$ sudo apt-get install samba -y
```
1. Criar um usuário sem diretório (Sem home e login no sistema)
```
$ sudo adduser --disabled-login -no-create-home joao
$ sudo adduser --disabled-login -no-create-home maria
```
2. Criar 2 grupos
```
$ sudo addgroup alunos
$ sudo addgroup professores
```
3. Adicionar os usuários aos grupos (João no grupo de professores e Maria no grupo de alunos)
```
$ sudo usermod -a -G professores joao
@ sudo usermod -a -G alunos maria
```
4. Criar um diretório pai para servir como recurso para fazer a associar esse diretório a esses usuários ou a esses grupos
```
$ sudo mkdir samba
```
5. Atribuir uma permissão total ao Root (dono do grupo), porém os demais só leitura e execução
```
$ sudo chmod 775 samba
$ ls -l
```
6. Entrar no diretório "samba" e criar 2 sub-diretórios, sendo, um diretório para os professores e outro para os alunos
```
$ cd samba
$ sudo mkdir professores
$ sudo mkdir alunos
```
7. Nesse diretório vamos mudar as permissões tanto de aluno como professores, tirando as permissões
```
# sudo chmod 770 alunos
$ sudo chmod 770 professores
```
8. Permitir que cada pasta tenha o seu respectivo dono
```
$ sudo chown -R root:alunos alunos/
$ sudo chown -R root:professores professores/
```
9. Criar uma senha de acesso aos usuários de compartilhamento
```
@ sudo smbpasswd -a maria
New SMB passowrd:
Retype SMB password
add user ***
```
Repetir o mesmo passo para os outros usuários.

10. Fazer a configuração no arquivo smb.conf fazendo um backup de segurança e criando um novo arquivo
```
$ cd /etc/samba
$ sudo mv smb.conf bkp.smb.conf
$ sudo nano smb.conf
```
11. Inserir as configurações
```
[global]
        netbios name = servidorSamba
        workgroup = WORKGROUP

[arquivos]
        path = /home/samba
        writeable = yes
        available = yes
        force group = alunos
        force group = professores
        create mask = 0770
        directory mask = 0770
        valid user = @alunos, @professores
```
Salvar e restartar o serviço
```
$ sudo systemctl restart smbd.service
```
12. Fazer o teste para verificar se as configurações estão corretas
13. Identificar o IP do linux
```
$ ip -br a
```
14. No windows acessar os usuários através do Windows Explorer na bara de endereço digitar: \\(ip do linux). Poderá solicitar o usuário e senha. Após isso vai abrir os arquivos, após isso poderá criar nova pasta e compartilhar arquivos.

15. Fim da prática
