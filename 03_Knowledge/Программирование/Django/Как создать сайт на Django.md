
Для начала а главной папке проекта в urls.py нужно отслеживать переходы по разным приложениям:
```python
urlpatterns = [  
    path('admin/', admin.site.urls), #Отслеживание админ панели
    path('', include('vlog.urls')),  #Отслеживание приложения vlog и файл urls
]
```

После в папке приложения создаем файл urls.py и прописываем отслеживание внутри его:
```Python
urlpatterns = [  
    path('', views.CameraViewSet), #Из файла views обрабатываем функцию CameraViewSet
]
```

В основной папке модуля создаем папку templates/(название модуля)/index.html
Чтобы вывести html шаблон нужно в файле views прописать отслеживаемый метод CameraViewSet:
```python
def CameraViewSet(request):  
    return render(request, 'vlog/index.html')
```

