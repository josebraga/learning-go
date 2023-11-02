# GO

* No implicit conversions
* No pointer arithmetic
* struct is a collection of fields
* Dereference pointer fields with ptr.X
* For range to iterate over slice/map `for i, v := range obj`
* Functions are first class objects (aka values, higher order)
* Go functions may be closures. A closure is a function value that references variables from outside its body.
  The function may access and assign to the referenced variables; in this sense the function is "bound" to the variables.
* A nil error denotes success; a non-nil error denotes failure.
    
## Packages

* Exported names start with capital letter



## Slices

* Slices are like references to arrays
* Create dynamically-sized arrays with `make` => `a := make([]int, 0, 5)`, ie., 0 length, 5 capacity

* Slice literals: prints `[{2 true} {3 false}]`

```
	s := []struct {
		i int
		b bool
	}{
		{2, true},
		{3, false},
	}
	fmt.Println(s)
```

* `append` to slice; if exceeds capacity, a new one is allocated



## Maps

* Check key exists with: `elem, ok = m[key]`

```
var m = map[string]Vertex{
	"Bell Labs": {40.68433, -74.39967},
	"Google":    {37.42202, -122.08408},
}
```

* Word count
```
func WordCount(s string) map[string]int {
    m := make(map[string]int)

	for _, word := range strings.Split(s, " ") {
		m[word]++
	}

    return m
}
```


## Functions

* `defer` statements executed when function returns (args are eval immediately!) ; also they are stacked, so executed in LIFO

```
func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}
```

* Named return values

```
func split(sum int) (x, y int) {   // as if declared at top of function
	x = sum * 4 / 9
	y = sum - x
	return
}

func main() {
	fmt.Println(split(17))
}
```

* Declarations: (Only) inside functions, `:=` is available for implicit types (non-const!)

* Function closures: `adder()` returns a closure, where each closure is bound to its own `sum` variable

```
func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}
```

or the fibonacci exercise

```
func fibonacci() func() int {
	x, y := 0, 1
	return func() int {
        x, y = y, x+y
        return y
	}
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}
```


## Methods

A method is a function with a special receiver argument.

The receiver appears in its own argument list between the `func` keyword and the method name.

In this example, the `Abs` method has a receiver of type `Vertex` named `v`.

```
type Vertex struct {
	X, Y float64
}

// copy
func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

// pointer
func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10)             // Go abstracts away to (&v).Scale(5)
	fmt.Println(v.Abs())    // 50
}
```

* You can only declare a method with a receiver whose type is defined in the same package as the method, so

```
type MyFloat float64

func (f MyFloat) noop() float64 {
	return float64(f)
}

func main() {
	f := MyFloat(-math.Sqrt2)
	fmt.Println(f.noop())
}
```


## Interfaces

* A type implements an interface by implementing its methods. There is no explicit declaration of intent, no "implements" keyword.


```
type Abser interface {
	Abs() float64
}

func main() {
}

// next two functions implicitly implement interface `Abser` but do not have to explicitly declare so

// is compatible with `Abser`
func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

// is compatible with `Abser`
func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

// NOT compatible!
func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

```

* If the concrete value inside the interface itself is `nil`, the method will be called with a `nil` receiver.
  Note that an interface value that holds a nil concrete value is itself non-nil.


```
type I interface {
	M()
}

func (t *T) M() {
	if t == nil {
		fmt.Println("<nil>")
		return
	}
	fmt.Println(t.S)
}
```

* A nil interface value holds neither value nor concrete type: run-time error

* Empty interface to handle values of unknown type, e.g, `fmt.Print` takes any number of arguments of type `interface{}`.

* Type assertion: access to an interface value's underlying concrete value

```
t := i.(T)  // panic if fails

t, ok := i.(T)
```

* Type Switch

```
switch v := i.(type) {
case T:
    // here v has type T
case S:
    // here v has type S
default:
    // no match; here v has the same type as i
}
```

#### Stringer interface

```
type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}

func main() {
	a := Person{"Arthur Dent", 42}
	z := Person{"Zaphod Beeblebrox", 9001}
	fmt.Println(a, z)
}
```

## Errors

* Avoid recursion on `func (e Type) Error() string` methods: `Sprintf` calls `e.Error()` again to get a string, thus a recursion

