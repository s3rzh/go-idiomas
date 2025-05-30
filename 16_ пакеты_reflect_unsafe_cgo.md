Иногда для решения задачи недостаточно стандартного Go-кода. Например, если тип данных невозможно определить на этапе компиляции, вы можете обеспечить взаимодействие с дан­ными и даже их конструирование с помощью средств поддержки рефлексии из пакета reflect . Если нужно воспользоваться преимуществами, которые дает вам схема размещения в памяти типов языка Go, можете задействовать пакет unsafe. А если некоторая функциональность может быть обеспечена только с помощью библиотек, написанных на языке C, можно вызывать C-код с по­мощью пакета cgo.

Рефлексия позволяет работать с типами на этапе выполнения. Те она позволяет исследовать типы на этапе выполнения, а также изучать, модифицировать и создавать переменные, функции и структуры на этапе выполнения.
Области применения рефлексии можно разделить на следующие основные категории:
- Чтение из базы данных и запись в нее. Пакет database/sql использует ре­флексию для того, чтобы отправлять записи в базы данных и, наоборот, читать записи из баз данных.
- Встроенные библиотеки шаблонизации языка Go text/template и html/template используют рефлексию для обработки значений, передаваемых шаблонам.
- Пакет fmt активно задействует рефлексию для определения типа предостав­ляемых параметров при вызове функции fmt.Println и других подобных функций.
- Пакет errors использует рефлексию для реализации функций errors.Is и errors.As.
- Пакет sort применяет рефлексию для реализации функций сортировки и проверки содержимого срезов любого типа: sort.Slice, sort.SliceStable и sort.SliceIsSorted.
- Последняя основная область применения рефлексии в стандартной библио­теке — преобразование данных в форматы JSON, XML и другие, поддержи­ваемые различными пакетами encoding, и обратно. Рефлексия используется
для доступа к тегам структур, а также для чтения и записи соответствующих полей структур.

Общей особенностью этих примеров является применение рефлексии для чтения или форматирования данных, которые импортируются в Go-программу или экс­портируются из нее. Рефлексия часто используется на границе между программой и внешним миром. У рефлексии есть своя цена -  она значительно замедляет выпол­нение операций. Еще одной областью применения пакета reflect из стандартной библиотеки Go является тестирование, па­кет reflect содержит функцию DeepEqual. Она реализована в пакете reflect по той причине, что для выполнения своей работы использует рефлексию. Функция reflect.DeepEqual сравнивает два значения более тщательно, чем
оператор ==, и такой способ сравнения задействуется в стандартной библио­теке для проверки результатов тестирования, а также сравнения чего-то, что нельзя сравнить с помощью оператора ==, например срезов или отображений или функций. В большинстве случаев можно обойтись без DeepEqual, потому что в Go 1.21 для сравнения срезов и отображений были добавлены более быстрые методы slices.Equal и maps.Equal.

Типы и функции, реализующие рефлексию в Go, опре­делены в пакете **reflect** стандартной библиотеки. Механизм действия рефлексии опирается на следующие три концепции: типы, разновидности типов и значения.
В контексте рефлексии тип является именно тем, что означает это слово. То есть он определяет, какими свойствами обладает переменная, какие значения она может содержать и как с ней можно взаимодействовать. При использовании рефлексии вы можете запросить у типа информацию об этих свойствах с по­мощью кода.
Получить рефлексивное представление типа переменной можно с помощью функции TypeOf из пакета reflect:
``` go
vType := reflect.TypeOf(v)
```
Функция reflect.TypeOf возвращает значение типа reflect.Type, представля­ющее тип переданной в эту функцию переменной. Тип reflect.Type обладает рядом методов, возвращающих информацию о типе переменной.
Вот некоторые методы:
``` go
var x int
xt := reflect.TypeOf(x)
fmt.Println(xt.Name()) // возвращает int

f := Foo{}
ft := reflect.TypeOf(f)
fmt.Println(ft.Name()) // возвращает Foo

xpt := reflect.TypeOf(&x)
fmt.Println(xpt.Name()) // возвращает пустую строку
```
Здесь мы объявляем переменную x типа int . Передаем ее в функцию reflect.TypeOf и получаем обратно экземпляр типа reflect.Type. Для простых типов, таких как int, метод Name() возвращает имя типа — в данном случае строку int для типа int. Для структур этот метод возвращает имя структуры. Некоторые типы, например срезы и указатели, не имеют имени типа, в таком случае метод Name возвращает пустую строку.
Метод Kind типа reflect.Type возвращает значение типа reflect.Kind — констан­ту, которая указывает, на основе чего создан тип: среза, отображения, указателя, структуры, интерфейса, строки, массива, функции, типа int или какого-то дру­гого простого типа. 

