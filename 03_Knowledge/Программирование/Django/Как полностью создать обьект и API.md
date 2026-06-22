## 1. Создание модели 
---

Для начала надо создать класс с название того обьекта в бд который будем описывать:
```Python
class Camera(models.Model):
```

Далее заполняем наш обьект теми элементами которые нам нужны:
```Python
class Camera(models.Model):  
    name=models.CharField(  
        verbose_name=_('Название камеры'),  
        max_length=100,  
        help_text="Именем может быть расположение камеры"  
    )  
  
    host=models.GenericIPAddressField(  
        verbose_name="IP камеры",  
        unique=True,  
    )  
  
    port=models.PositiveIntegerField(  
        verbose_name="PORT камеры"  
    )  
  
    create_date=models.DateTimeField(  
        auto_now_add=True,  
    )  
```

Name это поле модели в которую будет записаны параметры например: 
```Python
        verbose_name=_('Название камеры'),  
        max_length=100,  
        help_text="Именем может быть расположение камеры"  
```

- verbose name - это название которое будет отображаться в админ панели  как столбец ![[Pasted image 20241203203522.png]]
- max_lengh - это кол-во символов которыми можно подписать камеру  
- help_text - это текст который будет отображаться под полем ввода названия камеры 

Так же есть такое поле как:
```Python
    create_date=models.DateTimeField(  
        auto_now_add=True,  
    )  
```

Оно отвечает за то чтобы при создании обьекта камеры было написано время создания

Далее идет блок Meta. Это класс который нужен для корректного отображения в админке:
```Python
    class Meta:  
        verbose_name="Камера"  
        verbose_name_plural="Камеры"  
```

Meta 

Далее идет блок`def __str__(self):`он отвечает за вывод названия камеры в ее карточке :
```Python
    def __str__(self):  
        return self.name
```

Вот весь код:
```Python
class Camera(models.Model):  
    name=models.CharField(  
        verbose_name=('Название камеры'),  
        max_length=100,  
        help_text="Именем может быть расположение камеры"  
    )  
  
    host=models.GenericIPAddressField(  
        verbose_name="IP камеры",  
        unique=True,  
    )  
  
    port=models.PositiveIntegerField(  
        verbose_name="PORT камеры"  
    )  
  
    create_date=models.DateTimeField(  
        auto_now_add=True,  
    )  
  
  
    class Meta:  
        verbose_name="Камера"  
        verbose_name_plural="Камеры"  
  
    def __str__(self):  
        return self.name
```

### После всего этого надо обязательно сделать миграцию чтобы объект записался в бд 

`python manage.py makemigration`
Если джанго не увидел миграцию нужно в конце подписать модуль 
`python manage.py makemigration users`

`python manage.py migrate`
## 2. Добавление модели в админку
---

Не забываем импортировать модель из models
```Python
@admin.register(Camera) #Регистрация в админке в которую передаем нужный объект
class CameraAdmin(ModelAdmin): #Даем навзание 
    list_display = ('name','host', 'port', 'create_date',) #Указывает элементы которые будут видны пользователю в списке всех камер
    fieldsets = (  #Поля которык будут видны пользователю внутри карточки камеры
        ('Camera info', {'fields': ('name', 'host', 'port', 'create_date')}),  
    )  
    readonly_fields = ('create_date',) #Поля которые можно только читать в карточке
```

## 3. Пишем сериализатор
---
Сериализатор нужен для того чтобы выводились сообщения в API

```Python 
class CameraCreationSerializer(serializers.ModelSerializer): #Создаем класс
    class Meta: #Класс мета нужен лоя вывода полей в апи
        model = Camera #Указываем с каким обьектом работаем 
        fields = ['name', 'host', 'port'] #Указываем какие поля будут выводится в апи
        extra_kwargs = {  #Сообщения с ошибками 
            "name": {  
                "error_messages": {"required": "Невалидное имя.", "blank": "Пожалуйста, заполните поле имени."}},  
            "host": {  
                "error_messages": {"required": "Невалидный HOST.", "blank": "Пожалуйста, заполните поле HOST."}},  
            "port": {  
                "error_messages": {"required": "Невалидный PORT.", "blank": "Пожалуйста, заполните поле PORT."}},  
        }  
  
    def validate(self, data):  #Кастомная валидация которя будет разобрана дальше 
        custom_validate_camera_port(data) #Не забываем импортировать
        return data
```

## 4. Пишем кастомный валидатор
---

```Python
def custom_validate_camera_port(data): #Создаем валидатор 
    port = data.get('port') #Указываем какое поле будет проверяться
    if not port > 65535 or not port < 0:  #Пишем проверку НЕ ЗАБЫВАЕМ ПРО ОТРИЦАНИЕ NOT
    #Если условние не выполняется возвразаем ответ с сообщением 
        raise serializers.ValidationError({"port": "Значение PORT не верно (0 - 65535)."}) 
```

## 5. Пишем файл в папке views для создания страницы API
---

Создаем в папке views файл в котором будет реализовано управление API

```python
from rest_framework import viewsets, permissions #Импорты для апи 

#Импорты нашей модели и сериализатора для отправки сообщений
from users.models import Camera  
from users.serializers import CameraCreationSerializer  
  
  
class CameraViewSet(viewsets.ModelViewSet): #viewsets.ModelViewSet нужен для того чтобы страничка была шаблоном из рест фреймворка
    queryset = Camera.objects.all() #Вывод всех существующих элементов и вывод ответа апи на наши данные
    http_method_names = ['get', 'post' ,'head', 'options', 'list'] #Методы которые будут реалтзованы
    permission_classes = [permissions.IsAuthenticated] #Указываем что доступ будет только авторезированным пользователям 
    serializer_class = CameraCreationSerializer #Указываем на то что при преобразовании данных в ответ будет использоваться написанный ранее сериализатор 
```

## 6. Обработать запрос

В файле views прописать 
```Python
router = routers.DefaultRouter()
router.register(r'camera', CameraViewSet, basename='camera')
```
