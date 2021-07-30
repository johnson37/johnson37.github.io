# Django Web Development

## Environment Setup

### Differenet Django need different Python version requirment
- Django 1.8  **Python 2.7, 3.2 , 3.3, 3.4, 3.5**
- Django 1.9  **Python 2.7, 3.4, 3.5**
- Django 1.11 **Python 2.7, 3.4, 3.5, 3.6**
- Django 2.0  **Python 3.5+**


### Setup Environment
- 安装pip
- Install Django **pip install django** 

## The first Django APP
- django-admin startproject HelloWorld
- Open settings.py, ALLOWED_HOSTS = ['*']
- python manage.py runserver 0.0.0.0:8000

## Django 工作原理
![Django 工作原理](./django_whole_picture.PNG)


























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

```python

def search(request):  
    request.encoding='utf-8'
    print (request.GET)
    if "search_context" in request.GET:
        message = '你搜索的内容为: ' + request.GET["search_context"]
    else:
        message = '你提交了空表单'
    return HttpResponse(message)

```

```python
@csrf_exempt
def login_post(request):                                                                                             
    ctx = {}                                                                                                         
    if request.POST:                                                                                                 
        ctx['rlt'] = request.POST['login']                                                                           
                                                                                                                     
    return render(request, "index.html", ctx)                                                                        
```

### Django Admin
- Web site: 135.242.61.36:8000/admin/
- We need one super user:  **python manage.py createsuperuser**
- If we want to monitor the table , we need to add something:
  - In admin.py:
```Python
from blog.models import WebLogin                                                                                     
# Register your models here.                                                                                         
admin.site.register(WebLogin)  
```

### DB
- Create one DB Class, 
```python
class WebLogin(models.Model):                                                                                        
    name = models.CharField(max_length=20)                                                                           
    passwd = models.CharField(max_length=20)    
```

```python
def write_db(new_name, new_passwd):                                                                                  
    print("Enter Write DB, name is "+new_name+" passwd is "+ new_passwd )                                            
    login = WebLogin(name=new_name, passwd= new_passwd)                                                              
    login.save()     
```
```python
def read_all_db():                                                                                                   
    print("Enter read_all_db")                                                                                       
    response = ""                                                                                                    
    list = WebLogin.objects.all()                                                                                    
    for var in list:                                                                                                 
        response += var.name + " & Password is : " + var.passwd                                                      
    print (response)  
```

```python
def testdb(request):
    # 修改其中一个id=1的name字段，再save，相当于SQL中的UPDATE
    test1 = Test.objects.get(id=1)
    test1.name = 'Google'
    test1.save()
```

### Static File
In settings.py,
```Python
STATIC_URL = '/static/'
STATICFILES_DIRS = (                                                                                                 
    os.path.join(BASE_DIR, "static"),                                                                                
)
Check INSTALLED_APPS:
  django.contrib.staticfiles
```
In Root Directory, add one new directory **static**, put all cs/js in this directory.

HTML Part
```html
    load staticfiles                                                                                          
```

