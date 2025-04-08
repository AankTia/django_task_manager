# Creating a Step-by-Step CRUD Web Application with Django

Creating a basic CRUD (Create, Read, Update, Delete) web application using Django. A simple task management system.

Comprehensive tutorial for building a CRUD web application with Django. This guide walks through creating a task management system with all the essential CRUD operations.

The tutorial covers:

1. Setting up your development environment with Python and Django
2. Creating the Django project structure
3. Defining a Task model for your data
4. Implementing views for all CRUD operations
5. Creating templates with basic styling
6. Configuring URLs for your application
7. Setting up the admin interface

This step-by-step approach will help you understand how Django's MVT (Model-View-Template) architecture works together to create a functional web application.


# Building a CRUD Web Application with Django

This tutorial will guide you through creating a Task Management application with full CRUD functionality using Django.

## Prerequisites
- Python 3.8+ installed
- Basic understanding of Python
- Basic understanding of HTML/CSS

## Step 1: Set Up Your Development Environment

First, let's set up a virtual environment and install Django:

```bash
# Create a directory for your project
mkdir django_task_manager
cd django_task_manager

# Create and activate a virtual environment
python -m venv venv

# On Windows
venv\Scripts\activate

# On macOS/Linux
source venv/bin/activate

# Install Django
pip install django
```

## Step 2: Create a Django Project

```bash
# Create a new Django project
django-admin startproject task_manager .

# Create a new app for our tasks
python manage.py startapp tasks
```

## Step 3: Configure Settings

Open `task_manager/settings.py` and add your app to the `INSTALLED_APPS` list:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'tasks',  # Add your app here
]
```

## Step 4: Create the Task Model

In `tasks/models.py`, define your Task model:

```python
from django.db import models

class Task(models.Model):
    STATUS_CHOICES = [
        ('pending', 'Pending'),
        ('in_progress', 'In Progress'),
        ('completed', 'Completed'),
    ]
    
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    created_date = models.DateTimeField(auto_now_add=True)
    due_date = models.DateField(null=True, blank=True)
    status = models.CharField(
        max_length=20,
        choices=STATUS_CHOICES,
        default='pending'
    )
    
    def __str__(self):
        return self.title
```

## Step 5: Create and Apply Migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

## Step 6: Create the URLs

First, update the project-level URLs in `task_manager/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('tasks.urls')),
]
```

Then, create a new file `tasks/urls.py`:

```python
from django.urls import path
from . import views

app_name = 'tasks'

urlpatterns = [
    path('', views.task_list, name='task_list'),
    path('task/create/', views.task_create, name='task_create'),
    path('task/<int:pk>/', views.task_detail, name='task_detail'),
    path('task/<int:pk>/update/', views.task_update, name='task_update'),
    path('task/<int:pk>/delete/', views.task_delete, name='task_delete'),
]
```

## Step 7: Create the Views

In `tasks/views.py`, add the following code:

```python
from django.shortcuts import render, get_object_or_404, redirect
from .models import Task
from .forms import TaskForm

def task_list(request):
    tasks = Task.objects.all()
    return render(request, 'tasks/task_list.html', {'tasks': tasks})

def task_detail(request, pk):
    task = get_object_or_404(Task, pk=pk)
    return render(request, 'tasks/task_detail.html', {'task': task})

def task_create(request):
    if request.method == 'POST':
        form = TaskForm(request.POST)
        if form.is_valid():
            task = form.save()
            return redirect('tasks:task_detail', pk=task.pk)
    else:
        form = TaskForm()
    return render(request, 'tasks/task_form.html', {'form': form})

def task_update(request, pk):
    task = get_object_or_404(Task, pk=pk)
    if request.method == 'POST':
        form = TaskForm(request.POST, instance=task)
        if form.is_valid():
            task = form.save()
            return redirect('tasks:task_detail', pk=task.pk)
    else:
        form = TaskForm(instance=task)
    return render(request, 'tasks/task_form.html', {'form': form})

def task_delete(request, pk):
    task = get_object_or_404(Task, pk=pk)
    if request.method == 'POST':
        task.delete()
        return redirect('tasks:task_list')
    return render(request, 'tasks/task_confirm_delete.html', {'task': task})
```

## Step 8: Create a Form

Create a new file `tasks/forms.py`:

```python
from django import forms
from .models import Task

class TaskForm(forms.ModelForm):
    class Meta:
        model = Task
        fields = ['title', 'description', 'due_date', 'status']
        widgets = {
            'due_date': forms.DateInput(attrs={'type': 'date'}),
        }
```

## Step 9: Create the Templates

First, create a base template. Create a directory structure:

```
tasks/
    templates/
        tasks/
            base.html
            task_list.html
            task_detail.html
            task_form.html
            task_confirm_delete.html
