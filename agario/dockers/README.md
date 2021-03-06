В этом каталоге расположены docker-файлы, которые будут использоваться для запуска решений.  
В **каждом** docker-файле должны быть реализованы:
 * Установка необходимого инструментария (размещается в секции `RUN`) для соответствующего языка (включая все необходимые библиотеки, среди которых обязательно должна быть библиотека для работы с `json`)
 * Серия ENV-переменных, в которых описывается компиляция (если нужна) и запуск принятого от игрока решения (включая пути к файлам и директориям, размечаются секциями `ENV`)
 * Базовая простейшая стратегия, которая просто едет к ближайшей еде (при подготовке `pull-request`-а можно взять за основу любую из уже выложенных вместе с изначальными docker-файлами)

Каждый docker-файл предполагает, что решение участника будет лежать в недрах папки `/opt/client/`  
В каждом docker-файле доступна переменная `$MOUNT_POINT`, в ней находится путь, по которому можно найти залитое решение пользователя (в момент компиляции/запуска не компилируемого языка). В момент запуска компилируемого языка там будет лежать бинарник.

Предполагается, что авторы `pull-request`-ов будут писать docker-файл, ориентируясь на структуру уже существующих

### Требования к docker-файлам более развернуто:

 * Желательно, на каждый файл обойтись одной `RUN`-секцией (в случае нескольких вызовов, можно элементарно обойтись `&&`, например `RUN apt-get install python python-pip && pip install sklearn`)
 * Каждый docker-файл наследуется от `ubuntu:16.04`
 * В каждом docker-файле должны быть определены в секциях `ENV` по меньшей мере такие переменные:
  * `RUN_COMMAND` - Команда, находящаяся в этой переменной, будет исполнена для старта стратегии
  * `SOLUTION_CODE_PATH` - В ней содержится путь к папке с исходниками (например, для `java` это `ENV SOLUTION_CODE_PATH=/opt/client/src/main/java/`)
  * `SOLUTION_CODE_ENTRYPOINT` - Здесь содержится путь к файлу (относительно `SOLUTION_CODE_PATH`), который служит "точкой входа" в решении игрока. Для C++ это `ENV SOLUTION_CODE_ENTRYPOINT=main.cpp`, для Go это `ENV SOLUTION_CODE_ENTRYPOINT=main.go` итд

### Прочие возможные переменные в секции `ENV`

 * `COMPILATION_COMMAND`. Будет вызвана во время компиляции (разница между `COMPILATION_COMMAND` и `RUN_COMMAND` в том, что `COMPILATION_COMMAND` вызывается один раз для присланного решения, таким образом мы экономим время и имеем возможность сразу же показать участникам ошибки компиляции)

### Переменные по умолчанию

Если в docker-файле явно не определены `MOUNT_POINT` и `SOLUTION_CODE_PATH`, то применяется:
 * `ENV MOUNT_POINT=/opt/client/code`
 * `ENV SOLUTION_CODE_PATH=/opt/client/solution`

### На какой процесс запуска можно рассчитывать

В соответствии в docker-файлом для того языка, на котором прислано решение, происходит ровно следующее:
1. Архив с решением разархивируется в директорию, указанную в `SOLUTION_CODE_PATH` docker-файла. Если решение пришло просто исходником, оно просто копируется в `SOLUTION_CODE_PATH`/`SOLUTION_CODE_ENTRYPOINT`
2. Если необходима стадия компиляции, то из директории `/opt/client/` вызывается команда, лежащая в `COMPILATION_COMMAND`
3. Если всё хорошо, запускается tcp-клиент `aicups`, которому отдается команда, лежащая в `RUN_COMMAND`. Tcp-клиент запускает решение игрока, просто выполняя эту команду. С этого момента для решения игрока существует лишь `STDIN`, `STDOUT` и набор библиотек, описанных в соответствующем docker-файле