В чем разница между типом и разновидностью типа? Запомните следующее правило: если вы определяете струк­туру с именем Foo, то она обладает разновидностью reflect.Struct и типом Foo.

 При использовании рефлексии следует помнить о том, что любой код из пакета reflect исходит из предположе­ния, что вы знаете, что делаете. Некоторые из методов типа reflect.Type и других
типов пакета reflect имеют смысл только для определенных разновидностей типов. Так, например, у типа reflect.Type есть метод NumIn. Если экземпляр типа reflect.Type представляет функцию, то NumIn вернет количество ее входных параметров. Если экземпляр типа reflect.Type представляет что-то другое, то вызов метода NumIn сгенерирует панику.

Еще одним важным методом типа reflect.Type является метод Elem. Некоторые типы в Go содержат ссылки на другие типы, и метод Elem позволяет выяснить, что представляют собой эти вложенные типы. Например, вызовем функцию reflect.TypeOf, передав ей указатель на тип int:
``` go
var x int
xpt := reflect.TypeOf(&x)
fmt.Println(xpt.Name()) // возвращает пустую строку
fmt.Println(xpt.Kind()) // возвращает reflect.Ptr
fmt.Println(xpt.Elem().Name()) // возвращает int
fmt.Println(xpt.Elem().Kind()) // возвращает reflect.Int
```
В результате мы получим экземпляр типа reflect.Type с пустой строкой вме­сто имени и разновидностью reflect.Ptr, что подразумевает указатель. Когда экземпляр типа reflect.Type представляет указатель, метод Elem возвращает экземпляр типа reflect.Type, представляющий тот тип, на который указывает этот указатель. В данном случае метод Name возвращает строку int, а метод Kind — разновидность reflect.Int. Метод Elem можно использовать также для срезов, отображений, каналов и массивов.

У типа reflect.Type тоже есть методы для анализа структур. Метод NumField позволяет узнать, сколько полей содержит структура, а метод Field — извлечь поле структуры по индексу. Второй метод возвращает структуру каждого поля, как ее определяет тип reflect.StructField, что включает в себя имя, порядок, тип и имеющиеся в поле теги структур:
``` go
type Foo struct {
  A int `myTag:"value"`
  B string `myTag:"value2"`
}

var f Foo
ft := reflect.TypeOf(f)
for i := 0; i < ft.NumField(); i++ {
  curField := ft.Field(i)
  fmt.Println(curField.Name, curField.Type.Name(), curField.Tag.Get("myTag"))
}

// output:
// A int value
// B string value2
```
Мы создаем экземпляр типа Foo и с помощью функции reflect.TypeOf получаем экземпляр типа reflect.Type, представляющий переменную f. Затем с по­мощью метода NumField настраиваем цикл for так, чтобы он обошел индексы всех полей в переменной f. Далее с помощью метода Field получаем структуру reflect.StructField, представляющую отдельное поле. После этого мы можем использовать поля структуры reflect.StructField для получения дополнитель­ ной информации о поле.

Рефлексию можно использовать не только для выяснения типов, но и для чте­ния значений переменных, присваивания им значений и создания с нуля новых значений. С помощью функции reflect.ValueOf можно создать экземпляр типа reflect.Value, представляющий значение переменной:
``` go
vValue := reflect.ValueOf(v)
```
Поскольку каждая переменная в Go обладает типом, reflect.Value имеет метод Type, который возвращает экземпляр типа reflect.Type для экземпляра типа reflect.Value. Как и у типа reflect.Type, у типа reflect.Value есть метод Kind. Подобно тому как у типа reflect.Type есть методы для получения информации о типе переменной, у типа reflect.Value есть методы для получения информации о значении переменной.
Для начала посмотрим, как читать значения из экземпляров типа reflect.Value. Метод Interface возвращает значение переменной как интерфейс any. При этом теряется информация о типе, поэтому при записи возвращаемого значения в пере­менную его нужно снова привести к правильному типу с помощью операции утверждения типа:
``` go
s := []string{"a", "b", "c"}
sv := reflect.ValueOf(s) // переменная sv имеет тип reflect.Value
s2 := sv.Interface().([]string) // переменная s2 имеет тип []string
```
Метод Interface можно вызывать для экземпляров типа reflect.Value, содержа­щих значения любого типа, однако имеются и специализированные методы для случаев, когда переменная относится к одному из встроенных простых типов: Bool, Complex, Int, Uint, Float и String. Есть также метод Bytes для случая, когда переменная представляет собой байтовый срез. Вызов метода, который не со­ответствует типу значения, содержащегося в экземпляре типа reflect.Value, сгенерирует панику.

