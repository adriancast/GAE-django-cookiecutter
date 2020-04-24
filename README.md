# GAE-django-cookiecutter
The objective of this cookiecutter template is to build a production-ready Django 3.0 app that runs on App Engine standard environment and scales dynamically according to traffic.
The entire setup should take you less than 5 minutes and you will be using Google App Engine + PostgreSQL + Django 3.0.

## How to use this template

To execute the cookiecutter template wizard run:

```
$ pip install cookiecutter
$ cookiecutter gh:https://github.com/adriancast/GAE-django-cookiecutter
```
At this point, you will be asked some configuration data that you must extract from the Google Cloud Platform.
For a extended tutorial of how to extract this data check this medium article: https://medium.com/bluekiri/gae-django-3-0-production-ready-app-using-cookiecutter-in-5-minutes-adb6c35a2b3e

## Google App Engine configuration

The Google App Engine configuration is done in the app.yaml file. By default, this template should not autoscale the service once running in production. If you want to autoscaling threshold or bind more urls, you must change the configuration in this file.
````
runtime: python37
entrypoint: gunicorn -b :$PORT {{cookiecutter.project_name}}.wsgi

env_variables:
  SECRET_KEY: "{{cookiecutter.secret_key}}"
  DB_CONNECTION_NAME: "{{cookiecutter.db_connection_name}}"
  DB_NAME: "{{cookiecutter.db_name}}"
  DB_USER: "{{cookiecutter.db_user}}"
  DB_PASSWORD: "{{cookiecutter.db_password}}"

handlers:
# This configures Google App Engine to serve the files in the app's static
# directory.
- url: /static
  static_dir: static/

# This handler routes all requests not caught above to your main app. It is
# required when static routes are defined, but can be omitted (along with
# the entire handlers section) when there are no static files defined.
- url: /.*
  script: auto
````
## Django settings
This cookiecutter template will generate a Django app that runs un debug mode by default. When the code is running in the GAE, the Django settings override the debug and database settings with the ones from the app.yaml file. Note that sensetive data must be setted in the app.yaml file using environment variables.
````
# Production settings
if os.getenv('GAE_APPLICATION', None):
    """
    Remember to manage your migrations. To do it you should follow this steps:
    
    - Enable the Cloud SQL Admin API: Before using Cloud SQL in your dev machine,
      you must enable the Cloud SQL Admin API
     
     $ gcloud services enable sqladmin
     
    - Once the Cloud SQL Admin API is enabled you must use the cloud_sql_proxy script to connect to the database 
      production. You can find this script under /{{cookiecutter.project_name}}/cloud_sql_proxy
      
      $ chmod +x cloud_sql_proxy  # make sure the script can execute
      $ ./cloud_sql_proxy -instances="{{cookiecutter.db_connection_name}}"=tcp:3306
      
    
    - At this point, you will be able to update your database executing the migrations:
      $ python manage.py migrate
    """

    DEBUG = False

    # Redirects all non-HTTPS requests to HTTPS
    SECURE_SSL_REDIRECT = True
    SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',
            'HOST': '/'.join(['/cloudsql', os.getenv('DB_CONNECTION_NAME')]),
            'USER': os.getenv('DB_USERNAME'),
            'PASSWORD': os.getenv('DB_PASSWORD'),
            'NAME': os.getenv('DB_NAME')
        }
    }
````
