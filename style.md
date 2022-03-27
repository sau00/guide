<!--

Editing this document:

- Discuss all changes in GitHub issues first.
- Update the table of contents as new sections are added or removed.
- Use tables for side-by-side code samples. See below.

Code Samples:

Use 2 spaces to indent. Horizontal real estate is important in side-by-side
samples.

For side-by-side code samples, use the following snippet.

~~~
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
BAD CODE GOES HERE
```

</td><td>

```go
GOOD CODE GOES HERE
```

</td></tr>
</tbody></table>
~~~

(You need the empty lines between the <td> and code samples for it to be
treated as Markdown.)

If you need to add labels or descriptions below the code samples, add another
row before the </tbody></table> line.

~~~
<tr>
<td>DESCRIBE BAD CODE</td>
<td>DESCRIBE GOOD CODE</td>
</tr>
~~~

-->

# Uber Go Style Guide

## Дисклеймер
Для более глубокого понимания рекомендуется читать параллельно с английской версией.

## Содержание

- [Предисловие](#Предисловие)
- [Гайдлайны](#Гайдлайны)
  - [Указатели на интерфейсы](#Указатели-на-интерфейсы)
  - [Проверка соответствия интерфейса](#проверка-соответствия-интерфейса)
  - [Получатели и Интерфейсы](#получатели-и-интерфейсы)
  - [Допустимо использование Мьютексов с нулевыми значениями](#допустимо-использование-Мьютексов-с-нулевыми-значениями)
  - [Копирование слайсов и мап на границах](#копирование-слайсов-и-мап-на-границах)
  - [Отложенный вызов функции для высвобождения ресурсов](#отложенный-вызов-функции-для-высвобождения-ресурсов)
  - [Размер канала либо Один либо не указан](#размер-канала-либо-один-либо-не-указан)
  - [Начинайте перечисления с единицы](#начинайте-перечисления-с-единицы)
  - [Используйте пакет `"time"` для работы со временем](#используйте-пакет-"time"-для-работы-со-временем)
  - [Errors](#errors)
    - [Error Types](#error-types)
    - [Error Wrapping](#error-wrapping)
    - [Error Naming](#error-naming)
  - [Handle Type Assertion Failures](#handle-type-assertion-failures)
  - [Don't Panic](#dont-panic)
  - [Use go.uber.org/atomic](#use-gouberorgatomic)
  - [Avoid Mutable Globals](#avoid-mutable-globals)
  - [Avoid Embedding Types in Public Structs](#avoid-embedding-types-in-public-structs)
  - [Avoid Using Built-In Names](#avoid-using-built-in-names)
  - [Avoid `init()`](#avoid-init)
  - [Exit in Main](#exit-in-main)
    - [Exit Once](#exit-once)
- [Performance](#performance)
  - [Prefer strconv over fmt](#prefer-strconv-over-fmt)
  - [Avoid string-to-byte conversion](#avoid-string-to-byte-conversion)
  - [Prefer Specifying Container Capacity](#prefer-specifying-container-capacity)
      - [Specifying Map Capacity Hints](#specifying-map-capacity-hints)
      - [Specifying Slice Capacity](#specifying-slice-capacity)
- [Style](#style)
  - [Avoid overly long lines](#avoid-overly-long-lines)
  - [Be Consistent](#be-consistent)
  - [Group Similar Declarations](#group-similar-declarations)
  - [Import Group Ordering](#import-group-ordering)
  - [Package Names](#package-names)
  - [Function Names](#function-names)
  - [Import Aliasing](#import-aliasing)
  - [Function Grouping and Ordering](#function-grouping-and-ordering)
  - [Reduce Nesting](#reduce-nesting)
  - [Unnecessary Else](#unnecessary-else)
  - [Top-level Variable Declarations](#top-level-variable-declarations)
  - [Prefix Unexported Globals with _](#prefix-unexported-globals-with-_)
  - [Embedding in Structs](#embedding-in-structs)
  - [Local Variable Declarations](#local-variable-declarations)
  - [nil is a valid slice](#nil-is-a-valid-slice)
  - [Reduce Scope of Variables](#reduce-scope-of-variables)
  - [Avoid Naked Parameters](#avoid-naked-parameters)
  - [Use Raw String Literals to Avoid Escaping](#use-raw-string-literals-to-avoid-escaping)
  - [Initializing Structs](#initializing-structs)
      - [Use Field Names to Initialize Structs](#use-field-names-to-initialize-structs)
      - [Omit Zero Value Fields in Structs](#omit-zero-value-fields-in-structs)
      - [Use `var` for Zero Value Structs](#use-var-for-zero-value-structs)
      - [Initializing Struct References](#initializing-struct-references)
  - [Initializing Maps](#initializing-maps)
  - [Format Strings outside Printf](#format-strings-outside-printf)
  - [Naming Printf-style Functions](#naming-printf-style-functions)
- [Patterns](#patterns)
  - [Test Tables](#test-tables)
  - [Functional Options](#functional-options)
- [Linting](#linting)

## Предисловие

Стили - это соглашения, определяющие качество нашего кода. Термин стиль является не достаточно полным, так как данное соглашение описывает гораздо больше, чем просто форматирование исходного кода программы, c которым и так справляется `gofmt`.

Целью данного руководства является упрощение понимания, того как можно и нужно, а как нельзя писать код на Go в Uber. Эти правила необходимы для того, чтобы сохранить контроль над качеством кодовой базой проекта и при этом позволить программистам эффективно использовать возможности языка.

Данное руководство было создано [Prashant Varanasi] и [Simon Newton] чтобы помочь коллегам начать использовать Go. С течением времени в него были внесены изменения на основе обратной связи читателей.

  [Prashant Varanasi]: https://github.com/prashantv
  [Simon Newton]: https://github.com/nomis52

Данный документ является соглашением, которому мы следуем в Uber. Многое из руководства является общими рекомендациями написания кода на Go, в то время как некоторые идеи были почерпнуты из внешних источников:

1. [Effective Go](https://golang.org/doc/effective_go.html)
2. [Go Common Mistakes](https://github.com/golang/go/wiki/CommonMistakes)
3. [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)

Код не должен содержать ошибок при запуске `golint` и `go vet`. Мы рекомендуем настроить ваш редактор на:

- Автоматический запуск `goimports` во время сохранения
- Автоматический запуск `golint` и `go vet` для проверки на ошибки

Информацию по поддержке Go инструментов вашим редактором можно найти здесь:
<https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins>

## Гайдлайны

### Указатели на интерфейсы
Практически никогда нет необходимости использовать указатель на интерфейс. Интерфейсы необходимо передавать по значению, в то время как данные в интерфейсе могут содержать в себе указатель.

Интерфейс состоит из:

1. Указателя на специфичную для данного типа информацию. Можно думать об этом поле как о типе.
2. Указатель на данные. Если поле содержит указатель, то он сохраняется напрямую. Если поле содержит значение, то сохраняется указатель на это значение.

При этом, если вы хотите, чтобы методы интерфейса могли модифицировать содержащиеся в нем значения, необходимо использовать указатель.

### Проверка соответствия интерфейса

Рекомендуется проверять соответствие интерфейса во время компиляции, в местах где это необходимо:

- Экспортированные типы, необходимые для реализации определенных интерфейсов в рамках договоренностей API
- 

- Exported types that are required to implement specific interfaces as part of their API contract
- Экспортированные или неэкспортированные типы, являющиеся частью коллекции типов, реализующих один и тот же интерфейс
- Другие случаи, когда нарушение интерфейса может сбить с толку

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
type Handler struct {
  // ...
}



func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  ...
}
```

