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

## Проработка атаки


