## Очікувані результати
- PR з усіма змінами
- Скріншоти виконання кожного пункту завдання
- Короткий опис виконаних дій
- Опис будь-яких проблем, з якими ви зіткнулися
- Час який було витрачено на кожен етап та порівняння з попереднім результатом
## Завдання
1. **Скопіювати Dockerfile**
   - Cкопіюйте [Dockerfile](/Dockerfile) для створення образу з демо-сервісом у власний проект.
2. **Створити та запустити контейнер**
   - Відкотіть версії бібліотек у `requirements.txt`.
flask==3.0.0
psycopg2-binary==2.9.8

   - Побудуйте образ на основі цого Dockerfile.



Чтобы решить проблему с зависимостями в Dockerfile изменяем команду для виртуального окружения и установки пакетов:

# Создаем виртуальное окружение и устанавливаем пакеты через него
RUN python3 -m venv /venv && \
    /venv/bin/pip install --no-cache-dir -r requirements.txt
Наконец собираем image


И проверяем:

docker@docker:~/DOCKER-AND-KUBERNETES_MARTYNIUK_1$ docker images
REPOSITORY          TAG       IMAGE ID       CREATED          SIZE
app-docker          latest    4a83ec06946a   35 seconds ago   614MB
tonistiigi/binfmt   latest    69b92b14f1eb   2 months ago     80.9MB
docker@docker:~/DOCKER-AND-KUBERNETES_MARTYNIUK_1$ cat Dockerfile


   - Запустіть контейнер з цього образу та переконайтеся, що сервіс працює.
Запускаем контейнер
docker@docker:~/DOCKER-AND-KUBERNETES_MARTYNIUK_1$ docker run -d -p 5000:5000 app-docker
6061cf757a49ae152db96dd77a0a149d3ce294758e75b073484c99aa1f4e31e1
docker@docker:~/DOCKER-AND-KUBERNETES_MARTYNIUK_1$ docker ps
CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS         PORTS                                         NAMES
6061cf757a49   app-docker   "venv/bin/python3 ap…"   6 seconds ago   Up 5 seconds   0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp   elegant_curran

И проверяяем




3. **Внесіть зміни в код та оновіть залежності**
   - Змініть текст, що відображається на головній сторінці.
меняем app.py

<body>
    <h1>Demo Application ver 2.0</h1>
    <p>Current time: {{ current_time }}</p>

   - Оновіть версії бібліотек у `requirements.txt`.
меняем версию flask
flask==3.0.1

Меняем порт в Dockerfile
EXPOSE 5001
   - Перебудуйте образ і переконайтеся, що все працює з новими версіями.
Строим новый образ




4. **Запустити два сервіси різних версій одночасно**
   - Запустіть два контейнери: один зі старою версією сервісу, інший — з оновленою.
Запускаем второй контейнер

docker@docker:~/DOCKER-AND-KUBERNETES_MARTYNIUK_1$ docker run -d -p 5001:5000 app-docker2
4de08cc68b5c2eb58088385eeea747c1bf8fcd1eccec54fe429a7570cc2e9c68
docker@docker:~/DOCKER-AND-KUBERNETES_MARTYNIUK_1$ docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED              STATUS              PORTS                                         NAMES
4de08cc68b5c   app-docker2   "venv/bin/python3 ap…"   About a minute ago   Up About a minute   0.0.0.0:5001->5000/tcp, [::]:5001->5000/tcp   amazing_hodgkin
86519946b927   app-docker    "venv/bin/python3 ap…"   15 minutes ago       Up 15 minutes       0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp   happy_newton
docker@docker:~/DOCKER-AND-KUBERNETES_MARTYNIUK_1$

   - Переконайтеся, що обидва сервіси працюють на різних портах.
Проверяем 



																	
5. **Порівняйте підходи**
   - Порівняйте результати з першим ДЗ по часу і складності
Однозначно докер выигрывает, что не удивительно.

   - Далі трекати час не потрібно
Около 2.5 часов, но в основном из-за того что я только начал пользоваться githab.
6. **Оптимізувати Dockerfile**
   - Перепишіть Dockerfile, використовуючи методи оптимізації.
Переписанный файл докер

# Используем более легкий базовый образ с Python
FROM python:3.9-slim
# Устанавливаем только необходимые системные зависимости
RUN apt-get update && \
    apt-get install -y --no-install-recommends libpq-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
# Создаем и переходим в рабочую директорию
WORKDIR /app
# Сначала копируем только requirements.txt для кэширования слоя с зависимостями
COPY requirements.txt .
# Устанавливаем зависимости
RUN pip install --no-cache-dir -r requirements.txt
# Копируем остальные файлы
COPY . .
# Порт приложения
EXPOSE 5000
# Команда запуска
CMD ["python", "app.py"]

Размер уменьшился в 3.2 раза

docker@docker:~/DOCKER-AND-KUBERNETES_MARTYNIUK_1$ docker images
REPOSITORY          TAG       IMAGE ID       CREATED          SIZE
app-docker_opt      latest    2b41d327fecf   15 seconds ago   187MB
<none>              <none>    9f1ed3c7836b   18 minutes ago   614MB

   - Поясніть, які саме оптимізації ви застосували.

• Заменил ubuntu:24.04 на python:3.9-slim (экономия ~100MB)
• В slim-образе уже установлен Python и pip, не нужно ставить их отдельно
1. Объединение команд RUN:
• Объединил все apt-команды в один RUN для уменьшения количества слоев
• Добавил --no-install-recommends для установки только основных зависимостей
• Добавил очистку кэша apt (rm -rf /var/lib/apt/lists/*)
2. Оптимизация копирования файлов:
• Сначала копирую только requirements.txt
• Затем устанавливаю зависимости
• И только потом копирую остальные файлы
• Это позволяет использовать кэш слоя с зависимостями при изменении кода
3. Упрощение работы с Python:
• Убрал виртуальное окружение 
Использую системный Python из образа Прочие улучшения:
• Упростил CMD (не нужно указывать путь к Python в venv)
• Явно указал версию Python (3.9) для воспроизводимости

7. **Створити та завантажити оптимізований образ**
   - Побудуйте оптимізований образ.
Все аналогично предыдущему построению docker build -t app-docker_opt . 

   - Завантажте останні два образи на DockerHub.
Выгрузил первую версию (около 2-х минут)

   

 - Порівняйте їх розміри, час збірки та завантаження
Размеры сравнил ранее. Время загрузки оптимизированного образа менее 30 сек.