</td><td>

```go
type Handler struct {
  // ...
}

var _ http.Handler = (*Handler)(nil)

func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

</td></tr>
</tbody></table>

Присовение `var _ http.Handler = (*Handler)(nil)` не скомпилируется если `*Handler` перестанет имплементировать интерфейс `http.Handler`.

Правая часть присвоения должна быть нулевым значением типа, стоящего слева. Это `nil` для указателя (например `*Handler`), слайсов, мап, и пустых структур для структурных типов.

```go
type LogHandler struct {
  h   http.Handler
  log *zap.Logger
}

var _ http.Handler = LogHandler{}

func (h LogHandler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

### Получатели и Интерфейсы

Методы с получателями по значению могут также вызываться указателями.
Методы с получателями по указателю могут быть вызваны только указателями или [адресными значениями].

[адресные значения]: https://golang.org/ref/spec#Method_values

Например,

```go
type S struct {
  data string
}

func (s S) Read() string {
  return s.data
}

func (s *S) Write(str string) {
  s.data = str
}

sVals := map[int]S{1: {"A"}}

// Можно вызвать только .Read()
sVals[1].Read()

// Не скомпилируется:
// sVals[1].Write("test")

sPtrs := map[int]*S{1: {"A"}}

// Использовав указатель можно вызвать .Read() и .Write()
sPtrs[1].Read()
sPtrs[1].Write("test")
```

Аналогично интерфейс может быть имплементирован указателем, даже если получатель метода передан по значению.

```go
type F interface {
  f()
}

type S1 struct{}

func (s S1) f() {}

type S2 struct{}

func (s *S2) f() {}

s1Val := S1{}
s1Ptr := &S1{}
s2Val := S2{}
s2Ptr := &S2{}

var i F
i = s1Val
i = s1Ptr
i = s2Ptr

// Данный код не скомпилируется, s2Val является значением, нет реализованного получателя по значению метода f.
//   i = s2Val
```

В Effective Go об этом написано более подробно [Pointers vs. Values].

[Pointers vs. Values]: https://golang.org/doc/effective_go.html#pointers_vs_values

### Допустимо использование Мьютексов с нулевыми значениями

Нулевые значения `sync.Mutex` и `sync.RWMutex` допустимы, поэтому вам практически никогда не понадобится указатель на мьютекс.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
mu := new(sync.Mutex)
mu.Lock()
```

</td><td>

```go
var mu sync.Mutex
mu.Lock()
```

</td></tr>
</tbody></table>

Если используется структура по указателю, мьютекс должен быть полем по значению.

Не используйте эмбеддинг (встраивание) мьютекса в структуре, даже если это неэкспортируемая структура.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
type SMap struct {
  sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.Lock()
  defer m.Unlock()

  return m.data[k]
}
```

</td><td>

```go
type SMap struct {
  mu sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.mu.Lock()
  defer m.mu.Unlock()

  return m.data[k]
}
```

</td></tr>

<tr><td>

Поле `Mutex`, а также методы `Lock` и `Unlock` непреднамеренно становятся частью экспортируемого API структуры `SMap`.

</td><td>

Мьютекс и его методы в `SMap` спрятаны от внешнего вызова.

</td></tr>
</tbody></table>

### Копирование слайсов и мап на границах

Слайсы и мапы хранят указатели на содержащиеся в них данные, так что будьте осторожны в тех ситуациях, когда вам необходимо их копировать.

#### Получение слайсов и мап

Необходимо помнить, что значения мапы или слайса, переданной в качестве аргумента, могут быть изменены извне.

<table>
<thead><tr><th>Плохо</th> <th>Хорошо</th></tr></thead>
<tbody>
<tr>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// Вы собираетесь поменять значение в d1.trips?
trips[0] = ...
```

</td>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// Теперь мы можем поменять значение trips[0] без изменения значения d1.trips.
trips[0] = ...
```

</td>
</tr>

</tbody>
</table>

#### Возвращение слайсов или мап

Аналогично, имейте ввиду, что пользователи смогут изменить содержимое возвращаемой мапы или слайса.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

// Snapshot возвращает текущую статистику.
func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  return s.counters
}

// snapshot более не защищен мьютексом, любой может
// получить доступ к snapshot, что приведет к гонке.
snapshot := stats.Snapshot()
```

</td><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// Теперь snapshot является копией.
snapshot := stats.Snapshot()
```

</td></tr>
</tbody></table>

### Отложенный вызов функции для высвобождения ресурсов

