# Golang

- [Golang](#golang)
  - [General program structure](#general-program-structure)
    - [Sample program layout for a cmdline program](#sample-program-layout-for-a-cmdline-program)
  - [Syntax](#syntax)
    - [Variables/Constants/Literals](#variablesconstantsliterals)
    - [Functions](#functions)
      - [Defer](#defer)
    - [Data types](#data-types)
      - [Big numbers](#big-numbers)
    - [Pointers](#pointers)
    - [Operators](#operators)
    - [Arrays, slices](#arrays-slices)
      - [Slice behind-the-scenes concepts](#slice-behind-the-scenes-concepts)
    - [Maps](#maps)
    - [Structs](#structs)
      - [Composition](#composition)
      - [Methods](#methods)
      - [Interfaces](#interfaces)
    - [Type casting/assertion/switch](#type-castingassertionswitch)
    - [If/then/else, switch/case](#ifthenelse-switchcase)
    - [Loops](#loops)
      - [For](#for)
    - [Builtin functions](#builtin-functions)
    - [Errors](#errors)
  - [Concurrency](#concurrency)
    - [Goroutines](#goroutines)
    - [Channels](#channels)
    - [Non-blocking reads (select) + Interruptible threads (Context)](#non-blocking-reads-select--interruptible-threads-context)
    - [Base template of producers/consumers with queue processing](#base-template-of-producersconsumers-with-queue-processing)
  - [APIs](#apis)
    - [Strings](#strings)
      - [General concepts](#general-concepts)
      - [Formatting](#formatting)
      - [Parsing/Numerical/Character conversion](#parsingnumericalcharacter-conversion)
      - [Utils](#utils)
    - [Reflection](#reflection)
      - [DeepEqual](#deepequal)
    - [Regular expressions](#regular-expressions)
    - [Time](#time)
    - [Math](#math)
    - [Random](#random)
    - [Streams read/write, Buffer](#streams-readwrite-buffer)
    - [Files/Dirs/descriptors](#filesdirsdescriptors)
      - [File informations](#file-informations)
      - [Reading from stdin](#reading-from-stdin)
    - [JSON](#json)
    - [CSV](#csv)
    - [Gzip](#gzip)
    - [O/S, Environment variables, Runtime reflection](#os-environment-variables-runtime-reflection)
    - [Command line arguments/Options parsing](#command-line-argumentsoptions-parsing)
    - [Executing shell commands](#executing-shell-commands)
      - [Context](#context)
    - [Processes; Signals sending/handling](#processes-signals-sendinghandling)
    - [Logging](#logging)
    - [Atomic](#atomic)
    - [Sleep](#sleep)
    - [Memory allocation](#memory-allocation)
    - [Unsafe](#unsafe)
    - [CGo](#cgo)
  - [Testing](#testing)
    - [Strategies/tools](#strategiestools)
    - [Benchmarking](#benchmarking)
      - [Useful tools](#useful-tools)
  - [Management](#management)
    - [Packages/naming](#packagesnaming)
      - [init() function](#init-function)
    - [Tools](#tools)
    - [Modules](#modules)
    - [Build tags](#build-tags)
    - [Go environment variables, anc cross compiling](#go-environment-variables-anc-cross-compiling)
    - [Shared libraries (invoke from other languages)](#shared-libraries-invoke-from-other-languages)

## General program structure

```golang
package main

// Can also be one import per line, without round brackets.
import (
	"fmt"
)

func main() {
	fmt.Println("Hello world!")
}
```

then execute `go run main.go` (compile and run).

### Sample program layout for a cmdline program

```
todo
├── cmd
│   └── todo
│       ├── main.go       # cmdline (makes binary name `todo`)
│       └── main_test.go
├── go.mod
├── todo.go               # library
└── todo_test.go
```

## Syntax

### Variables/Constants/Literals

```golang
var v <type>
var v [<type>] = <value>

const c [<type>] = 1

a, b := 3, 4              // Multiple assignment; useful also for swapping.

largeNum := 1_000_000_000 // Cosmetic underscores
```

Constants, with `iota` and (optional) custom type:

```golang
type Position int

const (
  First Position = iota // 0, automatic
  Second                // 1
)
```

### Functions

General definition:

```golang
// Example signature, with multiple return values.
// If returning only one variable, the round brackets are not required.
//
func myFunc(myVar, myVar2 int) (bool, int) {
  return true, 1
}

// Variadic arguments (varargs)
// The `...` operator is called "Pack operator".
//
// In order to pass a slice, unpack it:
//
//   myFunc(mySlice...)
//
func myFunc(myVars ...string) { }

// "Named returns": variables declared in the function definition!
//
func myFunc() (myRetVal int) {
  // If no variables are specified ("Naked return"), the named returns are returned. Watch out
  // because this can be confusing.
  //
  return myRetVal
}
```

```golang
// Function variable
//
myFunc := func(phrase string) bool { return phrase == "" }

// Anonymous function
//
result := func(phrase string) bool { return phrase == "" }("abc")
```

Type function:

```golang
type MyFuncType func(int) bool

func AcceptsFunctions(myFunc MyFuncType, myVal int) {
	funcIsTrue := myFunc(myVal)
	fmt.Println("Func is true?:", funcIsTrue)
}

func main() {
	AcceptsFunctions(func(i int) bool { return i%2 == 0 }, 1) // false
	AcceptsFunctions(func(i int) bool { return i*2 == 4 }, 2) // true
}
```

```golang

// Closure! Has access to variables in scope. If the closure is invoked (from the caller), the `i`
// variable is updated!
//
func incrementI() func() int {
	var i int

	return func() int {
		i++
		return i
	}
}

incrementFx := incrementI()

fmt.Println(incrementFx()) // 1
fmt.Println(incrementFx()) // 2
```

#### Defer

Execution is in LIFO order, so the first deferred is invoked after the second defined.

```golang
func myFunc() {
	i := 0

  // The parameter is copied at `defer` time! So the `i` value is before the increment.
  //
	defer func(i2 int) { fmt.Println("Invoked after; value:", i2) }(i) // 0 !!

  // In case of closure, the variable values are the ones at execution time.
  //
	defer func() { fmt.Println("Invoked before; value:", i) }() // 1 !!

	i++

	fmt.Println("zero!")
}
```

### Data types

Data types:

- `float(32|64)`
- `[u]int(8|16|32|64)`
- `[u]int`: depends on the architecture
- `byte`: alias of `uint8`
- `rune`: alias of `int32`

Casting:

```golang
toType(fromType)
```

Typedef:

```golang
type Kind int
```

#### Big numbers

```golang
bigA := big.NewInt(123)
bigA.Add(bigA, big.NewInt(1))
```

### Pointers

Passing by value:

- can be less efficient;
- uses the stack, which is simple to manage.

Passign by reference:

- use the heap, which is managed by the garbage collector.

Basic usage:

```golang
// Alternative instantiations
var ptr *int
ptr := new(int)

val := 123
ptr = &val
*ptr = 456

fmt.Printf("%v\n", val)  // 456
fmt.Printf("%v\n", *ptr) // 456
```

### Operators

```golang
var++
var--
```

### Arrays, slices

Arrays can be compared (via `==`), if they have the same type and length. Slices can't be compared (via `==`).

Arrays:

```golang
var arr [10]int                // instantiate with a size and default values
arr := [...]string{"a", "b"}   // literal form
```

Slices:

```golang
var sl []int                               // instantiate without size
sl := make([]string, <size>[, <capacity>]) // with a size
bdsl := make([][]int, 10)                  // bidimensional slice; must initialize each entry separately
sl := []string{"a", "b"}                   // literal form
sl := array[[<start>]:[<end>]]             // make from an array; **end element is not included**
                                           // omitting start/end indexes indicate first/last indexes

sl = append(sl, 64)                        // append to a slice (can't append to/from an array)
sl2 := append(sl, sl...)                   // append a slice to another slice (variadic notation)

copy(dest[z:a], source[x:y])               // copy slices (indexing is optional); !! the destination capacity is equal to the source length !!
cap(sl)                                    // capacity of a slice

bytes.Equal(a, b []byte)                   // compares two byte arrays
```

Accessing:

- Golang doesn't have negative indexing! Use `len() - 1`.

#### Slice behind-the-scenes concepts

Slices are **views on arrays**!! If the underlying array data changes, the slice contents will change as well.

One needs to be aware of **which/when slice operations cause an underlying array to be copied**, since different slices will then point to different arrays!

**Underlying arrays are copied when the capacity is not enough**

### Maps

```golang
// Map literal

var OrbitalPeriods = map[Planet]float64{
  "Mercury": 0.2408467,
}


value, found := myMap["key"]              // Check if map contains an element
delete(myMap, "key")                      // Deletion; returns nothing!
```

### Structs

```golang
type mytype struct {
  field1  []int
  field2  byte
}

// Instantiation

// 1: fields are optional.
mt := mytype{
  field1: []int{1, 2, 3},
}

// 2: all fields must be specified.
mt := mytype{
  []int{1, 2, 3},
  1,
}

// 3: all fields are initialized with zero value
var mt mytype

// Anonymous structs

as := struct {
  field1 []int
  field2 byte
}{}
as.field1 = []int{1, 2, 3}
as.field2 = 3
```

Structs can be compared, as long as the types are the same, and they're comparable (so, no slices!).

#### Composition

```golang
type myints struct {
	i  int
	i2 int
}

type mycomposed struct {
	myints          // "Embedded" field

	i2 int
	f  float64
}

mt := mycomposed{
  myints: myints{ // member/struct names must be the same!
    i:  1,
    i2: 2,
  },
  i2: 3,
  f:  4.4,
}

mt.i = 10         // "promoted" -> can be accessed as member of the parent struct
mt.myints.i = 10  // other way of accessing the member
mt.myints.i2 = 20 // not "promoted", as there is overlapping
mt.i2 = 30
mt.f = 44.0
```

#### Methods

Methods are declared separately from the struct:

```golang
type Embedded struct {}

func (e *Embedded) method() {
	fmt.Println("Embedded!")
}
```

Methods of embeDynamic dispatching:

```golang
type Embedding struct{
	Embedded
}

func (e *Embedding) method() {
	fmt.Println("Embedding!")
}

val := Embedded{}
val.method() // "Embedded!"

val := Embedding{}
val.method() // "Embedding!"
```

#### Interfaces

Golang uses duck typing: interface implementation is not declared, instead, it's implicit through the implementation of the interface methods (which constitute the _Method set_).

Interface names idiomatically end with `-er`, and if there is only one method, they are named after the method.

```golang
type Speaker interface {
  void Speak()
}
```

The _Empty interface_ has no methods, and can represent any type (including the simple ones): `interface{}`.

General guidelines: APIs should accept interfaces and return concrete types.

### Type casting/assertion/switch

Simple typecast:

```golang
i := 333
f := float64(i)
```

Type assertion. Types must have the expected interface; it's not a casting:

```golang
func test(v interface{}) {
	f, ok := v.(string)

	if !ok {
		fmt.Println("casted!:", f)
	} else {
		fmt.Println("not casted!")
	}
}

test(123) // not casted!
```

Type switch:

```golang
func test(v interface{}) {
	switch t := v.(type) {
	case bool:
		fmt.Println(t, "is a bool!")
	case float32, float64:
		fmt.Println(t, "is a float!")
	default:
		fmt.Println(t, "is something else!")
	}
}
```

### If/then/else, switch/case

The `<initialization_expr>` can be an assignment (e.g. `err`).

```golang
if [<init_expr>]; <boolean_expr> {
} else if {
} else {
}
```

If `value_expr` is present, the `case` expressions are matching values; if not, then they're expressions.

```golang
switch [<init_expr>;] [<value_expr>] {
case <expr>[, <expr>]:
  <statements>
  [fallthrough]
default:
  <statements>
}
```

Note how in order to only have the `init_expr`, it must be terminated with a semicolon.

### Loops

#### For

```golang
for <boolean_expr> {}

// Multiple variables can be initialized (and also incremented) at once, via multiple assignment.
// Without any expression, it's a while true.
//
for [[<start_expr>]; [<condition>]; [<increment_expr>]] {
  if true {
    break
  } else {
    continue
  }
}

// For strings, `i` is the index of the byte in the string.
//
for i := range <array> {}                           // Array: index (!!), Map: key
for i, e := range <collection> {}                   // Array: index/entry, Map: key/value. !! the iteration order is randomized !!

// There is no easy way to iterate a range in reverse; the index-based for loop needs to be used.
```

### Builtin functions

```golang
len(collection)                // elements in a collection; String -> number of bytes
```

### Errors

```golang
err := fmt.Errorf("Unexpected string: %v", str)        // preferred
err := errors.New("Error in returnError() function!")  // alternative

err.Error()                        // get the message
panic(err)                         // panic throwing the given error
```

## Concurrency

### Goroutines

Goroutines don't share memory (differently from threads).

```golang
var cnt int

func updateCounter(wg *sync.WaitGroup, mtx *sync.Mutex) {
	mtx.Lock()

	cnt++

	mtx.Unlock()

	// Important!!
	wg.Done()
}

func main() {
	// The wait system is typically based on `WaitGroup`s
	wg := &sync.WaitGroup{}

	mtx := &sync.Mutex{}

	// Number of goroutines in the group
	wg.Add(2)

	// Goroutine(s)
	go updateCounter(wg, mtx)
	go updateCounter(wg, mtx)

	wg.Wait()
}
```

See the [Atomic](#atomic) section for the atomic operations support package, which are much faster than sync.Mutex!

### Channels

Instantiation and data transfer:

```golang
var ch chan int
ch = make(chan int)

// Unbuffered
ch := make(chan int)

// Buffered
ch := make(chan int, 10)

// Data transfer; must close when not needed anymore.
defer close(ch)
ch <- 2
i := <- ch

// Iteration; stops only when the channel is closed!
for i := range ch { ... }
```

No more data than the buffer can be read/written in the main thread, *without* a thread writing/reading on the other end, as operations are blocking.

```golang
// Error ("fatal error: all goroutines are asleep - deadlock!")
ch := make(chan int)
ch <- 1

// Error again (on second send)
ch := make(chan int, 1)
ch <- 1
ch <- 1

// No errors (but WaitGroup *must* be added)
ch := make(chan int)
go func() {
  time.Sleep(1 * time.Second) // no problem!
  fmt.Println(<-ch)
}()
ch <- 1
```

Channels can be passed to functions; they can also be used as simple `WaitGroup`s:

```golang
func worker(c chan int, w chan bool) {
	for i := range c {
		fmt.Println(i)
	}

	w <- true
}

func main() {
	c, w := make(chan int), make(chan bool)

	go worker(c, w)

	for i := 0; i < 10; i++ {
		c <- i
	}

  // If this is not closed, the workers waits in the range, and since nothing is reading from `w`,
  // an error happens on the `w` write!
	close(c)

  <-w
}
```

Typical concurrency patterns:

- pipeline: serial process, from one "_source_" to a "_sink_" (final stage)
- fan out/fan in: parallel processing, via workers workers reading from the same channel

### Non-blocking reads (select) + Interruptible threads (Context)

This shows also `select`, which doesn't block:

```golang
func longRunning(c context.Context, r chan string) {
exit:
	for {
		select {
		case <-c.Done():
			// !! Must use label (or return) !!
			break exit
		default:
			// crunch...
		}

		time.Sleep(10 * time.Millisecond)
	}

	fmt.Println("Cleanup")

	r <- "Completed"
}

func main() {
	c := context.TODO()
	cl, stop := context.WithCancel(c)

	r := make(chan string)

	go longRunning(cl, r)

	time.Sleep(100 * time.Millisecond)

	stop()

	// Without this, the goroutine cleanup may not execute
	fmt.Println(<-r)
}
```

### Base template of producers/consumers with queue processing

Synchronization is performed via `dataChan`, rather than via mutex, which is not considered idiomatic.

```golang
func producer(val int) ([]int, err error) {
	return []int{val, val, val}, nil
}

func concurrentProduceAndConsume() {
  // Add another channel, for error handling
	dataChan := make(chan []int)
  errorChan := make(chan error)
	completionChan := make(chan interface{})

	waitGroup := &sync.WaitGroup{}

	var result []int

	// Production ////////////////////////////////////////////////////////////////

	for i := 0; i < runtime.NumCPU(); i++ {
		waitGroup.Add(1)

		go func(i int, dataChan chan []int, waitGroup *sync.WaitGroup) {
      defer waitGroup.Done()

      if result, err := producer(i); err != nil {
        errChan <- err
      } else {
  			dataChan <- result
      }
		}(i, dataChan, waitGroup)
	}

	// Waiting ///////////////////////////////////////////////////////////////////

	go func(waitGroup *sync.WaitGroup) {
		waitGroup.Wait()
		completionChan <- struct{}{}
	}(waitGroup)

	// Consumption ///////////////////////////////////////////////////////////////

	for {
		select {
    case err := <-errorChan:
      return err
		case <-completionChan:
			fmt.Println(result)
			return
		case data := <-dataChan:
			result = append(result, data...)
		}
	}
}
```

## APIs

### Strings

```golang
// Raw string; the prefix spaces are not trimmed!
val := `foo
bar\nbaz`

// Interpreted string; can't be on separate lines.
val := "foo\nbar\nbaz"

// Rune
val := '🤯'

// Byte (the type must be specified)
var val byte = 'b'
```

#### General concepts

Strings are seen both as a collection of bytes, and a collection of `rune`.

Accessing a string via index, or using `len`, will access the individual bytes.

```golang
runes := []rune(str);
fmt.Printf("String chars len: %v\n", len(runes))

fmt.Printf("String chars len: %v\n", utf8.RuneCountInString(str)
```

Iterate runes:

```golang
for index, runeValue := range str {
  fmt.Printf("%#U starts at byte position %d\n", runeValue, index)
}
```

#### Formatting

Striong formatting:

```golang
fmt.Println(v1, v2)                         // a space is inserted between the variables
fmt.Print(v1, v2, "\n")
fmt.Printf("%s%d\n", v1, v2)

str := fmt.Sprintf("%s, %s", date, time)    // printf (to string) in Golang
```

Formatters (with support):

- Generic
  - `%v`: the value in a default format; when printing structs, the plus flag (%+v) adds field names
  - `%T`: variable type
- Numeric
  - `-`: left aligned; (w)idth; `0`-padding
  - `%d`: decimal: `-w0`
  - `%f`: float: `-w0` + `.p`recision (don't depend on width); width includes dot and decimals;
  - `%b`: base 2: `-w0`
  - `%x`/`%X`: base 16 (lower/upper case letters): `-w0`
- Characters
  - `%c`: the character represented by the corresponding Unicode code point
  - `%U`: Unicode format: U+1234; same as `U+%04X`

Using `%#`  increases the verbosity for some: at least `%v`, `%U`.

#### Parsing/Numerical/Character conversion

```golang
b, err := strconv.ParseBool("true")
f, err := strconv.ParseFloat(str, bitsize)           // excessive numbers cause an Inf log, but not an error
i, err := strconv.ParseInt(str, base, bitsize)
u, err := strconv.ParseUint(str, base, bitsize)

str := strconv.Itoa(97)                              // integer to string
string(intVar); string(byteVar)                      // alternative
string([]byte)                                       // array of bytes to string
// also see fmt.Sprintf
```

#### Utils

```golang
// `strings` package

strings.HasSuffix(str, suffix)
strings.ToUpper(str)
strings.Join(str, separator)
strings.ReplaceAll(str, search, replace)
strings.TrimSpace(str)
strings.Repeat("*", n)o
strings.Contains(str, str)

// `unicode` package

unicode.IsUpper(char)
unicode.IsLower(char)
unicode.IsNumber(char)
unicode.IsPunct(char)
unicode.IsSymbol(char)

// splitting

strings.Split(str, separator)                        // if separator is empty, split after every UTF-8 sequence
bytes.Split(str, separator []byte)                   // byte-based split; if separator is empty, split after every UTF-8 sequence
```

### Reflection

```golang
type Table struct {
	Name string
}

func main() {
	table := Table{
		Name: "MyTable",
	}

	fmt.Println("name: ", reflect.TypeOf(table).Name())                      // Table
	fmt.Println("field value: ", reflect.ValueOf(table).FieldByName("Type")) // MyTable
}
```

#### DeepEqual

Deep equality for complex objects. Conditions:

- arrays must have the same elements, in the same order;
- maps don't have an order concept, so there's no condition.

```golang
// true
reflect.DeepEqual(
  make([]int, 10),
  make([]int, 10),
)

// true
reflect.DeepEqual(
  map[int]string{1: "one", 2: "two"},
  map[int]string{2: "two", 1: "one"},
)
```

### Regular expressions

```golang
matches, err := regexp.MatchString("<pattern>", "<string>")

// Panics in case of compilation error.
//
r := regexp.MustCompile(`^(-?\d+(.\d+)?), (-?\d+(.\d+)?)$`)

// Same as the basic regexp.MatchString, but doesn't return err.
//
m := r.MatchString

// Finds all matching substrings. Non-capturing groups are included!
// -1: find all matches
//
s := r.FindAllString(coordinatesStr, -1)

// Finds capturing groups, as arrays of []string.
// -1: find all matches
//
g := r.FindAllStringSubmatch(coordinatesStr, -1)

// Search/replace
s := r.ReplaceAllString(s, replace)
```

Modifiers:

- `(?i)`: case insensitive

### Time

Instantiation:

```golang
now = time.Now()
t = time.Date(2009, 11, 17, 20, 34, 58, 651387237, time.UTC))
monday = time.Monday

time.Time{}       // "zero" time
```

Operations/Comparisons:

```golang
t2 := t.Add(25 * time.Second)
t3 := t.Sub(1 * time.Minute)

t2.Before(t)
t2.After(t)
t2.Equal(t)
```

Conversions:

```golang
t.Weekday()
t.Nanoseconds()
```

### Math

```golang
math.Abs(f float)
math.Inf(sign int)              // positive/negative sign = positive/negative infinity
math.IsInf(sign int)            // positive/negative, or 0 for indifferent
Pow(x, y float64)               // power (exponentiation)

math.NaN
math.IsNaN

math.Round(f * math.Pow10(precision)) / math.Pow10(precision) // round number to precision (Golang doesn't have a `round(f, p)` API)
```

Note that infinity _is_ a number.

### Random

```golang
import "math/rand"

rand.Seed(time.Now().UnixNano())

func (r *Rand) Intn(n int) int          // random non-negative int, in interval [0, n)
func Read(p []byte) (n int, err error)  // fill a slice with random numbers; returns bytes written and never fails
```

### Streams read/write, Buffer

Reading line by line:

```golang
scanner := bufio.NewScanner(f)

// Optionally set the `SplitFunc` (defaults to ScanLines otherwise)
// The bufio available are: ScanLines, ScanWords, ScanBytes, ScanRunes
//
scanner.Split(scanner.ScanWords)

for scanner.Scan() {
  fmt.Println(">", scanner.Text())
}
```

Alternative, via NewReader():

```golang
reader := bufio.NewReader(stdout)

for {
  line, err := reader.ReadString('\n')
  if err != nil {
    break
  }
  fmt.Print(line)
}
```

I/O operations:

```golang
io.WriteString(w Writer, s string) (n int, err error)                // write a string to any writer
io.Copy(dst Writer, src Reader) (written int64, err error)           // copy from reader to writer
io.CopyN(dst Writer, src Reader, n int64) (written int64, err error) // like Copy, but for the given number of bytes
```

Byte-based stream readers/writers:

```golang
// Bytes [Buffer](https://golang.org/pkg/bytes/#Buffer); implements Reader and Writer:
//
var buffer bytes.Buffer
log.SetOutput(&buffer)
fmt.Println(buffer.String())

// *Buffer from a string
buffer := bytes.NewBufferString("str")

// Reader for stream of bytes
reader := bytes.NewReader(slice)

// /dev/null writer!!!
ioutil.Discard
```

### Files/Dirs/descriptors

Create/delete a file:

```golang
f, err := os.OpenFile(LOGFILE, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
f, err := os.Open(filename)                                               // open for reading only

os.IsNotExist(err error) bool         // check if error on open is caused by file not existing

err := os.Remove(filename)

```

Directory operations:

```golang
os.MkdirAll(path string, perm FileMode) error   // mkdir -p; `perm` is used for the created dirs
err := os.RemoveAll(path)                       // recursive deletion
```

Read/write from/to files:

```golang
ioutil.ReadFile(filename string) ([]byte, error)
ioutil.WriteFile(filename string, data []byte, perm os.FileMode) error
```

Read file trees (directories):

```golang
// Return a list
func ioutil.ReadDir(dirname string) ([]os.FileInfo, error)

// Ignores errors; only possible is ErrBadPattern
func filepath.Glob(pattern string) (matches []string, err error)

// Walk a tree. Doesn't follow symlinks.
//
err = filepath.Walk(".", func(path string, info os.FileInfo, err error) error {
  // operations with `info`
})
```

Temporary files/dirs:

```golang
// Create a temporary file, close it, then get the name.
//
// A random string is appended to `prefix`, in order to generate the filename
//
func TempFile(dir, prefix string) (f *os.File, err error)
err := temp.Close()
temp.Name()

// Set `parentDir` to "" to use the default temporary dir.
//
tempDir, err := ioutil.TempDir(parentDir, namePrefix string)
```

Standard file descriptors:

- `os.Stdin`
- `os.stdOut`
- `os.Stderr`

File(name) operations:

```golang
import "path/filepath"

filepath.Base(string)               // file basename
filepath.Join(elem ...string)       // Filename.join()

filepath.Abs(string)                // absolute representation of the path (File.expand_path)
filepath.Ext(string)                // extension

_, err := os.Stat(filename)         // check if a file exists; watch out - not necessarily needed (see https://stackoverflow.com/q/12518876)
```

#### File informations

```golang
info, err := os.Stat(filename)

info.IsDir()
info.Name()
info.Size()
```

#### Reading from stdin

```golang
reader := bufio.NewScanner(os.Stdin)
buffer := ""

for reader.Scan() {
  buffer += reader.Text() + "\n"
}
```

### JSON

```golang
json.Marshal(interface{}) []byte, error   // Encode object into JSON
```

### CSV

```golang
csv.NewReader(r reader)

// Read at once

rows, err := csv.ReadAll()

for i, row := range rows {
  if i == 0 {
    continue // headers
  }
  strconv.ParseFloat(row[0], 64)  // Example; columns must be parsed
}

// Read line by line

csv.ReuseRecord = true            // Avoid reallocating new arrays

for i := 0; ; i++ {
  row, err := cr.Read()

  if err == io.EOF {
    break
  } else if i == 0 {
    continue
  }

  // ...
}
```

### Gzip

Compress via gzip. It's somewhat tricky to handle all the errors of writer and source. The book advises not to defer `Close()`, so that the errors can be handled depending on the error context.

```golang
gzWriter := gzip.NewWriter(w io.Writer) *Writer
gzWriter.Name = filename
io.Copy(gzWriter, inputReader)
if err := gzWriter.Close(); err != nil { return err }
```

### O/S, Environment variables, Runtime reflection

```golang
os.Exit(<exit_status>)                 // exit to the O/S

path, err := os.Getwd                  // current path ($PWD)
val := os.Getenv("ENV_VAR")            // get environment variable; empty if not found
val, found := os.LookupEnv("ENV_VAR")  // same, but false if not found

runtime.GOOS                           // e.g. "windows"
```

### Command line arguments/Options parsing

```golang
// commandline arguments
os.Args

// parsing options
// format (option with minus (= -l), default, comment); returns a reference
// generates help:
//
//     Usage of ./playground_golang:
//       -l	Count lines
//
lines    := flag.Bool("l", true, "Count lines")
task     := flag.String("task", "", "Task to be included in the ToDo list")
complete := flag.Int("complete", 0, "Item to be completed")

flag.Parse()
```

The `-h` flag is implemented automatically; it invokes `flag.Usage()`, which can be overwritten:

```golang
flag.Usage = func() {
  fmt.Fprintf("Custom help")
  flag.PrintDefaults()
}
```

Multiple commands can be defined via `FlagSet`:

```golang
countCommand := flag.NewFlagSet("count", flag.ExitOnError)
countTextPtr := countCommand.String("text", "", "Text to parse. (Required)")
// other commands/flags

if len(os.Args) < 2 { os.Exit(1) }

switch os.Args[1] {
case "count":
    countCommand.Parse(os.Args[2:])
// other commands
}

if countCommand.Parsed() {
  // handle flags
}
```

See [Building a Simple CLI Tool with Golang](https://blog.rapid7.com/2016/08/04/build-a-simple-cli-tool-with-golang) for the FlagSet example; it also includes custom Flag Types.

### Executing shell commands

```golang
command, err := exec.LookPath("binary")   // use if one needs to look for an executable in $PATH

command := exec.Command("ls", "-l")
command.Dir = "/new_path"

// Synchronous execution

var out, err bytes.Buffer
command.Stdout = &out                   // redirect
command.Stderr = &err

err := command.Run()                    // must set Stdout; `Output()` funcs can't be used

// Synchronous alternative

[]byte, err := command.Output()         // Runs, and stdout
[]byte, err := command.CombinedOutput() // stdout + stderr

// Asynchronous

err := command.Start()
err := command.Wait()

StdinPipe() (io.WriteCloser, error)     // `Pipe()` funcs require Wait()
StdoutPipe() (io.ReadCloser, error)
StderrPipe() (io.ReadCloser, error)
```

Example of reading stdout (without error checking):

```golang
command := exec.Command("ls", "-l")
stdout, _ := command.StdoutPipe()
reader := bufio.NewScanner(stdout)

command.Start()

for reader.Scan() {
  fmt.Println(reader.Text())
}

command.Wait()
```

If one needs to send data to the process, use `StdinPipe` the same way, but *don't forget to close after writing*, otherwise the process will hang waiting.

Not using bufio requires convoluted byte slices management (see https://is.gd/HDblbp).

#### Context

Context allows setting a timeout:

```golang
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Nanosecond)
defer cancel()

command := exec.CommandContext(ctx, "ls", "-l", "/tmp")

if result, err := command.Output(); err != nil {
  if ctx.Err() == context.DeadlineExceeded {
    os.Exit(1)
  }
} else {
  fmt.Println(string(result))
}
```

### Processes; Signals sending/handling

Get PID of current process, and send signal:

```golang
processPid := syscall.Getpid()
syscall.Kill(processPid, syscall.SIGINT)
```

Catching. It's not clear why this works when sending Ctrl+C, but not when signals are sent from another process, eg. via `pkill`.

```golang
// Set size of 1, so that at least one signal is handled, if many are sent
sig := make(chan os.Signal, 1)

// Specify signals to catch
signal.Notify(sig, syscall.SIGINT, syscall.SIGTERM)

// On more complex programs, integrate in the main events cycle
rec := <-sig

// Stop relaying signals to the channel
signal.Stop(sig)

fmt.Printf("Received signal: %s. Exiting\n", rec)
```

### Logging

Basic logging:

```golang
// Like `fmt`, but they prefix entries with a timestamp.
//
log.Println(); log.Printf(); log.Print()

// Log and exit!
//
log.Fatalln(); log.Fatalf(); log.Fatal()

// Log and panic!
//
log.Panicln(); log.Panicf(); log.Panic()

// Customize prefix. Sample output:
//
//    2020/04/17 23:34:41.008270 /path/to/play.go:8: Entry!
//
// Use log.SetFlags(0) for no additions.
//
log.SetFlags(log.Ldate | log.Lmicroseconds | log.Llongfile)
```

Syslog logging:

```golang
import "log/syslog"

sysLog, err := syslog.New(syslog.LOG_INFO | syslog.LOG_LOCAL7, programName)        // (priority|facility)

log.Fatal(v ...interface{})        // terminates the Go program, after logging
log.Panic(v ...interface{})        // also terminates, but prints more information

// Log levels: composition of: (Print|Fatal|Panic)(|f|ln)
// Standard log facilities: auth, authpriv, cron, daemon, kern, lpr, mail, mark, news, syslog, user, UUCP, local(0..7)

myLog := log.New(f, ogLinePrefix, log.LstdFlags | log.Lshortfile)    // Custom logger; LstdFlags prints the timestamp, Lshortfile filename+line num
```

### Atomic

Atomic operations package; supports operations on `[u]int(32|64)`. Much faster than using a mutex!

```golang
atomic.AddInt32(&myvar, myint32)
atomic.CompareAndSwapXXX()
atomic.LoadXXX()
atomic.StoreXXX()
atomic.SwapXXX()
```

Variants: `Int32`, `Int64`, `Uint32`, `Uint64`, `Pointer`.

### Sleep

```golang
time.Sleep(time.Second)   // Sleep one second
```

### Memory allocation

```golang
var m runtime.MemStats
runtime.ReadMemStats(&m)
fmt.Printf("TotalAlloc (Heap) = %v MiB\n", m.TotalAlloc/1048576)
```

### Unsafe

WATCH OUT! The unsafe package does not come with Go 1 compatibility guidelines - it could stop working in future versions.

```golang
func Float32bits(f float32) uint32
{
  return *(*uint32)(unsafe.Pointer(&f))
}
```

### CGo

The commented lines are the actual C code.

WATCH OUT!:

- There must be no empty lines between the C code and the import;
- The import must be on its own line

otherwise, the error `could not determine kind of name for C.<function_name>` will be raised.

```golang
package main

// #include <stdio.h>
// #include <stdlib.h>
//
// static void myprint(char* s) {
//   printf("%s\n", s);
// }
import "C"
import "unsafe"

func main() {
	cs := C.CString("Hello World!")
	C.myprint(cs)
	C.free(unsafe.Pointer(cs))
}
```

Example APIs:

```golang
// Converts Go string to C string
func C.CString(string) *C.char
// C data with explicit length to Go []byte
func C.GoBytes(unsafe.Pointer, C.int) []byte
```

## Testing

Compacted test case (`main_test.go`, for testing `main.go`):

```golang
// A `_test` suffix makes only exported types accessible.
//
package counter_test

// Helper functions should invoke t.Helper(), so that the error location is displayed in the callee.
//
func testHelper(t *testing.T, value interface{}) {
  t.Helper()
}

func TestCountWords(t *testing.T) {
  if res := count(b); res != 4 {
    t.Logf("Debug message\n")
    t.Errorf("Expected %d, got %d\n", 4, res)
  }
}
```

Table-based testing:

```golang
func TestRun(t *testing.T) {
  testCases := []struct {
    name     string
    // data ...
    expected string
  }{
    // ...
  }
  for _, tc := range testCases {
    t.Run(tc.name, func(t *testing.T) {
      res := // run test (data) ...
      if tc.expected != res {
        t.Errorf("Expected %q, got %q instead\n", tc.expected, res)
      }
    }
  }
```

### Strategies/tools

In order to use mocks, in the program code, instead of invoking a function directly, set it as unexported package variable; the testing code will be able to replace it!

Tools:

```golang
iotest.TimeoutReader(bytes.NewReader([]byte{0}))             // mock reader, that times out!
```

### Benchmarking

Code:

```golang
func BenchmarkRun(b *testing.B) {

  // `b.N` is automatically adjusted to make the benchmark run for 1 second.
  //
  for i := 0; i < b.N; i++ {
    // code to benchmark here
  }
}

// Extra functions:

b.ResetTimer()
```

Run:

```sh
# The run parameter avoids running anything except the benchmark function.
#
# - `benchtime` : use multiplier if the default (1 second) is too low.
# - `cpuprofile`: profile the cpu (slices of 10ms)
# - `memprofile`: profile the memory
# - `benchmem`  : display the total memory allocation
# - `trace`:    : tracing tool, for analyzing waits
#
go test -bench . -run ^$ -benchtime=10x \
  -cpuprofile cpu.pprof \
  -memprofile mem.pprof \
  -benchmem \
  -trace trace.out \
  | tee benchresults00.txt

# CPU profiler
#
go tool pprof cpu.pprof
(pprof) top -cum         # sort by `cum`ulative time
(pprof) list funcName    # accepts regex
(pprof) web              # display diagram

# Memory profiler
#
go tool pprof -alloc_space mem.pprof
(pprof) top -cum         # sort by `cum`ulative memory

# Tracing tool: automatically opens a browser
#
go tool trace trace.out
```

#### Useful tools

Useful tool to compare benchmark runs output:

```sh
go get -u -v golang.org/x/tools/cmd/benchcmp

benchcmp benchresults00.txt benchresults01.txt
# benchmark           old ns/op     new ns/op     delta
# BenchmarkConcat     523           68.6          -86.88%
#
# benchmark           old allocs    new allocs    delta
# BenchmarkConcat     3             1             -66.67%
#
# benchmark           old bytes     new bytes     delta
# BenchmarkConcat     80            48            -40.00%
```

## Management

### Packages/naming

A package is a directory with files; files in each package are generally divided by scope, e.g.:

- `strings`
  - `builder.go`
  - `compare.go`
  - `reader.go`
  - `replace.go`
  - `search.go`
  - `strings.go`

Package names are typically simple nouns (abbreviation is encouraged), without underscores, singular or plural, specific (non generic), e.g.:

- `strconv`
- `sync`
- `time`

"_Exported_" entities are accessible from outside the package, and start with a capital letter; "_Imported_" is the opposite.

Relevant env vars:

- `$GOROOT`: stdlib search path
- `$GOPATH`: user/imported libraries search path
  - `bin`: binaries
  - `pkg`: object files
  - `src`: source files
    - `src/github.com/Sav/Go/Chapter1` -> `import "github.com/Sav/Go/Chapter1"`

Using aliases for shared package names:

```golang
import f "fmt"
```

#### init() function

The `func main()` function are executed for each file in a package, before `main()`.

Each file can have multiple `init()` functions; the package order is the same of the [files passed to the compiler](https://stackoverflow.com/a/32829593).

### Tools

```sh
go build -o "$binary" "$sourcefile"
gofmt -w "$sourcefile"                # `w`rite changes
goimports -w "$sourcefile"            # add imports; `w`rite changes
go vet "$sourcefile"                  # !! static analysis !!
go run --race "$sourcefile"           # run with race detector
go get github.com/sav/project         # download a library
```

Test, and wildcard (`./...`; excludes `./vendor`):

```sh
# `v`erbose: prints also successful unit tests.
#
go test -v ./...
```

### Modules

[Modules](https://github.com/golang/go/wiki/Modules) basic usage:

```sh
go mod init github.com/my/repo
```

There are many other aspects, including:

- allow the project to be stored anywhere
- dependencies are automatically handled

Configuration is stored in a `go.mod` file.

### Build tags

Sample (complex) tagging; equals `(amd64 AND darwin) OR (i386 AND NOT gccgo)`

```golang
//+build amd64,darwin i386,!gccgo

package abc
```

Must leave one space after the `//+build` directive!

Filename suffixes can also be used as build tags; the formats is `filename[_GOOS][_GOARCH]`, e.g. `main_darwin.go`.

### Go environment variables, anc cross compiling

GO env variables, with the two most common:

```sh
$ go env
GOARCH="amd64"
GOOS="linux" # windows, darwin
...
```

There are many archs/OSs supported; see [source code](https://github.com/golang/go/blob/master/src/go/build/syslist.go).

Cross-compiling:

```sh
GOOS=darwin go build
```

### Shared libraries (invoke from other languages)

See https://github.com/vladimirvivien/go-cshared-examples.
