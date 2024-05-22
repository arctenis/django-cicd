# Création d'un pipeline CI/CD pour un projet Django

AVERTISSEMENT : en cours de rédaction

## Objectif

Mettre en place un pipeline CI/CD pour un projet Django. Le pipeline doit
automatiser les tâches suivantes :
- Tester le code
- Construire l'image Docker
- Pousser l'image Docker sur Docker Hub
- Déployer l'image Docker sur Render
- Mettre à jour l'image Docker sur Render à chaque push sur la branche principale

Prérequis : 
- Python
- Django
- Docker
- Git & Github
- YML

Outils :
- Django
- Gunicorn
- Docker
- Docker Hub
- Git
- Github
- Github Actions
- Render

## Projet Django

Créer un environnement virtuel, installer Django et les dépendances, créer une application.

```bash
python -m venv ~/.virtualenvs/django-cicd
source ~/.virtualenvs/django-cicd/bin/activate
mkdir django-cicd && cd django-cicd
pip install -U pip django gunicorn
django-admin startproject core .
python manage.py startapp pages
```

Ajouter l'application `pages` dans les `INSTALLED_APPS` du fichier `settings.py`.

```python
# core/settings.py
INSTALLED_APPS = [
    ...
    'pages',
]
```
 
```python
# pages/views.py
from django.shortcuts import render

def homepage(request):
    return render(request, "pages/homepage.html")
```

Ajouter un template HTML django-cicd/pages/templates/pages/homepage.html
```html
<!-- pages/templates/pages/homepage.html -->
<h1>Django app deployed !</h1>
```

Configurer les routes dans le fichier `urls.py`.

```python
# core/urls.py
from django.contrib import admin
from django.urls import path

from pages.views import homepage


urlpatterns = [
    path("admin/", admin.site.urls),
    path("", homepage),
]
```

Tester le projet :

```bash
python manage.py runserver
```

## Docker

Installer Docker en local. Soit Docker Desktop pour Windows et Mac, soit Docker Engine pour Linux.

Je suis sous Linux, j'ai suivi les instructions suivantes : https://github.com/docker/buildx?tab=readme-ov-file#linux-packages

Créer un Dockerfile à la racine du projet :

```Dockerfile
FROM python:3.13-rc-slim

# Set working directory
WORKDIR /usr/src/app

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Install dependencies
COPY requirements.txt .
RUN pip install -U pip && pip install -r requirements.txt

# Add app
COPY . .

# Open port 8000
EXPOSE 8000

# Run the app
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "core.wsgi:application"]
```

Créer un compte sur Docker Hub si ce n'est déjà fait.

Avec les commandes suivantes, on peut construire l'image Docker et la pousser sur Docker Hub.

```bash
docker build -t <username>/<repository>:<tag> .
docker login
docker tag <username>/<repository>:<tag> <username>/<repository>:<tag>
docker push <username>/<repository>:<tag>
```

## Render

Créer un compte sur Render si ce n'est déjà fait.

Créer un nouveau service web. Choisir de déployer à partir d'une image
provenant d'un registre.

Entrer l'URL de l'image Docker sur Docker Hub. 

```
hub.docker.com/r/<username>/<repository>
```