Рефлексию можно использовать также для присвоения значения переменной, однако эта операция выполняется в три этапа. Сначала нужно передать указа­тель на переменную в функцию reflect.ValueOf, которая вернет экземпляр типа reflect.Value, представляющий этот указатель:
``` go
i := 10
iv := reflect.ValueOf(&i)
```
Затем необходимо добраться непосредственно до значения, которое нужно поме­нять. Вызвав метод Elem в экземпляре типа reflect.Value, мы можем получить то значение, на которое указывает указатель, переданный в функцию reflect.ValueOf . Подобно тому как метод Elem типа reflect.Type возвращает тип, на который указы­вает вмещающий тип, метод Elem типа reflect.Value возвращает значение, на ко­торое указывает указатель, или значение, содержащееся в экземпляре интерфейса:
``` go
ivv := iv.Elem()
```
Теперь осталось непосредственно применить метод, используемый для установки значения. Наряду со специализированными методами для чтения простых типов в пакете reflect имеются и специализированные методы для установки значений простых типов: SetBool , SetInt, SetFloat, SetString и SetUint. В своем примере мы можем изменить значение переменной i, выполнив вызов ivv.SetInt(20).
Если выведем значение переменной i, то увидим, что теперь оно равно 20:
``` go
ivv.SetInt(20)
fmt.Println(i) // выводит 20
```
В случае всех остальных типов следует использовать метод Set, который прини­мает переменную типа reflect.Value. При этом присваиваемое значение может не быть указателем, потому что мы просто читаем это значение, не изменяя его. И подобно тому, как метод Interface(), помимо прочего, может применяться для чтения простых типов, метод Set может использоваться для записи простых типов.

Необходимость передачи указателя в функцию reflect.ValueOf для изменения значения входного параметра объясняется тем, что эта функция ведет себя так же, как любая другая функция в языке Go. Использование параметров указательного типа означает, что функция должна модифицировать значение параметра. Модификация производится путем разыменования указателя и при­ сваивания значения. Например, следующие две функции производят одно и то же действие:
``` go
func changeInt(i *int) {
 *i = 20
}
func changeIntReflect(i *int) {
 iv := reflect.ValueOf(i)
 iv.Elem().SetInt(20)
}
```
Попытка присвоить экземпляру reflect.Value значение неправильного типа приведет к панике.

Для создания переменной  служит функция reflect.New — рефлексивный аналог функции new. Она принимает экземпляр типа reflect.Type и возвращает экземпляр типа reflect.Value, представляющий
указатель на экземпляр типа reflect.Value, соответствующий указанному типу. Поскольку это указатель, можно модифицировать его, а затем присвоить моди­фицированное значение переменной с помощью метода Interface.
Подобно тому как метод reflect.New можно задействовать для создания указателя на скалярный тип, с помощью рефлексии можно создавать те же объекты, которые создает ключевое слово make, используя следующие функции:
``` go
func MakeChan(typ Type, buffer int) Value
func MakeMap(typ Type) Value
func MakeMapWithSize(typ Type, n int) Value
func MakeSlice(typ Type, len, cap int) Value
```
Эти функции принимают экземпляр типа reflect.Type, который вместо вложен­ного типа представляет составной тип.
Конструирование экземпляра типа reflect.Type всегда следует начинать со значе­ния. Однако если у вас нет значения, то для создания переменной, представляющей экземпляр типа reflect.Type, можно использовать следующий хитрый прием:
``` go
var stringType = reflect.TypeOf((*string)(nil)).Elem()
var stringSliceType = reflect.TypeOf([]string(nil))
```
Переменная stringType содержит экземпляр типа reflect.Type, представляющий тип string, а переменная stringSliceType — экземпляр типа reflect.Type, пред­ставляющий тип []string. 
В первой строке мы преобразуем здесь значение nil в указатель на тип string и за­действуем функцию reflect.TypeOf для создания экземпляра типа reflect.Type, представляющего этот указательный тип, после чего вызываем метод Elem этого экземпляра, чтобы получить базовый тип. В силу используемого в Go порядка выполнения операций мы заключили *string в скобки, чтобы компилятор не по­считал, что мы хотим преобразовать значение nil в значение типа string — эта операция недопустима.
В случае с переменной stringSliceType дело обстоит чуть проще, потому что срез может иметь значение nil. Нам остается лишь привести значение nil к типу []string и передать его в функцию reflect.TypeOf.
Теперь, располагая этими типами, можно вызвать методы reflect.New и re­flect.MakeSlice, как показано далее:
``` go
ssv := reflect.MakeSlice(stringSliceType, 0, 10)
sv := reflect.New(stringType).Elem()
sv.SetString("hello")
ssv = reflect.Append(ssv, sv)
ss := ssv.Interface().([]string)
fmt.Println(ss) // выводит [hello]
```

