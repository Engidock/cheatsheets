# Django Cheatsheet

Quick reference guide for Django development.

## 1. Project Setup and Configuration

### Initial Setup

Create virtual environment
```bash
python -m venv venv

source venv/bin/activate  # On Windows: venv\Scripts\activate
```

Install Django
```bash
pip install django
```

Create project
```bash
django-admin startproject myproject .
```

Create app
```bash
python manage.py startapp myapp
```

Create superuser
```bash
python manage.py createsuperuser
```

Run development server
```bash
python manage.py runserver
```

### settings.py Configuration

Add app to `INSTALLED_APPS`
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'myapp',  # Your app
]
```

Database configuration
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'dbname',
        'USER': 'user',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

Security settings
```python
SECRET_KEY = 'your-secret-key'
DEBUG = False  # Production
ALLOWED_HOSTS = ['example.com', 'www.example.com']
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
```

## 2. Models

### Model Definition

```python
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey('Author', on_delete=models.CASCADE)
    isbn = models.CharField(max_length=13, unique=True)
    published_date = models.DateField()
    pages = models.IntegerField()
    is_available = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-published_date']
        indexes = [models.Index(fields=['author', '-published_date'])]
        verbose_name_plural = 'Books'

    def __str__(self):
        return self.title
```

### Field Types and Options

| Purpose | Field |
|---|---|
| String (max_length required) | `CharField(max_length=100)` |
| Large text | `TextField()` |
| Integer numbers | `IntegerField(default=0)` |
| Decimal numbers (float) | `FloatField()` |
| Precise decimals | `DecimalField(max_digits=5, decimal_places=2)` |
| True/False | `BooleanField(default=False)` |
| Date only | `DateField(auto_now_add=True)` |
| Date and time | `DateTimeField(auto_now=True)` |
| Many-to-One relationship | `ForeignKey(Author, on_delete=CASCADE)` |
| One-to-One relationship | `OneToOneField(Profile, on_delete=CASCADE)` |
| Many-to-Many relationship | `ManyToManyField(Tag)` |
| Email validation | `EmailField()` |
| URL validation | `URLField()` |
| File upload | `FileField(upload_to='files/')` |
| Image upload | `ImageField(upload_to='images/')` |

### Field Options

Common field options
```python
field = models.CharField(
    max_length=100,               # Maximum length
    null=True,                    # Allow NULL in database
    blank=True,                   # Allow empty in forms
    default='value',              # Default value
    unique=True,                  # Must be unique
    db_index=True,                # Create database index
    choices=[('opt1', 'Option 1')],  # Limited choices
    help_text='Help text',        # Help text in forms
    verbose_name='Display Name',  # Human-readable name
)
```

## 3. QuerySets and ORM

### Basic QuerySet Operations

Creating objects
```python
Book.objects.create(title='Python', author=author, isbn='123')
```

Getting objects
```python
book = Book.objects.get(id=1)             # Single object
books = Book.objects.all()                # All objects
books = Book.objects.filter(author=author)  # Filter

books = Book.objects.exclude(pages__lt=100)  # Exclude
```

Ordering
```python
books = Book.objects.order_by('title')          # Ascending
books = Book.objects.order_by('-published_date')  # Descending
```

Slicing
```python
first_5 = Book.objects.all()[:5]

page_2 = Book.objects.all()[10:20]
```

Counting and existence
```python
count = Book.objects.count()

