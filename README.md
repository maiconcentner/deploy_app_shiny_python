# Guia de Deploy de Aplicativo Shiny em Python no Linux Mint

Este guia fornece instruções passo a passo para realizar o deploy de um aplicativo Shiny em Python no Linux Mint.

## 1. Instalação do Servidor Shiny

Certifique-se de que seu sistema está atualizado antes de começar.

```bash
$ sudo apt-get update
$ sudo apt-get install gdebi-core
$ wget https://download3.rstudio.org/ubuntu-14.04/x86_64/shiny-server-1.5.16.958-amd64.deb
$ sudo gdebi shiny-server-1.5.16.958-amd64.deb
```
## 2. Configurar o Servidor

É possível que seja necessário configurar o servidor, dependendo das especificidades do seu computador. Em alguns casos, essa configuração pode ser dispensada. No meu caso, não foi necessário configurar para apontar para o Python. Recomendo tentar rodar o app sem fazer essa configuração inicialmente. Se ocorrer algum erro, volte aqui.

1. Abra o arquivo de configuração do Shiny Server com um editor de texto. Utilizarei o nano, mas você pode escolher qualquer editor de sua preferência.

```bash
$ sudo nano /etc/shiny-server/shiny-server.conf
```
Adicione a linha que aponta para o seu interpretador Python ou ambiente virtual. Supondo que você esteja usando o Python 3, a linha pode ser algo como:

```bash
# Use Python 3
$ python /usr/bin/python3;
```
Ou, se estiver usando um ambiente virtual:
```bash
# Use o ambiente virtual
$ python /srv/shiny-server/python39-venv/bin/python;
```
Salve e saia do editor (no nano, você pode fazer isso pressionando Ctrl + X, seguido de Y, e depois Enter).
O resultado deve se parecer com algo assim:

```bash
# Use system python3 to run Shiny apps
python /usr/bin/python3;

# Instruct Shiny Server to run applications as the user "shiny"
run_as shiny;

# Define a server that listens on port 3838
server {
  listen 3838;

  # Define a location at the base URL
  location / {

    # Host the directory of Shiny Apps stored in this directory
    site_dir /srv/shiny-server;

    # Log all Shiny output to files in this directory
    log_dir /var/log/shiny-server;

    # When a user visits the base URL rather than a particular application,
    # an index of the applications available in this directory will be shown.
    directory_index on;
  }
}
```
Essa configuração aponta para o interpretador Python desejado. Salve as alterações e continue com o próximo passo do seu guia de implantação.
Lembre-se de ajustar as configurações conforme necessário, dependendo do seu ambiente e versão do Python.

## 3. Testar o Servidor

Após modificar o arquivo de configuração (/etc/shiny-server/shiny-server.conf) no passo anterior, primeiro, reinicie o servidor:

```bash
$ sudo systemctl restart shiny-server
```
Agora, verifique o status do servidor:

```bash
$ sudo systemctl status shiny-server
```
Se não aparecer nenhum erro, está tudo certo! O servidor está funcionando.

Se ocorrer algum erro, consulte o log gerado para identificar a causa do problema. Se você modificou o arquivo para apontar para o Python, há chances de que o caminho informado não esteja correto. Nesse caso, recomendo remover a linha que você inseriu, salvar e fechar o arquivo. Após isso, execute novamente o passo 3.

Para verificar o conteúdo do log, digite no terminal:
```bash
$ cat /var/log/shiny-server/shiny-server.log
```
Isso exibirá o conteúdo do arquivo de log, onde você poderá encontrar informações sobre possíveis erros ou problemas. Ajuste as configurações conforme necessário e repita os passos até que o servidor esteja funcionando corretamente.

## 4. Mover Seu Projeto Shiny para o Servidor

Seu projeto Shiny, juntamente com todos os arquivos necessários para seu funcionamento, deve estar localizado no diretório `/srv/shiny-server/nome-do-seu-projeto`.

Crie subdiretórios para cada aplicativo. Por exemplo:

```bash
/srv/shiny-server/meu-app/app.py
/srv/shiny-server/outro-app/app.py
```

