# Instalacao_Servidor_Apache2

1. Primeiramente faz a atualização do sistema:
```
$ sudo apt-get update
```
2. Logo após fazer a instalação do Apache2:
```
$ sudo apt-get install apache2 -y
```
3. Após a instalação verificar o status do servidor:
```
$ /etc/init.d/apache2 status
```
4. Estando rodando pode verificar se ja tem algum site de exemplo pelo navegador, mas para isso tem que saber qual o IP da máquina
```
$ ip a
```
5. Só digitar no navegador o ip da máquina, onde irá abrir um site de exemplo do Apache
6. Para fazer um teste com outro site:
```
$ cd /var/www/html
```
7. Criando uma nova página de html
```
$ sudo nano index.html
```
Editar:
```
<html>
<head><title>Título</title></head>
<body>
<center>
<h1>Bem vindo a minha primeira página HTML</h1>
</body>
</html>
```
Salvar

8. Para carregar outros site onde o html provem do github, primeiramente tem que instalar o Git para fazer um clone do repositório no GitHub

```
$ sudo apt-get install git -y
```
9. Ir no GitHub e copiar o endereço
```
$ sudo git clone (link)
```
