# Django Web Development
## Environment Setup
### Differenet Django need different Python version requirment
- Django 1.8  **Python 2.7, 3.2 , 3.3, 3.4, 3.5**
- Django 1.9  **Python 2.7, 3.4, 3.5**
- Django 1.11 **Python 2.7, 3.4, 3.5, 3.6**
- Django 2.0  **Python 3.5+**


### Setup Environment
- python setup
- Install python-setuptools **python setuptool.py install**
- Install Django **python setup.py install** 

## The first Django APP
**Here, my Environment is Django 1.9, Python 3.4**

- django-admin startproject HelloWorld
- Open settings.py, ALLOWED_HOSTS = ['*']
- python manage.py runserver 0.0.0.0:8000

## Some Tips
### Paint the Web
```python
from django.http import HttpResponse

return HttpResponse(message)
```

```python
from django.shortcuts import render_to_response

return render_to_response('search_form.html')
```

```python
from django.shortcuts import render

context          = {}
context['hello'] = 'Hello World!'
return render(request, 'hello.html', context)
```


### Form Handling
```html
    <form action="/search" method="get">
        <input type="text" name="search_context">
        <input type="submit" value="搜索">
    </form>    
```

```Python

def search(request):  
    request.encoding='utf-8'
    print (request.GET)
    if "search_context" in request.GET:
        message = '你搜索的内容为: ' + request.GET["search_context"]
    else:
        message = '你提交了空表单'
    return HttpResponse(message)

```

```Python
@csrf_exempt
def login_post(request):                                                                                             
    ctx = {}                                                                                                         
    if request.POST:                                                                                                 
        ctx['rlt'] = request.POST['login']                                                                           
                                                                                                                     
    return render(request, "index.html", ctx)                                                                        

```
