# (DEPLOYMENT DOCS COMING SOON...)
For now, please see https://github.com/havenfricke/vercel-django-example for deployment on Vercel.

Immediate use:

        cd client

then,

        npm install

lastly,

        cd..

        cd server

        python manage.py runserver

otherwise, please review the following for a more in depth understanding.

# Setting Up the Backend

SOURCE AUTHOR: Jordan Irabor

SOURCE CONTENT: https://www.digitalocean.com/community/tutorials/build-a-to-do-application-using-django-and-react

In this section, you will create a new project directory and install Django.

Open a new terminal window and run the following command to create a new project directory:

        mkdir your-server-name

Next, navigate into the directory:

        cd your-server-name

Now install Pipenv using pip:

        pip install pipenv

Note: Depending on your installation, you may need to use pip3 instead of pip.

And activate a new virtual environment:

        pipenv shell

Install Django using Pipenv:

        pipenv install django

Then create a new project called server:

        django-admin startproject server

Next, navigate into the newly created server directory:

        cd server

Start a new application called todo:

        python manage.py startapp todo

Run migrations:

        python manage.py migrate

And start up the server:

        python manage.py runserver

Navigate to http://localhost:8000 in your web browser:

At this point, you will see an instance of a Django application running successfully. Once you are finished, you can stop the server (CONTROL+C or CTRL+C).

# Registering the todo Application

Now that you have completed the setup for the server, you can begin registering the todo application as an installed app so that Django can recognize it.

Open the server/settings.py file in your code editor and add todo to the INSTALLED_APPS:

        server/settings.py:

        # Application definition

        INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'todo', <- [Add this]
        ]

Then, save your changes.

# Defining the Todo Model

Let’s create a model to define how the Todo items should be stored in the database.

Open the todo/models.py file in your code editor and add the following lines of code:

        todo/models.py

        from django.db import models

        # Create your models here.

        class Todo(models.Model):
        title = models.CharField(max_length=120)
        description = models.TextField()
        completed = models.BooleanField(default=False)

            def _str_(self):
                return self.title

The code snippet above describes three properties on the Todo model:

title
description
completed
The completed property is the status of a task. A task will either be completed or not completed at any time. Because you have created a Todo model, you will need to create a migration file:

now run command:

        python manage.py makemigrations todo

And apply the changes to the database with this command:

        python manage.py migrate todo

You can test to see that CRUD operations work on the Todo model you created by using the admin interface that Django provides by default.

Open the todo/admin.py file with your code editor and add the following lines of code:

        todo/admin.py:

        from django.contrib import admin
        from .models import Todo

        class TodoAdmin(admin.ModelAdmin):
        list_display = ('title', 'description', 'completed')

        # Register your models here.

        admin.site.register(Todo, TodoAdmin)

Then, save your changes.

You will need to create a “superuser” account to access the admin interface. Run the following command in your terminal:

run command:

        python manage.py createsuperuser

You will be prompted to enter a username, email, and password for the superuser. Be sure to enter details that you can remember because you will need them to log in to the admin dashboard.

Start the server once again:

        python manage.py runserver

Navigate to http://localhost:8000/admin in your web browser. And log in with the username and password that was created earlier:

You can create, edit, and, delete Todo items using this interface:

After experimenting with this interface, you can stop the server (CONTROL+C or CTRL+C).

Step 2 — Setting Up the APIs
In this section, you will create an API using the Django REST framework.

Install the djangorestframework and django-cors-headers using Pipenv:

run command:

        pipenv install djangorestframework django-cors-headers

You need to add rest_framework and corsheaders to the list of installed applications. Open the server/settings.py file in your code editor and update the INSTALLED_APPS and MIDDLEWARE sections:

         server/settings.py

        # Application definition

        INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'corsheaders', <- [Add this]
        'rest_framework', <- [Add this]
        'todo', <- [Add this for every model created]
        ]

        MIDDLEWARE = [
        'django.middleware.security.SecurityMiddleware',
        'django.contrib.sessions.middleware.SessionMiddleware',
        'django.middleware.common.CommonMiddleware',
        'django.middleware.csrf.CsrfViewMiddleware',
        'django.contrib.auth.middleware.AuthenticationMiddleware',
        'django.contrib.messages.middleware.MessageMiddleware',
        'django.middleware.clickjacking.XFrameOptionsMiddleware',
        'corsheaders.middleware.CorsMiddleware', <- [Add this]
        ]

Then, add these lines of code to the bottom of the server/settings.py file:
server/settings.py
CORS_ORIGIN_WHITELIST = [
'http://localhost:3000'
'http://yourfrontendurlhere:4202'
]

django-cors-headers is a Python library that will prevent the errors that you would normally get due to CORS rules. In the CORS_ORIGIN_WHITELIST code, you whitelisted localhost:3000 because you want the frontend (which will be served on that port) of the application to interact with the API.

Creating serializers
You will need serializers to convert model instances to JSON so that the frontend can work with the received data.

Create a todo/serializers.py file with your code editor. Open the serializers.py file and update it with the following lines of code:

        todo/serializers.py:

        from rest_framework import serializers
        from .models import Todo

        class TodoSerializer(serializers.ModelSerializer):
        class Meta:
        model = Todo
        fields = ('id', 'title', 'description', 'completed')

This code specifies the model to work with and the fields to be converted to JSON.

Creating the View
You will need to create a TodoView class in the todo/views.py file.

Open the todo/views.py file with your code editor and add the following lines of code:

        todo/views.py:

        from django.shortcuts import render
        from rest_framework import viewsets
        from .serializers import TodoSerializer
        from .models import Todo

        # Create your views here.

        class TodoView(viewsets.ModelViewSet):
        serializer_class = TodoSerializer
        queryset = Todo.objects.all()

The viewsets base class provides the implementation for CRUD operations by default. This code specifies the serializer_class and the queryset.

Open the server/urls.py file with your code editor and replace the contents with the following lines of code:

        server/urls.py:

        from django.contrib import admin
        from django.urls import path, include
        from rest_framework import routers
        from todo import views

        router = routers.DefaultRouter()
        router.register(r'todos', views.TodoView, 'todo')

        urlpatterns = [
        path('admin/', admin.site.urls),
        path('api/', include(router.urls)),
        ]

This code specifies the URL path for the API. This was the final step that completes the building of the API.

You can now perform CRUD operations on the Todo model. The router class allows you to make the following queries:

/todos/ - returns a list of all the Todo items. CREATE and READ operations can be performed here.
/todos/id - returns a single Todo item using the id primary key. UPDATE and DELETE operations can be performed here.
Let’s restart the server:

        python manage.py runserver

Navigate to http://localhost:8000/api/todos in your web browser:

You can CREATE a new Todo item using the interface:

If the Todo item is created successfully, you will be presented with a successful response:

You can also perform DELETE and UPDATE operations on specific Todo items using the id primary keys. Use the address structure /api/todos/{id} and provide an id.

Add 1 to the URL to examine the Todo item with the id of “1”. Navigate to http://localhost:8000/api/todos/1 in your web browser:

This completes the building of the server of the application.
