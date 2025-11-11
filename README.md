# 🤖 Автоматизация тестирования веб-формы «Пользователи»

![Статус](https://img.shields.io/badge/%D0%A1%D1%82%D0%B0%D1%82%D1%83%D1%81-%D0%9F%D1%80%D0%BE%D0%B5%D0%BA%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5_%D0%B8_%D1%87%D0%B0%D1%81%D1%82%D0%B8%D1%87%D0%BD%D0%B0%D1%8F_%D1%80%D0%B5%D0%B0%D0%BB%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F-blue)  
![Язык](https://img.shields.io/badge/Язык-Python_3.12-orange)  
![Фреймворк](https://img.shields.io/badge/Фреймворк-Pytest_+_Selenium-green)  
![Отчётность](https://img.shields.io/badge/Отчёты-Allure-lightgrey)  
![Дата](https://img.shields.io/badge/Дата-Ноябрь_2025-lightgrey)  

---
# 🧾 Описание задания

**Задание:** Спроектировать и частично реализовать автоматизацию для формы «Пользователи».

**Цель:** Создать структуру проекта автотестов, реализовать базовые Page Object'ы, стратегию тестовых данных, инструкции запуска и план дальнейшей автоматизации.

**Ожидаемые артефакты:**
1. Код и структура проекта (Page Object'ы, тесты, конфигурации).
2. README-файл с описанием шагов запуска и зависимостей.
3. Примеры тестов на Pytest с использованием Page Object'ов.
4. Краткий план автоматизации и сбор метрик.

---

**! Примечание**: Данное задание было выполнено с использованием нейросетей, так как я все еще процессе изучения автоматизации тестирования на Python. 

---

## 📁 Структура проекта

```
user_table_tests/
├── README.md                  # инструкция по запуску
├── requirements.txt            # зависимости проекта
├── pytest.ini                  # конфигурация Pytest
├── conftest.py                 # фикстуры (инициализация драйвера, URL и т.д.)
├── config/
│   └── settings.py             # BASE_URL, BROWSER, REMOTE и т.п.
├── data/
│   └── user_data.py            # тестовые данные и генерация пользователей
├── pages/
│   ├── base_page.py            # базовый класс Page Object
│   ├── user_table_page.py      # таблица пользователей
│   └── add_user_modal.py       # модальное окно "Добавить пользователя"
├── tests/
│   ├── test_add_user.py        # позитивные и негативные кейсы
│   ├── test_sorting.py         # проверка сортировки
│   ├── test_settings.py        # тесты сохранения настроек
│   └── test_validation.py      # проверки валидации форм
└── utils/
    └── helpers.py              # утилиты (генерация email, ожидания и т.д.)
```

---

## 🧩 Пример Page Object — `user_table_page.py`

```python
from selenium.webdriver.common.by import By

class UserTablePage:
    ADD_USER_BTN = (By.XPATH, "//button[contains(.,'Добавить')]")
    SEARCH_INPUT = (By.CSS_SELECTOR, "[placeholder='Поиск']")
    SORT_LOGIN = (By.XPATH, "//th[contains(.,'Логин')]")
    TABLE_ROWS = (By.CSS_SELECTOR, "tbody tr")

    def __init__(self, driver):
        self.driver = driver

    def open(self, base_url):
        self.driver.get(f"{base_url}/form/user-table")
        return self

    def click_add_user(self):
        self.driver.find_element(*self.ADD_USER_BTN).click()
        from pages.add_user_modal import AddUserModal
        return AddUserModal(self.driver)

    def search_user(self, login):
        field = self.driver.find_element(*self.SEARCH_INPUT)
        field.clear()
        field.send_keys(login)
        field.submit()
        return self

    def sort_by_login(self):
        self.driver.find_element(*self.SORT_LOGIN).click()
        return self

    def user_is_displayed(self, name):
        rows = self.driver.find_elements(*self.TABLE_ROWS)
        return any(name in row.text for row in rows)
```

---

## 🧩 Пример Page Object — `add_user_modal.py`

```python
from selenium.webdriver.common.by import By

class AddUserModal:
    TITLE = (By.CSS_SELECTOR, ".modal-title")
    NAME = (By.ID, "firstName")
    LASTNAME = (By.ID, "lastName")
    EMAIL = (By.ID, "email")
    ADD_BTN = (By.XPATH, "//button[contains(.,'Добавить')]")
    ERRORS = (By.CSS_SELECTOR, ".error-message")

    def __init__(self, driver):
        self.driver = driver

    def check_title(self, expected="Добавить пользователя"):
        assert expected in self.driver.find_element(*self.TITLE).text
        return self

    def fill_form(self, name, lastname, email):
        self.driver.find_element(*self.NAME).send_keys(name)
        self.driver.find_element(*self.LASTNAME).send_keys(lastname)
        self.driver.find_element(*self.EMAIL).send_keys(email)
        return self

    def submit(self):
        self.driver.find_element(*self.ADD_BTN).click()
        from pages.user_table_page import UserTablePage
        return UserTablePage(self.driver)

    def has_error(self, text):
        errors = self.driver.find_elements(*self.ERRORS)
        return any(text in e.text for e in errors)
```

---

## 🧪 Пример реального теста на Pytest — `test_add_user.py`

```python
import pytest
from pages.user_table_page import UserTablePage
from data.user_data import generate_user

@pytest.mark.usefixtures("driver")
class TestAddUser:

    def test_add_valid_user(self, driver, base_url):
        """Проверяет успешное добавление пользователя с корректными данными."""
        user_table = UserTablePage(driver).open(base_url)
        modal = user_table.click_add_user()
        new_user = generate_user(valid=True)

        modal.check_title().fill_form(new_user["name"], new_user["lastname"], new_user["email"]).submit()

        assert user_table.user_is_displayed(new_user["name"]), (
            f"Пользователь {new_user['name']} не отображается в таблице после добавления"
        )

    def test_add_user_with_short_name(self, driver, base_url):
        """Проверяет сообщение об ошибке при вводе имени короче 3 символов."""
        user_table = UserTablePage(driver).open(base_url)
        modal = user_table.click_add_user()

        invalid_user = {"name": "Al", "lastname": "Bo", "email": "test@example.com"}
        modal.fill_form(invalid_user["name"], invalid_user["lastname"], invalid_user["email"]).submit()

        assert modal.has_error("Слишком короткий текст"), "Сообщение об ошибке для короткого имени не отображается"

    def test_add_user_with_invalid_email(self, driver, base_url):
        """Проверяет отображение ошибки при вводе некорректного email."""
        user_table = UserTablePage(driver).open(base_url)
        modal = user_table.click_add_user()

        invalid_user = {"name": "Иван", "lastname": "Петров", "email": "ivan@wrongdomain"}
        modal.fill_form(invalid_user["name"], invalid_user["lastname"], invalid_user["email"]).submit()

        assert modal.has_error("Некорректный email"), "Ошибка о неправильном формате email не отображается"
```

---

## ⚙️ Стратегия тестовых данных

- **Подход:** генерация данных динамически через библиотеку `faker`.
- **Типы тестов:**
  - Позитивные — корректные имя, фамилия, email (`.com`, `.lt`, `.ru` и др.)
  - Негативные — пустые поля, короткие строки, неверный формат email
- **Изоляция:** каждый тест создает и удаляет свои данные — без зависимости между кейсами.
- **Модуль `data/user_data.py`** содержит фабрику данных для переиспользования.

```python
from faker import Faker
fake = Faker('ru_RU')

def generate_user(valid=True):
    if valid:
        return {
            "name": fake.first_name(),
            "lastname": fake.last_name(),
            "email": fake.email()
        }
    else:
        return {
            "name": "A",
            "lastname": "B",
            "email": "wrong-email"
        }
```

---

## 🚀 Инструкции запуска

### 🔧 Требования

- Python ≥ 3.10  
- Google Chrome / Brave  
- Установленный [Allure](https://docs.qameta.io/allure/)

### 📦 Установка зависимостей

```bash
pip install -r requirements.txt
```

### 📄 Пример `requirements.txt`

```
pytest
selenium
allure-pytest
faker
pytest-xdist
```

---

## ⚙️ Переменные окружения

| Переменная | Назначение |
|-------------|------------|
| `BASE_URL` | https://mes.inka.team |
| `BROWSER` | chrome / firefox |
| `REMOTE` | (опционально) адрес Selenium Grid или Selenoid |

---

## ▶️ Запуск тестов

Запуск всех тестов:

```bash
pytest -v --alluredir=allure-results
```

Просмотр отчета:

```bash
allure serve allure-results
```

---

## 🧭 План дальнейшей автоматизации

1. **Приоритетные тесты:**
   - Добавление пользователя (валидные и невалидные данные)
   - Проверка валидации полей (имя, фамилия, email)
   - Сортировка и фильтрация пользователей
   - Проверка сохранения пользовательских настроек
   - Smoke-тест на загрузку формы и доступность UI-элементов

2. **Следующие этапы:**
   - Добавить тесты для редактирования и удаления пользователей  
   - Настроить CI/CD (GitHub Actions, Jenkins)  
   - Реализовать запуск в Docker / Selenoid  
   - Подключить отчётность в Allure TestOps или Telegram

3. **Метрики:**
   - Успешность выполнения тестов  
   - Время прогонов  
   - Покрытие UI-функционала  
   - Динамика стабильности по сборкам

---

## 👤 Автор проекта

**Давид Гаргулия**  
QA Engineer | Автоматизация формы «Пользователи»  
📅 Ноябрь 2025

