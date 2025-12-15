# rs_2
# Лабораторная работа № 2. Проектирование и реализация клиент-серверной системы. HTTP, веб-серверы и RESTful веб-сервисы
## Цель работы
Изучить методы отправки и анализа HTTP-запросов с
использованием инструментов telnet и curl, освоить базовую настройку и
анализ работы HTTP-сервера nginx в качестве веб-сервера и обратного
прокси, а также изучить и применить на практике концепции архитектурного
стиля REST для создания веб-сервисов (API) на языке Python.

## Оборудование и программное обеспечение:
- Операционная система: Ubuntu 20.04.6 LTS (в рамках предоставленного образа).
- Сетевые утилиты: telnet, curl.
- Веб-сервер: nginx.
- Среда разработки:
- Интерпретатор Python 3.8+.
- Система управления пакетами python3-pip.
- Инструмент для создания виртуальных окружений python3-
venv.
- Микрофреймворк Flask для реализации REST API.
- Доступ к сети Интернет.

# Вариант 12
1. HTTP-анализ. Анализ ответа от публичного API Sberbank (sberbank.ru/ru/quotes/currencies).
2. Разработка REST API. API для "Заказы в магазине" (сущность: id, customer_name, items).
3. Настройка Nginx. Настроить блок upstream и балансировку (даже с одним сервером).
# Ход работы
Создаем диреекторию проекта, переходим в нее и устанавливаем необходимые утилиты
````
sudo apt install curl telnet libxml2-utils jq -y
````
## Задание 1
Необходимо провести анализ ответа от публичного API Sberbank (sberbank.ru/ru/quotes/currencies).

Выполним следующую команду:
````
curl -i https://www.sberbank.ru/ru/quotes/currencies
````

Результат:

<img width="616" height="430" alt="image" src="https://github.com/user-attachments/assets/eb75ce88-d971-4ed6-9c79-bd9bd402ddab" />

По этому скриншоту можно понять, что команда отрабатывает успешно на уровне HTTP, но Сбербанк блокирует запрос и вместо страниц с курсами отдает защитную HTML‑страницу. Рассмотрим, что же пошло не так: 
````
Возникла проблема при открытии сайта Сбербанка в этом браузере.Возможно у
Вас не установлены сертификаты Национального УЦ Минцифры России. Ознакомиться с
инструкциями по установке можно на "https://www.gosuslugi.ru/crt"
s://www.gosuslugi.ru/crt</a>.Либо попробуйте войти на сайт в другом браузере по ссылке
"https://www.sberbank.com/ru/certificates" https://www.sberbank.com/ru/certificates
Если ошибка повторится позвоните нам по номеру 900 или + 7495 500-55-50, если Вы за границей,
и сообщите ваш Support ID
````
При обращении к ресурсу Сбербанка https://www.sberbank.ru/ru/quotes/currencies с помощью curl вместо данных о курсах валют я получила HTML‑страницу, сгенерированную системой веб‑защиты с сообщением о необходимости выполнения определенных условий. Несмотря на статус HTTP/2 200, по данному URL фактически возвращается защита сайта, поэтому обработка ответа как XML с помощью xmllint не позволяет получить курсы валют.

## Задание 2
Необходимо реализовать: разработка REST API. API для "Заказы в магазине" (сущность: id, customer_name, items).

Сначала создадим папку проекта,виртуальное окружение и активируем его:
````
mkdir ordersapi
````
````
cd ordersapi.​
````

````
python3 -m venv venv
````
````
source venv/bin/activate
````
<img width="211" height="306" alt="image" src="https://github.com/user-attachments/assets/7981f742-fc55-4f4c-9674-de5ed1a8df10" />


Установлим Flask
````
pip install Flas
````
<img width="591" height="409" alt="image" src="https://github.com/user-attachments/assets/a4da55cc-c032-41b9-b232-fd79f6485203" />

В VS Code создаем новый файл app.py в папке ordersapi и вставьте в него следующий код:
````
from flask import Flask, jsonify, request
from datetime import datetime

#Инициализация Flask-приложения
app = Flask(name)

#Используем простой список словарей в качестве "базы данных" 
orders = [
{'id': 1, 'customer_name': 'Alice', 'items': ['book', 'pen'], 'created_at': datetime.now().isoformat()},
{'id': 2, 'customer_name': 'Bob', 'items': ['laptop'], 'created_at': datetime.now().isoformat()}
]