exists = Book.objects.filter(title='Python').exists()
```

### QuerySet Filtering

| Purpose | Example | Lookup |
|---|---|---|
| Exact match | `filter(title__exact='Django')` | `exact` |
| Case-insensitive | `filter(title__iexact='django')` | `iexact` |
| Contains substring | `filter(title__contains='Python')` | `contains` |
| Case-insensitive contains | `filter(title__icontains='python')` | `icontains` |
| Starts with | `filter(title__startswith='Python')` | `startswith` |
| Ends with | `filter(title__endswith='ing')` | `endswith` |
| Greater than | `filter(pages__gt=100)` | `gt` |
| Greater or equal | `filter(pages__gte=100)` | `gte` |
| Less than | `filter(pages__lt=500)` | `lt` |
| Less or equal | `filter(pages__lte=500)` | `lte` |
| In list | `filter(id__in=[1, 2, 3])` | `in` |
| Between range | `filter(pages__range=[100, 500])` | `range` |
| Is NULL | `filter(author__isnull=True)` | `isnull` |

### Query Optimization

```python
from django.db.models import Prefetch, Count, Q

# select_related for ForeignKey/OneToOne
books = Book.objects.select_related('author')

# prefetch_related for ManyToMany/reverse FK
books = Book.objects.prefetch_related('tags')

# only() - get specific fields
books = Book.objects.only('title', 'author')

# defer() - exclude specific fields
books = Book.objects.defer('description')

# values() - get dictionary
books = Book.objects.values('title', 'author__name')

# values_list() - get tuple
titles = Book.objects.values_list('title', flat=True)

# aggregate and annotate
from django.db.models import Count, Sum, Avg

stats = Book.objects.aggregate(
    total=Count('id'),
    avg_pages=Avg('pages')
)

# annotate
books = Book.objects.annotate(
    comment_count=Count('comments')
)

# Complex queries with Q objects
from django.db.models import Q

books = Book.objects.filter(
    Q(title__contains='Python') | Q(author__name='Guido')
)
```

## 4. Views

### Function-Based Views

```python
from django.shortcuts import render, get_object_or_404, redirect
from django.http import HttpResponse, JsonResponse
from django.views.decorators.http import require_http_methods
from django.views.decorators.csrf import csrf_exempt

@require_http_methods(["GET", "POST"])
def book_list(request):
    if request.method == 'POST':
        # Handle form submission
        title = request.POST.get('title')
        return redirect('book_detail', pk=1)
    # GET request
    books = Book.objects.all()
    return render(request, 'books/list.html', {'books': books})

def book_detail(request, pk):
    book = get_object_or_404(Book, pk=pk)
    return render(request, 'books/detail.html', {'book': book})

@csrf_exempt
def api_endpoint(request):
    data = {'message': 'Hello'}
    return JsonResponse(data)
```

### Class-Based Views

```python
from django.views import View
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy

# Function-like class view
class BookView(View):
    def get(self, request, pk):
        book = get_object_or_404(Book, pk=pk)
        return render(request, 'book.html', {'book': book})

    def post(self, request):
        # Handle POST
        return redirect('home')

# Generic list view
class BookListView(ListView):
    model = Book
    template_name = 'books/list.html'
    context_object_name = 'books'
    paginate_by = 20

    def get_queryset(self):
        return Book.objects.filter(is_available=True).order_by('-published_date')

# Generic detail view
class BookDetailView(DetailView):
    model = Book
    template_name = 'books/detail.html'
    context_object_name = 'book'

# Generic create view
class BookCreateView(CreateView):
    model = Book
    fields = ['title', 'author', 'isbn']
    template_name = 'books/form.html'
    success_url = reverse_lazy('book_list')

# Generic update view
class BookUpdateView(UpdateView):
    model = Book
    fields = ['title', 'author']
    success_url = reverse_lazy('book_list')

# Generic delete view
class BookDeleteView(DeleteView):
    model = Book
    success_url = reverse_lazy('book_list')
```

## 5. URLs and Routing

### URL Configuration

```python
# urls.py
from django.urls import path, include, re_path
from . import views

app_name = 'books'