Используйте `defer` для высвобождения таких ресурсов как файлы или блокировки.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
p.Lock()
if p.count < 10 {
  p.Unlock()
  return p.count
}

p.count++
newCount := p.count
p.Unlock()

return newCount

// легко потерять unlock из-за множественного return
```

</td><td>

```go
p.Lock()
defer p.Unlock()

if p.count < 10 {
  return p.count
}

p.count++
return p.count

// более читаемо
```

</td></tr>
</tbody></table>

Отложенный вызов функции имеет очень маленький оверхед по ресурсам и его следует избегать только если есть реальное обоснование того, что время выполнения функции не должно превышать порядков в наносекунды. Читаемость кода, получаемая благодаря использованию отложенных вызовов сильно выше накладных расходов. Это особенно заметно в больших методах, где есть больше чем просто чтение из памяти, а остальные вычисления значительно затратнее чем `defer`. 

### Размер канала либо Один либо не указан

Каналы должны обычно иметь либо размерность один, либо быть небуферизированными. По умолчанию каналы небуферизированы и имеют размер равный нулю. Любой другой размер должен быть предметом тщательных обсуждений. Заранее определенный размер канала предотвращает его переполнение под высокой нагрузкой, что позволяет избежать блокировок.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
//  64kb должно хватить на всех!
c := make(chan int, 64)
```

</td><td>

```go
// Размерность один
c := make(chan int, 1) // или
// Небуферизированный канал размерностью 0
c := make(chan int)
```

</td></tr>
</tbody></table>

### Начинайте перечисления с единицы

Стандартным способом объявления перечислений в Go является создание кастомного типа и группы `const` при помощи `iota`. Так как значением по умолчанию для переменных является 0, необходимо начинать перечисления с ненулевого значения, например с 1.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota
  Subtract
  Multiply
)

// Add=0, Subtract=1, Multiply=2
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

// Add=1, Subtract=2, Multiply=3
```

</td></tr>
</tbody></table>

Существуют случаи, когда использование нулевого значения имеет смысл, например,
в тех ситуациях, когда нулевое значение является ожидаемым значением по умолчанию.

```go
type LogOutput int

const (
  LogToStdout LogOutput = iota
  LogToFile
  LogToRemote
)

// LogToStdout=0, LogToFile=1, LogToRemote=2
```

### Используйте пакет `"time"` для работы со временем

Время - это сложная штука. Часто делаются неверные предположения о времени, которые включают в себя следующее:

1. День состоит из 24 часов
2. Час состоит из 60 минут
3. Неделя состоит из 7 дней
4. Год состоит из 365 дней
5. [Еще больше](https://infiniteundo.com/post/25326999628/falsehoods-programmers-believe-about-time)

Например, *1* означает, что добавление 24 часов к моменту времени не всегда дает новый календарный день.

Поэтому всегда пользуйтесь пакетом [`"time"`] при работе со временем, так как он помогает работать с этими неверными предположениями более безопасным и точным способом.

  [`"time"`]: https://golang.org/pkg/time/

#### Используйте `time.Time` для конкретных моментов времени

Используйте [`time.Time`] при работе с моментами времени и методы для `time.Time` при сравнении, добавлении или вычитании времени.

  [`time.Time`]: https://golang.org/pkg/time/#Time

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
func isActive(now, start, stop int) bool {
  return start <= now && now < stop
}
```

</td><td>

```go
func isActive(now, start, stop time.Time) bool {
  return (start.Before(now) || start.Equal(now)) && now.Before(stop)
}
```

</td></tr>
</tbody></table>

#### Используйте `time.Duration` для временных промежутков

Используйте [`time.Duration`] когда работаете с временными промежутками.

  [`time.Duration`]: https://golang.org/pkg/time/#Duration

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
func poll(delay int) {
  for {
    // ...
    time.Sleep(time.Duration(delay) * time.Millisecond)
  }
}

poll(10) // это 10 секунд или милисекунд?
```

</td><td>

```go
func poll(delay time.Duration) {
  for {
    // ...
    time.Sleep(delay)
  }
}

poll(10*time.Second)
```

</td></tr>
</tbody></table>

Возвращаясь к примеру с добавлением 24 часов, метод, который мы используем для добавления времени зависит от того что мы хотим получить в итоге. Если мы хотим то же время дня, но на следующий календарный день, мы должны использовать [`Time.AddDate`]. Однако, если мы хотим, чтобы момент времени гарантированно был через 24 часа после предыдущего времени, мы должны использовать [`Time.Add`].

  [`Time.AddDate`]: https://golang.org/pkg/time/#Time.AddDate
  [`Time.Add`]: https://golang.org/pkg/time/#Time.Add

```go
newDay := t.AddDate(0 /* года */, 0 /* месяцы */, 1 /* дни */)
maybeNewDay := t.Add(24 * time.Hour)
```

#### Используйте `time.Time` и `time.Duration` при взаимодействии с внешними системами

Используйте `time.Duration` и `time.Time` при взаимодействии с внешними системами, когда это возможно. Например:

- Флаги командной строки: [`flag`] поддерживает `time.Duration` при помощи [`time.ParseDuration`]
- JSON: [`encoding/json`] поддерживает `time.Time` по стандарту [RFC 3339] строки при помощи метода [`UnmarshalJSON`]
- SQL: [`database/sql`] поддерживает конвертацию `DATETIME` или `TIMESTAMP` в `time.Time` и обратно в зависимости от используемого драйвера
- YAML: [`gopkg.in/yaml.v2`] поддерживает `time.Time` по стандарту [RFC 3339] строки, а также `time.Duration` при помощи [`time.ParseDuration`].

  [`flag`]: https://golang.org/pkg/flag/
  [`time.ParseDuration`]: https://golang.org/pkg/time/#ParseDuration
  [`encoding/json`]: https://golang.org/pkg/encoding/json/
  [RFC 3339]: https://tools.ietf.org/html/rfc3339
  [`UnmarshalJSON`]: https://golang.org/pkg/time/#Time.UnmarshalJSON
  [`database/sql`]: https://golang.org/pkg/database/sql/
  [`gopkg.in/yaml.v2`]: https://godoc.org/gopkg.in/yaml.v2

В случае если нет возможности воспользоваться `time.Duration`, используйте `int` или `float64` и включите единицы измерения в название поля.

Например, так как `encoding/json` не поддерживает `time.Duration`, в примере ниже единица измерения включена в название поля.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
// {"interval": 2}
type Config struct {
  Interval int `json:"interval"`
}
```

