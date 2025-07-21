# ![ClipWindowsGIF](https://github.com/user-attachments/assets/d89e53a8-fbe6-4f9a-a5a8-362b9c188c3f)
# 🚀 Instalacao-Webmin

1. Fazer o Download com o seguinte comando:
```
$ wget https://www.webmin.com/download/deb/webmin-current.deb
```
2. Descompactar o arquivo baixado
```
$ sudo dpkg -i webmin-current.deb
```
Aparecerão alguns erros de dependências, mas é normal

3. Seguir com o comando para fazer o processo de instalação:
```
$ sudo apt -f install -y
```
4. Abre o navegador e digitar:
```
localhost:100000
```
Tudo dando certo vai aparecer uma mensagem de alerta, clica em "Avançado" e depois em "Aceito o risco e continuar"
Vai abrir uma janela pedindo usuário e senha (a mesma utilizada no computador). 
Ele vai mostrar um interface gráfica mostrando os servidores, os sites que estão sendo utilizados, as zonas de DNS.