urlpatterns = [
    # Basic path
    path('', views.book_list, name='list'),
    # With path converter
    path('<int:pk>/', views.book_detail, name='detail'),
    path('<slug:slug>/', views.book_by_slug, name='by_slug'),
    # Multiple parameters
    path('<int:author_id>/<int:book_id>/', views.author_book, name='author_book'),
    # Class-based views
    path('list/', views.BookListView.as_view(), name='list'),
    path('<int:pk>/', views.BookDetailView.as_view(), name='detail'),
    # Include app URLs
    path('api/', include('api.urls')),
    # Regex paths (older style)
    re_path(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive, name='year_archive'),
]

# Main urls.py
urlpatterns = [
    path('admin/', admin.site.urls),
    path('books/', include('books.urls')),
]
```

### Path Converters

| Type | Example |
|---|---|
| Any string (default) | `path('<str:slug>/', ...)` |
| Integer | `path('<int:id>/', ...)` |
| ASCII slug | `path('<slug:slug>/', ...)` |
| UUID | `path('<uuid:id>/', ...)` |
| Path with slashes | `path('<path:file>/', ...)` |

## 6. Forms

### Django Forms

```python
from django import forms
from .models import Book

# Form from scratch
class SearchForm(forms.Form):
    query = forms.CharField(max_length=100)
    category = forms.ChoiceField(choices=[('fiction', 'Fiction'), ('non-fiction', 'Non-Fiction')])
    published_after = forms.DateField(required=False)

# ModelForm
class BookForm(forms.ModelForm):
    class Meta:
        model = Book
        fields = ['title', 'author', 'isbn', 'published_date', 'pages']
        widgets = {
            'title': forms.TextInput(attrs={'class': 'form-control', 'placeholder': 'Enter title'}),
            'published_date': forms.DateInput(attrs={'type': 'date'}),
            'pages': forms.NumberInput(attrs={'min': 1}),
        }

    def clean_isbn(self):
        isbn = self.cleaned_data.get('isbn')
        if len(isbn) != 13:
            raise forms.ValidationError('ISBN must be 13 digits')
        return isbn

# Using forms in views
def create_book(request):
    if request.method == 'POST':
        form = BookForm(request.POST, request.FILES)
        if form.is_valid():
            form.save()
            return redirect('book_list')
    else:
        form = BookForm()
    return render(request, 'book_form.html', {'form': form})
```

### Form Fields

```python
# TextField
name = forms.CharField(max_length=100, widget=forms.TextInput)

# TextArea
description = forms.CharField(widget=forms.Textarea)

# Email
email = forms.EmailField()

# URL
website = forms.URLField()

# Integer
quantity = forms.IntegerField(min_value=0, max_value=100)

# Decimal
price = forms.DecimalField(decimal_places=2)

# Boolean
agree = forms.BooleanField(required=False)

# Choice
category = forms.ChoiceField(choices=[('a', 'Option A'), ('b', 'Option B')])

# Multiple Choice
tags = forms.MultipleChoiceField(choices=[...])

# Date
birth_date = forms.DateField()

# DateTime
created_at = forms.DateTimeField()

# Time
start_time = forms.TimeField()

# File
document = forms.FileField()

# Image
photo = forms.ImageField()
```

## 7. Templates

### Template Tags and Filters

Variable output
```html
{{ variable }}

{{ object.attribute }}

{{ dictionary.key }}

{{ list.0 }}
```

Filters
```html
{{ name|lower }}

{{ name|upper }}

{{ text|truncatewords:10 }}

{{ date|date:"Y-m-d" }}

{{ price|floatformat:2 }}

{{ list|length }}

{{ text|slugify }}

{{ html|safe }}

{{ value|default:"N/A" }}

{{ items|join:", " }}
```

Conditionals
```html
{% if user.is_authenticated %}
  Welcome, {{ user.username }}
{% elif user.is_anonymous %}
  Please log in
{% else %}
  Guest
{% endif %}
```

Loops
```html
{% for item in items %}
  {{ item }}
  {% if forloop.first %}First item{% endif %}
  {% if forloop.last %}Last item{% endif %}
  Iteration: {{ forloop.counter }}
{% empty %}
  No items
{% endfor %}
```

Includes and inheritance
```html
{% include "header.html" %}