</td><td>

```go
// {"intervalMillis": 2000}
type Config struct {
  IntervalMillis int `json:"intervalMillis"`
}
```

</td></tr>
</tbody></table>

В случае если нет возможности воспользоваться `time.Time`, и альтернативный формат несогласован, используйте `string` по стандарту [RFC 3339]. Этот формат используется по умолчанию методом [`Time.UnmarshalText`] и доступен для использования в `Time.Format` и `time.Parse` по [`time.RFC3339`].

  [`Time.UnmarshalText`]: https://golang.org/pkg/time/#Time.UnmarshalText
  [`time.RFC3339`]: https://golang.org/pkg/time/#RFC3339

Хотя на практике это обычно не представляет проблемы, имейте в виду, что пакет `"time"` не поддерживает синтаксический анализ временных меток с дополнительными секундами ([8728]) и не учитывает дополнительные секунды в расчетах ([15190]). Если вы сравните два момента времени, разница не будет включать високосные секунды, которые могли произойти между этими двумя моментами.

  [8728]: https://github.com/golang/go/issues/8728
  [15190]: https://github.com/golang/go/issues/15190

<!-- TODO: section on String methods for enums -->

### Типизация ошибок

Существует несколько вариантов создания ошибок:

- [`errors.New`] для ошибок с простой статичной строкой
- [`fmt.Errorf`] для ошибок с форматируемой строкой
- Пользовательские типы ошибок, которые имплементируют метод `Error()`
- Обернутые ошибки при помощи [`"pkg/errors".Wrap`]

Во время возвращения ошибок, учтите следующие пункты, для выбора наиболее оптимального решения:

- Возвращается простая ошибка, которая не несет в себе дополнительной информации? [`errors.New`] будет подходящим выбором.
- Клиентам необходимо получать и обрабатывать эту ошибку? Тогда необходимо использовать кастомный тип и имплементировать метод `Error()`.
- Передаете ошибку из функции, которая расположена ниже по стеку вызовов? Тогда обратите внимание на [section on error wrapping](#error-wrapping).
- Во всех остальных случаях [`fmt.Errorf`] будет хорошим выбором.

  [`errors.New`]: https://golang.org/pkg/errors/#New
  [`fmt.Errorf`]: https://golang.org/pkg/fmt/#Errorf
  [`"pkg/errors".Wrap`]: https://godoc.org/github.com/pkg/errors#Wrap

Если клиенту необходимо определять ошибку и вы создали простую ошибку при помощи [`errors.New`], используйте var для инициализации ошибки.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

func use() {
  if err := foo.Open(); err != nil {
    if err.Error() == "could not open" {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td><td>

```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if err == foo.ErrCouldNotOpen {
    // handle
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

Если у вас ошибка, с которой могут работать клиенты и вы хотели бы добавить
больше информации к ней, тогда необходимо использовать пользовательский тип.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
func open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

func use() {
  if err := open(); err != nil {
    if strings.Contains(err.Error(), "not found") {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td><td>

```go
type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func open(file string) error {
  return errNotFound{file: file}
}

func use() {
  if err := open(); err != nil {
    if _, ok := err.(errNotFound); ok {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td></tr>
</tbody></table>

Будьте осторожны с экспортом пользовательских ошибок, так как они становятся частью публичного API пакета. Желательно экспортировать функцию, которая в свою очередь проверяет ошибку.

```go
// package foo

type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func IsNotFoundError(err error) bool {
  _, ok := err.(errNotFound)
  return ok
}

func Open(file string) error {
  return errNotFound{file: file}
}

// package bar

if err := foo.Open("foo"); err != nil {
  if foo.IsNotFoundError(err) {
    // handle
  } else {
    panic("unknown error")
  }
}
```

<!-- TODO: Exposing the information to callers with accessor functions. -->

### Error Wrapping

There are three main options for propagating errors if a call fails:

- Return the original error if there is no additional context to add and you
  want to maintain the original error type.
- Add context using [`"pkg/errors".Wrap`] so that the error message provides
  more context and [`"pkg/errors".Cause`] can be used to extract the original
  error.
- Use [`fmt.Errorf`] if the callers do not need to detect or handle that
  specific error case.

It is recommended to add context where possible so that instead of a vague
error such as "connection refused", you get more useful errors such as
"call service foo: connection refused".

When adding context to returned errors, keep the context succinct by avoiding
phrases like "failed to", which state the obvious and pile up as the error
percolates up through the stack:

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "failed to create new store: %s", err)
}
```

</td><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "new store: %s", err)
}
```

<tr><td>

```
failed to x: failed to y: failed to create new store: the error
```

</td><td>

```
x: y: new store: the error
```

</td></tr>
</tbody></table>

However once the error is sent to another system, it should be clear the
message is an error (e.g. an `err` tag or "Failed" prefix in logs).

See also [Don't just check errors, handle them gracefully].

  [`"pkg/errors".Cause`]: https://godoc.org/github.com/pkg/errors#Cause
  [Don't just check errors, handle them gracefully]: https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully

### Handle Type Assertion Failures

The single return value form of a [type assertion] will panic on an incorrect
type. Therefore, always use the "comma ok" idiom.

  [type assertion]: https://golang.org/ref/spec#Type_assertions

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
t := i.(string)
```

</td><td>

```go
t, ok := i.(string)
if !ok {
  // handle the error gracefully
}
```

</td></tr>
</tbody></table>

<!-- TODO: There are a few situations where the single assignment form is
fine. -->

### Не паниковать

Код, запущенный в продакшене должен избегать паник. Паники являются главным источником [cascading failures]. Если вызываемая функция закончилась ошибкой, ее необходимо вернуть выше и позволить вызывающей функции решить, как ее обработать.

  [cascading failures]: https://en.wikipedia.org/wiki/Cascading_failure

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
func foo(bar string) {
  if len(bar) == 0 {
    panic("bar must not be empty")
  }
  // ...
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  foo(os.Args[1])
}
```

</td><td>

```go
func foo(bar string) error {
  if len(bar) == 0 {
    return errors.New("bar must not be empty")
  }
  // ...
  return nil
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  if err := foo(os.Args[1]); err != nil {
    panic(err)
  }
}
```

</td></tr>
</tbody></table>

Panic/recover не подходит в качестве механизма обработки ошибок. Программа должна
паниковать только в безвыходных ситуациях, например разыменование структуры в nil.
Исключением является инициализация программы: ошибки на старте могут прервать ее выполнение, вызвав панику.

```go
var _statusTemplate = template.Must(template.New("name").Parse("_statusHTML"))
```

Even in tests, prefer `t.Fatal` or `t.FailNow` over panics to ensure that the
test is marked as failed.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestFoo(t *testing.T)

f, err := ioutil.TempFile("", "test")
if err != nil {
  panic("failed to set up test")
}
```

</td><td>

```go
// func TestFoo(t *testing.T)

f, err := ioutil.TempFile("", "test")
if err != nil {
  t.Fatal("failed to set up test")
}
```

</td></tr>
</tbody></table>

<!-- TODO: Explain how to use _test packages. -->

### Use go.uber.org/atomic

Atomic operations with the [sync/atomic] package operate on the raw types
(`int32`, `int64`, etc.) so it is easy to forget to use the atomic operation to
read or modify the variables.

[go.uber.org/atomic] adds type safety to these operations by hiding the
underlying type. Additionally, it includes a convenient `atomic.Bool` type.

  [go.uber.org/atomic]: https://godoc.org/go.uber.org/atomic
  [sync/atomic]: https://golang.org/pkg/sync/atomic/

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type foo struct {
  running int32  // atomic
}

func (f* foo) start() {
  if atomic.SwapInt32(&f.running, 1) == 1 {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running == 1  // race!
}
```

</td><td>

```go
type foo struct {
  running atomic.Bool
}

func (f *foo) start() {
  if f.running.Swap(true) {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running.Load()
}
```

</td></tr>
</tbody></table>

## Производительность

Performance-specific guidelines apply only to the hot path.

### Используйте strconv вместо fmt

При конвертации типов в/из строки, `strconv` быстрее, чем `fmt`.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  s := fmt.Sprint(rand.Int())
}
```

</td><td>

```go
for i := 0; i < b.N; i++ {
  s := strconv.Itoa(rand.Int())
}
```

</td></tr>
<tr><td>

```
BenchmarkFmtSprint-4    143 ns/op    2 allocs/op
```

</td><td>

```
BenchmarkStrconv-4    64.2 ns/op    1 allocs/op
```

</td></tr>
</tbody></table>

### Избегайте приведения string-to-byte

Не приводите строку в слайс байтов много раз. Вместо этого 
выполните преобразование один раз и сохраните результат.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  w.Write([]byte("Hello world"))
}
```

</td><td>

```go
data := []byte("Hello world")
for i := 0; i < b.N; i++ {
  w.Write(data)
}
```

</tr>
<tr><td>

```
BenchmarkBad-4   50000000   22.2 ns/op
```

</td><td>

```
BenchmarkGood-4  500000000   3.25 ns/op
```

</td></tr>
</tbody></table>

### Старайтесь определять Capacity для мапов

Где возможно, старайтесь объявлять значение capacity при инициализации
мап при помощи `make()`.

```go
make(map[T1]T2, hint)
```

Задание capacity во время вызова `make()` выделяет память для заранее известного количества элементов, что уменьшает количество операций выделения памяти во время записи значений в мапу. Помните, что capacity для мап не гарантирует того, что присваивание элементов в мапу не будет выделять под них дополнительную память.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[string]os.FileInfo)

files, _ := ioutil.ReadDir("./files")
for _, f := range files {
    m[f.Name()] = f
}
```

</td><td>

```go

files, _ := ioutil.ReadDir("./files")

m := make(map[string]os.FileInfo, len(files))
for _, f := range files {
    m[f.Name()] = f
}
```

</td></tr>
<tr><td>

`m` создается без объявления capacity; возможно дополнительное выделение памяти во время присвоения значений.

</td><td>

`m` создается с объявлением capacity; возможно меньшее или отсутствие операций выделения памяти во время присвоения значений.

</td></tr>
</tbody></table>

## Style

### Be Consistent

Some of the guidelines outlined in this document can be evaluated objectively;
others are situational, contextual, or subjective.

Above all else, **be consistent**.

Consistent code is easier to maintain, is easier to rationalize, requires less
cognitive overhead, and is easier to migrate or update as new conventions emerge
or classes of bugs are fixed.

Conversely, having multiple disparate or conflicting styles within a single
codebase causes maintenance overhead, uncertainty, and cognitive dissonance,
all of which can directly contribute to lower velocity, painful code reviews,
and bugs.

When applying these guidelines to a codebase, it is recommended that changes
are made at a package (or larger) level: application at a sub-package level
violates the above concern by introducing multiple styles into the same code.

### Группируйте похожие объявления

Go поддерживает группировку похожих объявлений.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
import "a"
import "b"
```

</td><td>

```go
import (
  "a"
  "b"
)
```

</td></tr>
</tbody></table>

Это также применимо к константам, переменным и объявлениям типов.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go

const a = 1
const b = 2



var a = 1
var b = 2



type Area float64
type Volume float64
```

</td><td>

```go
const (
  a = 1
  b = 2
)

var (
  a = 1
  b = 2
)

type (
  Area float64
  Volume float64
)
```

</td></tr>
</tbody></table>

Группируйте только близкие по смыслу объявления. Не следует группировать все подряд.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
  ENV_VAR = "MY_ENV"
)
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

const ENV_VAR = "MY_ENV"
```

</td></tr>
</tbody></table>

Группы не ограничиваются местом, где могут быть использованы. Например, вы можете
использовать их внутри функций.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
func f() string {
  var red = color.New(0xff0000)
  var green = color.New(0x00ff00)
  var blue = color.New(0x0000ff)

  ...
}
```

</td><td>

```go
func f() string {
  var (
    red   = color.New(0xff0000)
    green = color.New(0x00ff00)
    blue  = color.New(0x0000ff)
  )

  ...
}
```

</td></tr>
</tbody></table>

### Порядок импорта пакетов

Импортируемые пакеты должны быть разделены на две группы:

- Стандартная библиотека
- Все остальные пакеты

Такой порядок сортировки применяется утилитой goimports по умолчанию.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"
  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td><td>

```go
import (
  "fmt"
  "os"

  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td></tr>
</tbody></table>

### Названия пакетов

При наименовании пакетов руководствуйтесь следующими принципами:

- Название должно состоять только из символов нижнего регистра. Использовать заглавные буквы и подчеркивания запрещено.
- Название при котором для большинства вызовов нет необходимости использовать именованный импорт.
- Коротко и ясно. Помните, что к имени пакета необходимо обращаться при каждом вызове.
- В единственном числе. Например, `net/url` вместо `net/urls`.
- Не "common", "util", "shared", или "lib". Это плохие и неинформативные названия.

Также смотрите [Package Names] и [Style guidline for Go packages].

  [Package Names]: https://blog.golang.org/package-names
  [Style guideline for Go packages]: https://rakyll.org/style-packages/

### Названия функций

Мы следуем соглашению сообщества Go о наименовании функций [MixedCaps for function
names]. Исключением являются функции тестов, которые могут содержать нижнее подчеркивание
с целью объединения родственных тест-кейсов, например
`TestMyFunction_WhatIsBeingTested`.

  [MixedCaps for function names]: https://golang.org/doc/effective_go.html#mixed-caps

### Псевдонимы импортов

Используйте псевдонимы импортов, в случае если название пакета не совпадает с 
последним элементом в пути импорта. 

```go
import (
  "net/http"

  client "example.com/client-go"
  trace "example.com/trace/v2"
)
```

Во всех остальных случаях использование псевдонимов необходимо избегать, исключением
является прямой конфликт в названиях импортов.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"


  nettrace "golang.net/x/trace"
)
```

</td><td>

```go
import (
  "fmt"
  "os"
  "runtime/trace"

  nettrace "golang.net/x/trace"
)
```

</td></tr>
</tbody></table>

### Группировка и упорядочивание функций

- Функции должны быть отсортированы в порядке приблизительного вызова.
- Функции в файле должны быть сгруппированы по получателю.

Таким образом, экспортируемые функции должны располагаться в файле первыми, сразу после
объявления `struct`, `const` и `var`.

Методы `newXYZ()`/`NewXYZ()` должны располагаться после определения типов, но до остальных
методов получателя.

Поскольку функции сгруппированы по получателю, утилитарные функции должны располагаться
в конце файла.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
func (s *something) Cost() {
  return calcCost(s.weights)
}

type something struct{ ... }

func calcCost(n []int) int {...}

func (s *something) Stop() {...}

func newSomething() *something {
    return &something{}
}
```

</td><td>

```go
type something struct{ ... }

func newSomething() *something {
    return &something{}
}

func (s *something) Cost() {
  return calcCost(s.weights)
}

func (s *something) Stop() {...}

func calcCost(n []int) int {...}
```

</td></tr>
</tbody></table>

### Уменьшение вложенности

Уменьшайте уровень вложенности кода, где это возможно. Старайтесь сперва
обрабатывать ошибки / специальные условия и возвращать результат или 
continue внутри циклов. Уменьшайте количество многоуровневого вложенного кода.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
for _, v := range data {
  if v.F1 == 1 {
    v = process(v)
    if err := v.Call(); err == nil {
      v.Send()
    } else {
      return err
    }
  } else {
    log.Printf("Invalid v: %v", v)
  }
}
```

</td><td>

```go
for _, v := range data {
  if v.F1 != 1 {
    log.Printf("Invalid v: %v", v)
    continue
  }

  v = process(v)
  if err := v.Call(); err != nil {
    return err
  }
  v.Send()
}
```

</td></tr>
</tbody></table>

### Излишние Else

Если переменной присваивается значение в обоих ветвях if/else, то это может быть заменено
единственным вызовом if.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
var a int
if b {
  a = 100
} else {
  a = 10
}
```

</td><td>

```go
a := 10
if b {
  a = 100
}
```

</td></tr>
</tbody></table>

### Объявление верхнеуровневых переменных

Для объявления верхнеуровневых переменных используйте `var`. Не указывайте тип,
за исключением тех случаев, когда выражение не совпадает с типом.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
var _s string = F()

func F() string { return "A" }
```

</td><td>

```go
var _s = F()
// Поскольку F возвращает строку, нам не нужно явно указывать
// тип еще раз.

func F() string { return "A" }
```

</td></tr>
</tbody></table>

Указывайте тип, если тип выражения не совпадает явно с желаемым типом.

```go
type myError struct{}

func (myError) Error() string { return "error" }

func F() myError { return myError{} }

var _e error = F()
// F возвращает объект типа myError, при этом мы хотим вернуть error.
```

### Используйте префикс _ для глобальных неэкспортируемых переменных

Используйте префикс `_` для верхнеуровневых переменных `var` и констант `const`, для
явного обозначения глобальных переменных.

Исключения: Неэкспортируемые значения ошибок, которые должны быть с префиксом `err`.

Объяснение: Верхнеуровневые переменные и константы находятся в области видимости всего пакета.
Использование общих имен может привести к случайному использованию не тех переменных в
разных файлах.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
// foo.go

const (
  defaultPort = 8080
  defaultUser = "user"
)

// bar.go

func Bar() {
  defaultPort := 9090
  ...
  fmt.Println("Default port", defaultPort)

  // Мы не увидим ошибку компиляции, если первая строка 
  // Bar() будет удалена.
}
```

</td><td>

```go
// foo.go

const (
  _defaultPort = 8080
  _defaultUser = "user"
)
```

</td></tr>
</tbody></table>

### Встраивание в структуры

Встраиваемые типы (такие, как мьютексы) следует определять в самом начале
списка структуры, также необходимо разделять встраиваемые поля от обычных переносом
строки.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
type Client struct {
  version int
  http.Client
}
```

</td><td>

```go
type Client struct {
  http.Client

  version int
}
```

</td></tr>
</tbody></table>

### Используйте названия полей при инициализации структур

Вам практически всегда потребуется использовать названия полей при инициализации
структур. Этого требует [`go vet`].

  [`go vet`]: https://golang.org/cmd/vet/

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
k := User{"John", "Doe", true}
```

</td><td>

```go
k := User{
    FirstName: "John",
    LastName: "Doe",
    Admin: true,
}
```

</td></tr>
</tbody></table>

Исключения: Названия полей могут быть опущены в тестовых таблицах, где 
присутствует менее 3-ех полей.

```go
tests := []struct{
  op Operation
  want string
}{
  {Add, "add"},
  {Subtract, "subtract"},
}
```

### Определение локальных переменных

Короткое определение переменных (`:=`) должно использоваться в случаях если переменная
определяется явным значением.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
var s = "foo"
```

</td><td>

```go
s := "foo"
```

</td></tr>
</tbody></table>

Тем не менее, существуют случаи, когда определение через `var` выглядит понятнее. [Declaring Empty Slices], например.

  [Declaring Empty Slices]: https://github.com/golang/go/wiki/CodeReviewComments#declaring-empty-slices

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
func f(list []int) {
  filtered := []int{}
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td><td>

```go
func f(list []int) {
  var filtered []int
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td></tr>
</tbody></table>

### nil это полноценный срез

`nil` является полноценный срезом длины 0. Это означает, что,

- Не следует возвращать срез длины 0 явным образом. Вместо этого необходимо возвращать `nil`.

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  if x == "" {
    return []int{}
  }
  ```

  </td><td>

  ```go
  if x == "" {
    return nil
  }
  ```

  </td></tr>
  </tbody></table>

- Для проверки, является ли срез пустым всегда используйте `len(s) == 0`. Не проверяйте его на `nil`.

  <table>
  <thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  func isEmpty(s []string) bool {
    return s == nil
  }
  ```

  </td><td>

  ```go
  func isEmpty(s []string) bool {
    return len(s) == 0
  }
  ```

  </td></tr>
  </tbody></table>

- Срез инициализированный через `var` сразу готов к использованию. (без `make()`).

  <table>
  <thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  nums := []int{}
  // or, nums := make([]int)

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td><td>

  ```go
  var nums []int

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td></tr>
  </tbody></table>

### Уменьшайте область видимости переменных

Где возможно, уменьшайте область видимости переменных, только если это не ведет к увеличению вложенности. Данное правило не должно конфликтовать с [Уменьшением вложенности](#reduce-nesting).

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
err := ioutil.WriteFile(name, data, 0644)
if err != nil {
 return err
}
```

</td><td>

```go
if err := ioutil.WriteFile(name, data, 0644); err != nil {
 return err
}
```

</td></tr>
</tbody></table>

Если вам необходим результат, вызовите функцию снаружи if, тогда в этом случае
нет необходимости пытаться уменьшать область видимости.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
if data, err := ioutil.ReadFile(name); err == nil {
  err = cfg.Decode(data)
  if err != nil {
    return err
  }

  fmt.Println(cfg)
  return nil
} else {
  return err
}
```

</td><td>

```go
data, err := ioutil.ReadFile(name)
if err != nil {
   return err
}

if err := cfg.Decode(data); err != nil {
  return err
}

fmt.Println(cfg)
return nil
```

</td></tr>
</tbody></table>

### Избегайте прямых аргументов

Прямые аргументы в функциях могут навредить читабельности. Добавляйте C-style комментарии
(`/* ... */`) для аргументов в тех случаях, когда их значения неочевидны.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true, true)
```

</td><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true /* isLocal */, true /* done */)
```

</td></tr>
</tbody></table>

Еще лучше, если заменить типы `bool` кастомными типами для повышения
читаемости и типо-безопасности. Также это позволит хранить и передавать
для заданного параметра больше, чем два состояния (true/false).

```go
type Region int

const (
  UnknownRegion Region = iota
  Local
)

type Status int

const (
  StatusReady = iota + 1
  StatusDone
  // Maybe we will have a StatusInProgress in the future.
)

func printInfo(name string, region Region, status Status)
```

### Use Raw String Literals to Avoid Escaping

Go supports [raw string literals](https://golang.org/ref/spec#raw_string_lit),
which can span multiple lines and include quotes. Use these to avoid
hand-escaped strings which are much harder to read.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
wantError := "unknown name:\"test\""
```

</td><td>

```go
wantError := `unknown error:"test"`
```

</td></tr>
</tbody></table>

### Инициализация ссылок на структуры

Используйте `&T{}` вместо `new(T)` при инициализации ссылок на структуры, так как
таким методом вы можете сразу инициализировать значения структуры.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
sval := T{Name: "foo"}

// inconsistent
sptr := new(T)
sptr.Name = "bar"
```

</td><td>

```go
sval := T{Name: "foo"}

sptr := &T{Name: "bar"}
```

</td></tr>
</tbody></table>

### Инициализация мап

Используйте `make(..)` для пустых мапов, и мапов заполняемыми
в рантайме. Это позволяет визуально отличить инициализацию мапы от 
ее объявления, также это позволяет в дальнейшем добавить размер мапы
при инициализации в случае необходимости.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
var (
  // m1 is safe to read and write;
  // m2 will panic on writes.
  m1 = map[T1]T2{}
  m2 map[T1]T2
)
```

</td><td>

```go
var (
  // m1 is safe to read and write;
  // m2 will panic on writes.
  m1 = make(map[T1]T2)
  m2 map[T1]T2
)
```

</td></tr>
<tr><td>

Объявление и инициализация внешне похожи.

</td><td>

Объявление и инициализация внешне различаются.

</td></tr>
</tbody></table>

Где возможно, указывайте capacity мапы при инициализации
через `make()`.  Смотрите [Prefer Specifying Map Capacity Hints](#prefer-specifying-map-capacity-hints)
для более подробной информации.

С другой стороны, если мапа содержит фиксированное количество элементов
используйте прямую инициализацию мапы.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[T1]T2, 3)
m[k1] = v1
m[k2] = v2
m[k3] = v3
```

</td><td>

```go
m := map[T1]T2{
  k1: v1,
  k2: v2,
  k3: v3,
}
```

</td></tr>
</tbody></table>

Проще говоря, используйте явное определение мапы если заранее известно
количество элементов и сами элементы, которые будут содержаться в мапе,
во всех остальных случаях используйте `make` (также старайтесь указывать capacity)


### Строки форматирования за Printf

Если вы определяете строки форматирования для `Printf`-style функций вне сигнатуры
функции, то обозначайте их как `const`.

Это поможет `go vet` проводить статический анализ строк форматирования.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
msg := "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td><td>

```go
const msg = "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td></tr>
</tbody></table>

### Naming Printf-style Functions

When you declare a `Printf`-style function, make sure that `go vet` can detect
it and check the format string.

This means that you should use predefined `Printf`-style function
names if possible. `go vet` will check these by default. See [Printf family]
for more information.

  [Printf family]: https://golang.org/cmd/vet/#hdr-Printf_family

If using the predefined names is not an option, end the name you choose with
f: `Wrapf`, not `Wrap`. `go vet` can be asked to check specific `Printf`-style
names but they must end with f.

```shell
$ go vet -printfuncs=wrapf,statusf
```

See also [go vet: Printf family check].

  [go vet: Printf family check]: https://kuzminva.wordpress.com/2017/11/07/go-vet-printf-family-check/

## Паттерны

### Test Tables

Use table-driven tests with [subtests] to avoid duplicating code when the core
test logic is repetitive.

  [subtests]: https://blog.golang.org/subtests

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestSplitHostPort(t *testing.T)

host, port, err := net.SplitHostPort("192.0.2.0:8000")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("192.0.2.0:http")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "http", port)

host, port, err = net.SplitHostPort(":8000")
require.NoError(t, err)
assert.Equal(t, "", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("1:8")
require.NoError(t, err)
assert.Equal(t, "1", host)
assert.Equal(t, "8", port)
```

</td><td>

```go
// func TestSplitHostPort(t *testing.T)

tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  {
    give:     "192.0.2.0:8000",
    wantHost: "192.0.2.0",
    wantPort: "8000",
  },
  {
    give:     "192.0.2.0:http",
    wantHost: "192.0.2.0",
    wantPort: "http",
  },
  {
    give:     ":8000",
    wantHost: "",
    wantPort: "8000",
  },
  {
    give:     "1:8",
    wantHost: "1",
    wantPort: "8",
  },
}

for _, tt := range tests {
  t.Run(tt.give, func(t *testing.T) {
    host, port, err := net.SplitHostPort(tt.give)
    require.NoError(t, err)
    assert.Equal(t, tt.wantHost, host)
    assert.Equal(t, tt.wantPort, port)
  })
}
```

</td></tr>
</tbody></table>

Test tables make it easier to add context to error messages, reduce duplicate
logic, and add new test cases.

We follow the convention that the slice of structs is referred to as `tests`
and each test case `tt`. Further, we encourage explicating the input and output
values for each test case with `give` and `want` prefixes.

```go
tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  // ...
}

for _, tt := range tests {
  // ...
}
```

### Параметры функций (Functional Options)

Параметры функций (Functional Options) это паттерн, в котором вы определяете интерфейсный 
тип `Option` который записывает информацию в какую-то внутреннюю структуру. Вы можете принимать
некоторое количество таких опций и работать со всей информацией записанной опциями во внутреннюю
структуру.

Используйте данный паттерн для необязательных аргументов в конструкторах или в других
методах публичных API которые будут потенциально расширяться, особенно если в этих методах
уже есть три или более аргументов.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
// package db

func Connect(
  addr string,
  timeout time.Duration,
  caching bool,
) (*Connection, error) {
  // ...
}

// Timeout and caching must always be provided,
// even if the user wants to use the default.

db.Connect(addr, db.DefaultTimeout, db.DefaultCaching)
db.Connect(addr, newTimeout, db.DefaultCaching)
db.Connect(addr, db.DefaultTimeout, false /* caching */)
db.Connect(addr, newTimeout, false /* caching */)
```

</td><td>

```go
type options struct {
  timeout time.Duration
  caching bool
}

// Option overrides behavior of Connect.
type Option interface {
  apply(*options)
}

type optionFunc func(*options)

func (f optionFunc) apply(o *options) {
  f(o)
}

func WithTimeout(t time.Duration) Option {
  return optionFunc(func(o *options) {
    o.timeout = t
  })
}

func WithCaching(cache bool) Option {
  return optionFunc(func(o *options) {
    o.caching = cache
  })
}

// Connect creates a connection.
func Connect(
  addr string,
  opts ...Option,
) (*Connection, error) {
  options := options{
    timeout: defaultTimeout,
    caching: defaultCaching,
  }

  for _, o := range opts {
    o.apply(&options)
  }

  // ...
}

// Options must be provided only if needed.

db.Connect(addr)
db.Connect(addr, db.WithTimeout(newTimeout))
db.Connect(addr, db.WithCaching(false))
db.Connect(
  addr,
  db.WithCaching(false),
  db.WithTimeout(newTimeout),
)
```

</td></tr>
</tbody></table>

Смотрите также,

- [Self-referential functions and the design of options]
- [Functional options for friendly APIs]

  [Self-referential functions and the design of options]: https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html
  [Functional options for friendly APIs]: https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis

<!-- TODO: replace this with parameter structs and functional options, when to
use one vs other -->
