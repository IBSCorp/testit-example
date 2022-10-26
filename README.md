# Пример интеграции с TEST IT

## Настройки

Для интеграции необходимо выставить настройки.

Данные, доступные для настройки:

* url – URL сервера Test IT (**обязательный параметр**)
* privateToken – секретный ключ в профиле Test IT (**обязательный параметр**)
* projectId – id рабочего проекта в системе Test IT (**обязательный параметр**)
* configurationId – id конфигурации в рабочем проекте в системе Test IT (**обязательный параметр**)
* testRunId - id TestRun в который необходимо залить результаты (можно не задавать, если хотите создать новый)
* testRunName - имя TestRun (если не задать, то имя TestRun будет равно текущей даты запуска)
* proxy - proxy для сервера Test IT (можно не задавать)
* enabled - параметр для отправки результатов в Test IT:  true - если хотите отправлять результаты в Test IT, false - если не хотите (можно не задавать, по умолчанию true)
* configFile - путь до конфигурационного файла json, в котором можно задать все настройки выше (можно не задавать - в таком случае будет искаться конфигурационный файл с именем по умолчанию)

Памятка, как определить значения нужных параметров из системы Test IT:
* url – скопировать из адресной строки браузера, включая обозначение протокола https|https и имя сервера (опционально может быть добавлен номер порта через двоеточие)
* privateToken – нажать иконку пользователя в верхнем правом углу, далее: Настройки профиля
* projectId -  получить через api-запрос swagger /api/v2/projects/{id}, в качестве параметра передать id проекта из url (при открытии проекта в приложении Test IT) 
* configurationId -  получить через api-запрос swagger /api/v2/configurations/{id}, в качестве параметра передать id конфигурации из url (при открытии конфигурации в приложении Test IT) 

Настройки можно задать следующими способами:

1. Конфигурационный файл
2. env (переменные окружения)
3. CLI (command line interface = system properties)
4. QualIT Plugin (если работаете локально в IntelliJ Idea)

### Конфигурационный файл

Имя конфигурационного файла можно указать через CLI или переменные окружения через параметр testit.configFile, в котором необходимо указать путь до конфигурационного файла
от каталога src/test/resources (файл должен находиться в этом каталоге или его подкаталогах)

Пример через CLI, если файл лежит в корне src/test/resources:

```
-Dtestit.configFile=dev_config.json
```

Или если файл лежит src/test/resources/config:

```
-Dtestit.configFile=config/config.json
```

Также можно не задавать путь до конфигурационного файла вовсе, если он лежит в корне каталога src/test/resources и
его имя config.json. Это будет считаться конфигурационным файлом по умолчанию и адаптер сам его найдет и считает параметры из этого файла.

Пример конфигурационного файла:

```json
{
  "url": "URL сервера Test IT",
  "privateToken": "Секретный ключ в профиле Test IT",
  "projectId": "Id рабочего проекта в системе Test IT",
  "configurationId": "Id конфигурации в рабочем проекте в системе Test IT",
  "testRunId": "Id TestRun, в который необходимо залить результаты",
  "testRunName": "Имя TestRun",
  "proxy": "Proxy для сервера Test IT"
}
```
Если в конфигурационном файле указывается PrivateToken, то в логах вы увидите warning.

### env (переменные окружения) и CLI (system properties)

Настройки Adapter и Client также можно задавать через переменные окружения или CLI. Параметры в таком случае будут
переопределяться от низкого к высокому:

1. Конфигурационный файл
2. env (переменные окружения)
3. CLI (system properties) - будут иметь наивысший приоритет.

Пример установки параметров через CLI:

```
-Dtestit.projectId=123 -Dtestit.configurationId=321 -Dtestit.privateToken=token -Dtestit.url=url -Dtestit.proxy=proxy 
-Dtestit.testRunName=name -Dtestit.testRunId=111 -Dtestit.enabled=true
```

В переменных окружения указывайте параметры аналогичным образом, но уже без ключа -D

```
testit.configurationId=321 
testit.configFile=config/dev_config.json
```

### QualIT Plugin

Параметры, заданные в QualIT Plugin, будут переданы как system properties и иметь наивысший приоритет.

## Вспомогательные методы

Адаптер предоставляет пользователю вспомогательные методы для отправки данных в результаты автотестов:

1. createAttachments() – массив, от 1 до N. Прикрепляемые к результату файлы.
2. addLinks() – массив, от 1 до N. Ссылки в результатах автотеста.
3. addMessage() – string, который будет дописываться, а в конце теста заливать собранное информативное сообщение об автотесте (в результатах автотеста), добавляя errorMessage (не stack trace) от тестового фреймворка.

Для использования метода createAttachments() необходимо импортировать класс AttachmentSteps и объявить его переменную через аннотацию @Autowired.
Для использования методов addLinks() и addMessage() необходимо импортировать класс TestITSteps и также объявить его переменную через аннотацию @Autowired.

```java
import ru.ibsqa.qualit.steps.AttachmentSteps;
import ru.ibsqa.qualit.testit.steps.TestITSteps;

@Component
public class TestSteps {
    @Autowired
    private TestITSteps testITSteps;
    @Autowired
    private AttachmentSteps attachmentSteps;
}
```

Далее можно пользоваться методами, которые предоставляют классы TestITSteps и AttachmentSteps.

### Примеры

```
 attachmentSteps.createAttachment(new File("путь к файлу"), "mimeType")
 
 attachmentSteps.createAttachment("Имя_Файла.txt", "Текст, который будет в файле", "text/plain");
 
 testITSteps.addLink("https://www.ibs.ru/", "Title", "Description", LinkType.RELATED);
 
 testITSteps.addMessage("Шаг прошел успешно.");
```

## Тестовый сценарий

Адаптер позволяет составлять тестовые сценарии на языке Gherkin в feature файлах. В сценариях можно задавать метаданные через тэги:

* @ExternalId – ID автотеста в рабочем проекте системы Test IT (**обязательный параметр**).
* @WorkItemId – привязка автотеста к ручному тест-кейсу.
* @DisplayName – название автотеста в списке автотестов.
* @Description – описание в карточке автотеста.
* @Link – ссылки в карточке автотеста.
* @label – тэги в карточке автотеста. Задаются через символ @ без знака равно.
* Сценарий – название в карточке автотеста. Задается не через тег, а в поле "Сценарий".

### Пример тестового сценария

```gherkin
Функция: Тестовая функция

  @ExternalId=0001
  @Description=Очень_важный_тест
  @WorkItemId=1234
  @DisplayName=TestScript
  @Link=https://testit.ru @Link=https://ibs.ru
  @testLabel @newLabel @topLabel
  Сценарий: Тестовый сценарий

    * Авторизация

    * Проверка

    * Выход
```

Некоторые вспомогательные методы можно также использовать напрямую в тестовом сценарии. Например:

```gherkin
    * добавить к результату теста файл "путь к файлу" с типом "image/jpeg"

    * добавить к результату теста сообщение "Тест завершился успешно!"
```