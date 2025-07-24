# Запуск
Первый делом клонируем репозиторий на вашу машину:
```bash
git clone https://github.com/OPG-Team/gph_services.git
```

Переходим в директорию с проектом:
```bash
cd gph_sevices
```
Теперь в папку cd rabbitmq, создаём папку mkdir rabbitmq_logs и задаём права доступа: chown -R 999:999 rabbitmq_logs/

Возвращаемся в корень и копируем настройки из `.env.example` в `.env`:
```bash
cp .env.example .env
```

Выставляем нужные настройки в `.env`

KC_HOSTNAME=ваш_ip                           ### Укажите на каком ip будут работать сервисы ###
BASE_URL=http://ваш_ip:16580/    
KC_SERVER_PUBLIC_URL=http://ваш_ip:8080/   

## Настройки по `keycloak` для `api_gateway`
Чтобы правильно настроить работу keycloak с api_gateway нужно запустить первым делом keycloak_web (и keycloakdb) и настроить его через web-интерфейс.
```bash
docker-compose up --build -d keycloak_web
```

Запускается на http://ваш_ip:8080.

Данные для админа что вы указали в `.env`, как `KEYCLOAK_ADMIN` и `KEYCLOAK_ADMIN_PASSWORD`, по дефолту:
- Логин: `admin`
- Пароль: `admin`

### 1. Переходим в "Administration Console"
### 2. Создаем realm
Когда вы находитесь в админ панели, сверху слева вы увидите надпись `master` - это и есть текущий realm

Для нашего проекта создаем новый realm, для этого нажимаем на `master` и нажимаем "Create realm"

В открывшемся окне указываем только "Realm name" - данное название указываем в `.env` как `KC_REALM_NAME`

### 3. Создаем "Client"
**Client** - это сервис который будет работать в нашем realm (а не user!!!), в нашем случае это api_gateway 

- Переходим в "Client"
- Нажимаем "Create client"
    - Client type: `OpenID Connect`
    - Client ID: Название для client'а, это название -> `.env`, как `KC_CLIENT_ID`
    - Нажимаем "Next"
- На 2 странице всключаем флаг "Client authentication" и "Next"
- На 3 странице не меняем настройки и нажимаем "Save"

Теперь должна открыться страница с данным клиентом и его настройки, вам нужно добавить настройки (пример настроек):
- Valid redirect URIs (URL нашей API, для корректного редиректа с Keycloak):
    - http://ваш_ip:16580/docs/oauth2-redirect (для swagger)
    - http://ваш_ip:16580/oauth2-redirect (для swagger)
    - http://ваш_ip:16580/auth/callback
    - http://ваш_ip:16580/auth/login
- Web origins (Основной URL вашего API):
    - http://ваш_ip:16580
- Нажимаем "Save"

### 4. Получаем `KC_CLIENT_SECRET_KEY`
На странице нашего клиента находим "Credential"

И на странице копируем ключ "Client Secret" и вписываем в `.env`, как `KC_CLIENT_SECRET_KEY`

На этом настройка keycloak завершена.

### 5. Создадим user для теста авторизации
1. Переходим в "User"
2. Нажимам "Add user"
3. Указываем только "Username"
4. Нажимаем "Create user"
5. Заходим в "Credentials" и устанавливаем пароль для пользователя (убрать галочку с Temporary)

И запускам проект через **docker-compose**:
```bash
docker-compose up --build -d
```

### 5. Теперь можем запускать `api_gateway`

### 6. Тест авторизации
Для проверки работы API в целом можем пойти этим путем:
1. Открываем в браузере http://ваш_ip:16580/auth/login

Нас должно перенести на форму для авторизации, там указываем данные тестового пользователя

2. Далее нам должно отправить на http://ваш_ip:16580/auth/callback... и вернуть JSON с **access** и **refresh** токенами

Для проверки в Swagger:
1. Переходим на http://ваш_ip:16580/docs 
2. Нажимаем кнопку "Authorize" и вводим в "client_secret" ключ из keycloak (мы его внесли в `.env` как `KC_CLIENT_SECRET_KEY`), и снова нажимаем на "Authorize"
3. Если все успешно то нас перекинет на форму для авторизации (если вы уже авторизовались  с браузера то он сразу войдет по куки)
4. Теперь нас вернуло на Swagger и теперь мы авторизованы