{% include "header.html" with title="My Title" %}

{% extends "base.html" %}

{% block content %}
  Content here
{% endblock %}
```

Static files and CSRF
```html
{% load static %}

{% csrf_token %}
```

## 8. Authentication and Authorization

### User Authentication

```python
from django.contrib.auth.models import User
from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.decorators import login_required, permission_required
from django.views.generic import LoginView, LogoutView

# Create user
user = User.objects.create_user(username='john', email='john@example.com', password='pass123')

# Authenticate
user = authenticate(username='john', password='pass123')

# Login
login(request, user)

# Logout
logout(request)

# Check if authenticated
if request.user.is_authenticated:
    print(f"Welcome, {request.user.username}")

# Function-based view protection
@login_required(login_url='login')
def protected_view(request):
    return render(request, 'protected.html')

# Class-based view protection
from django.contrib.auth.mixins import LoginRequiredMixin

class ProtectedView(LoginRequiredMixin, View):
    login_url = 'login'

    def get(self, request):
        return render(request, 'protected.html')

# Permission checks
@permission_required('app.can_edit_book')
def edit_book(request, pk):
    pass
```

Check in template
```html
{% if user.is_authenticated %}
  Welcome, {{ user.username }}
{% endif %}

{% if perms.app.can_edit_book %}
{% endif %}
```

### Permissions

```python
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType

# Add permission to user
user.user_permissions.add(permission)

# Remove permission
user.user_permissions.remove(permission)

# Check permission
user.has_perm('app.can_edit_book')
user.has_perms(['app.can_edit_book', 'app.can_delete_book'])
```

Check in template
```html
{% if perms.app.can_edit_book %}
  Show edit button
{% endif %}
```

Define custom permissions in model `Meta`
```python
class Book(models.Model):
    class Meta:
        permissions = [
            ('can_publish', 'Can publish books'),
            ('can_archive', 'Can archive books'),
        ]
```

## 9. Admin Interface

### Admin Registration

```python
from django.contrib import admin
from .models import Book, Author

# Basic registration
admin.site.register(Book)

# Custom admin class
@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    list_display = ('title', 'author', 'published_date', 'pages')
    list_filter = ('published_date', 'is_available')
    search_fields = ('title', 'author__name')
    list_editable = ('is_available',)
    readonly_fields = ('created_at', 'updated_at')
    fieldsets = (
        ('Basic Information', {
            'fields': ('title', 'isbn')
        }),
        ('Details', {
            'fields': ('author', 'pages', 'published_date')
        }),
        ('Status', {
            'fields': ('is_available', 'created_at', 'updated_at'),
            'classes': ('collapse',)
        }),
    )

    def get_readonly_fields(self, request, obj=None):
        if obj:  # Editing existing
            return self.readonly_fields + ('isbn',)
        return self.readonly_fields
```

## 10. Signals

### Using Signals

```python
from django.db.models.signals import post_save, pre_save, post_delete
from django.dispatch import receiver

# Receiver as decorator
@receiver(post_save, sender=Book)
def create_book_log(sender, instance, created, **kwargs):
    if created:
        print(f"New book created: {instance.title}")

# Receiver as function
def update_author_count(sender, instance, **kwargs):
    instance.author.book_count = instance.author.books.count()
    instance.author.save()

post_save.connect(update_author_count, sender=Book)

# Multiple models
@receiver(post_save, sender=[Book, Magazine])
def handle_publication(sender, instance, created, **kwargs):
    if created:
        print(f"New {sender.__name__}: {instance.title}")

# Pre-save signal
@receiver(pre_save, sender=Book)
def validate_isbn(sender, instance, **kwargs):
    if len(instance.isbn) != 13:
        raise ValueError('Invalid ISBN')

# Post-delete signal
@receiver(post_delete, sender=Book)
def log_deletion(sender, instance, **kwargs):
    print(f"Book deleted: {instance.title}")