```

### base.html
```html
<!DOCTYPE html>
<html>
<head>
    <title>Task Manager</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            margin: 0;
            padding: 20px;
            max-width: 1000px;
            margin: 0 auto;
        }
        nav {
            margin-bottom: 20px;
            background-color: #f4f4f4;
            padding: 10px;
        }
        nav a {
            margin-right: 15px;
            text-decoration: none;
            color: #333;
        }
        .task {
            background: #f4f4f4;
            margin-bottom: 10px;
            padding: 10px;
            border-radius: 5px;
        }
        .task h3 {
            margin-top: 0;
        }
        form {
            background: #f4f4f4;
            padding: 20px;
            border-radius: 5px;
        }
        form label {
            display: block;
            margin-bottom: 5px;
        }
        form input[type="text"],
        form textarea,
        form select {
            width: 100%;
            padding: 8px;
            margin-bottom: 15px;
            border-radius: 5px;
            border: 1px solid #ddd;
        }
        form button {
            background: #333;
            color: #fff;
            padding: 10px 15px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        .button {
            display: inline-block;
            background: #333;
            color: #fff;
            padding: 5px 10px;
            text-decoration: none;
            border-radius: 5px;
            font-size: 0.9em;
        }
    </style>
</head>
<body>
    <header>
        <h1>Task Manager</h1>
        <nav>
            <a href="{% url 'tasks:task_list' %}">Home</a>
            <a href="{% url 'tasks:task_create' %}">Add New Task</a>
        </nav>
    </header>
    
    <main>
        {% block content %}
        {% endblock %}
    </main>
</body>
</html>
```

### task_list.html
```html
{% extends 'tasks/base.html' %}

{% block content %}
    <h2>Task List</h2>
    
    {% if tasks %}
        {% for task in tasks %}
            <div class="task">
                <h3>{{ task.title }}</h3>
                <p>Status: {{ task.get_status_display }}</p>
                <p>
                    <a href="{% url 'tasks:task_detail' task.pk %}" class="button">View</a>
                    <a href="{% url 'tasks:task_update' task.pk %}" class="button">Edit</a>
                    <a href="{% url 'tasks:task_delete' task.pk %}" class="button">Delete</a>
                </p>
            </div>
        {% endfor %}
    {% else %}
        <p>No tasks available.</p>
    {% endif %}
{% endblock %}
```

### task_detail.html
```html
{% extends 'tasks/base.html' %}

{% block content %}
    <h2>{{ task.title }}</h2>
    
    <div class="task">
        <p><strong>Description:</strong> {{ task.description }}</p>
        <p><strong>Created:</strong> {{ task.created_date }}</p>
        <p><strong>Due Date:</strong> {{ task.due_date|default:"Not specified" }}</p>
        <p><strong>Status:</strong> {{ task.get_status_display }}</p>
        
        <p>
            <a href="{% url 'tasks:task_update' task.pk %}" class="button">Edit</a>
            <a href="{% url 'tasks:task_delete' task.pk %}" class="button">Delete</a>
            <a href="{% url 'tasks:task_list' %}" class="button">Back to List</a>
        </p>
    </div>
{% endblock %}
```

### task_form.html
```html
{% extends 'tasks/base.html' %}

{% block content %}
    <h2>{% if form.instance.pk %}Edit Task{% else %}Create New Task{% endif %}</h2>
    
    <form method="post">
        {% csrf_token %}
        <div>
            <label for="{{ form.title.id_for_label }}">Title:</label>
            {{ form.title }}
        </div>
        <div>
            <label for="{{ form.description.id_for_label }}">Description:</label>
            {{ form.description }}
        </div>
        <div>
            <label for="{{ form.due_date.id_for_label }}">Due Date:</label>
            {{ form.due_date }}
        </div>
        <div>
            <label for="{{ form.status.id_for_label }}">Status:</label>
            {{ form.status }}
        </div>
        <button type="submit">Save</button>
        <a href="{% url 'tasks:task_list' %}" class="button">Cancel</a>
    </form>
{% endblock %}
```

### task_confirm_delete.html
```html
{% extends 'tasks/base.html' %}

{% block content %}
    <h2>Delete Task</h2>
    
    <p>Are you sure you want to delete "{{ task.title }}"?</p>
    
    <form method="post">
        {% csrf_token %}
        <button type="submit">Yes, Delete</button>
        <a href="{% url 'tasks:task_detail' task.pk %}" class="button">Cancel</a>
    </form>
{% endblock %}
```

## Step 10: Register Your Model with the Admin Site

In `tasks/admin.py`, add:

```python
from django.contrib import admin
from .models import Task

admin.site.register(Task)
```

## Step 11: Create a Superuser (Optional)

```bash
python manage.py createsuperuser
```

Follow the prompts to create an admin account.

## Step 12: Run the Development Server

```bash
python manage.py runserver
```

Visit `http://127.0.0.1:8000/` to see your application.
Visit `http://127.0.0.1:8000/admin/` to access the admin interface.

## Summary

Congratulations! You've built a complete CRUD application with Django that allows you to:

1. **Create** new tasks
2. **Read** task details
3. **Update** existing tasks
4. **Delete** tasks you no longer need

This basic structure can be extended for more complex applications by adding more models, views, and functionality.