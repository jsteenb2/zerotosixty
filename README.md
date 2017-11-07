# Golang or Go Home

a programmer's introduction to Go (*not an intro to programming in general*).  Pull down the repo, ```cd``` into it and follow along.

## Why Go?

Is it the lightning fast compile and execution times that make Go so appealing? In short, yes, that is a definite plus for working in Go. However, that's not all there is to love about Go. To get the speed you don't have to load up on complexity in Go. 

30+ years of lessons learned have been applied to Go. The creators, the likes of which include Rob Pike & Ken Thompson (former bell labs, creators of large parts of Unix and the Plan-9 OS), Robert Griesemer(create of V8), and Russ Cox (another Plan-9 contributor and systems programming heavyweight), only add something to the language if it is agreed upon by the group. So what did the creators include in Go...? 

* native support for testing, crypto, networking.... you name it
	* built right into the std lib
* memory integrity/management
	* zero values
	* pointers but no pointer arithmetic
	* Garbage collected
* native support for multi-core computers
	* concurrency modeled after the again famous [CSP](https://en.wikipedia.org/wiki/Communicating_sequential_processes)(communicating sequential processes)
* Wealth of tools to support you in development
	* gofmt
	* benchmarking
	* memory profiling
	* code coverage analyzer

You get all that and more. The best thing about Go, above all else, is how simple it is to work in big teams. 

## Warning

Go is NOT object/class oriented

LEAVE YOUR OBJECTED ORIENTED HAT AT THE DOOR

## Go Type System

### Primitives

```go
var boolean bool

var aString string

var (
	anInt   int
	anInt8  int8
	anInt32 int32
	anInt64 int64
	uInt    uint64
)
	
var (
	aFloat32 float32
	aFloat64 float64
)

// more still
```

All your expected types and then some. Note for ints, you want to default to using ```int``` for integers unless you are absolutely sure you need something else. ```int``` changes depending on your systems architecture (address size).

Each time a var is created without declaring it to a specific value, it will assume the zero type. See the ```var_declarations_test.go``` file for an example of what that looks like. Memory is allocated for you, memory integrity comes first.

run example: ```$ go test -v var_declarations_test.go -run TestVariableDeclarationsToZeroValues```

### Var Declarations

There are a number of ways to declare variables and we'll look at a few of the most commonly used ones.

```go
// setting to zero values
var boolean bool
var aString string
var anInt   int

// long way
var maxInt uint64 = 1<<64 - 1
var falsey bool = false

// shorthand, sets type equal to the right hand assignment
anInt := 2017  // int type default
aFloat := 32.0 // float64 type default
boolean := true
```

The shorthand is very commonly used when you are setting the value of something at declaration. If you want the zero value, you should use the first set, guarantees zero value.

run example: ```$ go test -v var_declarations_test.go -run 'DelcarationAssignment|Shorthand'```

### Type Conversion

There is no type-casting in Go, but there is conversions. They look a lot like constructor functions...

```go
var anInt8 int8 = 2
var aFloat32 float32 = 32.00

var anInt16 int16 = 2
var aFloat64 float64 = 32.00
	
int16(anInt8) == anInt16 // output: true
float64(aFloat32) == aFloat64 // output: true

// anInt8 == anInt16 { // doesn't compile, cannot do operations on different types
```

Different types are not interchangeable without conversion. Compiler will crap the bed and tell you to fix it.... again an emphasis on data integrity.

run example: ```$ go test -v var_declarations_test.go -run Conversion```

### Data Structures

#### Arrays

Nothing to far off the beaten path. The most efficient data structure from the point of view of the hardware. Go runs on the metal, take a mental note of this.

```go
var zeroArray [4]string // zero value is array with all elements at their zero value

filledArray := [4]string{"Asynchrony", "World", "Wide", "Technology"}

unFilledArray := [4]string{"Missed", "By", "1"} // all 4 entries initialized, 3 to indicated values, 1 to zero value of "" for string

unFilledArray[3] = "covfefe"

for ele := range filledArray {
	fmt.Println(ele)
}
```

Arrays are straight forward, more of the same you've probably seen before. When you initialize an array, it will be put into memory as indicated. Any empty values will assume the zero value of the type the element is. The ```range``` operator iterates over elements of the array, similar to a foreach loop.

run example: ```$ go test -v data_structures_test.go -run Array```

#### Slices

Slices are where Go makes it's money. They are basically dynamic arrays that can manipulate capacity at runtime. A slice is a 3 word data structure that looks like:

1. pointer to backing array
2. length
3. capacity

```go
strSlice := []string{"like", "arrays", "only", "better"}

for idx, value := range strSlice {
	fmt.Printf("id: %d, value: %v\n", idx, value)
}

// can extend with append
strSlice = append(strSlice, "and", "extendable") // append(dst, ...src)

fmt.Println(strSlice)
```

Adding to a slice is easy enough with append. First arg being the destination, the rest of the arguments being the source. 

run example: ```$ go test -v data_structures_test.go -run SliceTypeWithRange```

There are several ways to create a slice and reassign elements of that slice.

```go
slice := make([]string, 0, 0) // initializes slice with 0 length and 0 capacity

slice = append(slice, "with", "great", "power", "comes", "great", "responsibility")
fmt.Println(strings.Join(slice, " "))

newSlice := slice[:] // returns copy of original slice
newSlice[2] = "food"
fmt.Println(strings.Join(newSlice, " "))

newerSlice := append(slice[:len(newSlice)-1], "indigestion")
fmt.Println(strings.Join(newerSlice, " "))
```

Pretty straightforward for reassignments. Note, you can call for a range of indices by using the ```newSlice[start:end]``` syntax. The end is exclusive. Anytime you use the ```:``` it will created a copy in memory of the backing array elements that are included in your range. Don't jump into arrays right away, the performance gain isn't world changing but it is horribly restrictive.

run example: ```$ go test -v data_structures_test.go -run SliceReassign```

#### Maps

Maps in Go are akin to the object in Javascript, maps in Java/C++, the dictionary in Python, yada yada. A key can be any type that can be equated(```==```), which includes primitives, structs, you name it. Anything except an array, slice, or another map should work. Note: the hashing function is done for you, no need to roll your own. It's robust as all get out, thank you Google!

Assignments are like you saw with arrays and slices.

```go
aMap := map[string][]string{
	"marvel": {"spiderman", "hulk", "ironman"},
	"dc":     {"batman", "superman", "wonderwoman"},
}

for key, value := range aMap {
	fmt.Printf("%s", key)
	for _, hero := range value {
		fmt.Printf("\t%s", hero)
	}
}
```

You get access to range operator on maps as well. This example illustrate some Go syntax sugar. Since the type of the value was called out in the declaration of the map, you do not need to initialize the embedded slices with the full ```[]string{}``` syntax. Go's compiler fills in the gaps for you.

You can access k/v pairs using the same syntax you are use to in other languages. You also can delete k/v pairs from a map using the ```delete``` keyword. Each access also has a lookup flag that indicates if the k/v pair exists.

```go
aMap := map[string][]string{
	"marvel": {"spiderman", "hulk", "ironman"},
	"dc":     {"batman", "superman", "wonderwoman"},
}

delete(aMap, "dc") // drops k/v pair

_, ok := aMap["dc"]

fmt.Printf("dc found: %t", ok) // output: dc found: false
```

run example: ```$ go test -v data_structures_test.go -run MapType```

#### Zero Value of Reference Types

Reference types, ```map``` and ```slice```, have a reference value of nil. 

```go
var zeroSlice []bool
var zeroMap map[string]bool

if zeroSlice == nil {
	fmt.Println("zeroSlice == nil")
}

if zeroMap == nil {
	fmt.Println("zeroMap == nil")
}
```

run example: ```$ go test -v data_structures_test.go -run ZeroTypes```

### Structs

Structs are whatever you make them be. You can tack on 
