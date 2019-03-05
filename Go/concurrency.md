### Goroutines

1. Goroutines is a light weight thread which shares the same address space. 
Access to shared memory should be synchronized.

```
go fun(x, y, z) //starts a goroutine
```

```
func say(s string) {
	for i := 0; i < 5; i++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Println(s)
	}
}

func main() {
	go say("hello")
	say("world")
}
```

### Channels

Channel is used to coordinate information between goroutines.

1. create a chanel

```
//"chan int" means the channel stores int values
ch := make(chan int) 
```

2. send and retrieve from the channel

```
ch <- v    // Send v to channel ch.
v := <-ch  // Receive from ch, and assign value to v.

//retrieve multiple values
x, y := <-ch, <-ch
```

3. buffered channels

```
ch := make(chan int, 100)
```

4. close channel

```
c := make(chan in, 10)
for i := range c {    //receives values from the channel until it is closed
                      // if it is not closed, it will panic
	
}
```

5. select

select lets a goroutine wait on multiple communication operations

```
	select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
	}

	//default selection
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
```