```
func (e ErrNegativeSqrt) Error() string {
  return fmt.Sprintf("cannot Sqrt negative number: %v", float64(e))
}
```

## Readers

* `io.Reader` interface, which represents the read end of a stream of data.

```
func main() {
	r := strings.NewReader("Hello, Reader!")

	b := make([]byte, 8)
	for {
		n, err := r.Read(b)
		fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
		fmt.Printf("b[:n] = %q\n", b[:n])
		if err == io.EOF {
			break
		}
	}
}
```

* A common pattern is an `io.Reader` that wraps another `io.Reader`, modifying the stream in some way.
  For example, the `gzip.NewReader` function takes an `io.Reader` (a stream of compressed data) and
  returns a `*gzip.Reader` that also implements `io.Reader` (a stream of the decompressed data).
  
```
type rot13Reader struct {
	r io.Reader
}

// We read current stream, then we alter it
func (r *rot13Reader) Read(in []byte) (n int, err error) {
	n, err = r.r.Read(in)
	
	for i := 0; i < n; i++ {
		in[i] = rot13(in[i])
	}
	return
}

func main() {
	s := strings.NewReader("Lbh penpxrq gur pbqr!")
	r := rot13Reader{s}
	io.Copy(os.Stdout, &r)
}
```

## Generics

#### Constrained Generic Functions

```
// Index returns the index of x in s, or -1 if not found.
func Index[T comparable](s []T, x T) int {
	for i, v := range s {
		// v and x are type T, which has the comparable
		// constraint, so we can use == here.
		if v == x {
			return i
		}
	}
	return -1
}

func main() {
	// Index works on a slice of ints
	si := []int{10, 20, 15, -10}
	fmt.Println(Index(si, 15))            // 2

	// Index also works on a slice of strings
	ss := []string{"foo", "bar", "baz"}
	fmt.Println(Index(ss, "hello"))      // -1
}
```

#### Generic Types

```
type List[T any] struct {
	next *List[T]
	val  T
}
```

## Goroutines

* A goroutine is a lightweight thread managed by the Go runtime.

`go f(x, y, z)` starts a new goroutine running `f(x, y, z)`

The evaluation of `f`, `x`, `y`, and `z` happens in the current goroutine and the *execution* of **f** happens in the **new** goroutine.

Goroutines run in the same address space, so access to shared memory must be synchronized.
The `sync` package provides useful primitives, although you won't need them much in Go as there are other primitives.


#### Channels

* Channels are a typed conduit through which you can send and receive values with the channel operator, `<-`.

```
ch <- v    // Send v to channel ch.
v := <-ch  // Receive from ch, and assign value to v.
```

* Create channel `ch := make(chan int)`

* Sends/receives are blocked until other side is ready: so Goroutines sync without locks or cond vars

```
func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // send sum to c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // receive from c

	fmt.Println(x, y, x+y)       // -5 17 12
}
```

#### Buffered Channels

Sends to a buffered channel block only when the buffer is full. Receives block when the buffer is empty

`ch := make(chan int, 100)`

#### Range and Close

* A sender can close a channel to indicate that no more values will be sent `v, ok := <-ch`

* The loop `for i := range c` receives values from the channel repeatedly until it is closed.

* Only the sender should close a channel, **never** the receiver. Sending on a closed channel will cause a *panic*.

* Channels aren't like files; *you don't usually need to close* them. Closing is only necessary when
  the receiver must be told there are no more values coming, such as to terminate a range loop.

```
func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}
```


#### Select

* The `select` statement lets a goroutine wait on multiple communication operations.

* A `select` blocks until one of its cases can run, then it executes that case.
  It chooses one at random if multiple are ready.

```
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
        // whatever channel gets data, it is selected
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
    
    // immediately func() is running and populating channels
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}
```

* Default selection if no channel is ready

```
func main() {
	tick := time.Tick(100 * time.Millisecond)
	boom := time.After(500 * time.Millisecond)
	for {
		select {
		case <-tick:
			fmt.Println("tick.")
		case <-boom:
			fmt.Println("BOOM!")
			return
		default:
			fmt.Println("    .")
			time.Sleep(50 * time.Millisecond)
		}
	}
}
```

prints


```
    .
    .
tick.
    .
    .
tick.
BOOM!
```