Você pode mover ou copiar seus arquivos para o diretório mencionado.

No mesmo diretório do seu projeto `app.py`, crie um arquivo chamado `requirements.txt`.

No conteúdo do arquivo, liste os pacotes utilizados no projeto, incluindo suas versões, se necessário. Por exemplo:

```plaintext
scipy==1.8.0
pandas>=1.4.1
numpy
shiny
```
Coloque uma linha para cada pacote utilizado. Salve e feche o arquivo.

Isso ajuda a garantir que as dependências necessárias para o seu projeto estejam instaladas no ambiente do Shiny Server. Certifique-se de fornecer versões específicas de pacotes se a compatibilidade for uma preocupação.

Agora, seu projeto está pronto para ser servido pelo Shiny Server. Continue com os próximos passos do guia conforme necessário para concluir o deploy.


## 5. Instalação do Docker (Opcional, mas Recomendado)

Recomendo a instalação do Docker para facilitar a gestão e isolamento de ambientes. Siga os passos abaixo para instalar o Docker:

1. Atualize o índice de pacotes:

```bash
$ sudo apt-get update
```
Instale as dependências necessárias para permitir o uso de repositórios via HTTPS:
```bash
$ sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
Adicione a chave GPG oficial do Docker:
```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
Adicione o repositório Docker:
```bash
$ echo "deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Instale o Docker Engine:
```bash
$ sudo apt-get update
$ sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```
Verifique a instalação:
```bash
$ sudo docker --version
```
Caso surja algum erro durante a instalação, você pode fornecer o erro ao Chat GPT para obter ajuda na resolução.

Se desejar refazer a instalação do Docker utilizando um tutorial específico, desfaça o processo com os seguintes comandos:

```bash
$ sudo apt-get purge -y docker-ce docker-ce-cli containerd.io
$ sudo rm -rf /var/lib/docker
$ sudo rm -rf /etc/docker
$ sudo rm -rf /usr/share/keyrings/docker-archive-keyring.gpg
$ sudo rm /etc/apt/sources.list.d/docker.list
$ sudo apt-get purge -y apt-transport-https ca-certificates curl gnupg
```
Esses comandos desinstalarão o Docker e removerão seus arquivos de configuração e chaves GPG.

## 6. Deploy para https://www.shinyapps.io/

Antes de tudo, reinicie o servidor para que ele reconheça o aplicativo que você moveu para o diretório do servidor:

```bash
$ sudo systemctl restart shiny-server
$ sudo systemctl status shiny-server
```
Em caso de problemas, verifique os logs:

```bash
$ cat /var/log/shiny-server.log
$ cat /var/log/shiny-server/*.log
```
Esses logs geralmente fornecem informações úteis para diagnosticar problemas.

Agora, crie sua conta no `shinyapps.io` (se ainda não tiver) e siga os passos fornecidos na tela inicial da sua conta.

Instale o pacote `rsconnect`:
```bash
$ pip install rsconnect-python --upgrade
```
O restante do processo é mais fácil de seguir diretamente no site do `shinyapps.io`, pois lá você encontrará seu token e chave para o deploy. Basta copiar e colar no seu terminal.

Depois, vá para o diretório do seu projeto:

```bash
$ cd /srv/shiny-server/caminho-do-seu-projeto
```
É importante que todos os arquivos necessários para seu aplicativo funcionar estejam neste diretório.

Teste seu aplicativo:
```bash
$ shiny run app.py
```
Acesse o endereço que apareceu no terminal, algo como:

`http://seu-ip:8000`

Se tudo estiver funcionando, seu aplicativo está pronto para ser publicado.

Para fazer o deploy, utilize o seguinte comando (lembre-se de estar no diretório do `app.py` para executar o comando):
```bash
$ rsconnect deploy shiny . --entrypoint app:app
```
Isso iniciará o processo de deploy do seu aplicativo para o `shinyapps.io`. Siga as instruções adicionais fornecidas durante o processo.

#Referencias:
[1] https://docs.posit.co/shiny-server/
[2] https://shiny.posit.co/py/docs/deploy.html#rsconnect-deploy
[3] https://www.shinyapps.io/