```

## 11. Caching

### Caching Strategies

```python
from django.core.cache import cache
from django.views.decorators.cache import cache_page, cache_control
```

Cache settings in `settings.py`
```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
    }
}
```

Manual caching
```python
books = cache.get('all_books')
if books is None:
    books = list(Book.objects.all())
    cache.set('all_books', books, 3600)  # 1 hour
```

Cache with `get_or_set`
```python
books = cache.get_or_set('all_books', lambda: list(Book.objects.all()), 3600)
```

Delete from cache
```python
cache.delete('all_books')

cache.clear()  # Clear all cache
```

Cache decorator for views
```python
@cache_page(60 * 5)  # 5 minutes
def book_list(request):
    books = Book.objects.all()
    return render(request, 'books/list.html', {'books': books})
```

Cache control
```python
@cache_control(max_age=3600)
def view_func(request):
    pass
```

In templates
```html
{% load cache %}
{% cache 500 sidebar %}
  Expensive sidebar content
{% endcache %}
```

## 12. Middleware and Decorators

### Custom Middleware

```python
from django.utils.deprecation import MiddlewareMixin

class CustomMiddleware(MiddlewareMixin):
    def process_request(self, request):
        # Called before view
        request.custom_attr = 'value'
        return None  # Continue processing

    def process_view(self, request, view_func, view_args, view_kwargs):
        # Called just before view
        return None  # None = continue, HttpResponse = short-circuit

    def process_response(self, request, response):
        # Called after view
        response['Custom-Header'] = 'value'
        return response

    def process_exception(self, request, exception):
        # Called if view raises exception
        if isinstance(exception, Http404):
            return render(request, '404.html')
        return None
```

Register in `settings.py`
```python
MIDDLEWARE = [
    'myapp.middleware.CustomMiddleware',
]
```

### Common Decorators

```python
from django.views.decorators.http import require_http_methods, require_GET, require_POST
from django.views.decorators.cache import cache_page
from django.contrib.auth.decorators import login_required

# HTTP method decorators
@require_GET
def only_get(request):
    pass

@require_POST
def only_post(request):
    pass

@require_http_methods(["GET", "POST"])
def get_or_post(request):
    pass

# Authentication
@login_required
def protected(request):
    pass

# Caching
@cache_page(60 * 5)
def cached_view(request):
    pass

# Permission
@permission_required('app.can_edit')
def edit_view(request):
    pass
```

## 13. Celery and Async Tasks

### Celery Setup and Usage

`settings.py`
```python
CELERY_BROKER_URL = 'redis://localhost:6379'
CELERY_RESULT_BACKEND = 'redis://localhost:6379'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
```

`celery.py`
```python
from celery import Celery

app = Celery('myproject')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()
```

`tasks.py`
```python
from celery import shared_task
from django.core.mail import send_mail

@shared_task
def send_email_task(email, subject, message):
    send_mail(subject, message, 'from@example.com', [email])

@shared_task(bind=True, max_retries=3)
def process_data(self, data):
    try:
        # Process data
        return result
    except Exception as exc:
        # Retry after 60 seconds
        raise self.retry(exc=exc, countdown=60)
```

Calling tasks
```python
send_email_task.delay(email, subject, message)
send_email_task.apply_async(args=[...], countdown=10)  # 10 seconds later
```

Monitor tasks
```python
from celery.result import AsyncResult

result = AsyncResult(task_id)

print(result.state)

print(result.result)
```

## 14. Testing

### Writing Tests

```python
from django.test import TestCase, Client
from django.contrib.auth.models import User
from .models import Book

class BookModelTest(TestCase):
    def setUp(self):
        self.book = Book.objects.create(title='Django', pages=500)

    def test_book_creation(self):
        self.assertEqual(self.book.title, 'Django')
        self.assertEqual(self.book.pages, 500)

    def test_book_str(self):
        self.assertEqual(str(self.book), 'Django')