Используйте рефлексию для проверки значения интерфейса на равенство nil.

Если вам нужно проверить, равно ли значению nil ассоциированное с интерфейсом значение, то это можно сделать с помощью рефлексии и методов IsValid и IsNil:
``` go
func hasNoValue(i interface{}) bool {
 iv := reflect.ValueOf(i)
 if !iv.IsValid() {
  return true
 }

 switch iv.Kind() {
  case reflect.Pointer, reflect.Slice, reflect.Map, reflect.Func,reflect.Interface:
   return iv.IsNil()
  default:
   return false
 }
}
```
Метод IsValid возвращает true , если экземпляр типа reflect.Value содержит любое другое значение, кроме интерфейса, равного nil. Это нужно проверять в первую очередь, потому что, когда метод IsValid возвращает false, вызов любого другого метода типа reflect.Value, что неудивительно, генерирует панику. Метод IsNil возвращает true, если экземпляр типа reflect.Value содержит значение nil, но его можно использовать лишь в том случае, когда разновидность типа (reflect.Kind) допускает равенство значению nil. Если вызвать этот метод для типа, нулевое значение которого отличается от nil, то это, как вы уже догадались, приведет к панике.

Используйте рефлексию для создания маршалера данных.

Создавайте с помощью рефлексии функции для автоматизации повторяющихся задач. Например, вот как может выглядеть фабричная функция, снабжающая любую переданную ей функцию информацией о времени выполнения:
``` go
func MakeTimedFunction(f any) any {
 ft := reflect.TypeOf(f)
 fv := reflect.ValueOf(f)
 wrapperF := reflect.MakeFunc(ft, func(in []reflect.Value) []reflect.Value {
 start := time.Now()
 out := fv.Call(in)
 end := time.Now()
 fmt.Println(end.Sub(start))
  return out
 })
 return wrapperF.Interface()
}
```
Поскольку эта функция должна принимать на входе любую функцию, ее параметр объявлен с типом any. Она передает экземпляр типа reflect.Type, представля­ющий полученную функцию, в вызов reflect.MakeFunc вместе с замыканием, которое фиксирует начальный момент времени, вызывает исходную функцию с помощью рефлексии, фиксирует конечный момент времени, выводит разницу между началом и концом и возвращает значение, вычисленное исходной функ­цией. Поскольку reflect.MakeFunc возвращает экземпляр типа reflect.Value, мы вызываем метод Interface этого типа, чтобы получить возвращаемое значение. Использовать эту функцию можно следующим образом:
``` go
func timeMe(a int) int {
 time.Sleep(time.Duration(a) * time.Second)
 result := a * 2
 return result
}

func main() {
 timed:= MakeTimedFunction(timeMe).(func(int) int)
 fmt.Println(timed(2))
}
```

Рефлексию можно использовать для создания структур, но лучше этого не делать.  Функ­ция reflect.StructOf принимает срез полей типа reflect.StructField и воз­вращает экземпляр типа reflect.Type, представляющий новую структуру. Такие структуры можно присваивать только переменным типа any, а их поля можно читать и изменять только с использованием рефлексии. Эта функция представляет интерес фактически только с научной точки зрения.

Рефлексия не позволяет создавать методы. Те при помощи *рефлексии* можно создавать структуры, но нельзя снабдить их дополнительными методами. Это означает, что вы не сможете с помощью рефлексии создать новый тип, реализующий некоторый интерфейс.

Как было сказано раньше - рефлексия может играть важную роль при преобразовании данных на внешней границе Go-кода.

