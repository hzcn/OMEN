### 数据库：models.py

from django.db import models



class cal(models.Model): 

​    value_a = models.CharField(max_length=10)

​    value_b = models.FloatField(max_length=10)

​    result = models.CharField(max_length=10)

### 路由：urls.py

from django.contrib import admin

from django.urls import path

from firstWEB import views



urlpatterns = [

​    path('admin/', admin.site.urls),

​    path('',views.index),

​    path('calpage/',views.Cal),

​    path('cal',views.Cal),

​    path('list',views.Calist),

​    path('del',views.DelData)

]

### 视图：views.py

from django.shortcuts import render

from firstWEB.models import cal

from django.http import HttpReponse



def index(request):

​    return render(request,'index.html')

def CalPage(request):

​    return render(request,'cal.html')

def Cal(request):

​    if request.POST:

​        value_a = request.POST['valueA']

​        value_b = request.POST['valueB']

​        result = int(value_a) + int(value_b)

​        cal.objects.create(value_a=value_a,value_b=value_b,result=result)

​        return render(request,'result.html',context={'data':result})

​    else:

​        return HttpReponse('please visit us with POST')



def CalList(request):

​    data = cal.objects.all()

​    return render(request,'list.html',context={'data':data})

def DelData(request):

​    cal.object.all().delete()

​    return HttpReponse('data Deleted')        