## Фриланс для репетиторов
Этот проект представляет собой API, часть backend-приложения для просмотра и принятия заказов репетиторами. В проекте реализовано:
- Различные виды связи в БД;
- Интеграция со стороним API;
- Взаимодействие между пользователями;
- Права доступа к API.
## Оглавление

1. [Используемый стек технологий](#используемый-стек-технологий)
2. [Запуск](#запуск)
3. [Тестирование](#тестирование)
4. [Документация](#Документация)
5. [Автор](#автор)

## Используемый стек технологий
- FastAPI
- SQLAlchemy
- PostgreSQL(asyncpg)
- Alembic
- Pytest(pytest-asyncio)
- Redis
- Celery
- Git
- Docker

## Запуск:
Клонируйте репозиторий на локальную машину:
```commandline
$ git clone https://github.com/kolyaslow/freelance_tutor.git
```
Выполните команды Docker.
```docker
$ sudo docker compose up
```

## Тестирование:
Выполните команды Docker.
```docker
$ sudo docker compose exec app pytest -s -v
```

## Документация

<details>
<summary>Схема БД</summary>

![photo](/photo/db.png)

>Сущность User
```
id(PK) - уникальный идентификатор записи
email - email пользователя указанный при регистрации
hashed_password - хэш пароля
is_active - показатель, что пользователь пользуется аккаунтом
is_superuser - поле показывающее, что пользователь суперпользователь
is_verified - поле показывающее, что пользователь подтвердил email
role - роль пользователя (репетитор, ученик)
```

>Сущность Profile
```
id(PK) - уникальный идентификатор записи
fullname - ФИО репетитора
description - описание профиля
```

>Сущность Subject
```
name(PK) - название предмета
```

>Сущность SubjectUserAssociation
```
id(PK) - уникальный идентификатор записи
user_id(FK) - id пользователя (репетитора)
subject_name(FK) - название предмета
```

>Сущность Order
```
id(PK) - уникальный идентификатор записи
user_id(FK) - id пользователя (ученика)
subject_name(FK) - название предмета
description - описание профиля
is_active - поле показывающее, открыт ли заказ или закрыт
```

>Сущность Response
```
id(PK) - уникальный идентификатор записи
order_id(FK) - id заказа
user_id(FK) - id пользователя (репетитора)
status - показатель принятие репетитора, как исполнителя
```

>Сущность ConfirmationKeys
```
id(PK) - уникальный идентификатор записи
user_id(FK) - id пользователя (репетитора)
email_confirmation_code - код подтверждения email пользователя
```
</details>

<details>

<summary>Схема проекта</summary>

```commandline
|   main.py             # Точка входа проекта
|   pyproject.toml      # Зависимости проекта
|
+---alembic     # Модуль миграции БД
+---api_v1      # Модуль API_V1
|   |   schemas_confirmation_keys.py    # Pydentic схемы для таблицы confirmation_keys
|   |   __init__.py                     # Инициализатор пакета, где все роутеры собираются для последуещего импорта в экземпляр fastapi(app)
|   |
|   +---common  # Модуль с общими функциями необходимыми API
|   |   |   crud.py                     # Модуль для взаимодействия с базой данных
|   |   |   dependencies.py             # Модуль для описания зависимостей
|   |
|   +---order
|   |   |   crud.py
|   |   |   dependencies.py
|   |   |   schemas.py
|   |   |   views.py        # Модуль для описание endpoint API
|   |
|   +---profile
|   |   |   crud.py
|   |   |   dependencies.py
|   |   |   schemas.py
|   |   |   views.py
|   |
|   +---subject
|   |   |   crud.py
|   |   |   dependencies.py
|   |   |   schemas.py
|   |   |   views.py
|   |
|   +---task_selery
|   |   |   config.py       # Конфигурация даных для модуля
|   |   |   send_email.py   # Модуль отправки письма на email
|   |
|   +---user
|   |   |   config.py
|   |   |   crud.py
|   |   |   fastapi_user.py     # Модуль создания экземпляра FastapiUser
|   |   |   schemas.py
|   |   |   views.py
|   |
+---core
|   |   config.py           # Конфигурация проекта, в том числе, бд
|   |   db_helper.py        # Создание AsyncEngine, AsycSessionFactory
|   |   __init__.py
|   |
|   +---models
|   |   |   base.py                         # Модуль базовой модели ORM
|   |   |   confirmation_keys.py
|   |   |   mixins.py                       # Модуль примесей для создания связей между таблицами БД
|   |   |   order.py
|   |   |   profile.py
|   |   |   subject.py
|   |   |   subject_user_association.py     # Таблица для связи "многие-ко-многим" между таблицами subject и user
|   |   |   user.py
|   |   |   __init__.py                     # Инициализация всех элементов для работы с БД через SQLalchemy.
+---tests   # Модуль с тестами проекта
|   |   conftest.py       # Общие фикстуры необходимые тестам
|   |   test_inaccessibility_api.py     # Тесты проверки авторизации API
|   |
|   +---common
|   |   |   base_request_api.py     # Модуль формирования и оправки тестовых запросов
|   |   |   fixture_profile_management.py       # Модуль фикстур, отвечающих за управление профилем
|   |   |   subject_fixture.py
|   |   |   user_authentication_fixture.py      # Модуль аутентификации пользователей с разными правами
|   |   |   __init__.py
|   |
|   +---order
|   |   |   conftest.py
|   |   |   test_router_create_order.py     # Тесты для роутера create_order
|   |   |   test_router_delete_order.py
|   |   |   test_router_getting_orders_for_tutor.py
|   |   |   test_router_get_all_orders.py
|   |
|   +---profile
|   |   |   test_router_create.py
|   |   |   test_router_delete.py
|   |   |   test_router_update.py
|   |
|   +---user
|   |   |   test_router_get_subjects_by_user.py
|   |   |   test_router_show_all_tutor_by_subject.py
|   |
```
</details>

<details>

<summary>Описание API</summary>
  <div style="margin-left: 20px;">

  После запуска интерактивная документация доступна по адресу (Реализовано через OpenAPI(Swagger)):
  ```
  http://127.0.0.1:8008/docs#/
  ```
Все API, кроме API аутентификации, доступны *только* аутентифицированным пользователем,
которые подтвердили свою почту

 <details>

<summary>API аутентификации</summary>
<div style="margin-left: 20px;">
  <details>

  <summary>Регистрация пользователя</summary>

- Описание: Регистрирует пользователя в системе, а также отправляет письмо с кодом подтверждения на указанный при регистрации email.
- Метод: POST.
- Запрос:

```
/auth/register
```

- Параметры запроса:
<table>
  <tr>
    <th>Поле</th>
    <th>Тип</th>
    <th>Обязательный параметр</th>
    <th>Описание</th>
  </tr>
  <tr>
    <td>email</td>
    <td>string</td>
    <td>Да</td>
    <td>email пользователя</td>
  </tr>
  <tr>
    <td>password</td>
    <td>string</td>
    <td>Да</td>
    <td>пароль</td>
  </tr>
  <tr>
    <td>role</td>
    <td>string</td>
    <td>Да</td>
    <td>роль пользователя, соответствующая значениям: tutor, customer</td>
  </tr>
</table>

- Тело ответа:
```json
{
  "id": 16,   # id записи в БД
  "email": "use4r@example.com",   # email указанный при регистрации
  "is_active": true,    # показатель блокировки пользователя, всегда проставляется в значение True
  "is_superuser": false,    # суперпользователь, всегда проставляется в значение False
  "is_verified": false,   # показатель подтверждения почты пользователем
  "role": "tutor"   # роль указанная при регистрации
}
```

- Ошибки:
<table>
  <tr>
    <th>Статус код</th>
    <th>Описание</th>
    <th>Возвращаемый ответ</th>
  </tr>
  <tr>
    <td>400</td>
    <td>Попытка повторной регистрации пользователя</td>
    <td>

```
{
  "detail": "REGISTER_USER_ALREADY_EXISTS"
}
```
</td>
  </tr>
</table>

  </details>
<details>

  <summary>Подтвердение почты</summary>

- Описание: Проверка кода подтверждения отправленного при регистрации и установка поля is_verified=True,
при неверном указании почты исключение не выкидывается.
При успешном подтверждении вернется статус код 200.
- Метод: POST.
- Запрос:

```
/user/verify_user
```
- Параметры запроса:
<table>
  <tr>
    <th>Поле</th>
    <th>Тип</th>
    <th>Обязательный параметр</th>
    <th>Описание</th>
  </tr>
  <tr>
    <td>user_email </td>
    <td>string</td>
    <td>Да</td>
    <td>email указанный при регистрации</td>
  </tr>
  <tr>
    <td>code</td>
    <td>string</td>
    <td>Да</td>
    <td>Код подтверждления отправленный на почту</td>
  </tr>
</table>

- Ошибки:
<table>
  <tr>
    <th>Статус код</th>
    <th>Описание</th>
    <th>Возвращаемый ответ</th>
  </tr>
  <tr>
    <td>401</td>
    <td>Возникает при неверном указании кода</td>
    <td>None</td>
  </tr>
</table>
  </details>
<hr style="width: 100%;">
</div>
</details>
<details>

  <summary>API для работы с профилем</summary>

 <div style="margin-left: 20px;">

API доступно лишь репетиторам.
При использовании API другими пользователями вызывается исключение:
ошибка недоступности API см. [общие исключения](#общие_исключения).
  <details>

  <summary>Создание профиля</summary>

- Описание: создание профиля для репетиторов.
- Метод: POST.

- Запрос:
```
/profile/create_profile
```
- Параметры запроса:
<table>
  <tr>
    <th>Поле</th>
    <th>Тип</th>
    <th>Обязательный параметр</th>
    <th>Описание</th>
  </tr>
  <tr>
    <td>fullname</td>
    <td>string</td>
    <td>Нет</td>
    <td>ФИО пользователя</td>
  </tr>
  <tr>
    <td>description</td>
    <td>string</td>
    <td>Нет</td>
    <td>Описание профиля репетитора</td>
  </tr>
  <tr>
    <td>user_id</td>
    <td>int</td>
    <td>Нет</td>
    <td>id репетитора, заполняется автоматически</td>
  </tr>
</table>

- Тело ответа:
```json
{
  "fullname": "string",   # полное имя пользователя, указанного при создании профиля
  "description": "string"   # описание, указанное при создании профиля
}
```
- Ошибки:
<table>
  <tr>
    <th>Статус код</th>
    <th>Описание</th>
    <th>Возвращаемый ответ</th>
  </tr>
  <tr>
    <td>422</td>
    <td>Попытка повторного создания профиля</td>
    <td>

```
{
  "detail": "Профиль для пользователя с именем user@example.com уже создан",
}
```
</td>
  </tr>
</table>

</details>

  <details>

  <summary>Обновление профиля</summary>

- Описание: запрос позволяет обновить профиль репетитора.
- Метод: PATCH.
- Запрос:
```json
/profile/update_profile
```

- Параметры запроса:
<table>
  <tr>
    <th>Поле</th>
    <th>Тип</th>
    <th>Обязательный параметр</th>
    <th>Описание</th>
  </tr>
  <tr>
    <td>fullname</td>
    <td>string</td>
    <td>Нет</td>
    <td>ФИО пользователя</td>
  </tr>
  <tr>
    <td>description</td>
    <td>string</td>
    <td>Нет</td>
    <td>Описание профиля репетитора</td>
  </tr>
  <tr>
    <td>user_id</td>
    <td>int</td>
    <td>Нет</td>
    <td>id репетитора, заполняется автоматически</td>
  </tr>
</table>

- Тело ответа:
```json
{
  "fullname": "string",   # ФИО, указанное при обновлении
  "description": "string", # описание, указанное при обновлнеи
  "user_id": 0    # id пользователя, репетитора
}
```
</details>

<details>

  <summary>Удаление профиля</summary>

- Описание: Удаление профиля репетитора. При успешном удалении возвращается статус код 204.
- Метод: DELETE.
- Запрос:
```json
/profile/delete_profile
```

- Ошибки:
<table>
  <tr>
    <th>Статус код</th>
    <th>Описание</th>
    <th>Возвращаемый ответ</th>
  </tr>
  <tr>
    <td>404</td>
    <td>Попытка удалить несуществующий профиль</td>
    <td>

```
{
  "detail": "No profile was found for user user@gmail.com"
}
```
</td>
  </tr>
</table>
</details>

<details>
  <summary>Добавление предметов, которые ведет репетитор</summary>

- Описание: добавление в профиль **списка предметов**, которые репетитор может вести.
При успешном добавлении, вернется статус код 200.
- Метод: POST.

- Запрос:
```
/user/add_subject
```

- Тело запроса:
```json
[
  "name_subject"    # название предмета, соответсвующее предметам из таблици Subject
]
```


</details>
</div>
<hr style="width: 100%;">
</details>

<details>
  <summary>API для работы с заказом</summary>
<div style="margin-left: 20px;">

  API доступны лишь пользователям, являющиеся заказчиками, т.е. поле `role=customer`.
  При попытке получить доступ не заказчикам вызывает исключение:
  ошибка недоступности API см. [общие исключения](#общие_исключения).

  <details>

<summary>Создание заказа</summary>

- Описание: cоздание заказа, при повторном создании заказа, исключение не выкидывается.
- Метод: POST.
- Запрос:

```json
/order/create_order
```
- Параметры запроса:
<table>
  <tr>
    <th>Поле</th>
    <th>Тип</th>
    <th>Обязательный параметр</th>
    <th>Описание</th>
  </tr>
  <tr>
    <td>description</td>
    <td>string</td>
    <td>Да</td>
    <td>Описание заказа</td>
  </tr>
  <tr>
    <td>is_active</td>
    <td>bool</td>
    <td>Нет</td>
    <td>Готовность получать отклики на заказ.
При значение True, заказа открыт к откликам.
Значение по умолчанию True.</td>
  </tr>
  <tr>
    <td>subject_name</td>
    <td>str</td>
    <td>Да</td>
    <td>Предмет по которому ищется репетитор</td>
  </tr>
<tr>
    <td>user_id</td>
    <td>int</td>
    <td>Нет</td>
    <td>id заказчика, заполняется автоматически</td>
  </tr>
</table>

- Тело ответа:

```json
{
  "description": "string",
  "is_active": true,
  "subject_name": "mathematics",
  "user_id": 0,
  "id": 0   # id заказа
}
```

</details>
    <details>

<summary>Получение своих заказов, заказчиком</summary>

- Описание: Получение всех заказов, которые создал закзачик.
При отсутсвии заказавов, вернет пустой список.
- Метод: GET.

- Запрос:

```json
/order/get_all_orders
```
- Тело ответа:
```json
[
  {
    "description": "string",
    "is_active": true,
    "subject_name": "mathematics",
    "user_id": 0,   # id заказчика
    "id": 0
  },
]
```
</details>
<details>

<summary>Получение всех заказов для репетитора</summary>

- Описание: получение всех заказов, которые репетитор может вести,
то есть предметы в заказе и те, что ведет репетитор совпадают, а также закза открыт для откликов
(поле заказа is_active=True )
При отсутсвии заказов, вернется пустой список. *Доступно лишь репетиторам.*
- Метод: GET.
- Запрос:
```
/order/getting_orders_for_tutor

```
- Параметры запроса:
<table>
  <tr>
    <th>Поле</th>
    <th>Тип</th>
    <th>Обязательный параметр</th>
    <th>Описание</th>
  </tr>
  <tr>
    <td>page</td>
    <td>int</td>
    <td>Нет</td>
    <td>

Указывает страницу пагинации, *значения должны быть больше 0*.
При указании недопутимого значения, выкидывается исключение:
ошибка валидации см. [общие исключения](#общие_исключения). *Значение по умолчанию 0.*
</td>
  </tr>
  <tr>
    <td>size</td>
    <td>int</td>
    <td>Нет</td>
    <td>

Количетво элемнтов выдаваемых за раз.
*Допустимые значения от 10 до 100*.
При указании недопутимого значения, выкидывается исключение:
ошибка валидации см. [общие исключения](#общие_исключения).
*Значение по умолчанию 10.*
</td>
  </tr>
</table>

- Тело ответа:
```json
[
  {
    "description": "string",    # описание заказа
    "is_active": true,    # открыт ли заказа, для откликов
    "subject_name": "mathematics",  # предмет, который требуется проводить
    "user_id": 0  # id заказачика
  }
]
```
</details>
<details>

<summary>Удаление заказа</summary>

- Описание: при успешном удалении заказа, вернется код 204.
- Метод: DELETE.
- Запрос:
```
/order/delete_order/id_order

id_order: int - id заказа
```
- Ошибки:
<table>
  <tr>
    <th>Статус код</th>
    <th>Описание</th>
    <th>Возвращаемый ответ</th>
  </tr>
  <tr>
    <td>404</td>
    <td>Попытка удаления несуществующего закза</td>
    <td>

```
{
  "detail": "Незвозможно получить объект по его id"
}
```
</td>
  </tr>
</table>

</details>
</div>
 <hr style="width: 100%;">
</details>


<details>

  <summary> Просмотр репетиторов</summary>

- Описание: Получения списка репетиторов по опредленному предмету,
если таких репетиторов нет, вернет пустой список.
- Метод: GET.
- Запрос:
```
/user/show_all_tutor_by_subject/{name_subject}
```

- Параметры запроса:
<table>
  <tr>
    <th>Поле</th>
    <th>Тип</th>
    <th>Обязательный параметр</th>
    <th>Описание</th>
  </tr>
  <tr>
    <td>name_subject</td>
    <td>string</td>
    <td>Да</td>
    <td>Название предмета, по которому ищете репетиторов</td>
  </tr>
  <tr>
    <td>page</td>
    <td>int</td>
    <td>Нет</td>
    <td>

Указывает страницу пагинации, *значения должны быть больше 0*.
При указании недопутимого значения, выкидывается исключение:
ошибка валидации см. [общие исключения](#общие_исключения). *Значение по умолчанию 0.*
</td>
  </tr>
  <tr>
    <td>size</td>
    <td>int</td>
    <td>Нет</td>
    <td>

Количетво элемнтов выдаваемых за раз.
*Допустимые значения от 10 до 100*.
При указании недопутимого значения, выкидывается исключение:
ошибка валидации см. [общие исключения](#общие_исключения).
*Значение по умолчанию 10.*
</td>
  </tr>

<tr>
    <td>price_sorting</td>
    <td>bool</td>
    <td>Нет</td>
    <td>
Сортировка по цене за услугу, при значение:

- True - сортируется по возрастанию цены;
- False - сортируется по убыванию цены;

Значение по умолчанию False.
</td>
  </tr>

<tr>
    <td>rating_sorting</td>
    <td>bool</td>
    <td>Нет</td>
    <td>
Сортировка по рейтингу репетиторов, при значение:

- True - сортируется по возрастанию рейтинга;
- False - сортируется по убыванию рейтинга;

Значение по умолчанию False.
</td>
  </tr>
</table>

- Тело ответа:
```json
]
  {
    "fullname": "string", # ФИО репетитора
    "description": "string" # описание профиля репетитора
  },
]
```
- ВАЖНО: Соровка вначале идет по цене, потом по рейтингу.

</details>
<details>

  <summary id="общие_исключения">Общие исключения</summary>

<table>
  <tr>
    <th>Статус код</th>
    <th>Описание</th>
    <th>Возвращаемый ответ</th>
  </tr>
  <tr>
    <td>422</td>
    <td>Ошибка валидации входных данных</td>
    <td>

```json
{
  "detail": [
    {
      "loc": [
        "string",
        0
      ],
      "msg": "string",
      "type": "string"
    }
  ]
}
```
</td>
  </tr>


  <tr>
    <td>401</td>
    <td>Возникает при попытки получить доступ к недостуцпным API</td>
    <td>

```json
{
  "detail": "Unauthorized"
}
```
</td>
  </tr>
</table>
</details>
</div>
</details>


## Автор
**Николай Пышенко**
- email: pysenkon@gmail.com
- Telegram: [@koliaslow](https://t.me/koliaslow)
- VK: [@koliaslow](https://vk.com/koliaslow)