Пакет **unsafe** позволяет манипулировать памятью. Так же пакет unsafe применяется для преоб­разования двоичных данных. Большинство случаев использования unsafe обусловлено необходимостью интеграции с операционными системами и кодом на C. Разработчики часто задействуют unsafe, чтобы повысить эффективность кода на Go. Главное предназначение unsafe — организация взаимодействий между система­ми. Стандартная библиотека Go использует unsafe для обмена данными с опе­рационной системой.

Тип unsafe.Pointer является особенным в том плане, что он существует лишь для того, чтобы преобразовывать указатели любых типов в тип unsafe.Pointer и об­ратно.  Помимо указателей, в тип unsafe.Pointer или из него можно преобразовы­вать также значения специального целочисленного типа uintptr. Над значениями этого типа можно производить математические действия, как в случае любого другого целочисленного типа. Это позволяет заходить в экземпляр определенного типа и извлекать отдельные байты. Можно также использовать адресную арифме­тику. Эти байтовые манипуляции ведут к изменению значения переменной.

Существует два основных паттерна применения unsafe. Первый паттерн — пре­образование друг в друга значений двух разных типов, которые не поддерживают такое преобразование. Это обеспечивается путем использования цепочки пре­образований типа с unsafe.Pointer посередине. Второй паттерн — чтение или запись байтов переменной путем ее преобразования в unsafe.Pointer, затем преобразования unsafe.Pointer в указатель и копирования или модификации байтов содержимого переменной. Оба паттерна требуют знать размер (и, воз­можно, местоположение) обрабатываемых данных. Эту информацию можно получить с помощью функций Sizeof и Offsetof , которые тоже определены в пакете unsafe.

Некоторые функции в unsafe помогают узнать, как байты, составляющие значе­ния разных типов, располагаются в памяти. Функция **Sizeof** как следует из названия, возвращает размер в байтах всего, что ей передается. 
В то время как размеры для числовых типов довольно очевидны ( int16 — это 16 бит, или 2 байта, byte — это 1 байт и т. д.), ситуация с другими типами немного сложнее. Для указателя вы получите объем памяти, необходимый для хранения указателя (обычно 8 байт в 64-битной системе), а не размер данных, на которые он указывает. Вот почему Sizeof считает, что любой срез имеет длину 24 байта в 64-битной системе: он реализован как два поля int, хранящие длину и емкость, и указатель на содержимое среза. Любая строка имеет длину 16 байт в 64-битной системе (поле int для хранения длины и указатель на содержимое строки). Любое
отображение в 64-битной системе имеет длину 8 байт, потому что в Go любое отображение — это указатель на довольно сложную структуру данных.

Массивы являются типами значений, поэтому их размер вычисляется умноже­нием длины массива на размер одного элемента.

Размер структуры — это сумма размеров полей плюс некоторые корректировки для учета выравнивания. Процессоры быстрее обрабатывают данные, которые за­нимают в памяти целое число машинных слов и начинаются и заканчиваются на границах этих слов, и медленнее, когда обрабатываемое значение начинается в се­редине одного машинного слова и заканчивается в середине другого. Поэтому для достижения наибольшей эффективности компилятор добавляет отступы между полями, чтобы они выстраивались по границам машинных слов. Компилятор также стремится правильно выровнять всю структуру. В 64-битной системе он добавит отступы в конец структуры, чтобы довести ее размер до кратного 8 байтам.

Функция, **Offsetof** , сообщает позицию поля в структуре.

Иногда простым переупорядочением полей в часто используемых структурах для минимизации объема отступов, необходи­ мых для выравнивания, можно добиться значительной экономии памяти (особенно манипулиру­ющих большими объемами данных).

Как упоминалось ранее, одна из главных причин применения пакета unsafe — производительность, особенно при чтении данных из сети. Если вам нужно обеспечить сопоставление данных, передаваемых в структуру данных языка Go или из нее, то это можно очень быстро сделать с помощью типа unsafe.Pointer.

При пересылке данных по сети обычно используется прямой порядок сле­дования байтов (когда первыми следуют старшие байты), или, как его еще называют, сетевой порядок байтов. Поскольку в большинстве современных
процессоров применяется обратный (little-endian) порядок следования бай­тов (или оба, но в ходе работы в режиме с обратным порядком), следует быть крайне осторожными при чтении данных из сети и их записи в сеть.