next_id = 3

#Эндпоинт для получения всех заказов
@app.route('/api/orders', methods=['GET'])
def get_orders():
   """Возвращает список всех заказов."""
   return jsonify({'orders': orders})

#Эндпоинт для получения одного заказа по id
@app.route('/api/orders/int:order_id', methods=['GET'])
def get_order(order_id):
   """Возвращает один заказ по его идентификатору."""
   order = next((o for o in orders if o['id'] == order_id), None)
   if order is None:
      return jsonify({'error': 'Заказ с таким id не найден'}), 404
   return jsonify(order)

#Эндпоинт для создания нового заказа
@app.route('/api/orders', methods=['POST'])
def add_order():
   """Добавляет новый заказ в список."""
   global next_id
   data = request.json
   if not data or not all(k in data for k in ['customer_name', 'items']):
      return jsonify({'error': 'Поля customer_name и items обязательны'}), 400
   if not isinstance(data['items'], list):
      return jsonify({'error': 'Поле items должно быть списком'}), 400
   new_order = {
      'id': next_id,
      'customer_name': data['customer_name'],
      'items': data['items'],
      'created_at': datetime.now().isoformat()
    }
   orders.append(new_order)
   next_id += 1
   return jsonify(new_order), 201

#Точка входа для запуска сервера
if name == 'main':
   app.run(host='0.0.0.0', port=5000)
````

<img width="634" height="412" alt="image" src="https://github.com/user-attachments/assets/c4e1ef84-08b1-4b21-8d6f-1e8adca10af9" />

Запускаем API
````
python3 app.py
````
<img width="577" height="247" alt="image" src="https://github.com/user-attachments/assets/284ab7c4-8e87-40b3-9d12-f81f65229fbf" />

В другом терминале запрашиваем информацию о существующих заказах
````
curl -s http://127.0.0.1:5000/api/orders | jq
````
<img width="592" height="340" alt="image" src="https://github.com/user-attachments/assets/e84be088-9062-4caa-aab3-63238fe63482" />

<img width="580" height="306" alt="image" src="https://github.com/user-attachments/assets/22f361c8-2308-419f-8c3d-3f01abd5bfb1" />


Добавим еще одну покупку

````
 curl -s -X POST -H "Content-Type: application/json" \  
    -d '{"customer_name":"Kristina","items":["phone","case"]}' \  
    http://127.0.0.1:5000/api/orders | jq  
````
<img width="589" height="291" alt="image" src="https://github.com/user-attachments/assets/096f0f24-8c9e-4ccf-9cfe-2c82aea53403" />

Задание выполнено, созданный API выводит запрашиваемые данные и в него добавляются новые покупки.

## Задание 3
Переходим к выполнению следующей задачи: настройка Nginx. Настроить блок upstream и балансировку (даже с одним сервером).

Установка и подключение Nginx
````
sudo apt update  
sudo apt install nginx -y  
sudo systemctl start nginx  
sudo systemctl enable nginx
````
Проверка дефолтной страницы
````
curl http://localhost
````

<img width="598" height="301" alt="image" src="https://github.com/user-attachments/assets/177e7fb8-29cd-49b0-adc1-15a822c50ac9" />

Продолжим настройка upstream. Откроем конфигурационный файл: 
````
sudo nano /etc/nginx/nginx.conf
````
<img width="595" height="379" alt="image" src="https://github.com/user-attachments/assets/e2f5348e-6bb9-48ec-b47e-07ab4da61997" />


Внутри блока http { ... } добавим описание upstream для Flask‑API заказов:
````
 upstream orders_backend {  
     server 127.0.0.1:5000;    
}
````
Внутри server в блок location добавим новый блок для API заказов
````
    location /api/orders {  
       # перенаправляем запросы к пулу backend-серверов  
        proxy_pass http://orders_backend;  
        proxy_set_header Host $host;  
        proxy_set_header X-Real-IP $remote_addr;  
  }
````

Проверяем синтаксис

````
sudo nginx -t
````

<img width="579" height="133" alt="image" src="https://github.com/user-attachments/assets/6ccf1d87-7b3a-4c5d-801d-c2cec58c190e" />

Теперь перейдем к тестированию, будем работать в двух терминалах: Flask и client. В терминале Flask запустим сервис.
