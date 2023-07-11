# Deploy django
Configurações mínimas para hospedar um projeto em Django em serviços como Railway, Heroku e etc...
Se você já tem o projeto desenvolvido e gostaria apenas de realizar o deploy, vá direto para o tópico de <a href="## Ocultando a configuração da instância">Ocultando a configuração da instância</a>

## Crie a pasta do projeto
```
mkdir directory_name
```
```
cd directory_name
```

## Criar e ativar o ambiente virtual
```
python -m venv venv
```
```
venv\Scripts\activate
```

## Instalar o Django
```
* pip install django
```
## Criar um projeto Django
```
django-admin startproject core .
```

## Criar o repositório git
```
git init
```
* Criar um arquivo chamado de `.gitignore` contendo o seguinte:
```
# See the name for you IDE
.idea
# If you are using sqlite3
*.sqlite3
# Name of your virtuan env
.vEnv
*pyc
```
Se você criar um repositório diretamente pelo github definindo python como linguagem de desenvolvimento esse arquivo gitignore já estará preenchido corretamente.
```
git add .
```
```
git commit -m 'First commit'
```

## Ocultando a configuração da instância
É necessário a instalação do Python Decouple para que seja possível utilizar um arquivo de configurações, onde ficarão salvos os dados sensíveis da aplicação Django.
```
pip install python-decouple
```

Crie um arquivo .env na raíz do projeto ao lado do manage.py e insira as seguintes variáveis
```
SECRET_KEY=Your$eCretKeyHere (Você consegue o valor dessa chave no arquivo settings.py)
DEBUG=True
```
A chave secreta(SECRET_KEY) não pode conter aspas simnples nem duplas, deve conter apenas os caracteres de composição da chave.

### Settings.py
No arquivo Settings também é necessário modificações, inciando pela importação do config.

```
* from decouple import config
```

Agora é necessário inserir o trecho a seguir exatamente como está abaixo. Isso irá garantir que seja possível utilizar as variáveis no ambiente de desenvolvimento e que essas informações não sejam vistas após o deploy.
```
SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
```

## Configurando uma Base de Dados (Você não precisa disso se já tiver um banco de dados).
```
pip install dj-database-url
```

### Settings.py

Necessário import do dj_database_url
```
from dj_database_url import parse as dburl
```
O código abaixo permite a utilização do sqlite3 quando estiver rodando localmente, e faz uso do banco de dados de produção quando estiver online.
```
default_dburl = 'sqlite:///' + os.path.join(BASE_DIR, 'db.sqlite3')

DATABASES = {
    'default': config('DATABASE_URL', default=default_dburl, cast=dburl),
}
```
No arquivo `.env` adicione o seguinte comando:
```
DATABASE_URL=postgres://user:password@localhost:5432/dbname
```
Neste exemplo:

- user: substitua pelo nome de usuário do PostgreSQL.
- password: substitua pela senha do usuário do PostgreSQL.
- localhost: substitua pelo endereço do servidor do PostgreSQL (pode ser um endereço IP ou nome de domínio).
- 5432: substitua pela porta em que o PostgreSQL está sendo executado (o padrão é 5432).
- dbname: substitua pelo nome do banco de dados PostgreSQL que você está usando.

## HOSTS
É necessário a configuração do `ALLOWED HOSTS` dentro do arquivo seetings.py. Se você ainda não sabe qual será os hosts e DNS utilizado deixe da seguinte forma.
```
ALLOWED_HOSTS = ['*']
```
Isso vai permitir acesso de qualquer endereço de internet.

Caso você já tenha um DNS configurado poderá utilizar normalmente da seguinte maneira:
```
ALLOWED_HOSTS = ['seusite.com.br']
```

Incluir também as permissões para uso do CSRF
```
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
#CSRF_TRUSTED_ORIGINS = ['https://seu.site.com.br']
#CSRF_COOKIE_SECURE = True
#SESSION_COOKIE_SECURE = True
```


Ainda no Arquivo Settings.py

Importar a bibliote OS
```
import os
```
### SERVINDO ARQUIVOS ESTÁTICOS
Copie e cole o código abaixo para arquivos estáticos.
```
STATIC_URL = 'static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static'),]
```

Instalar o Whitenoise para servir os arquivos estáticos em produção.
```
pip install whitenoise
```

Adicionar o código abaixo na segunda linha do MIDDLEWARE
```
'whitenoise.middleware.WhiteNoiseMiddleware',
```
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]


Para inserir os arquivos estáticos do admin Django necessários para o deploy utilize o comando:
```
python manage.py collectstatic
```

## Instalar gunicorn
```
pip install gunicorn
```
Crie na raiz do projeto ao lado do manage.py 2 arquivos, são eles:

```
Procfile
runtime.txt
```

### runtime
Dentro do arquivo `runtime.txt` coloque a versão do python utilizada no desenvolvimento.
```
python-3.11.4
```
Se nao sabe qual a versão está utilizando use o comando `python --version`
Atente-se ao uso de letras minúsculas.

### Procfile
Dentro do arquivo `Procfile` coloque o seguinte código.
```
web: gunicorn core.wsgi --log-file -
```
Lembrando que `core` deverá ser o nome do seu projeto.

### BANCO DE DADOS POSTGRESQL

Recomendo a utilização do banco de dados PostgreSQL caso seja uma necessidade de seu projeto. Para isso precisamos instalar uma biblioteca chamada de psycopg2.
```
pip instalar psycopg2
```

### CHECK --DEPLOY
Verificar requerimentos para DEPLOY
```
python manage.py check --deploy
```

### Requerimentos básicos

Vamos criar um arquivo contendo os requsitos funcionais para nosso sistema.
```
pip freeze > requirements.txt
```

----- EM CONSTRUÇÃO -------
----- NÃO EXECUTE NADA DOS COMANDO ABAIXO ---------

### wsgi 
* from dj_static import Cling
* application = Cling(get_wsgi_application())
* Also don't forget to check "DJANGO_SETTINGS_MODULE". It is prone to frequent mistakes.

