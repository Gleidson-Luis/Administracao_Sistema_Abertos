# Configurar o Apache2

1. Para criar multiplos sites no Servidor Apache. Nesse exemplos vamos colocar 2 sites para isso vamos criar 2 diretorios:
```
$ cd /var/www
```
2. Criar os diretórios:
```
$ sudo mkdir -p site1/public_html
$ sudo mkdir -p site2/public_html
```
3. Configurar essas pastas que fiquem acessiveis ao usuário sem ter permissão do sudo:

```
$ sudo chown -R $USER:$USER /var/www/site1/public_html
# sudo chown -R $USER:$USER /var/www/site2/public_html
```
4. Criar os arquivos (páginas web) vamos acessar os diretórios
```
$ cd /var/www/site1/public_html
```
5. Criar um arquivo HTML
```
$ sudo nano index.html
```
Criar tanto o Site1 e o Site2
```
<html>
<head>
<title>Pagina do site 1</title>
</head>
<body>
<center>
<h1>Site 1</h1>
</body>
</html>
```
6. Configuração para que o Apache consiga encontrar o Site1 e Site2. Acessar o diretório
```
$ cd /etc/apache2/sites-available
```
7 Fazer uma cópia do arquivo padrão (.conf)
```
$ sudo cp 000-default.conf site1.conf
$ sudo cp 000-default.conf site2.conf
```
8. Apos copiar, editar o arquivo
```
$ sudo nano site.conf
```
Mudar as linhas
```
ServerAdmin admin@site1
ServerName site1
ServerAlias www.site1
DocumentRoot /var/www/site1/public_html
```
Salvar e fazer a mesma coisa com o Site2
9. Para finalizar precisa ativar no apache essas configurações
```
$ sudo a2ensite site1.conf
$ sudo a2ensite site2.conf

```
10. Desabilitr o site padrão
```
$ sudo a2dissite 000=default.conf
```
11. Reiniciar o apache
```
$ sudo systemctl restart apache2
```
12. Renomear no arquivo host para quando digitar o site1 ou site2, ir direto na página
```
$ sudo nano /etc/hosts
```
Editar o aquivo
```
127.0.0.1    site1
127.0.0.1    site2
127.0.0.1    usuario-VirtualBo
