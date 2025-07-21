# 🖥️📲 Utilizando-dois-servidores-com-Apache-e-DNS

1. 🚀 Instalar uma nova Máquina Virtual para se conectar ao servidor criado em aulas anteriores
2. 🚀 Instalar o Apache na nova máquina virtual
```
$ sudo apt-get update
$ sudo apt-get install apache2 -y
```
3. Verificar se está funcionando corretamente
```
$ sudo /etc/init.d/apache2 status
```
4. 🧩 Entrar no diretório
```
$ cd /var/www/
```
Criar um novo diretório
```
$ sudo mkdir -p ufrn/public_html
```
Entrar nesse novo diretório criado
```
$ cd ufrn/public_html
```
5. ✍ Criar um html simples nesse diretório e criar as tags padrões
```
$ sudo nano index.html
```
Editar o arquivo colocar as tags padrão
```
<html>
<head><title>Olá seja bem vindo ao UFRN</title>
</head>
<body>
<center>
<h1>Bem vindo ao site do UFRN</h1>
</body>
</html>
```
Salvar

6. ✍ Criar o arquivo de configuração desse site
```
$ cd /etc/apache2/sites-available/
```
7. 🖨 Fazer uma cópia do padrão
```
$ sudo cp 000-defaut.conf ufrn.conf
```
8. ⚒ Editar o arquivo
```
$ sudo nano ufrn.conf
```
9. Definir os parâmetros
```
ServerAdmin admin@ufrn.com
ServerName ufrn.com
ServerAlias www.ufrn.com
DocumentRoot /var/www/ufrn/public_html
```
Salvar

10. Reiniciar o servidor e habilitar o "ufrn.com" e desabilitar o "000-default.conf" (sempre dando enter após cada comando)
```
$ sudo /etc/init.d/apache2 restart
$ sudo a2ensite ufrn.conf
$ sudo a2dissite 000-default.conf
$ sudo /etc/init.d/apache2 restart
```
11. Após isso, editar as configurações de rede na VM2 colocando o IP manual IP: 192.168.0.11; Mascara: 255.255.255.0 e Gateway: 192.168.0.1.
O DNS ficará desmarcado a opção automático e indicar o DNS do outro servidor que tem o Bind instalado que é o: 192.168.0.10. Salva e aplica essas configurações.

12. Desligar a VM2 e colocá-la em uma "Rede Interna" nas configurações do Virtual Box
13. Na VM1, vamos fazer algumas configurações no DNS
14. Na VM1 criar alguns arquivos
```
$ cd/etc/bind/
```
15. Copiar o arquivo "db.ifrn.com" e alterar o arquivo "db.reverso10" e no "named.conf.local
```
$ sudo cp db.ifrn.com db.ufrn.com
```
16. Alterar esse arquivo
```
$ sudo nano db.ufrn.com
```
17. Onde tem "ifrn" trocar por "ufrn"
18. Trocar os endereços: Onde tem 192.168.0.10 por 192.168.0.11 e excluir as linhas "Site1" e "Site2"
Salvar

19. Editar o arquivo "named.conf.local" e configurar uma nova Zona Direta
```
//Zona Direta
zone "ufrn.com" {
    type master;
    file "/etc/bind/db.ufrn.com";
};
```
Salva e reinicia
```
$ sudo /etc/init.d/named restart
```

20. Editar o arquivo "db.reverso10"
```
$ sudo nano db.reverso10
```

21. Adicionar uma nova linha
```
11    IN    PTR    ufrn.com.
```
Salvar e reiniciar

22. Desligar a VM1 e colocá-la em uma "Rede Interna" nas configurações do Virtual Box

23. Com as VMs ligadas, dar um ping em cada ip para verificar se as máquinas estão respondendo. Estando tudo ok, abrir o navegador e verificar se os sites estão abrindo nas duas máquinas VMs.

24. Estando tudo ok, a tarefa está concluída!