class BookViewTest(TestCase):
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user('testuser', password='pass')
        self.book = Book.objects.create(title='Django', pages=500)

    def test_book_list_view(self):
        response = self.client.get('/books/')
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'Django')

    def test_book_detail_view(self):
        response = self.client.get(f'/books/{self.book.id}/')
        self.assertEqual(response.status_code, 200)

    def test_login_required(self):
        response = self.client.get('/books/create/')
        self.assertEqual(response.status_code, 302)  # Redirect to login

    def test_post_request(self):
        self.client.login(username='testuser', password='pass')
        response = self.client.post('/books/create/', {
            'title': 'New Book',
            'pages': 300
        })
        self.assertEqual(response.status_code, 302)
        self.assertTrue(Book.objects.filter(title='New Book').exists())

class FormTest(TestCase):
    def test_valid_form(self):
        form = BookForm(data={'title': 'Django', 'pages': 500})
        self.assertTrue(form.is_valid())

    def test_invalid_form(self):
        form = BookForm(data={'title': '', 'pages': 500})
        self.assertFalse(form.is_valid())
```

## 15. Common Commands

### Management Commands

| Purpose | Command |
|---|---|
| Create new app | `python manage.py startapp appname` |
| Create migrations from model changes | `python manage.py makemigrations` |
| Apply migrations to database | `python manage.py migrate` |
| Migrate to specific migration | `python manage.py migrate app 0001` |
| Create empty migration | `python manage.py makemigrations --empty app --name migration_name` |
| Show migration status | `python manage.py showmigrations` |
| Create admin user | `python manage.py createsuperuser` |
| Start development server | `python manage.py runserver` |
| Run tests | `python manage.py test` |
| Run specific test | `python manage.py test app.tests.TestClass` |
| Interactive Python shell | `python manage.py shell` |
| Database shell | `python manage.py dbshell` |
| Export data | `python manage.py dumpdata > data.json` |
| Import data | `python manage.py loaddata data.json` |
| Collect static files | `python manage.py collectstatic` |
| Check for issues | `python manage.py check` |

**Quick Tip:** Use `python manage.py --help` to see all available commands, and `python manage.py command --help` for command-specific help.

## 16. Useful Snippets

### Common Patterns

Get or create
```python
book, created = Book.objects.get_or_create(
    isbn='123456789',
    defaults={'title': 'Default Title'}
)
```

Bulk create
```python
books = [Book(title=f'Book {i}') for i in range(100)]
Book.objects.bulk_create(books)
```

Bulk update
```python
Book.objects.filter(pages__lt=100).update(is_available=False)
```

Exists vs count
```python
if Book.objects.filter(title='Django').exists():  # Better for checking existence
    pass
```

Pagination
```python
from django.core.paginator import Paginator

paginator = Paginator(Book.objects.all(), 20)
page = paginator.get_page(request.GET.get('page'))
```

Raw SQL
```python
from django.db import connection

with connection.cursor() as cursor:
    cursor.execute("SELECT * FROM books_book WHERE pages > %s", [100])
```

F expressions for database-level operations
```python
from django.db.models import F

Book.objects.all().update(views=F('views') + 1)
```

Q objects for complex queries
```python
from django.db.models import Q

Book.objects.filter(Q(title__contains='Python') | Q(author__name='Guido'))
```

Transaction handling
```python
from django.db import transaction

with transaction.atomic():
    book = Book.objects.create(title='Test')
    author = Author.objects.create(name='Test')
```

Custom manager
```python
class PublishedManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(is_published=True)

class Book(models.Model):
    objects = models.Manager()
    published = PublishedManager()

# Use: Book.published.all()
```

**Security:** Always use parameterized queries to prevent SQL injection. Never use string formatting in SQL queries.

**Note:** Django ORM handles transactions, connection pooling, and query optimization automatically. Avoid raw SQL unless absolutely necessary.

---
*Source: adapted from the Django cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
