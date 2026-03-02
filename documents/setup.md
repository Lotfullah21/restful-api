## Creating a virtual environment and starting a project using virtualenv and pip

```py
# install python and pip using Homebrew
brew install python

# install virtualenv using pip
python3 -m pip install virtualenv

# make a main directory
mkdir directory_name
cd directory_name

# create a new virtual environment
virtualenv env

# activate virtual environment (macOS)
source env/bin/activate

# add .gitignore and requirements.txt
touch .gitignore
touch requirements.txt

# install Django in the virtual environment
pip install django
pip install python-dotenv

# check if Django is installed
django-admin --version
# install all listed packages present in requirement.txt
pip install -r requirements.txt
# create a new Django project (corrected command)
django-admin startproject project_name

# go to the project directory
cd project_name

# create an app inside the project
django-admin startapp appname
# or
python manage.py startapp appname

# install django rest framework
pip3 install djangorestframework

# start the development server
python manage.py runserver
```

## Add the app

add the `rest_framework` in settings.py at installed apps.

```py
# rest_api/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    ... rest of the apps
    'rest_framework',
    # add the newly created app.
    'CourseListAPI'
]
```

## Create an api end point

## Creating a virtual environment and starting a project using pipevn

```sh
# a tool for managing virtual environments
pip3 install pipenv
# creates a new virtual environment
pipenv install django
# initialize virtual env and install django in it
pipenv shell
# to start a project
django-admin startproject project_name
cd project_name
# to create an app
python manage.py startapp app_name
# install django rest framework
pipenv install djangorestframework
# to run the django server
python manage.py runserver
# Integrate virtual environment with vs code and apply commands from integrated terminal
```

### Integrating the virtual environment with VS Code:

- Open the Command Palette (Command + Shift + P on Mac, Ctrl + Shift + P on Windows/Linux).
- Search for "Python: Select Interpreter."
- Choose the Python interpreter from the virtual environment created by pipenv for your project.
- In the terminal, navigate to your project directory and run the Django server.

## Creating a virtual environment and starting a project using virtualenv and pip

```py
# install python and pip using Homebrew
brew install python

# install virtualenv using pip
python3 -m pip install virtualenv

# make a main directory
mkdir directory_name
cd directory_name

# create a new virtual environment
virtualenv env

# activate virtual environment (macOS)
source env/bin/activate

# add .gitignore and requirements.txt
touch .gitignore
touch requirements.txt

# install Django in the virtual environment
pip install django
pip install python-dotenv

# check if Django is installed
django-admin --version
# install all listed packages present in requirement.txt
pip install -r requirements.txt
# create a new Django project (corrected command)
django-admin startproject project_name

# go to the project directory
cd project_name

# create an app inside the project
django-admin startapp appname
# or
python manage.py startapp appname

# install django rest framework
pip3 install djangorestframework

# start the development server
python manage.py runserver
```

add the `rest_framework` in settings.py at installed apps.

```py
# rest_api/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    ... rest of the apps
    'rest_framework',
    # add the newly created app.
    'CourseListAPI'
]
```

before running the django server, make sure to run the following command.

```sh
python manage.py makemigrations
python manage.py migrate
```

### Resources

- [pipenv](https://pipenv.pypa.io/en/latest/)
- [django](https://www.djangoproject.com/)

### Resources

- [pipenv](https://pipenv.pypa.io/en/latest/)
- [django](https://www.djangoproject.com/)