``` go
func DataFromBytesUnsafe(b [dataSize]byte) Data {
 data := *(*Data)(unsafe.Pointer(&b))
 if isLE {
  data.Value = bits.ReverseBytes32(data.Value)
 }
 return data
}
```
Сначала мы принима­ем указатель на байтовый массив и преобразуем его в небезопасный указатель (unsafe.Pointer). Затем преобразуем небезопасный указатель в указатель *Data (его нужно заключить в круглые скобки, чтобы получить правильный порядок выполнения операций). Поскольку функция должна вернуть структуру, а не указатель на нее, мы разыменовываем указатель. Далее проверяем флаг — при­знак работы на платформе с обратным порядком следования байтов. Если флаг установлен, то инвертируем порядок байтов в поле Value. Наконец, возвращаем значение.
Массивы, как и структуры, — это **типы значений**, для которых память выделяется напря­мую (для ссылочных типов, аля-слайс, мапа, канал и др - нужно явно выделять память, например через make). 

А как устанавливается признак работы на платформе с обратным порядком сле­дования байтов? Это делает следующий код:
``` go
var isLE bool
func init() {
 var x uint16 = 0xFF00
 xb := *(*[2]byte)(unsafe.Pointer(&x))
 isLE = (xb[0] == 0x00)
}
```
На платформе с обратным (*little-endian*) порядком следования байтов перемен­ная x будет представлена в памяти как [00 FF], а с прямым (*big-endian*) поряд­ком — как [FF 00]. Мы используем unsafe.Pointer, чтобы преобразовать число в массив байтов, а затем проверяем первый байт, чтобы определить значение переменной isLE.

Использование unsafe во время работы с массивами дает ускорение примерно в 2,0–2,5 раза по сравнению со стандартным подходом. Если в вашей программе производится много подобных преобразований, то применение описанных низко­уровневых приемов будет вполне оправданным. Но в подавляющем большинстве программ лучше обойтись без небезопасного кода.

Вы можете объ­единить рефлексию и unsafe для чтения и изменения неэкспортируемых полей в структурах. Сначала определим структуру в одном пакете:
``` go
type HasUnexportedField struct {
  A int
  b bool  // Обычный код за пределами пакета не сможет получить доступ к полю b
  C string
}

func SetBUnsafe(huf *one_package.HasUnexportedField) {
  sf, _ := reflect.TypeOf(huf).Elem().FieldByName("b")
  offset := sf.Offset
  start := unsafe.Pointer(huf)
  pos := unsafe.Add(start, offset) // тут unsafe.Pointer возвращается
  b := (*bool)(pos)
  fmt.Println(*b) // читает значение
  *b = true // записывает значение
}
```
Здесь с помощью механизма рефлексии извлекается информация о типе поля **b**. Метод FieldByName возвращает экземпляр reflect.StructField для любого поля в структуре, даже неэкспортированного. Этот экземпляр содержит также смеще­ние связанного с ним поля. Далее huf преобразуется в unsafe.Pointer и исполь­зуется метод unsafe.Add для добавления смещения к указателю, чтобы перейти к местоположению **b** в структуре. После этого остается только привести указатель unsafe.Pointer, полученный вызовом Add, к типу *bool . Теперь можно прочитать или изменить значение b.

Go предоставляет флаг компилятора, позволяющий выявлять случаи неправиль­ного применения типов Pointer и unsafe.Pointer. Запустив свой код с флагом **-gcflags=-d=checkptr** , вы обеспечите проведение такой дополнительной проверки на этапе выполнения. Как и детектор состояний гонки, эта проверка не гаран­тирует выявление абсолютно всех проблем с небезопасным кодом и замедляет работу программы. В то же время использование этого флага вполне уместно на этапе тестирования кода.

Пакет **unsafe** — мощное и низкоуровневое средство! Используйте его, только когда точно знаете, что делаете, и вам действительно нужен обеспечиваемый им прирост производительности.

Пакет *cgo* обеспечивает интеграцию, а не повышает производительность. Пакет *cgo* чаще всего используется на внешней границе Go-программы. Пакет *cgo* предназначен главным образом для интеграции с C-библиотеками.Учитывая то, что cgo не позволяет повысить производительность и его слож­но использовать в нетривиальных программах, этот пакет стоит применять лишь в случаях, когда требуется задействовать C-библиотеку, которую нель­зя заменить кодом, написанным на Go. Вместо того чтобы применять пакет cgo самостоятельно, стоит поискать сторонний модуль, предоставляющий нужную обертку.
