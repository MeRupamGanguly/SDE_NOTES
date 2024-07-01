
# Notes by RupamGanguly

The SDE - 3 Journey:
---------------------------------------
---------------------------------------

### Tell me about Yourself ?

- I have a background in BTech IT, passout of 2020. After completing my degree, I joined Sensibol as a Golang Backend Developer. My primary role involved developing microservices for our clients such as PDL and SingShala using Golang, MongoDB, AWS S3, Lambda, SQS, SNS, and Elasticsearch. 
I recently resigned in April and now seeking new and exciting career opportunities.

### Describe Advantages and Disadvantes of Microservice and Monolith.

- Microservices are better for Big Project where Scaling and Zero downtime required. Bug Fix and Maintainability is easy for Micrroservices. Disadvantage is Inter Service Netowrk call.

### Describe GoRoutine vs Thread.

- Goroutines are managed by the Go runtime, while threads are managed by the operating system kernel.
- Goroutines are cheaper to create and manage than threads.
- Goroutines communicate using channels, which are safer than shared memory used by threads.
- Goroutine have dynamicall sized stacks managed by GORuntime, Thread have fixed-size stacks allocated by operating system.

### Describe Concurrency primitives.

- Concurrency primitives are fundamental tools or mechanisms provided by programming languages to help manage and control the execution of multiple processes that can run concurrently. Some common concurency primitives are Mutex, Semaphore, Condition Variable, Atomic Operation, Channels etc.

- Mutexes are used to protect shared resources (such as variables or data structures) from being accessed simultaneously by multiple threads or goroutines.

- A semaphore is essentially a counter that can be incremented or decremented atomically. When a thread wants to enter a critical section or access a shared resource, it performs a P(WAIT) operation on the semaphore. When a thread exits a critical section or finishes using a shared resource, it performs a V(SIGNAL) operation on the semaphore.
	- Wait/P op: If the semaphore's value is greater than zero (> 0), the semaphore decrements its value (semaphore_value--) and allows the thread to proceed.   If the semaphore's value is zero (== 0), the thread is blocked (put into a waiting state) until the semaphore's value becomes positive.
	- Signal/V op: This operation increments the semaphore's value (semaphore_value++).   If there are threads waiting (blocked on P), it unblocks one of them, allowing it to proceed.

- Condition Variables: Condition variables allow threads to wait until a certain condition is true before proceeding. They are typically used in conjunction with mutexes to manage complex synchronization patterns.

- Atomic Operations: They are often used for operations like incrementing counters or updating shared flags. Go provides the sync/atomic package for atomic operations on primitive types (int32, int64, uint32, uint64, etc.). Operations in this package are designed to be lock-free and efficient.
	- Atomic operations are suitable for simple operations on primitive types. For more complex operations or shared data structures, consider using mutexes or other synchronization primitives.

- Channels are a higher-level concurrency primitive provided by some programming languages (like Go and Rust).They facilitate communication and synchronization between concurrent threads or goroutines by allowing them to send and receive values. 
	- Channels can be used for synchronization, signaling, and data exchange between concurrent entities. 
	- Channels help prevent race conditions by ensuring that only one goroutine can access data at a time.  
  - Channels can be buffered, allowing goroutines to send multiple values without blocking until the buffer is full. This can improve performance in scenarios where the producer and consumer operate at different speeds.
	- Channels can be used with Go's select statement, enabling multiplexing of communication operations. This allows for more complex control flow and non-blocking operations. 
	
### Describe WaitGroup in golang.

- wg.Add(1) increments the WaitGroup counter before each goroutine starts.  Each worker goroutine calls wg.Done() when it completes its task, which decrements the counter. wg.Wait() blocks the main goroutine until the counter becomes zero, indicating that all workers have finished.

```go
func worker(i int, wg *sync.WaitGroup){
	defer wg.Done()
	time.Sleep(time.Second)
}
func main(){
	wg:=sync.WaitGroup{}
	for i:0;i<5;i++{
		wg.Add(1)
		go worker(i,&wg)
	}
	wg.Wait()
}
```

### Describe Map Synchronization.

- In Go (Golang), maps are not inherently safe for concurrent access by multiple goroutines. If one goroutine is writing to a map while another goroutine is reading from or writing to the same map concurrently, it can result in unpredictable behavior or panics. They require synchronization mechanisms such as mutexes to ensure safe concurrent read and write operations. 

```go
var m map[string]int
var mu sync.Mutex // var mu sync.RWMutex : It allows multiple goroutines to read the map simultaneously but ensures exclusive access for writes.

// Writing to map
mu.Lock()
m["key"] = 123
mu.Unlock()

// Reading from map
mu.Lock() // mu.RLock() : Acquires a read lock. Multiple goroutines can hold a read lock simultaneously, allowing for concurrent read access to a shared resource.
value := m["key"]
mu.Unlock() // mu.RUnlock()

```

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	testMap := make(map[string]string)
	testMap["Jio"] = "Reliance"
	testMap["Airtel"] = "Bharti Airtel"

	for k, v := range testMap {
		fmt.Println("Key is: ", k, "Value is: ", v)
	}
	var wg sync.WaitGroup

	mu := sync.RWMutex{}

	for i := 0; i < 10; i++ {
		wg.Add(2)
		go mapWriter(&testMap, &wg, &mu)
		go mapConsumer(&testMap, &wg, &mu)
	}
	wg.Wait()
}

func mapConsumer(testMap *map[string]string, wg *sync.WaitGroup, mu *sync.RWMutex) {
	// Go does not allow you to use range directly on a pointer to a map because it expects a map type.
	// you need to dereference the pointer to get the actual map value, and then you can range over it.
	defer wg.Done()
	vTestMap := *testMap
	for k, v := range vTestMap {
		mu.RLock()
		fmt.Println("Consumer Key is : ", k, "Value is : ", v)
		mu.RUnlock()
	}
}
func mapWriter(testMap *map[string]string, wg *sync.WaitGroup, mu *sync.RWMutex) {
	defer wg.Done()
	for i := 0; i < 10; i++ {
		// Go does not allow you to use range directly on a pointer to a map because it expects a map type.
		// you need to dereference the pointer to get the actual map value, and then you can range over it.
		mu.Lock()
		vTestMap := *testMap
		vTestMap["Agni"] = fmt.Sprint("Jal", i)
		testMap = &vTestMap
		mu.Unlock()
	}
}
```

### Describe Channel Comuniation:

```go
func producer(ch chan<-int){ //ch <- value // Sends value into channel ch
	for i:=0;i<5;i++{
		ch<-i
	}
	close(ch)
}
func consumer(ch <-chan int){ //value := <-ch // Receives from channel ch and assigns it to value
	for n:=range ch{
		fmt.Print("RECV: ",num)
	}
}
for main(){
	ch:=make(chan int)
	go producer(ch)
	consumer(ch)
}
```

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// <-After for Sender
func producer(ch chan<- int, wg *sync.WaitGroup) {
	for i := 0; i < 5; i++ {
		time.Sleep(time.Second)
		ch <- i
	}
	close(ch)
	wg.Done()
}

// <-Before for Reciver
func consumer(ch <-chan int, wg *sync.WaitGroup) {
	for d := range ch {
		fmt.Println(d)
	}
	wg.Done()
}
func main() {
	var wg sync.WaitGroup
	ch := make(chan int) // Do not use var For nil channel Deadlock Error
	wg.Add(2)
	go producer(ch, &wg)
	go consumer(ch, &wg)
	fmt.Print("Main is Doing its own Task")
	wg.Wait()
}
```

### Describe Select Statement.

- select statement is used for concurrent communication among goroutines. It allows you to wait on multiple communication operations simultaneously and proceed as soon as one of them can proceed.

- The select statement blocks until one of its cases can proceed. If multiple cases can proceed, one is chosen at random.

- Each case inside select must be a channel operation (<-channel for receive, channel <- value for send), default case (optional) runs if no other case is ready.

- If no case is ready and there's a default case, it executes immediately. If there's no default case, select blocks until at least one case is ready.

```go
func main(){
	ch1:=make (chan string)
	ch2:=make(chan string)

	go func(){
		time.Sleep(time.Second)
		ch1<-"one"
		} ()

	go func(){
		time.Sleep(time.Second)
		ch2<-"two"
		}

	select{
		case ms1:=<-ch1:
			fmt.Print(ms1)
		case ms2:=<-ch2:
			fmt.Print(ms2)
		case <-time.After(3 * time.Second):
			fmt.Print("Time Out")
	}
}
```

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	go func() {
		rand.NewSource(time.Now().Unix())
		i := rand.Intn(6)
		fmt.Println("ch1 Rand ", i)
		time.Sleep(time.Duration(i) * time.Second)
		ch1 <- 1
	}()
	go func() {
		rand.NewSource(time.Now().Unix())
		i := rand.Intn(6)
		fmt.Println("ch2 Rand ", i)
		time.Sleep(time.Second * time.Duration(i))
		ch2 <- 2
	}()

	select {
	case <-ch1:
		fmt.Print("ch1 is Done")
	case <-ch2:
		fmt.Print("ch2 is Done")
	case <-time.After(time.Second * time.Duration(4)):
		fmt.Print("Context Expire")
	}
}

```
```go
package main

import (
	"context"
	"fmt"
	"math/rand"
	"time"
)

func coreProcess(ctx context.Context, i int, ch chan<- int) {
	rand.NewSource(time.Now().Unix())
	r := rand.Intn(4)
	t := time.Duration(r) * time.Second
	ctx, cancel := context.WithTimeout(ctx, t)
	defer cancel()
	fmt.Printf("Doing Some Work for Process: %d with random %d", i, r)
	fmt.Println()
	select {
	case <-time.After(t): //Send 1 via Channel to Caller Function After Wait for Random Seconds.
		ch <- 1
	case <-ctx.Done(): // If context expire then send 0 via Channel to Caller Function
		ch <- 0
	}

}
func main() {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	ch := make(chan int)
	defer close(ch)
	for i := 0; i < 6; i++ { // Calling CoreProcess concurently 6 times.
		go coreProcess(ctx, i, ch)
	}
	// gather result from al Concurent Processes.
	for i := 0; i < 6; i++ {
		select {
		case res := <-ch: // To store it a res is important , otherwise in Print statement channel read twice
			fmt.Println(res) // fmt.Println(<-ch)  X:Wrong Syntax
		case <-ctx.Done():
			fmt.Println("Context Done")
		}
	}
}
```

### When to chose which Concurent Primitives.

- Use Mutexes: When protecting shared data with simple critical sections or when only one thread should access a resource at a time.

- Use Semaphores: When limiting access to a pool of resources or coordinating multiple threads with a fixed capacity.

- Use Condition Variables: When threads need to wait for specific conditions or notifications before proceeding.

- Use Atomic Operations: When performing simple, atomic read-modify-write operations on shared variables without full mutual exclusion.
	
- Use Channels: When implementing communication and synchronization between goroutines, especially in Go programming.


### Describe a challenging technical problem you've solved.

- At PDL, I encountered a challenging problem related to optimizing the search functionality. The platform was experiencing slow search and struggling to efficiently handle complex queries across a vast database.

- One major issue was the inefficient indexing in the database.

- I proposed and implemented a solution using Elasticsearch to power the search engine. Also I integrated Redis caching mechanisms to store frequently accessed search results, that reducing the load on the Elasticsearch cluster. I also implemented asynchronous processing using AWS Lambda and SQS to handle indexing updates in real-time.

- MongoDB creates indexes based on the fields specified, Each document needs to be evaluated and indexed according to the specified index keys. Compound indexes require more processing to create, because they involve combinations of fields. MongoDB's indexing process consumes CPU, memory, and disk I/O resources.

- When new Bulk Music Add or Updates metadata were received, instead of directly processing them synchronously within the search service, they were sent to an SQS queue. SQS acts as a messaging buffer between the main application (which generates data updates) and the indexing process. This queue will store messages containing data that need to be indexed by Elasticsearch. I Configured queue settings such as visibility timeout, message retention period, and access policies according to my requirements.

- A Lambda function was configured to poll the SQS queue for incoming messages. When a message (indicating new indexing data) was found in the queue, the Lambda function would trigger. I Configured the Lambda function to listen to the SQS queue by setting up an event source mapping in the Lambda console and Implement the logic to retrieve data from the message payload, transform it if necessary, and then index the data into Elasticsearch using Elasticsearch APIs or AWS SDKs. Inside the Lambda function, the indexing updates were processed asynchronously.

- By decoupling the indexing process from the main search service, the platform achieved several benefits: Users experienced faster search response times because the main search service was no longer burdened with synchronous indexing tasks. The platform could handle spikes in indexing workload or user queries more efficiently by scaling Lambda functions and SQS queues based on demand. SQS ensured reliable message delivery, even in the event of temporary service disruptions or failures.

- Serverless architectures like Lambda and SQS reduce costs by charging only for the compute time and messaging throughput used. Elasticsearch on AWS can be cost-effective compared to managing your own Elasticsearch cluster.

---------------------------------------

## SOLID principle:
SOLID principles are guidelines for designing software that are easy to understand, maintain, and extend over time.
- Single Responsibility Principle : A class should have one, and only one, reason to change. This means each class should have a single responsibility or job to do, making it easier to understand and modify without affecting other parts of the system.
```go
type Book struct{
  ISIN string
  Name String
  AuthorID string
}
type Author struct{
  ID string
  Name String
}
```

```go
// Assume One Author doesnot want to Disclose its Real Name to Spread. So we can Serve Frontend by Its Alias instead of Real Name of the Author. Without Changing Book Class/Struct.
type Book struct{
  ISIN string
  Name String
  AuthorID string
}
type Author struct{
  ID string
  Name String
  Alias String
}
```
- Open/Closed Principle: Software components like classes, modules, functions, etc, should be open for extension but closed for modification. This principle encourages you to design your systems in a way that allows new functionality to be added without changing existing code.
```go
// The Shape interface defines a contract for shapes that can calculate area.
type Shape interface {
	Area() float64
}
type Rectangle struct {
	Width  float64
	Height float64
}
type Circle struct {
	Radius float64
}
// The Rectangle and Circle structs implement the Shape interface with their respective Area methods.
func (r Rectangle) Area() float64 {
	return r.Width * r.Height
}
func (c Circle) Area() float64 {
	return 3.14 * c.Radius * c.Radius
}
//The PrintArea function accepts any Shape and calculates its area without modifying existing code. This adheres to Open/Closed Principle because new shapes (like Triangle) can be added without changing PrintArea function.
func PrintArea(shape Shape) {
	fmt.Printf("Area of the shape: %f\n", shape.Area())
}
func main() {
	rect := Rectangle{Width: 5, Height: 3}
	circ := Circle{Radius: 2}

	PrintArea(rect)
	PrintArea(circ)
}
```
- Liskov Substitution Principle : Objects of a superclass should be replaceable with objects of its subclasses without affecting the correctness of the program.
```go
// The Bird interface defines a contract for birds with a Fly method.
type Bird interface {
	Fly() string
}
type Sparrow struct {
	Name string
}
type Penguin struct {
	Name string
}
// Sparrow and Penguin structs implement the Bird interface with their respective Fly methods.
func (s Sparrow) Fly() string {
	return "Sparrow flying"
}
func (p Penguin) Fly() string {
	return "Penguins can't fly!"
}
//The FlyAndDisplay function accepts any Bird and displays how it flies without errors. This adheres to Liskov Substitution Principle because Penguin, despite not being able to fly, still satisfies the Bird interface.
func FlyAndDisplay(b Bird) { // b is the Object of Bird Class
	fmt.Println(b.Fly())
}
func main() {
	sparrow := Sparrow{Name: "Sparrow"}
	penguin := Penguin{Name: "Penguin"}
  // SuperClass is Bird,  Sparrow, Penguin are the SubClass
	FlyAndDisplay(sparrow)
	FlyAndDisplay(penguin)
}
```
- Interface Segregation Principle : Implementor Class should not be forced to depend on interfaces they do not use. This principle advises that interfaces should be specific to the needs of clients so that they are not burdened with unnecessary methods.
```go
// The Printer interface defines a contract for printers with a Print method.
type Printer interface {
	Print()
}
// The Scanner interface defines a contract for scanners with a Scan method.
type Scanner interface {
	Scan()
}
// The MultiFunctionDevice interface combines Printer and Scanner interfaces for multifunction devices.
type MultiFunctionDevice interface {
	Printer
	Scanner
}

type SimplePrinter struct{}
type SimpleScanner struct{}

// SimplePrinter and SimpleScanner structs implement Printer and Scanner interfaces, respectively. They follow ISP by only implementing the methods they need.
func (sp SimplePrinter) Print() {
	fmt.Println("Printing...")
}

func (ss SimpleScanner) Scan() {
	fmt.Println("Scanning...")
}

func main() {
	printer := SimplePrinter{}
	scanner := SimpleScanner{}

	printer.Print()
	scanner.Scan()
}
```
- Dependency Inversion Principle : High-level modules like Class, should not depend on low-level modules; both should depend on abstractions like interfaces or abstract classes. Abstractions should not depend on details; details should depend on abstractions. This principle promotes decoupling by ensuring that high-level modules (such as classes) rely on interfaces or abstract classes rather than specific implementations.
```go
// The MessageSender interface defines a contract for sending messages with a SendMessage method.
type MessageSender interface {
	SendMessage(msg string) error
}
// EmailSender and SMSClient structs implement the MessageSender interface with their respective SendMessage methods.
type EmailSender struct{}

func (es EmailSender) SendMessage(msg string) error {
	fmt.Println("Sending email:", msg)
	return nil
}
type SMSClient struct{}

func (sc SMSClient) SendMessage(msg string) error {
	fmt.Println("Sending SMS:", msg)
	return nil
}
// The NotificationService struct depends on MessageSender interface, not on concrete implementations (EmailSender or SMSClient). This adheres to DIP because high-level modules (NotificationService) depend on abstractions (MessageSender) rather than details.
type NotificationService struct {
	Sender MessageSender
}
func (ns NotificationService) SendNotification(msg string) error {
	return ns.Sender.SendMessage(msg)
}
func main() {
	emailSender := EmailSender{}
	smsClient := SMSClient{}

	emailNotification := NotificationService{Sender: emailSender}
	smsNotification := NotificationService{Sender: smsClient}

	emailNotification.SendNotification("Hello, this is an email notification!")
	smsNotification.SendNotification("Hello, this is an SMS notification!")
} 
```

SOLID :  
- Class/Struct should have a single responsibility. Book And Author Different Class. Order and Product Different Class. 
- New Features-Function can be added without changing Old Featers-Function.
- Objects of a superclass should be replaceable with objects of its subclasses without affecting the correctness of the program.
- Do not Club different interafces together. Implementor Class should not be forced to implement methods that they do not use.
- Promotes decoupling by ensuring that high-level modules such as classes rely on interfaces or abstract classes rather than specific implementations.
---------------------------------------
---------------------------------------

## DB Selection:
- Vertical Scaling: Hardware Upgrades. 
- Horizontal Scaling : Distributing data and workload across multiple servers or nodes, Sharding: Dividing the dataset into smaller, manageable parts called shards and distributing these shards across multiple servers. Each server (or shard) handles a subset of the data.

- MongoDB, as a NoSQL document-oriented database, typically excels in scenarios where flexibility and horizontal scalability are crucial.
- MongoDB supports multi-document ACID transactions (from version 4.0 onwards), which improves its ability to handle concurrent operations and transactional workloads.
- PostgreSQL is ideal for applications requiring strong consistency, complex joins, and analytical queries, such as financial systems, CRM applications, and data warehousing.
- PostgreSQL ensures full ACID compliance, which guarantees transactional integrity but can impact OPS(Operation Per Second) under heavy write operations compared to NoSQL databases like MongoDB.
- Redis is an in-memory data structure store known for its exceptional speed and simplicity, It excels in scenarios requiring low latency and high throughput for read and write operations.

- The type of operations (read-heavy, write-heavy, transactional, analytical) significantly impacts OPS performance for each database system.
- MongoDB and Redis excel in horizontal scaling, while PostgreSQL supports vertical scaling better. The ability to scale horizontally can directly influence OPS performance in distributed and large-scale applications.
- MongoDB’s document-oriented model and Redis’s key-value store offer performance advantages for certain use cases compared to PostgreSQL’s relational model, which may require more indexing and query optimization for high OPS.

## MongoDB Notes

```bash
# Run Docker Container
docker run -d --name mongodb-server \
-e MONGO_INITDB_ROOT_USERNAME=admin \
-e MONGO_INITDB_ROOT_PASSWORD=adminpass \
-p 27017:27017 \
-v mongodb-data-vol:/data/db/ \
-v mongodb-log-vol:/var/log/mongodb/ \
mongo
```
```
db.Company.find({ "employee.is_active":true}); || filter1 := bson.M{"employee.is_active": true}

db.Company.find({ "employee.roles":{$in:["Sell","Finance"]}}); || filter2 := bson.M{"employee.roles": bson.M{"$in": []string{"Sell", "Finance"}}}
```
```go
// Golang Mongo Driver Example
mongoOption := options.Client().ApplyURI("mongodb://admin:adminpass@localhost:27017/")
mongoContext, contextCancle := context.WithTimeout(context.Background(), 10*time.Second)
defer contextCancle()
mongoClient, err := mongo.Connect(mongoContext, mongoOption)
if err != nil {
	log.Fatal(err)
}
mgop := mongoClient.Database("RGGMA").Collection("Company")
```

### Operators
- $eq: Matches values that are equal to a specified value.
- $ne: Matches all values that are not equal to a specified value.
- $gt, $gte, $lt, $lte: Greater than, greater than or equal to, less than, less than or equal to, respectively.
- $in: Matches any of the values specified in an array.
- $nin: Matches none of the values specified in an array.
----
- $set: Sets the value of a field in a document.
- $unset: Removes the specified field from a document.
- $inc: Increments the value of the field by a specified amount.
- $push: Adds an element to an array.
- $addToSet: Adds elements to an array only if they do not already exist.
- $pull: Removes all instances of a value from an array.
- $rename: Renames a field.
- $bit: Performs bitwise AND, OR, or XOR updates on integer fields.
----
- $unwind: Deconstructs an array field from the input documents to output a document for each element.
- $match: Filters documents.
- $group: Groups documents by a specified identifier.
- $project: Reshapes documents by including, excluding, or transforming fields.
- $sort: Orders documents.
- $limit: Limits the number of documents.
- $skip: Skips a specified number of documents.
----
- $elemMatch: Matches documents that contain an array field with at least one element that matches all the specified query criteria.
- $all: Matches arrays that contain all elements specified in the query.
- $size: Matches arrays with a specific number of elements.
----
- $text: Performs text search.
----
- $lookup stage in MongoDB aggregation allows you to perform a left outer join to retrieve documents from another collection and include them in your result set.

#### Indexing
MongoDB supports various types of indexes to accommodate different query patterns and optimize performance:

- Single Field Index: Indexes on a single field of a document.

- Compound Index: Indexes on multiple fields within a document.

- Multikey Index: Indexes on arrays of values (each value in the array is indexed).

- Text Index: Special index type for performing full-text searches on string content.

- Geospatial Index: Indexes for geospatial data, supporting queries that calculate geometries based on proximity.

- Hashed Index: Indexes where MongoDB hashes the indexed field's values, typically used for sharding.

- Partial Indexing : Partial indexing involves creating an index on documents that satisfy a filter expression. MongoDB indexes only those documents that match the filter, ignoring documents that do not meet the criteria.  Benefits of Partial Indexing: Reduced Index Size: By indexing only a subset of documents, you reduce the index size compared to indexing the entire collection.Improved Performance: Queries that match the indexed subset of documents can benefit from faster query execution because MongoDB only needs to scan the indexed subset. 
Let's consider an example where we have a collection of books, and we want to create a partial index on books that are currently available (not sold out):

Considerations for Effective Indexing

- Index Selectivity: Ensure indexes are selective enough to reduce the number of documents MongoDB needs to scan.

- Index Size: Consider the size of indexes and their impact on disk usage and memory.

- Index Usage: Monitor index usage and adjust as necessary based on query patterns.

```bash 
db.collection.createIndex({ fieldName: 1 }); 
db.collection.createIndex({ field1: 1, field2: -1 });
db.books.createIndex(
  { price: 1 }, # specifies that we are creating an ascending index on the price field.
  { partialFilterExpression: { available: true } } # specifies that the index should only include documents where the available field is true.
);
```
```bash
# This query can utilize our partial index on price where available is true to efficiently retrieve and sort books that are currently available.
db.books.find({ available: true }).sort({ price: 1 });
```

#### $text
- Text search is case insensitive by default.
- Text search is also diacritic insensitive (e.g., "café" would match "cafe").
- MongoDB calculates a relevance score (score) for each document based on the frequency and proximity of the search terms in the indexed fields.
```json
// Example Document
{
  "_id": ObjectId("60d02e9c25c156ae22df2b73"),
  "title": "The Catcher in the Rye",
  "author": "J.D. Salinger",
  "genre": "Fiction",
  "summary": "The Catcher in the Rye is a novel by J. D. Salinger, partially published in serial form in 1945–1946 and as a novel in 1951. It was originally intended for adults but is often read by adolescents for its themes of angst and alienation, and as a critique on superficiality in society."
}
```
```bash
# Text searches in MongoDB require a text index on the field(s) you want to search.
db.books.createIndex({ summary: "text" });
```

```bash
db.books.find(
  { $text: { $search: "adolescents" } },
  { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } });
```
This query searches for documents in the books collection where the summary field contains the word "adolescents". The $text operator specifies that this is a text search, and $search is used to specify the search string. This query not only finds documents matching "adolescents" but also sorts them by the relevance score in descending order.


#### Updates in Documents:
```json
// Document Structure
{
  "_id": ObjectId("60d5b4d28b7f9f12a46b587c"),
  "username": "johndoe",
  "email": "johndoe@example.com",
  "age": 30,
  "address": {
    "city": "New York",
    "state": "NY",
    "zipcode": "10001"
  },
  "skills": ["JavaScript", "MongoDB", "Node.js"]
}
```
To update the age field of a user document with a specific _id:
```bash
db.users.updateOne(
  { _id: ObjectId("60d5b4d28b7f9f12a46b587c") },
  { $set: { age: 31 } }
);
```
To update the zipcode in the address subdocument:
```bash
db.users.updateOne(
  { _id: ObjectId("60d5b4d28b7f9f12a46b587c") },
  { $set: { "address.zipcode": "10002" } }
);

```
To add a new skill to the skills array:
```bash
db.users.updateOne(
  { _id: ObjectId("60d5b4d28b7f9f12a46b587c") },
  { $push: { skills: "React" } }
);
```
To update multiple documents that match a specific condition:
```bash
db.users.updateMany(
  { age: { $gte: 25 } }, // Update all users where age is 25 or older
  { $set: { status: "active" } } // Set the status field to "active"
);
```
Let's say you want to increment the age field of the user document with _id 
```bash
db.users.updateOne(
  { _id: ObjectId("60d5b4d28b7f9f12a46b587c") },
  { $inc: { age: 1 } }
);
```
Let's say you want to add a skill to the skills array of the user document with _id 

```bash
db.users.updateOne(
  { _id: ObjectId("60d5b4d28b7f9f12a46b587c") },
  { 
    $addToSet: { skills: "React" }
  }
);
# If the skills array already contains "React", MongoDB will not add it again. If "React" is not already in the array, MongoDB will add it.
```

You can combine multiple conditions and update operators to perform more complex updates:
```bash 
# This example updates documents matching both conditions: age greater than or equal to 30 and living in New York City. It sets the status to "premium", updates the zipcode to "10003", and adds "GraphQL" to the skills array.
db.users.updateMany(
  { age: { $gte: 30 }, "address.city": "New York" },
  { $set: { status: "premium", "address.zipcode": "10003" }, $push: { skills: "GraphQL" } }
);
```
Let's say we want to find users who are either from New York ("NY") or are aged 30 or older:
```bash
# $or operator is used to match documents where either "address.state" is "NY" or age is greater than or equal to 30.
db.users.find({
  $or: [
    { "address.state": "NY" },
    { age: { $gte: 30 } }
  ]
});
```
Suppose we want to find users who have the skill "MongoDB" and are either from "NY" or "CA":
```bash 
# "skills": "MongoDB" matches documents where the skills array contains "MongoDB".
# $or operator is used to match documents where "address.state" is either "NY" or "CA".
db.users.find({
  "skills": "MongoDB",
  $or: [
    { "address.state": "NY" },
    { "address.state": "CA" }
  ]
});
```
Let's find users whose skills array contains both "JavaScript" and "Node.js":
```bash 
# $all operator is used to match documents where the skills array contains all specified elements ("JavaScript" and "Node.js").
db.users.find({
  skills: { $all: ["JavaScript", "Node.js"] }
});
```
Suppose we want to find users whose email addresses end with ".com":

Regular expression /.*\.com$/ is used to match documents where email ends with ".com".
```bash 
db.users.find({
  email: /.*\.com$/
});
```
Let's find users aged between 25 and 35, and project only their username, email, and age, sorted by age in descending order:
```bash 
db.users.find({
  age: { $gte: 25, $lte: 35 } # age: { $gte: 25, $lte: 35 } filters documents where age is between 25 and 35.
},
# { username: 1, email: 1, age: 1, _id: 0 } specifies to include username, email, and age fields in the result (_id is excluded).
{
  username: 1,
  email: 1,
  age: 1,
  _id: 0
}).sort({ age: -1 }); # .sort({ age: -1 }) sorts the results by age in descending order (-1 for descending, 1 for ascending).
```
Let's find users who are not from "CA":
```bash 
# $not operator negates the condition, matching documents where "address.state" is not equal ($ne) to "CA".
db.users.find({
  "address.state": { $not: { $eq: "CA" } }
});
```
Suppose we have documents with an array of embedded documents (orders), and we want to find users whose orders include items with quantity greater than 10:

```bash 
# $elemMatch is used to match documents where at least one element in the orders array matches the specified condition (items array contains an element where quantity is greater than 10).
db.users.find({
  orders: {
    $elemMatch: {
      items: { $elemMatch: { quantity: { $gt: 10 } } }
    }
  }
});
```

Let's find users whose age is within a certain range and aggregate the count of such users by their address.state:
```bash
db.users.aggregate([
  {
    $match: { # $match stage filters documents where age is between 25 and 35.
      age: { $gte: 25, $lte: 35 }
    }
  },
  {
    $group: { # $group stage groups documents by address.state (_id) and calculates the count of matching documents ($sum: 1).
      _id: "$address.state",
      count: { $sum: 1 }
    }
  }
]);
```


#### $elemMatch v/s . operator:

```json
[
  {
    "_id": "6671d6fa65fcd889d2dc222b",
    "name": "Company 3",
    "employee": [
      { "name": "Employee 0", "age": 26, "is_active": true, "roles": ["IT", "Finance", "Law"] },
      {  "name": "Employee 40", "age": 29, "is_active": true, "roles": ["Sell"] }
    ]
  },
  {
    "_id": "6671d6fa65fcd889d2dc2255",
    "name": "Company 5",
    "employee": [
      { "name": "Employee 8", "age": 38, "is_active": true, "roles": [] },
      {  "name": "Employee 11", "age": 26, "is_active": true, "roles": ["Law"] }
    ]
  },
  {
    "_id": "6671d6fa65fcd889d2dc222a",
    "name": "Company 4",
    "employee": [
      {  "name": "Employee 3", "age": 36, "is_active": true, "roles": ["IT", "Finance", "Law"] },
      {  "name": "Employee 80", "age": 41, "is_active": true, "roles": ["Finance"] },
      {  "name": "Employee 20", "age": 37, "is_active": false, "roles": ["Finance"] }
    ]
  }
]
```
```bash
db.Company.find({"employee.age":{$gt:30},"employee.is_active":true, "employee.roles":{$in:["Law"]}});
# Result: Company 5,Company 4 
# Why: Employee 8 has age 38 and role nil. but Employee 11 has age 26 and role Law. This document satisfy two condition with two different array element.

db.Company.find({"employee":{$elemMatch:{"age":{$gt:30},"is_active":true,"roles":{$in:["Law"]}}}});
# Result: Company 4 
# Why: Employee 8 has age 38 and role nil. but Employee 11 has age 26 and role Law. This document does not satisfy two condition with same array element. So Company 5 not satisfied the Criteria.

```
The dot operator allows us to get a result where each of the two arrays in a document meets one condition individually, but neither array meets all conditions on its own.


#### $lookup
$lookup is use to Perform Left Outer Join to another Collection in Same DB,based on some condition. $lookup resource intensive so Proper Indexing is must for Large DataSet.
We have 2 Collections one is Order another is Products. Order collection has field product_id which is _id of Product collection.
```json
// Orders Collection:
{ "_id": 1, "order_id": "A001", "product_id": 101, "quantity": 2 }
{ "_id": 2, "order_id": "A002", "product_id": 102, "quantity": 1 }
```

```json
// Products Collection:
{ "_id": 101, "name": "Laptop", "price": 1200 }
{ "_id": 102, "name": "Mouse", "price": 30 }
```
```bash
db.orders.aggregate([
  {
    $lookup: {
      from: "products",
      localField: "product_id",
      foreignField: "_id",
      as: "product_details"
    }
  }
]);
```
```json
[
  {
    "_id": 1,
    "order_id": "A001",
    "product_id": 101,
    "quantity": 2,
    "product_details": [
      { "_id": 101, "name": "Laptop", "price": 1200 }
    ]
  },
  {
    "_id": 2,
    "order_id": "A002",
    "product_id": 102,
    "quantity": 1,
    "product_details": [
      { "_id": 102, "name": "Mouse", "price": 30 }
    ]
  }
]
```

#### $Unwind:
Imagine you have a collection where each document contains an array field.
When you apply $unwind to an array field in a document, MongoDB will create separate documents where Each new document will have the same values for all other fields except the one array field, for each element of the array.
```json
{
  "_id": 1,
  "name": "Product ABC",
  "tags": ["electronics", "smartphone", "tech"]
}
```
```bash 
db.products.aggregate([
  { $unwind: "$tags" }
]);
```
```json
{ "_id": 1, "name": "Product ABC", "tags": "electronics" }
{ "_id": 1, "name": "Product ABC", "tags": "smartphone" }
{ "_id": 1, "name": "Product ABC", "tags": "tech" }
```

#### $group:
```json
[
  { "_id": 1, "item": "book", "price": 10, "quantity": 2, "date": ISODate("2024-06-20T08:00:00Z") },
  { "_id": 2, "item": "pen", "price": 5, "quantity": 5, "date": ISODate("2024-06-20T10:00:00Z") },
  { "_id": 3, "item": "book", "price": 10, "quantity": 1, "date": ISODate("2024-06-21T09:00:00Z") },
  { "_id": 4, "item": "eraser", "price": 2, "quantity": 3, "date": ISODate("2024-06-21T11:00:00Z") }
]
```

```bash
db.sales.aggregate([
  {
    $group: {
      _id: "$item",  // Group by the "item" field
      totalSales: { $sum: { $multiply: ["$price", "$quantity"] } }  // Calculate total sales for each item
    }
  }
]);
```
```json
[
  { "_id": "book", "totalSales": 30 },
  { "_id": "pen", "totalSales": 25 },
  { "_id": "eraser", "totalSales": 6 }
]
```

### Aggregation
Aggregation operations are constructed using a sequence of stages, where each stage transforms the documents as they pass through the pipeline. Each stage performs a specific operation on the data. Common stages include $match, $group, $project, $sort, $limit, and $lookup. Aggregation pipeline stages use operators to specify transformations or computations.
```json
{
  "_id": 1,
  "item": "book",
  "price": 10,
  "quantity": 2,
  "date": ISODate("2024-06-20T08:00:00Z"),
  "tags": ["school", "notebook"]
}
{
  "_id": 2,
  "item": "pen",
  "price": 5,
  "quantity": 5,
  "date": ISODate("2024-06-20T10:00:00Z"),
  "tags": ["school", "stationery"]
}
```
```bash
db.sales.aggregate([
  { $match: { tags: "school" } }, #  Filters documents to pass only those that match the specified condition.
  { $group: { _id: "$item", totalAmount: { $sum: { $multiply: ["$price", "$quantity"] } } } }, #  Groups documents by a specified identifier expression and applies accumulator expressions. Operators like $sum, $multiply, etc., perform specific computations.
  { $sort: { totalAmount: -1 } }, #  Orders the documents.
  { $limit: 3 } # Limits the number of documents passed to the next stage.
]);
```
```go
// Golang Mongo Driver Example
filter4 := bson.D{{Key: "$match", Value: bson.D{{Key: "employee.is_active", Value: true}}}}
filter5 := bson.D{{Key: "$count", Value: "Active User Count"}}
pipe := mongo.Pipeline{filter4, filter5}
cur, err := mgop.Aggregate(mongoContext, pipe) 

if err != nil {
	log.Fatal(err)
}
var res []bson.D
err = cur.All(mongoContext, &res)
if err != nil {
	log.Fatal(err)
}
for _, r := range res {
	fmt.Println(r)
}
```
### Describe how MongoDB achieves horizontal scalability and high availability in its architecture. 

- MongoDB uses sharding to horizontally partition data across multiple machines or nodes called shards. 
- Each shard contains a subset of the data, distributed based on a shard key. This allows MongoDB to distribute read and write operations across shards.
- Sharding enables MongoDB to handle large volumes of data and high throughput by scaling out horizontally.
- The shard key determines how data is distributed across shards. It’s a crucial design decision that impacts data distribution and query performance.
- When a client application sends a query to MongoDB, the query is routed through the mongos instance. The mongos examines the query to determine which shard(s) contain the relevant data based on the shard key.
- mongos instances also manage load balancing across the shards. They distribute incoming queries evenly across shards to ensure that no individual shard becomes overwhelmed with requests, thereby optimizing performance.
- Adding more mongos instances can improve the throughput and scalability of a MongoDB deployment, as they handle query routing and load balancing.
- MongoDB uses config servers to store metadata about the sharded cluster, including the mapping between shards and ranges of shard keys.
- Config servers provide configuration and coordination services, allowing MongoDB routers (mongos instances) to direct queries to the appropriate shards based on the shard key.
- MongoDB uses replica sets to provide redundancy and automatic failover.
- Each replica set consists of multiple nodes (typically three or more): one primary node for read and write operations and secondary nodes that replicate data from the primary. If the primary node fails, a new primary is elected from the remaining nodes in the replica set, ensuring continuous availability.
- MongoDB’s oplog is a capped collection that records all write operations (inserts, updates, deletes) in the order they occur.
- MongoDB replica sets support automatic failover. If the primary node becomes unavailable, a secondary node is automatically promoted to primary.
- Clients can continue to read and write data from the new primary node without interruption, ensuring high availability and reliability.

---------------------------------------
---------------------------------------

## Redis Notes

REmote DIctionary Server - Redis primarily stores data in RAM, which allows it to achieve exceptionally fast read and write operations. This makes it suitable for applications requiring low latency and high throughput. Redis supports optional persistence by periodically dumping the dataset to disk or by appending each command to a log. 

Redis supports transactions, grouping multiple commands into a single atomic operation. Redis Cluster provides automatic partitioning of data across multiple Redis nodes for scalability and high availability. 

Redis is commonly used as a caching layer to store frequently accessed data, reducing latency and load on backend databases. Storing session data for web applications, ensuring fast access and scalability. 

Redis' fast read and write speeds make it suitable for real-time analytics, counting, and metrics. As a message broker, Redis facilitates communication between different parts of an application using its pub/sub capabilities. 
 
Redis has a maximum allowed message size (512 MB).

Redis internally uses hash tables to store hashes. Hash tables allow for fast lookups, inserts, and updates under normal circumstances.

When you execute HGET key field, Redis computes the hash of the key to locate the hash table where the hash is stored. Then, it directly accesses the field within that hash table using the field name.

The O(1) time complexity of HGET remains consistent even as the size of the hash (number of fields) grows. This scalability makes Redis well-suited for scenarios where fast access to individual hash fields is crucial, such as caching, session management, and storing structured data.

```bash
docker run -d --name redis-server -p 6379:6379 redis
```
```go
// Initialize Redis client
ctx := context.Background()
redisClient := redis.NewClient(&redis.Options{Addr: "localhost:6379", Password: "", DB: 0})

// Add User To Redis
userJson, err := json.Marshal(user)
	if err != nil {
		return
	}
	err = repo.redisClient.Set(ctx, user.ID, string(userJson), 0).Err()
	if err != nil {
		return
	}
// Get User From Redis
res, err := repo.redisClient.Get(ctx, id).Result()
	if err != nil {
		if err == redis.Nil {
			err = domain.ErrNoDocumentFound
		}
		return
	}
	err = json.Unmarshal([]byte(res), &user)

// PUB-SUB SERVER:
	// Publish messages to "notifications" channel every second
	ticker := time.NewTicker(1 * time.Second)
	defer ticker.Stop()

	for {
		select {
		case <-ctx.Done():
			return
		case <-ticker.C:
			err := redisClient.Publish(ctx, "notifications", "Hello from publisher").Err()
			if err != nil {
				return
			}
		}
	}
// PUB-SUB CLIENT:
	// Subscribe to "notifications" channel
	pubsub := redisClient.Subscribe(ctx, "notifications")
	defer pubsub.Close()

	// Receive messages in a separate goroutine
	go func() {
		for {
			msg, err := pubsub.ReceiveMessage(ctx)
			if err != nil {
			return
		}
	}()
}
```
```bash
# HSET provides efficient O(1) time complexity for setting or updating individual fields.
HSET user:1001 username alice email alice@example.com
# user:1001 is the key of the hash.
# username is the field within the hash.
# alice is the value that will be stored at the username field.
# email is another field within the same hash.
# alice@example.com is the value that will be stored at the email field.

# If the field (field) does not exist in the hash, it is created and set to the provided value.
# If the field already exists in the hash, its value is overwritten with the new value.

# HSET returns 1 if the field is a new field in the hash and the value was set, or 0 if the field already existed and the value was updated.
# Using hashes allows efficient updates of individual fields without having to replace the entire object.

HGET user:1001 username
# If the field does not exist in the hash or if the key does not exist at all, HGET returns nil.
# Using HGET allows you to fetch individual fields efficiently without needing to retrieve the entire hash, which is particularly beneficial when dealing with large hashes.
# HGET command has a time complexity of O(1)
```
Redis provides two main mechanisms for persistence: 

RDB (Redis Database File) snapshots : Periodically saves the dataset to disk as a binary file (dump.rdb).

AOF (Append-Only File) logs : Logs every write operation received by the server, which can be replayed to rebuild the dataset.


To enable persistence in Redis, modify the redis.conf file 
```bash
# RDB
save 900 1     # Save the dataset to disk if at least 1 key changed and 900 seconds (15 minutes) elapsed
save 300 10    # Save the dataset to disk if at least 10 keys changed and 300 seconds (5 minutes) elapsed
save 60 10000  # Save the dataset to disk if at least 10000 keys changed and 60 seconds elapsed.

# AOF
appendonly yes        # Enable the AOF persistence
appendfilename "appendonly.aof"  # Specify the AOF file name
appendfsync everysec  # Fsync every second (you can also use 'always' for every write, or 'no' for disabling)

```

Synchronous operations like SAVE can block Redis for the duration of the save operation, impacting performance temporarily. BGSAVE and BGREWRITEAOF are preferred for production environments.
Changes made to redis.conf require a Redis server restart to take effect, but many configuration options can be adjusted at runtime using CONFIG SET.

## PostgreSQL Notes

PostgreSQL: Released under the PostgreSQL License, which is a open-source license. It Scales well vertically and horizontally with support for partitioning and table inheritance. Supports a wider range of data types including custom types and arrays. Known for handling complex queries and large volumes of data efficiently. Optimized for read-heavy workloads.

```go

package repository

import (
    "database/sql"
    "log"

    _ "github.com/lib/pq"
)

type UserRepository struct {
    db *sql.DB
}

func NewUserRepository(db *sql.DB) *UserRepository {
    return &UserRepository{db}
}

// Example function to fetch a user by ID
func (ur *UserRepository) GetUserByID(id int) (string, error) {
    var name string
    err := ur.db.QueryRow("SELECT name FROM users WHERE id = $1", id).Scan(&name)
    if err != nil {
        log.Println("Error querying user:", err)
        return "", err
    }
    return name, nil
}

// Example function to create a new user
func (ur *UserRepository) CreateUser(name string, age int) error {
    _, err := ur.db.Exec("INSERT INTO users (name, age) VALUES ($1, $2)", name, age)
    if err != nil {
        log.Println("Error inserting user:", err)
        return err
    }
    return nil
}

// Example function to update user details
func (ur *UserRepository) UpdateUser(id int, name string, age int) error {
    _, err := ur.db.Exec("UPDATE users SET name = $1, age = $2 WHERE id = $3", name, age, id)
    if err != nil {
        log.Println("Error updating user:", err)
        return err
    }
    return nil
}

// Example function to delete a user
func (ur *UserRepository) DeleteUser(id int) error {
    _, err := ur.db.Exec("DELETE FROM users WHERE id = $1", id)
    if err != nil {
        log.Println("Error deleting user:", err)
        return err
    }
    return nil
}
```

```go
package main

import (
    "database/sql"
    "fmt"
    "log"

    _ "github.com/lib/pq"
    "github.com/yourusername/project/repository"
)

const (
    host     = "localhost"
    port     = 5432
    user     = "your_username"
    password = "your_password"
    dbname   = "your_database"
)

func main() {
    // Construct connection string
    connStr := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable",
        host, port, user, password, dbname)

    // Open database connection
    db, err := sql.Open("postgres", connStr)
    if err != nil {
        log.Fatal("Error connecting to the database: ", err)
    }
    defer db.Close()

    // Verify the connection
    err = db.Ping()
    if err != nil {
        log.Fatal("Error verifying database connection: ", err)
    }
    fmt.Println("Successfully connected to the database!")

    // Initialize repository
    userRepository := repository.NewUserRepository(db)

    // Example usage of repository methods
    // Create a user
    err = userRepository.CreateUser("John Doe", 30)
    if err != nil {
        log.Fatal("Error creating user: ", err)
    }

    // Get a user by ID
    userName, err := userRepository.GetUserByID(1)
    if err != nil {
        log.Fatal("Error fetching user: ", err)
    }
    fmt.Println("User name:", userName)

    // Update a user
    err = userRepository.UpdateUser(1, "Jane Smith", 35)
    if err != nil {
        log.Fatal("Error updating user: ", err)
    }

    // Delete a user
    err = userRepository.DeleteUser(1)
    if err != nil {
        log.Fatal("Error deleting user: ", err)
    }

    fmt.Println("CRUD operations completed successfully!")
}
```
Creating the Employee Table

```bash
CREATE TABLE employee (
    emp_id SERIAL PRIMARY KEY,
    emp_name VARCHAR(100) NOT NULL,
    emp_salary NUMERIC(10, 2),
    emp_dept VARCHAR(50)
);
```

```bash
INSERT INTO employee (emp_name, emp_salary, emp_dept) VALUES
('John Doe', 50000.00, 'HR'),
('Jane Smith', 60000.00, 'IT'),
('Michael Johnson', 55000.00, 'Sales'),
('Emily Davis', 52000.00, 'IT'),
('William Brown', 48000.00, 'HR');
```

```bash
# Select all employees:
SELECT * FROM employee;
```

```bash
# Select employees in a specific department
SELECT * FROM employee 
WHERE emp_dept = 'IT';
```

```bash
# Select employees earning more than a certain amount
SELECT * FROM employee 
WHERE emp_salary > 55000;
```

```bash
# Select employees ordered by salary in descending order
SELECT * FROM employee 
ORDER BY emp_salary DESC;
```

```bash
# Count the number of employees in each department
SELECT emp_dept, COUNT(*) AS num_employees
FROM employee
GROUP BY emp_dept;
```

```bash
# Update an employee's salary
UPDATE employee SET emp_salary = emp_salary * 1.1
WHERE emp_name = 'John Doe';
```

```bash
# Delete an employee
DELETE FROM employee 
WHERE emp_name = 'William Brown';
```

```bash
# Find the highest salary among employees
SELECT MAX(emp_salary) AS max_salary FROM employee;
```

```bash
# Calculate the average salary of employees
SELECT AVG(emp_salary) AS avg_salary FROM employee;
```

```bash
# Calculate the total salary expense per department
SELECT emp_dept, SUM(emp_salary) AS total_salary_expense
FROM employee
GROUP BY emp_dept;
```

```bash
# Find employees whose salaries are in the top 10%
SELECT emp_name, emp_salary
FROM (
    SELECT emp_name, emp_salary,
           NTILE(10) OVER (ORDER BY emp_salary DESC) AS salary_percentile
    FROM employee
) top_10_percent
WHERE salary_percentile = 1;
```

```bash
# Find the department with the highest average salary
SELECT emp_dept, AVG(emp_salary) AS avg_salary
FROM employee
GROUP BY emp_dept
ORDER BY avg_salary DESC
LIMIT 1;
```

```bash
# Select employees who have a salary greater than the average salary of their department
SELECT emp_name, emp_salary, emp_dept
FROM employee e
WHERE emp_salary > (
    SELECT AVG(emp_salary)
    FROM employee
    WHERE emp_dept = e.emp_dept
);
```
 Create Another Table employee_details
```bash
CREATE TABLE employee_details (
    emp_id SERIAL PRIMARY KEY,
    emp_name VARCHAR(100) NOT NULL,
    emp_address VARCHAR(255),
    emp_phone VARCHAR(20),
    emp_email VARCHAR(100)
);
```

```bash
INSERT INTO employee_details (emp_name, emp_address, emp_phone, emp_email) VALUES
('John Doe', '123 Main St, Anytown, USA', '123-456-7890', 'john.doe@example.com'),
('Jane Smith', '456 Oak Ave, Sometown, USA', '987-654-3210', 'jane.smith@example.com'),
('Michael Johnson', '789 Elm Blvd, Othercity, USA', '456-789-0123', 'michael.johnson@example.com'),
('Emily Davis', '321 Pine Dr, Yourtown, USA', '789-012-3456', 'emily.davis@example.com'),
('William Brown', '567 Maple Ln, Smalltown, USA', '234-567-8901', 'william.brown@example.com');
```


```bash
# Select all employees along with their details
SELECT e.emp_id, e.emp_name, e.emp_salary, d.emp_address, d.emp_phone, d.emp_email
FROM employee e
JOIN employee_details d ON e.emp_id = d.emp_id;
```

```bash
# Inner Join: Returns rows when there is a match in both tables.

SELECT e.emp_id, e.emp_name, e.emp_salary, d.emp_address, d.emp_phone
FROM employee e
INNER JOIN employee_details d ON e.emp_id = d.emp_id;
```

```bash
# Left Join (or Left Outer Join): Returns all rows from the left table (employee), and the matched rows from the right table (employee_details). If there is no match, NULL values are returned for the right side.

SELECT e.emp_name, e.emp_dept, d.emp_address, d.emp_phone
FROM employee e
LEFT JOIN employee_details d ON e.emp_id = d.emp_id;
```

```bash
# Right Join (or Right Outer Join): Returns all rows from the right table (employee_details), and the matched rows from the left table (employee). If there is no match, NULL values are returned for the left side.

SELECT e.emp_name, e.emp_dept, d.emp_address, d.emp_phone
FROM employee_details d
RIGHT JOIN employee e ON e.emp_id = d.emp_id;
```

```bash
# Full Join (or Full Outer Join): Returns all rows when there is a match in either table. If there is no match, NULL values are returned for missing side.

SELECT e.emp_name, e.emp_dept, d.emp_address, d.emp_phone
FROM employee e
FULL JOIN employee_details d ON e.emp_id = d.emp_id;
```

```bash
# Self Join: Joining a table with itself, typically to find related records within the same table.

SELECT e1.emp_name AS employee_name, e2.emp_name AS manager_name
FROM employee e1
LEFT JOIN employee e2 ON e1.manager_id = e2.emp_id;
```

```bash
# Cross Join: Cartesian join that returns the Cartesian product of the two tables. It matches each row from the first table with each row from the second table.

SELECT e.emp_name, d.emp_address
FROM employee e
CROSS JOIN employee_details d;
```


```bash
# Select employees in a specific department along with their details
SELECT e.emp_id, e.emp_name, e.emp_salary, d.emp_address, d.emp_phone, d.emp_email
FROM employee e
JOIN employee_details d ON e.emp_id = d.emp_id
WHERE e.emp_dept = 'IT';
```

```bash
# Update an employee's details
UPDATE employee_details
SET emp_address = 'New Address', emp_phone = '999-888-7777'
WHERE emp_id = (SELECT emp_id FROM employee WHERE emp_name = 'John Doe');
```

```bash
# Delete an employee's details
UPDATE employee_details
SET emp_email = NULL
WHERE emp_id = (SELECT emp_id FROM employee WHERE emp_name = 'William Brown');
```

## GRPC 

gRPC (gRPC Remote Procedure Calls) is an open-source framework developed by Google that enables efficient and scalable communication between distributed systems.

gRPC supports Transport Layer Security (TLS) by default, providing secure communication between clients and servers. It also supports authentication mechanisms like OAuth, JWT, etc., for securing RPC endpoints.

gRPC uses HTTP/2 as its underlying protocol for communication. This results in lower latency and reduced overhead compared to HTTP/1.x.

It uses Protocol Buffers (protobuf) as its interface definition language and allows you to define both the service interface and the data types in a single .proto file. 

Protocol Buffers serialize structured data into a binary format, which is more compact and efficient than JSON or XML. This helps in reducing the size of the payload transmitted over the network, improving performance.

gRPC supports two types of RPCs: unary (single request and response) and streaming (multiple messages over a single connection). Streaming can be unary-to-unary, unary-to-streaming, streaming-to-unary, or streaming-to-streaming.

Unary RPC: The simplest form where the client sends a single request to the server and gets back a single response. It’s similar to traditional synchronous function calls.

Server Streaming RPC: The client sends a request and the server responds with a stream of messages. The client reads these messages until the server completes sending them.

Client Streaming RPC: The client sends a stream of messages to the server, and then the server responds with a single message.

Bidirectional Streaming RPC: Both the client and server send a stream of messages independently. The streams operate independently, allowing for full-duplex communication.

### Exmple of Grpc with Server-side-streaming and Client-side streaming

Create a user.proto file that defines the message types, RPC methods and gRPC service that can be called remotely:

```proto
// user.proto

syntax = "proto3";

package user;

service UserService {
    rpc CreateUser(CreateUserRequest) returns (UserResponse);
    rpc GetUser(GetUserRequest) returns (UserResponse);
    rpc UpdateUser(UpdateUserRequest) returns (UserResponse);
    rpc DeleteUser(DeleteUserRequest) returns (DeleteUserResponse);

    rpc StreamUsers(StreamUsersRequest) returns (stream UserResponse);  // Server-side-streaming
    rpc RecordUserActivity(stream UserActivityRequest) returns (UserActivitySummary); // Client-side streaming
}

message User {
    int32 id = 1;
    string name = 2;
    int32 age = 3;
}

message CreateUserRequest {
    string name = 1;
    int32 age = 2;
}

message GetUserRequest {
    int32 id = 1;
}

message UpdateUserRequest {
    int32 id = 1;
    string name = 2;
    int32 age = 3;
}

message DeleteUserRequest {
    int32 id = 1;
}

message DeleteUserResponse {
    bool success = 1;
}

message UserResponse {
    User user = 1;
}

message StreamUsersRequest {
    int32 max_users = 1;
}

message UserActivityRequest {
    int32 user_id = 1;
    string activity = 2;
}

message UserActivitySummary {
    int32 total_activities = 1;
}
```
Generate Go Code from .proto File
```bash
# This will generate user.pb.go and user_grpc.pb.go files.
protoc user.proto --go_out=plugins=grpc:.
```
```go
// server.go

package main

import (
    "context"
    "log"
    "net"
    "strconv"
    "time"

    pb "your_module_path/user" // Update with your actual module path
    "google.golang.org/grpc"
)

type userServiceServer struct {
    users []*pb.User // Mimic DB
}

func (s *userServiceServer) CreateUser(ctx context.Context, req *pb.CreateUserRequest) (*pb.UserResponse, error) {
    user := &pb.User{
        Id:   int32(len(s.users) + 1), // Generate ID (simple increment for example)
        Name: req.Name,
        Age:  req.Age,
    }
    s.users = append(s.users, user)
    return &pb.UserResponse{User: user}, nil
}

func (s *userServiceServer) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.UserResponse, error) {
    for _, user := range s.users {
        if user.Id == req.Id {
            return &pb.UserResponse{User: user}, nil
        }
    }
    return nil, grpc.Errorf(grpc.CodeNotFound, "User not found")
}

func (s *userServiceServer) UpdateUser(ctx context.Context, req *pb.UpdateUserRequest) (*pb.UserResponse, error) {
    for _, user := range s.users {
        if user.Id == req.Id {
            user.Name = req.Name
            user.Age = req.Age
            return &pb.UserResponse{User: user}, nil
        }
    }
    return nil, grpc.Errorf(grpc.CodeNotFound, "User not found")
}

func (s *userServiceServer) DeleteUser(ctx context.Context, req *pb.DeleteUserRequest) (*pb.DeleteUserResponse, error) {
    for i, user := range s.users {
        if user.Id == req.Id {
            s.users = append(s.users[:i], s.users[i+1:]...)
            return &pb.DeleteUserResponse{Success: true}, nil
        }
    }
    return nil, grpc.Errorf(grpc.CodeNotFound, "User not found")
}

func (s *userServiceServer) StreamUsers(req *pb.StreamUsersRequest, stream pb.UserService_StreamUsersServer) error {
    maxUsers := req.MaxUsers
    for _, user := range s.users {
        if maxUsers <= 0 {
            break
        }
        if err := stream.Send(&pb.UserResponse{User: user}); err != nil {
            return err
        }
        maxUsers--
    }
    return nil
}

func (s *userServiceServer) RecordUserActivity(stream pb.UserService_RecordUserActivityServer) error {
    var totalActivities int32
    for {
        req, err := stream.Recv()
        if err != nil {
            return err
        }
        log.Printf("Received activity for user ID %d: %s\n", req.UserId, req.Activity)
        totalActivities++
        if err := stream.SendAndClose(&pb.UserActivitySummary{TotalActivities: totalActivities}); err != nil {
            return err
        }
    }
}

func main() {
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("Failed to listen: %v", err)
    }
    s := grpc.NewServer()
    pb.RegisterUserServiceServer(s, &userServiceServer{})
    log.Println("gRPC server running on port 50051")
    if err := s.Serve(lis); err != nil {
        log.Fatalf("Failed to serve: %v", err)
    }
}
```

```go
// client.go

package main

import (
    "context"
    "log"
    "os"
    "strconv"
    "time"

    pb "your_module_path/user" // Update with your actual module path
    "google.golang.org/grpc"
)

func main() {
    conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure())
    if err != nil {
        log.Fatalf("Failed to connect: %v", err)
    }
    defer conn.Close()

    c := pb.NewUserServiceClient(conn)

    // Call CreateUser RPC
    ctx, cancel := context.WithTimeout(context.Background(), time.Second)
    defer cancel()
    createUserResp, err := c.CreateUser(ctx, &pb.CreateUserRequest{Name: "Alice", Age: 30})
    if err != nil {
        log.Fatalf("CreateUser RPC failed: %v", err)
    }
    log.Printf("Created user: %v\n", createUserResp.User)

    // Call GetUser RPC
    getUserResp, err := c.GetUser(ctx, &pb.GetUserRequest{Id: createUserResp.User.Id})
    if err != nil {
        log.Fatalf("GetUser RPC failed: %v", err)
    }
    log.Printf("Got user: %v\n", getUserResp.User)

    // Call UpdateUser RPC
    updateUserResp, err := c.UpdateUser(ctx, &pb.UpdateUserRequest{Id: createUserResp.User.Id, Name: "Alice Smith", Age: 32})
    if err != nil {
        log.Fatalf("UpdateUser RPC failed: %v", err)
    }
    log.Printf("Updated user: %v\n", updateUserResp.User)

    // Call DeleteUser RPC
    deleteUserResp, err := c.DeleteUser(ctx, &pb.DeleteUserRequest{Id: createUserResp.User.Id})
    if err != nil {
        log.Fatalf("DeleteUser RPC failed: %v", err)
    }
    log.Printf("Deleted user success: %v\n", deleteUserResp.Success)

    // Call StreamUsers RPC (Server-side streaming)
    streamUsersResp, err := c.StreamUsers(ctx, &pb.StreamUsersRequest{MaxUsers: 3})
    if err != nil {
        log.Fatalf("StreamUsers RPC failed: %v", err)
    }
    for {
        userResp, err := streamUsersResp.Recv()
        if err != nil {
            break
        }
        log.Printf("Streaming user: %v\n", userResp.User)
    }

    // Call RecordUserActivity RPC (Client-side streaming)
    recordUserActivityStream, err := c.RecordUserActivity(ctx)
    if err != nil {
        log.Fatalf("RecordUserActivity RPC failed: %v", err)
    }
    for i := 1; i <= 5; i++ {
        activity := "Activity " + strconv.Itoa(i)
        err := recordUserActivityStream.Send(&pb.UserActivityRequest{UserId: 1, Activity: activity})
        if err != nil {
            log.Fatalf("Failed to send activity: %v", err)
        }
    }
    summaryResp, err := recordUserActivityStream.CloseAndRecv()
    if err != nil {
        log.Fatalf("Failed to receive summary response: %v", err)
    }
    log.Printf("Recorded user activities. Total activities: %d\n", summaryResp.TotalActivities)
}
```

#### Exmple of Grpc with InterService Comunication

```bash
sudo apt install protobuf-compiler
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest


protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative order.proto
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative user.proto
```

```proto
syntax = "proto3";

package user;

option go_package = "grpc_ultimate/user";

service UserService{
    rpc LoginUser (LoginReq) returns (LoginResp);
    rpc GetUser (UserReq) returns (UserResp);
}

message LoginReq {
    string username =1;
    string pass =2;
}

message LoginResp{
    string token =1;
}

message UserReq{
    string userId = 1;
}

message UserResp{
    string userID = 1;
    string name = 2;
    string email= 3;
}
```

```go
// User/user.go
package user

import (
	"context"
	"fmt"
	"log"
	"time"

	"github.com/dgrijalva/jwt-go"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/metadata"
	"google.golang.org/grpc/status"
)

var key = []byte("t7mkzwoxlihzd8fteABssEVzz9xKiinn51i8u6urmiu=")

type userService struct {
}

func NewUserService() *userService {
	return &userService{}
}

func (svc *userService) LoginUser(ctx context.Context, req *LoginReq) (res *LoginResp, err error) {
	claim := jwt.MapClaims{
		"username": req.Username,
		"exp":      time.Now().Add(time.Hour).Unix(),
	}
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claim)
	signToken, err := token.SignedString(key)
	if err != nil {
		return nil, status.Error(codes.Internal, err)
	}
	res = &LoginResp{Token: signToken}
	return res, nil
}
func (svc *userService) GetUser(ctx context.Context, req *UserReq) (u *UserResp, err error) {
	md, ok := metadata.FromIncomingContext(ctx)
	if !ok {
		return
	}
	tkSTr := md.Get("authorization")[0]
	// The jwt.Parse function is used to parse and validate a JWT (JSON Web Token) string (tkSTr).
	// tkSTr (JWT Token String): This is the JWT token string that you want to parse and validate.
	// The second parameter is a validation function, that jwt.Parse invokes to retrieve the key used to validate the token's signature.
	token, err := jwt.Parse(tkSTr, func(t *jwt.Token) (interface{}, error) {
		return []byte(key), nil // Here, []byte(key) converts the key string into a byte slice ([]byte), which is the expected type for the key interface in the jwt-go library.
	})
	if err != nil {
		return
	}
	if token.Valid {
		u = &UserResp{
			UserID: req.UserId,
			Name:   "Urmi",
			Email:  "urmi@gendi.com",
		}
	} else {
		return nil, status.Error(codes.Unauthenticated, err)
	}
	return
}

func (svc *userService) mustEmbedUnimplementedUserServiceServer() {

}
```

```go
// User/cmd/main.go
package main

import (
	"grpc_ultimate/user"
	"log"
	"net"

	"google.golang.org/grpc"
)

func main() {
	lis, err := net.Listen("tcp", ":7000")
	if err != nil {
		log.Fatal(err)
	}
	userSvc := user.NewUserService()
	grpcServer := grpc.NewServer()
	user.RegisterUserServiceServer(grpcServer, userSvc)
	if err := grpcServer.Serve(lis); err != nil {
		log.Fatal(err)
	}
}
```
```proto
syntax = "proto3";

package order;

option go_package = "grpc_ultimate/order";

service OrderService{
    rpc CreateOrder(OrderRequest) returns (OrderResponse);
}

message OrderRequest{
    string orderId=1;
    string userId=2;
    repeated string items=3;
}

message OrderResponse{
    string orderId=1;
    string userId=2;
    repeated string items=3;
    string message=4;
}
```

```go
// Order/order.go
package order

import (
	"context"
	"errors"
	"grpc_ultimate/user"

	"google.golang.org/grpc/metadata"
)

type orderService struct {
	userClient user.UserServiceClient
	token      string
}

func NewOrderService(userClient user.UserServiceClient, token string) *orderService {
	return &orderService{
		userClient: userClient,
		token:      token,
	}

}
func (svc *orderService) CreateOrder(ctx context.Context, req *OrderRequest) (res *OrderResponse, err error) {
	md := metadata.Pairs(
		"authorization", svc.token,
	)
	// Attach metadata to the outgoing context
	ctx = metadata.NewOutgoingContext(ctx, md)
	user, err := svc.userClient.GetUser(ctx, &user.UserReq{UserId: req.UserId})
	if err != nil {
		return
	}
	if len(user.UserID) > 0 {
		order := OrderResponse{
			OrderId: req.OrderId,
			UserId:  req.UserId,
			Items:   req.Items,
			Message: "Done",
		}
		return &order, nil
	} else {
		return
	}
}

func (svc *orderService) mustEmbedUnimplementedOrderServiceServer() {}
```

```go
// Order/cmd/order.go
package main

import (
	"context"
	"grpc_ultimate/order"
	"grpc_ultimate/user"
	"log"
	"net"

	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
)

func main() {
	conn, err := grpc.NewClient("localhost:7000", grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()
	userClient := user.NewUserServiceClient(conn)
	loginResp, err := userClient.LoginUser(context.Background(), &user.LoginReq{Username: "Urmi", Pass: "9653"})
	if err != nil {
		log.Fatal(err)
	}
	orderSvc := order.NewOrderService(userClient, loginResp.Token)
	// orderSvc.CreateOrder(context.Background(), &order.OrderRequest{OrderId: "67", UserId: "67", Items: []string{"uyu", "fd", "tff"}})
	lis, err := net.Listen("tcp", ":7001")
	if err != nil {
		log.Fatal(err)
	}
	grpcServer := grpc.NewServer()
	order.RegisterOrderServiceServer(grpcServer, orderSvc)
	if err := grpcServer.Serve(lis); err != nil {
		log.Fatal(err)
	}
}
```

## AWS S3 and SNS Operation :

### Upload File to S3 :

```go
package main
import (
    "os"
    "fmt"
    "strings"
    "path/filepath"
    "github.com/aws/aws-sdk-go/aws"
    "github.com/aws/aws-sdk-go/aws/session"
    "github.com/aws/aws-sdk-go/service/s3"
)
func main() {
    bucket := "your-bucket-name"
    region := "your-aws-region"
    sess := session.Must(session.NewSession(&aws.Config{
        Region: aws.String(region),
    }))
    svc := s3.New(sess)
    filePath := "path/to/your/file.txt"
    file, err := os.Open(filePath)
    if err != nil {
        return
    }
    defer file.Close()
    fileInfo, _ := file.Stat()
    size := fileInfo.Size()
    contentType := strings.ToLower(filepath.Ext(file.Name()))
    params := &s3.PutObjectInput{
        Bucket:        aws.String(bucket),
        Key:           aws.String(filepath.Base(filePath)), // Object key (filename)
        Body:          file,
        ContentLength: aws.Int64(size),
        ContentType:   aws.String("text/plain"), // Replace with appropriate content type
    }
    _, err = svc.PutObject(params)
    if err != nil {
        return
    }
}
```
 ### SNS And Download file From S3 :

```go
package main
import (
    "fmt"
    "os"
    "github.com/aws/aws-sdk-go/aws"
    "github.com/aws/aws-sdk-go/aws/session"
    "github.com/aws/aws-sdk-go/service/sns"
    "github.com/aws/aws-sdk-go/service/s3"
    "github.com/aws/aws-sdk-go/aws/awserr"
    "github.com/aws/aws-sdk-go/service/s3/s3manager"
)
func downloadFromS3(bucket, key, filename string) error {
    sess := session.Must(session.NewSession(&aws.Config{
        Region: aws.String("your-aws-region"),
    }))
    downloader := s3manager.NewDownloader(sess)
    file, err := os.Create(filename)
    if err != nil {
        return err
    }
    _, err = downloader.Download(file,
        &s3.GetObjectInput{
            Bucket: aws.String(bucket),
            Key:    aws.String(key),
        })
    if err != nil {
       return
    }
    return nil
}

func main() {
    region := "your-aws-region"
    sess := session.Must(session.NewSession(&aws.Config{
        Region: aws.String(region),
    }))
    svc := sns.New(sess)
    topicArn := "arn:aws:sns:your-region:your-account-id:your-topic-name"
    subscribeInput := &sns.SubscribeInput{
        Protocol: aws.String("email"), // Change protocol as needed
        Endpoint: aws.String("your-email@example.com"), // Change endpoint as needed
        TopicArn: aws.String(topicArn),
    }
    subscribeOutput, err := svc.Subscribe(subscribeInput)
    if err != nil {
        return
    }
    fmt.Println("subscribed successfully:", *subscribeOutput.SubscriptionArn)
    for {
        receiveInput := &sns.ReceiveMessageInput{
            MaxNumberOfMessages: aws.Int64(10), // Adjust as needed
            QueueUrl:            aws.String("queue-url"), // Provide SQS queue URL if using SQS as subscriber endpoint
        }
        receiveOutput, err := svc.ReceiveMessage(receiveInput)
        if err != nil {
            continue
        }
        for _, msg := range receiveOutput.Messages {
            fmt.Println("received message:", *msg.MessageId, *msg.Body)
            bucket := "your-bucket-name"
            key := *msg.Body // Assuming the message body is the S3 object key
            err := downloadFromS3(bucket, key, key) // Saves the file with the same name as the key
            if err != nil {
                continue
            }
            fmt.Printf("file %s downloaded successfully\n", key)
            _, err = svc.DeleteMessage(&sns.DeleteMessageInput{
                QueueUrl:      aws.String("queue-url"), // Provide SQS queue URL if using SQS as subscriber endpoint
                ReceiptHandle: msg.ReceiptHandle,
            })
            if err != nil {
                continue
            }
            fmt.Println("message deleted from queue")
        }
    }
}
```
- The program subscribes to an SNS topic using the Subscribe method of the sns service client. Replace topicArn, Protocol, and Endpoint with your actual values.

- It continuously polls for messages using ReceiveMessage and processes each message. The assumed message body is used as the S3 key to download the file.

- The downloadFromS3 function initializes an AWS session, creates an S3 downloader, and downloads the file specified by bucket and key to the local filesystem with the same name as the S3 key.

- After successfully downloading the file, it deletes the message from the queue (assuming you are using an SQS queue as the SNS endpoint) to mark it as processed.

### BIG FILE UPLOAD AND DOWNLAOD

```go
package main

import (
    "os"
    "fmt"
    "math"
    "github.com/aws/aws-sdk-go/aws"
    "github.com/aws/aws-sdk-go/aws/session"
    "github.com/aws/aws-sdk-go/service/s3"
    "github.com/aws/aws-sdk-go/service/s3/s3manager"
)

func main() {
    // Specify your AWS region and S3 bucket name
    region := "us-west-2"
    bucket := "your-bucket-name"
    
    // Initialize AWS session
    sess := session.Must(session.NewSession(&aws.Config{
        Region: aws.String(region),
    }))
    
    // Initialize S3 uploader
    uploader := s3manager.NewUploader(sess)
    
    // File to be uploaded
    filePath := "/path/to/your/big/file"
    
    file, err := os.Open(filePath)
    if err != nil {
        fmt.Println("Failed to open file", err)
        return
    }
    defer file.Close()
    
    fileInfo, _ := file.Stat()
    fileSize := fileInfo.Size()
    
    // Part size in bytes (e.g., 5 MB)
    const partSize = 5 * 1024 * 1024  // 5 MB
    
    // Calculate number of parts needed
    numParts := int(math.Ceil(float64(fileSize) / float64(partSize)))
    
    // Initialize slice to hold upload part inputs
    var uploadParts []*s3manager.UploadPartOutput
    
    // Iterate through the file, upload each part concurrently
    for i := 0; i < numParts; i++ {
        // Calculate the byte range for the part
        var start int64 = int64(i) * partSize
        var end int64 = int64(math.Min(float64(start+partSize), float64(fileSize)))
        partSize := end - start
        
        // Create buffer for the part
        buffer := make([]byte, partSize)
        _, err := file.ReadAt(buffer, start)
        if err != nil {
            fmt.Println("Failed to read file segment", err)
            return
        }
        
        // Upload part to S3
        result, err := uploader.Upload(&s3manager.UploadInput{
            Bucket:      aws.String(bucket),
            Key:         aws.String("your-s3-object-key"),
            PartNumber:  aws.Int64(int64(i) + 1), // Part numbers must be > 0
            Body:        aws.ReadSeekCloser(bytes.NewReader(buffer)),
            ContentLength: aws.Int64(partSize),
        })
        if err != nil {
            fmt.Println("Failed to upload part", err)
            return
        }
        
        // Collect uploaded part information
        uploadParts = append(uploadParts, result)
    }
    
    // Complete the multipart upload
    _, err = uploader.CompleteUpload(&s3manager.CompleteMultipartUploadInput{
        Bucket: aws.String(bucket),
        Key:    aws.String("your-s3-object-key"),
        UploadId: aws.String(uploadParts[0].UploadId), // Use Upload ID from first part
        MultipartUpload: &s3.CompletedMultipartUpload{
            Parts: uploadParts,
        },
    })
    if err != nil {
        fmt.Println("Failed to complete multipart upload", err)
        return
    }
    
    fmt.Println("File upload successful")
}
```

```go
package main

import (
    "os"
    "fmt"
    "github.com/aws/aws-sdk-go/aws"
    "github.com/aws/aws-sdk-go/aws/session"
    "github.com/aws/aws-sdk-go/service/s3"
)

func main() {
    // Specify your AWS region and S3 bucket name
    region := "us-west-2"
    bucket := "your-bucket-name"
    objectKey := "your-s3-object-key"
    
    // Initialize AWS session
    sess := session.Must(session.NewSession(&aws.Config{
        Region: aws.String(region),
    }))
    
    // Initialize S3 downloader
    downloader := s3.New(sess)
    
    // Get object metadata to determine file size
    resp, err := downloader.HeadObject(&s3.HeadObjectInput{
        Bucket: aws.String(bucket),
        Key:    aws.String(objectKey),
    })
    if err != nil {
        fmt.Println("Failed to get object metadata", err)
        return
    }
    
    fileSize := *resp.ContentLength
    
    // Open a file to write to
    file, err := os.Create("/path/to/local/file")
    if err != nil {
        fmt.Println("Failed to create file", err)
        return
    }
    defer file.Close()
    
    // Part size in bytes (e.g., 5 MB)
    const partSize = 5 * 1024 * 1024  // 5 MB
    
    // Calculate number of parts needed
    numParts := int(math.Ceil(float64(fileSize) / float64(partSize)))
    
    // Iterate through the file, download each part concurrently
    for i := 0; i < numParts; i++ {
        // Calculate the byte range for the part
        var start int64 = int64(i) * partSize
        var end int64 = int64(math.Min(float64(start+partSize), float64(fileSize)))
        rangeStr := fmt.Sprintf("bytes=%d-%d", start, end-1)
        
        // Download part from S3
        resp, err := downloader.GetObject(&s3.GetObjectInput{
            Bucket: aws.String(bucket),
            Key:    aws.String(objectKey),
            Range:  aws.String(rangeStr),
        })
        if err != nil {
            fmt.Println("Failed to download part", err)
            return
        }
        
        // Write part to local file
        _, err = io.Copy(file, resp.Body)
        if err != nil {
            fmt.Println("Failed to write part to file", err)
            return
        }
        
        resp.Body.Close()
    }
    
    fmt.Println("File download successful")
}
```

## Elastic-Search:

Elasticsearch is a powerful distributed search engine that is commonly used for a variety of use cases including full-text search, log analytics, and more. 

To interact with AWS Elasticsearch from AWS Lambda using Go (Golang), you typically perform operations like adding documents (indexing) and retrieving documents (searching). 
To integrate AWS Lambda with Elasticsearch, you typically need to make HTTP requests to the Elasticsearch service endpoint using Lambda's runtime environment.

Ensure you have an Elasticsearch domain set up and accessible. Ensure that your AWS Lambda function has the necessary IAM permissions to interact with AWS Elasticsearch.


### Service to Put Data into SQS Queue :

```go
package main
import (
	"encoding/json"
	"log"
	"os"
	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/sqs"
)
var (
	queueURL = "your-sqs-queue-url"
)
type Document struct {
	ID    string `json:"id"`
	Title string `json:"title"`
}
func main() {
	sess := session.Must(session.NewSession())
	sqsSvc := sqs.New(sess)
	document := Document{
		ID:    "1",
		Title: "Example Document Title",
	}
	docBytes, err := json.Marshal(document)
	if err != nil {
		log.Fatalf("Error marshalling document: %v", err)
	}
	_, err = sqsSvc.SendMessage(&sqs.SendMessageInput{
		MessageBody: aws.String(string(docBytes)),
		QueueUrl:    aws.String(queueURL),
	})
	if err != nil {
		log.Fatal(err)
	}

	log.Printf(document.ID)
}
```
### This Lambda function will listen to an SQS queue, retrieve messages (documents), and update their corresponding entries in Elasticsearch : 

```go
package main
import (
	"context"
	"encoding/json"
	"fmt"
	"log"
	"os"
	"time"
	"github.com/aws/aws-lambda-go/events"
	"github.com/aws/aws-lambda-go/lambda"
	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/elasticsearchservice"
	"github.com/aws/aws-sdk-go/service/sqs"
)
var (
	esService   *elasticsearchservice.ElasticsearchService
	sqsService  *sqs.SQS
	esIndexName = "your-index-name"
)
type Document struct {
	ID    string `json:"id"`
	Title string `json:"title"`
}
func init() {
	sess := session.Must(session.NewSession())
	esService = elasticsearchservice.New(sess)
	sqsService = sqs.New(sess)
}
func handler(ctx context.Context, sqsEvent events.SQSEvent) error {
	for _, message := range sqsEvent.Records {
		var document Document
		err := json.Unmarshal([]byte(message.Body), &document)
		if err != nil {
			log.Printf(err)
			continue
		}
		err = updateElasticsearch(document)
		if err != nil {
			log.Printf(err)
			continue
		}
		log.Printf(document.ID)
	}
	return nil
}

func updateElasticsearch(document Document) error {
	docBytes, err := json.Marshal(document)
	if err != nil {
		return err
	}
	req, err := http.NewRequest("PUT", fmt.Sprintf("https://%s/%s/_doc/%s", os.Getenv("ES_ENDPOINT"), esIndexName, document.ID), bytes.NewReader(docBytes))
	if err != nil {
		return err
	}
	req.Header.Set("Content-Type", "application/json")
	client := &http.Client{Timeout: 10 * time.Second}
	res, err := client.Do(req)
	if err != nil {
		return err
	}
	defer res.Body.Close()
	if res.StatusCode != http.StatusOK && res.StatusCode != http.StatusCreated {
		return fmt.Errorf("unexpected status code: %d", res.StatusCode)
	}
	return nil
}
func main() {
	lambda.Start(handler)
}
```

- Compile your Go code (GOOS=linux GOARCH=amd64 go build -o main main.go).
- Create a ZIP file (main.zip) containing your compiled binary (main) and any necessary dependencies.
- Go to the AWS Lambda Console.
- Create a new Lambda function.
- Upload the ZIP file (main.zip).
- Set the Lambda handler to main (or your binary name).
- Configure the function environment variables (ES_ENDPOINT, INDEX_NAME, QUEUE_URL, etc.) if required.


```go
import "github.com/aws/aws-sdk-go/aws/session"
import es "github.com/aws/aws-sdk-go/service/elasticsearchservice"
type item struct{
	ID string `json:"id"`
	Name string `json:"name"
	Price float32 `json:"price"
}
func AddInES(data item)(err error){
	els:=es.New(session.New())
	doc:=map[string]interface{}{
		"id":item.ID,
		"name":item.Name,
		"price":item.Price,
	}
	dataBytes,err:=json.Marshal(doc)
	if err!=nil{
		return
	}
	req:=es.UpdateElasticsearchDomainConfigInput{
		DomainName:"",
		ElasticsearchClusterConfig: &es.ElasticsearchClusterConfig{InstanceCount: aws.Int64(1),},
		AccessPolicies: aws.String("")
	}
	_,err=els.UpdateElasticsearchDomainConfig(req)
	if err!=nil{
		return
	}
	return nil
}
func Handler(ctx context.Context, request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
	vat item Item
	err:=json.Unmarshal([]byte(request.Body), &item)
	if err!=nil{
		return events.APIGatewayProxyResponse{StatusCode: http.StatusBadRequest}, nil
	}
	err=AddInES(item)
	if err!=nil{
		return events.APIGatewayProxyResponse{StatusCode: http.StatusInternalServerError}, nil
	}
	res:=events.APIGatewayProxyResponse{
        StatusCode: http.StatusOK,
        Body:       "Item added to Elasticsearch",
    }
	return response, nil
}
func main() {
    lambda.Start(Handler)
}
// GET From Elastic Search
func GetFromES(itemID string) (map[string]interface{}, error) {
    els := elasticsearchservice.New(session.New())

    req := &es.GetCompatibleElasticsearchVersionsInput{
        DomainName: aws.String(""),
        ElasticsearchClusterConfig: &es.ElasticsearchClusterConfig{
            InstanceCount: aws.Int64(1),
        },
        AccessPolicies: aws.String(""),
    }
    resp, err := els.GetCompatibleElasticsearchVersions(req)
    if err != nil {
        return nil, fmt.Errorf("failed to get item from Elasticsearch: %v", err)
    }

    var document map[string]interface{}
    if err := json.Unmarshal([]byte(resp), &document); err != nil {
        return nil, err
    }
    
    return document, nil
}
func Handler(ctx context.Context, request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
	itemID := request.PathParameters["id"]
	item, err := GetFromElasticsearch(itemID)
	if err!=nil{
		return events.APIGatewayProxyResponse{StatusCode: http.StatusInternalServerError}, nil
	}
	response, err := json.Marshal(item)
	if err!=nil{
		return events.APIGatewayProxyResponse{StatusCode: http.StatusInternalServerError}, nil
	}
	return events.APIGatewayProxyResponse{
        StatusCode: http.StatusOK,
        Body:       string(response),
    }, nil
}
```


# System Design


### Forward Proxy

When a client make a request to access a webpage the request first goes through the forward proxy server.
forward proxy server hides the IP address from CLient by masking. forward proxy server can fiter based on content type, url, or other criteria. forward proxy server can restrict access to specific resources or domains based on predifined rules. forward proxy server can cache frequently requested resources.
In a corporate environment, a forward proxy might be used to control and monitor employees' internet access, enforce security policies, or speed up access to commonly accessed resources.

### Reverse Proxy
Reverse Proxy receives client requestes and forward them to the appropiate backend servers. Reverse Proxy can load balance. Reverse Proxy can handle SSL/TLS encryption decryption. Reverse Proxy can cache the requested data to reduce load on backend servers. Reverse Proxy can act as a firewall by isolating backend servers from direct access to clients.

### CAP theorem
- Consistency: This means that all parts of your system show the same data at the same time. So, if you update something, all users will see that update immediately. Every read receives the most recent write or an error.

- Availability: This means that your system is always responsive and can handle requests even if there's a problem somewhere. Users won't see errors; they'll always get some kind of response. Every request receives a response, without guarantee that it contains the most recent write. 

- Partition Tolerance: This means your system can keep working even if parts of it can't talk to each other because of network problems. It stays operational despite these "partitions."


If you choose consistency and partition tolerance (CP), your system might not always be available when there are network issues. If you choose availability and partition tolerance (AP), you might sacrifice some consistency because different parts of your system could show slightly different data for a short time.

a banking system needs strong consistency (so your balance is always accurate) and partition tolerance (so you can still access your money even if there’s a network issue), even if it means sometimes the system might be temporarily unavailable. On the other hand, a social media feed might prioritize availability (you always see posts) and partition tolerance (you can use the app even if your connection is spotty), even if it means posts might not appear in perfect order right away.

### ACID and BASE 
ACID and BASE represent different approaches to managing data consistency in distributed systems.
ACID is suitable for applications that require strong guarantees about data integrity and consistency, such as financial transactions. BASE is suitable for applications where high availability and scalability are more important than strong consistency, such as social media feeds or content delivery networks.

ACID stands for Atomicity, Consistency, Isolation, Durability.

- Atomicity: Ensures that a transaction is treated as a single unit of work, either all of its operations are executed or none are. If any part of the transaction fails, the entire transaction is rolled back.

- Consistency: Ensures that the database remains in a consistent state before and after the transaction.

- Isolation: Ensures that the execution of transactions concurrently produces a result that is equivalent to a serial execution. 

- Durability: Ensures that once a transaction has been committed, it will remain committed even in the event of system failures (e.g., power outage, crash). The changes made by the transaction are permanently stored in the database.

BASE stands for Basically Available, Soft state, Eventually consistent.

- Basically Available: Guarantees that the system remains operational and responsive, even in the face of failures. This means that every request receives a response, even if it might be a failure response.

- Soft state: Allows the system to relax consistency constraints. Instead of requiring immediate and perfect consistency everywhere, it accepts that there may be temporary differences or delays in syncing data across different parts of the system. They allow the system to keep operating even if some parts are temporarily out of sync, which is crucial for maintaining overall system responsiveness and avoiding disruptions.

- Eventually consistent: Consistency is achieved over time, rather than immediately. Updates to a system propagate asynchronously to all replicas, and eventually, all replicas will converge to a consistent state.


### Design Patterns
- Singleton Pattern:  The Singleton pattern ensures a class has only one instance and provides a global point of access to it. The singleton instance is created only when it is first requested, rather than at the time of class loading. It should be designed to be thread-safe, ensuring that multiple threads can safely access the singleton instance concurrently without creating multiple instances. Singletone design pattern can use at Storing and accessing global settings or configurations throughout the application, database connection pool or a logging service where multiple parts of an application need access to a single resource.

```go
package main

import (
    "fmt"
    "sync"
)

// ConfigManager is a singleton struct for managing configuration settings.
type ConfigManager struct {
    config map[string]string //config is a map storing key-value pairs of configuration settings.
    // other fields as needed
}

var (
    instance *ConfigManager
    once     sync.Once
)

// GetConfigManager returns the singleton instance of ConfigManager.
// This function provides global access to the singleton instance of ConfigManager
func GetConfigManager() *ConfigManager {
    once.Do(func() { // sync.Once ensures that, the initialization code, inside once.Do() is executed exactly once, preventing multiple initializations even with concurrent calls.
        instance = &ConfigManager{
            config: make(map[string]string),
        }
        // Initialize configuration settings here
        instance.initConfig()
    })
    return instance
}

// initConfig simulates loading initial configuration settings.
// This method initializes the initial configuration settings. In a real application, this might involve reading settings from a configuration file, a database, or environment variables.
func (cm *ConfigManager) initConfig() {
    // Load configuration settings from file, database, etc.
    cm.config["server_address"] = "localhost"
    cm.config["port"] = "8080"
    // Add more configuration settings as needed
}

// GetConfig retrieves a specific configuration setting.
func (cm *ConfigManager) GetConfig(key string) string {
    return cm.config[key]
}

func main() {
    // Get the singleton instance of ConfigManager
    configManager := GetConfigManager()

    // Access configuration settings
    fmt.Println("Server Address:", configManager.GetConfig("server_address"))
    fmt.Println("Port:", configManager.GetConfig("port"))
}

```

- Builder Pattern:  The Builder pattern in Go (Golang) is used to construct complex objects step by step. 

```go
package main

import "fmt"

// Product represents the complex object we want to build.
type Product struct {
    Part1 string
    Part2 int
    Part3 bool
}

// Builder interface defines the steps to build the Product.
type Builder interface {
    SetPart1(part1 string)
    SetPart2(part2 int)
    SetPart3(part3 bool)
    Build() Product
}

// ConcreteBuilder implements the Builder interface.
type ConcreteBuilder struct {
    part1 string
    part2 int
    part3 bool
}

func (b *ConcreteBuilder) SetPart1(part1 string) {
    b.part1 = part1
}

func (b *ConcreteBuilder) SetPart2(part2 int) {
    b.part2 = part2
}

func (b *ConcreteBuilder) SetPart3(part3 bool) {
    b.part3 = part3
}

func (b *ConcreteBuilder) Build() Product {
    // Normally, here we could implement additional logic or validation
    // before returning the final product.
    return Product{
        Part1: b.part1,
        Part2: b.part2,
        Part3: b.part3,
    }
}

// Director controls the construction process using a Builder.
type Director struct {
    builder Builder
}

func NewDirector(builder Builder) *Director {
    return &Director{builder: builder}
}

func (d *Director) Construct(part1 string, part2 int, part3 bool) Product {
    d.builder.SetPart1(part1)
    d.builder.SetPart2(part2)
    d.builder.SetPart3(part3)
    return d.builder.Build()
}

func main() {
    // Create a concrete builder instance
    builder := &ConcreteBuilder{}

    // Create a director with the concrete builder
    director := NewDirector(builder)

    // Construct the product
    product := director.Construct("Example", 42, true)

    // Print the constructed product
    fmt.Printf("Constructed Product: %+v\n", product)
}
```


- Factory Pattern: Factory pattern is typically used to encapsulate the instantiation of objects, allowing the client code to create objects without knowing the exact type being created.

Here's an example of implementing the Factory pattern with repository selection based on database type:

```go
package main

import (
	"fmt"
	"errors"
)

// Repository interface defines the methods that all concrete repository implementations must implement.
type Repository interface {
	GetByID(id int) (interface{}, error)
	Save(data interface{}) error
	// Add other methods as needed
}

// MySQLRepository represents a concrete implementation of Repository interface for MySQL.
type MySQLRepository struct {
	// MySQL connection details or any necessary configuration
}

func (r *MySQLRepository) GetByID(id int) (interface{}, error) {
	// Implement MySQL specific logic to fetch data by ID
	return nil, errors.New("not implemented")
}

func (r *MySQLRepository) Save(data interface{}) error {
	// Implement MySQL specific logic to save data
	return errors.New("not implemented")
}

// PostgreSQLRepository represents a concrete implementation of Repository interface for PostgreSQL.
type PostgreSQLRepository struct {
	// PostgreSQL connection details or any necessary configuration
}

func (r *PostgreSQLRepository) GetByID(id int) (interface{}, error) {
	// Implement PostgreSQL specific logic to fetch data by ID
	return nil, errors.New("not implemented")
}

func (r *PostgreSQLRepository) Save(data interface{}) error {
	// Implement PostgreSQL specific logic to save data
	return errors.New("not implemented")
}

// RepositoryFactory is the factory that creates different types of repositories based on the database type.
type RepositoryFactory struct{}

func (f *RepositoryFactory) CreateRepository(databaseType string) (Repository, error) {
	switch databaseType {
	case "mysql":
		return &MySQLRepository{}, nil
	case "postgresql":
		return &PostgreSQLRepository{}, nil
	default:
		return nil, fmt.Errorf("unsupported database type: %s", databaseType)
	}
}

func main() {
	factory := RepositoryFactory{}

	// Create a MySQL repository
	mysqlRepo, err := factory.CreateRepository("mysql")
	if err != nil {
		fmt.Println("Error creating MySQL repository:", err)
	} else {
		fmt.Println("Created MySQL repository successfully")
		// Use mysqlRepo as needed
	}

	// Create a PostgreSQL repository
	postgresqlRepo, err := factory.CreateRepository("postgresql")
	if err != nil {
		fmt.Println("Error creating PostgreSQL repository:", err)
	} else {
		fmt.Println("Created PostgreSQL repository successfully")
		// Use postgresqlRepo as needed
	}

	// Try to create a repository with an unsupported database type
	_, err = factory.CreateRepository("mongodb")
	if err != nil {
		fmt.Println("Error creating MongoDB repository:", err)
	}
}
```


- Observer Pattern: The Observer pattern defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.
```go
package main

import (
	"fmt"
	"time"
)

// Observer defines the interface that all observers (subscribers) must implement.
type Observer interface {
	Update(article string)
}

// Subject defines the interface for the subject (publisher) that observers will observe.
type Subject interface {
	Register(observer Observer)
	Deregister(observer Observer)
	Notify(article string)
}

// NewsService represents a concrete implementation of the Subject interface.
type NewsService struct {
	observers []Observer
}

func (n *NewsService) Register(observer Observer) {
	n.observers = append(n.observers, observer)
}

func (n *NewsService) Deregister(observer Observer) {
	for i, obs := range n.observers {
		if obs == observer {
			n.observers = append(n.observers[:i], n.observers[i+1:]...)
			break
		}
	}
}

func (n *NewsService) Notify(article string) {
	for _, observer := range n.observers {
		observer.Update(article)
	}
}

// NewsSubscriber represents a concrete implementation of the Observer interface.
type NewsSubscriber struct {
	Name string
}

func (s *NewsSubscriber) Update(article string) {
	fmt.Printf("[%s] Received article update: %s\n", s.Name, article)
}

func main() {
	// Create a news service
	newsService := &NewsService{}

	// Create news subscribers (observers)
	subscriber1 := &NewsSubscriber{Name: "John"}
	subscriber2 := &NewsSubscriber{Name: "Alice"}
	subscriber3 := &NewsSubscriber{Name: "Bob"}

	// Register subscribers with the news service
	newsService.Register(subscriber1)
	newsService.Register(subscriber2)
	newsService.Register(subscriber3)

	// Simulate publishing new articles
	go func() {
		for i := 1; i <= 5; i++ {
			article := fmt.Sprintf("Article %d", i)
			newsService.Notify(article)
			time.Sleep(time.Second)
		}
	}()

	// Let the program run for a moment to see the output
	time.Sleep(6 * time.Second)

	// Deregister subscriber2
	newsService.Deregister(subscriber2)

	// Simulate publishing another article after deregistration
	article := "Final Article"
	newsService.Notify(article)

	// Let the program run for a moment to see the final output
	time.Sleep(time.Second)
}

```
Advantages of the Observer Pattern:

    Loose coupling: The subject  and observers they don't need to know each other's details beyond the Observer interface.

    Supports multiple observers: The pattern allows for any number of observers to be registered with a subject, and they can all react independently to changes.

    Ease of extension: You can easily add new observers without modifying the subject or other observers.

When to Use the Observer Pattern:

    Event-driven systems: When changes in one object require updates in other related objects (e.g., UI components reflecting changes in underlying data).

    Decoupling behavior: When you want to decouple an abstraction from its implementation so that the two can vary independently.


- Decorator Pattern: The Decorator pattern allows behavior to be added to individual objects, dynamically, without affecting the behavior of other objects from the same class. 

In this example, the Decorator pattern allows us to dynamically add behaviors (enhancements to attack or defense) to an object (the player) without affecting its core functionality. This pattern is useful when you need to extend an object's functionality in a flexible and modular way, which is especially relevant in game development and other software scenarios where objects can have varied and dynamic behaviors.

```go

package main

import "fmt"

// Player interface represents the basic functionalities a player must have.
type Player interface {
	Attack() int
	Defense() int
}

// BasePlayer represents the basic attributes of a player.
type BasePlayer struct {
	attack  int
	defense int
}

func (p *BasePlayer) Attack() int {
	return p.attack
}

func (p *BasePlayer) Defense() int {
	return p.defense
}

// ArmorDecorator enhances a player's defense.
// ArmorDecorator enhances a player's defense by adding an additional armor value.
type ArmorDecorator struct {
	player Player
	armor  int
}

func (a *ArmorDecorator) Attack() int {
	return a.player.Attack()
}

func (a *ArmorDecorator) Defense() int {
	return a.player.Defense() + a.armor
}

// WeaponDecorator enhances a player's attack.
// WeaponDecorator enhances a player's attack by adding an additional attack value.
type WeaponDecorator struct {
	player Player
	attack int
}

func (w *WeaponDecorator) Attack() int {
	return w.player.Attack() + w.attack
}

func (w *WeaponDecorator) Defense() int {
	return w.player.Defense()
}

func main() {
	// Create a base player
	player := &BasePlayer{
		attack:  10,
		defense: 5,
	}

	fmt.Println("Base Player:")
	fmt.Printf("Attack: %d, Defense: %d\n", player.Attack(), player.Defense())

	// Add armor to the player
	playerWithArmor := &ArmorDecorator{
		player: player,
		armor:  5,
	}

	fmt.Println("Player with Armor:")
	fmt.Printf("Attack: %d, Defense: %d\n", playerWithArmor.Attack(), playerWithArmor.Defense())

	// Add a weapon to the player with armor
	playerWithArmorAndWeapon := &WeaponDecorator{
		player: playerWithArmor,
		attack: 8,
	}

	fmt.Println("Player with Armor and Weapon:")
	fmt.Printf("Attack: %d, Defense: %d\n", playerWithArmorAndWeapon.Attack(), playerWithArmorAndWeapon.Defense())
}
```
```bash
# OUTPUT
Base Player:
Attack: 10, Defense: 5
Player with Armor:
Attack: 10, Defense: 10
Player with Armor and Weapon:
Attack: 18, Defense: 10
```
- Strategy Pattern: The Strategy pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable. It lets the algorithm vary independently from clients that use it. The strategy pattern is a behavioral design pattern that enables selecting an algorithm at runtime from a family of algorithms. It allows a client to choose from a variety of algorithms or strategies without altering their structure. This pattern is useful when you have multiple algorithms that can be interchangeable depending on the context.

```go
// Defines a contract for sorting algorithms. Any concrete sorting algorithm (BubbleSort, QuickSort) must implement this interface.
type SortStrategy interface {
    Sort([]int) []int
}
```
```go
ype BubbleSort struct{}

func (bs *BubbleSort) Sort(arr []int) []int {
    n := len(arr)
    for i := 0; i < n-1; i++ {
        for j := 0; j < n-i-1; j++ {
            if arr[j] > arr[j+1] {
                arr[j], arr[j+1] = arr[j+1], arr[j]
            }
        }
    }
    return arr
}
```

```go
type QuickSort struct{}

func (qs *QuickSort) Sort(arr []int) []int {
    if len(arr) < 2 {
        return arr
    }
    pivot := arr[0]
    var less, greater []int
    for _, v := range arr[1:] {
        if v <= pivot {
            less = append(less, v)
        } else {
            greater = append(greater, v)
        }
    }
    sorted := append(append(qs.Sort(less), pivot), qs.Sort(greater)...)
    return sorted
}

```
```go
// Holds a reference to a strategy object (SortStrategy) and provides methods to set and use different sorting algorithms. It delegates the sorting task to the current strategy.
type SortContext struct {
    strategy SortStrategy
}

func NewSortContext(strategy SortStrategy) *SortContext {
    return &SortContext{strategy: strategy}
}

func (sc *SortContext) SetStrategy(strategy SortStrategy) {
    sc.strategy = strategy
}

func (sc *SortContext) SortArray(arr []int) []int {
    return sc.strategy.Sort(arr)
}

```
```go
func main() {
    arr := []int{64, 25, 12, 22, 11}

    // Use Bubble Sort
    bubbleSort := &BubbleSort{}
    context := NewSortContext(bubbleSort)
    sorted := context.SortArray(arr)
    fmt.Println("Sorted using Bubble Sort:", sorted)

    // Use Quick Sort
    quickSort := &QuickSort{}
    context.SetStrategy(quickSort)
    sorted = context.SortArray(arr)
    fmt.Println("Sorted using Quick Sort:", sorted)
}
```
The strategy pattern allows the client (main function) to select different algorithms dynamically at runtime.
Algorithms are encapsulated in their own classes (BubbleSort, QuickSort), adhering to the Single Responsibility Principle.
Adding new sorting algorithms (MergeSort, InsertionSort, etc.) involves creating new classes that implement SortStrategy.

## Development Patterns

### DDD - Domain Driven Design 
Focus on the Core Domain: 
- Concentrate effort and resources on understanding and solving the core problems of the business domain. 
- Develop a common understanding of the domain among all team members using a shared language. 
- Invest time in understanding the domain, including business processes, rules, and terminology. 
- Clearly define and design models (entities, value objects, aggregates) that reflect the domain and business rules.  
- Start with a core domain and refine the model iteratively based on feedback and evolving understanding. 
- Foster collaboration between domain experts, developers, and other stakeholders throughout the development process. 
- Promotes a design that is easier to maintain and evolve as the domain evolves. 
- Manages complexity by breaking down large problems into smaller, manageable parts.

- By using DDD we get benifits of Aligns development efforts with business requirements. Encapsulates domain logic in well-defined models and services. Enables scaling of development efforts by breaking down the system into manageable parts.


Imagine we're building an online shopping platform where users can browse products, add them to their cart, and place orders.

##### Firstly, we need to deeply understand the domain of online shopping:
- Core Concepts: Products, Users, Orders, Cart, Payment, Shipping, etc.
- Business Rules: Discounts, Shipping Policies, Payment Gateways, Inventory Management, etc.
- Workflows: Adding items to cart, Checkout Process, Order Fulfillment, etc.

##### Establish a common language that everyone in the team understands:
- Common Terms: Product, User, Order, Cart, Payment, Shipping, Checkout, etc.
- Define what each term means and how they interact.

##### Identify bounded contexts(Boundary of Models- Product and Order Model should not be Mixed) to divide and conquer complexity:
- Product Catalog Context: Manages products, categories, and inventory.
- Order Management Context: Handles order placement, fulfillment, and status tracking.
- User Authentication Context: Manages user authentication and authorization.
- Payment Integration Context: Integrates with payment gateways for processing transactions.

##### Define domain models based on identified entities, value objects, aggregates, and services:
Bounded contexts help divide a complex domain into manageable parts, value objects represent immutable pieces of data, and aggregates group related objects together to maintain consistency and manage complexity in the domain model.

- Entities: Product, Order, User (with unique IDs and lifecycle).
- Value Objects: CartItem (with attributes like product ID, quantity).
- Aggregates: Aggregates are clusters of related objects that are treated as a single unit. Order (with associated OrderItems and shipping details). The Order entity acts as the root of the aggregate, managing the lifecycle of associated objects like OrderItems and PaymentDetails.
- Services: PaymentService (handling payment transactions), ShippingService (calculating shipping costs). 

##### Implement repositories to abstract data access:
- ProductRepository: Save, FindByID, SearchByCategory, etc.
- OrderRepository: Save, FindByID, SearchByUser, etc.
- UserRepository: Save, FindByID, Authenticate, etc.
- Persistence Layer: Implement using SQL databases, NoSQL databases, or in-memory stores.

##### Use domain events to handle cross-cutting concerns or communicate state changes:
- OrderPlacedEvent: Triggered after successful order placement.
- PaymentProcessedEvent: Triggered after successful payment processing.
- InventoryUpdatedEvent: Triggered after stock levels change.

##### Develop application services to orchestrate business use cases:
- PlaceOrderService: Coordinates creating an order, updating inventory, and triggering payment.
- AddToCartService: Manages adding items to the user's cart and updating totals.
- CheckoutService: Handles the checkout process, including validation and order finalization.

##### Implement infrastructure concerns such as:
- Database Connections: Connecting to databases for persistence.
- External Services Integration: Integrating with third-party APIs (payment gateways, shipping providers).
- Event Bus: Implementing event-driven architecture for handling domain events.

#### Define Entities and Value Objects
```go
// Product entity
type Product struct {
    ID       uuid.UUID
    Name     string
    Price    float64
    Quantity int
}

// CartItem value object
// A CartItem is defined by its attributes such as ProductID and Quantity. These attributes determine the nature of the CartItem, but the CartItem itself does not have an identity independent of these attributes.
// Two CartItem instances are considered equal if their attributes (ProductID and Quantity) are the same. In domain-driven design, equality of value objects is determined based on their attribute values rather than their identity.
type CartItem struct {
    ProductID uuid.UUID
    Quantity  int
}

// Order entity
type Order struct {
    ID         uuid.UUID
    CustomerID uuid.UUID
    Items      []CartItem
    Total      float64
    CreatedAt   time.Time
}
```
### Aggregates group related entities and value objects together, with an aggregate root enforcing consistency.

```go
// Order aggregate root
// An Order often comprises multiple related entities and value objects such as OrderItems, CustomerDetails, and PaymentDetails
// when an order is placed or updated, all related data within the aggregate is managed consistently.
// An order may have rules regarding minimum quantities, total prices, or eligibility for discounts. By defining these rules within the aggregate, you ensure they are consistently enforced whenever an order is manipulated.
// The CustomerDetails and PaymentDetails are correctly associated with the order.

type OrderAggregate struct {
    Order *Order
}

func (oa *OrderAggregate) AddItem(productID uuid.UUID, quantity int) error {
    // Business logic to add items to the order
}
```
### Repositories abstract the data persistence layer for entities.
```go
// ProductRepository interface
type ProductRepository interface {
    Save(product *Product) error
    FindByID(id uuid.UUID) (*Product, error)
    // Other methods like FindByName, Delete, etc.
}

// OrderRepository interface
type OrderRepository interface {
    Save(order *Order) error
    FindByID(id uuid.UUID) (*Order, error)
    // Other methods like FindByCustomerID, UpdateStatus, etc.
}
```

### Application services orchestrate interactions between domain models and external systems.

```go
// ProductService handles operations related to products
type ProductService struct {
    productRepo ProductRepository
}

func (ps *ProductService) AddProduct(name string, price float64, quantity int) (*Product, error) {
    // Business logic to add a new product
}

// OrderService handles operations related to orders
type OrderService struct {
    orderRepo OrderRepository
}

func (os *OrderService) PlaceOrder(customerID uuid.UUID, items []CartItem) (*Order, error) {
    // Business logic to place a new order
}

```
### Implement the infrastructure layer for persistence, messaging, etc. For simplicity, we'll use in-memory repositories.
```go
// InMemoryProductRepository implements ProductRepository using in-memory storage
type InMemoryProductRepository struct {
    products map[uuid.UUID]*Product
}

func (r *InMemoryProductRepository) Save(product *Product) error {
    r.products[product.ID] = product
    return nil
}

func (r *InMemoryProductRepository) FindByID(id uuid.UUID) (*Product, error) {
    if product, ok := r.products[id]; ok {
        return product, nil
    }
    return nil, errors.New("product not found")
}

// InMemoryOrderRepository implements OrderRepository using in-memory storage
type InMemoryOrderRepository struct {
    orders map[uuid.UUID]*Order
}

func (r *InMemoryOrderRepository) Save(order *Order) error {
    r.orders[order.ID] = order
    return nil
}

func (r *InMemoryOrderRepository) FindByID(id uuid.UUID) (*Order, error) {
    if order, ok := r.orders[id]; ok {
        return order, nil
    }
    return nil, errors.New("order not found")
}
```
### Now, let's see how we would use these components in a hypothetical scenario:

```go
func main() {
    // Initialize repositories
    productRepo := &InMemoryProductRepository{
        products: make(map[uuid.UUID]*Product),
    }
    orderRepo := &InMemoryOrderRepository{
        orders: make(map[uuid.UUID]*Order),
    }

    // Initialize services
    productService := &ProductService{productRepo}
    orderService := &OrderService{orderRepo}

    // Example usage: Add a new product
    newProduct, err := productService.AddProduct("Smartphone", 999.99, 50)
    if err != nil {
        fmt.Println("Error adding product:", err)
        return
    }
    fmt.Println("New Product ID:", newProduct.ID)

    // Example usage: Place a new order
    customerID := uuid.New()
    items := []CartItem{
        {ProductID: newProduct.ID, Quantity: 2},
    }
    newOrder, err := orderService.PlaceOrder(customerID, items)
    if err != nil {
        fmt.Println("Error placing order:", err)
        return
    }
    fmt.Println("New Order ID:", newOrder.ID)
}
```

### TDD:
Test-Driven Development (TDD) is a software development approach where you write tests before writing the actual code. 

Red-Green-Refactor Cycle:
- Red: Write a test that fails. This ensures you have a clear goal and helps you define the expected behavior of your code.
- Green: Write the minimum amount of code to pass the test. This encourages you to focus on implementing just enough to satisfy the test.
- Refactor: Improve the code without changing its behavior. This step ensures your code remains clean, maintainable, and efficient.

Implement the simplest solution that passes the test case.

Refactor the code to improve its design, readability, and efficiency while keeping the tests passing.

TDD is iterative, meaning you repeat the Red-Green-Refactor cycle for each small piece of functionality.


Forces you to think about the design upfront and leads to more modular, testable code.

Gives confidence that the code works as intended, especially after each successful test pass.

Go has a built-in testing framework (testing package) that supports writing tests in a straightforward manner.Tests are typically written in files named *_test.go alongside the source files they test. 

Use functions prefixed with Test to define test cases, and functions like t.Errorf() to report test failures. Tests in Go often follow a pattern where you set up initial conditions, perform actions, and then validate results using assertions.

Adapting to TDD requires practice and discipline, especially for developers new to the methodology. Writing tests before code can feel counterintuitive initially but pays off in the long run.

Tests should be updated alongside code changes to ensure they remain accurate and relevant.

We'll create a basic inventory management microservice that handles adding products, retrieving product details, and updating product information.

```bash
/shopping-platform
  |- main.go
  |- product.go
  |- product_test.go
```
```go
// product_test.go

package main

import (
	"testing"
)

func TestAddProduct(t *testing.T) {
	// Initialize your product service or mock dependencies as needed
	// For simplicity, we assume a Product struct and an AddProduct function

	// Test case 1: Add a new product successfully
	t.Run("Add New Product", func(t *testing.T) {
		// Set up input data
		name := "Laptop"
		price := 1500.0
		quantity := 10

		// Call the function to add the product
		err := AddProduct(name, price, quantity)

		// Assert the expected outcome
		if err != nil {
			t.Errorf("Unexpected error: %v", err)
		}
	})

	// Test case 2: Handle invalid input (e.g., negative price or quantity)
	t.Run("Handle Invalid Input", func(t *testing.T) {
		// Set up invalid input data
		name := "Invalid Product"
		price := -100.0
		quantity := -5

		// Call the function to add the product with invalid input
		err := AddProduct(name, price, quantity)

		// Assert the expected outcome
		if err == nil {
			t.Errorf("Expected error for invalid input, but got none")
		}
	})
}
// Run the tests using go test. Since we haven't implemented the AddProduct function yet, both test cases should fail.
```

```go
// product.go

package main

import (
	"errors"
)

// Product represents a product entity
type Product struct {
	Name     string
	Price    float64
	Quantity int
}

var inventory []Product

// AddProduct adds a new product to the inventory
func AddProduct(name string, price float64, quantity int) error {
	// Validate input
	if price <= 0 || quantity <= 0 {
		return errors.New("price and quantity must be positive")
	}

	// Create the product object
	product := Product{
		Name:     name,
		Price:    price,
		Quantity: quantity,
	}

	// Add the product to the inventory (simulated storage, not shown in this example)
	inventory = append(inventory, product)

	return nil
}
// Run go test again. The tests should now pass, confirming that the AddProduct function behaves as expected.
```
Continue writing additional tests for other functionalities such as retrieving product details (GetProduct function), updating product information (UpdateProduct function), error handling, edge cases (e.g., product not found), and so on. Follow the same Red-Green-Refactor cycle for each new test.


### Interview Questions of System Design

1. What is system design, and why is it important?

System design involves designing the architecture, components, and modules of a system to satisfy specified requirements. It's crucial for creating scalable, reliable, and maintainable systems.

2. Explain the principles of system design.

Principles include scalability, reliability, availability, performance, maintainability, and security.

3. How do you approach designing a system from scratch?

Discuss the steps: 
Understanding Requirements: Gather and document what the system needs to do (functional requirements) and how it should perform (non-functional requirements like speed and security).

Defining Use Cases: Outline specific scenarios or interactions users will have with the system to achieve their goals.

Identifying Components: Break down the system into manageable parts or modules that perform specific tasks.

Considering Architectural Patterns: Choose how components will interact (e.g., client-server, microservices) to achieve scalability and flexibility.

Designing Interfaces: Define how different parts of the system will communicate and interact with each other.

Applying Design Patterns: Use established solutions (like MVC for structure, observer for event handling) to address common design challenges.

Choosing Technologies: Select appropriate tools and technologies that align with system requirements and team expertise.

Considering Data Storage: Decide on databases or file systems for storing and managing data, ensuring efficient data flow.

Following Coding Best Practices: Adhere to principles like DRY (Don't Repeat Yourself) and SOLID (Single Responsibility, Open-Closed, Liskov Substitution, Interface Segregation, Dependency Inversion) for maintainable and scalable code.

Implementing Error Handling: Create mechanisms to catch and manage errors gracefully, and set up logging for tracking system activities.

Defining Test Cases: Develop tests that verify system functionality against requirements and cover edge cases to ensure robustness.

Addressing Security: Identify potential threats (like unauthorized access) and implement strategies (authentication, authorization, encryption) to protect the system and user data.

Ensuring Scalability: Design the system to handle increasing loads by considering horizontal (adding more servers) and vertical (increasing server capacity) scaling.

Defining Deployment Strategy: Plan how to move the system from development (staging) to live (production) environments while minimizing disruption.

Planning for Monitoring and Maintenance: Set up tools and processes for monitoring system performance and ensuring ongoing maintenance and updates.

4. Describe the difference between scalability and elasticity.

Scalability refers to the ability to handle growth, whereas elasticity is the ability to dynamically allocate and deallocate resources based on demand.

5. What factors should you consider when designing a system for high availability?

Replication, fault tolerance, load balancing, failover mechanisms, and distributed architecture.

6. Explain CAP theorem and its implications for distributed systems.

CAP theorem states that a distributed system cannot simultaneously guarantee Consistency, Availability, and Partition tolerance. Understanding this helps in making trade-offs based on system requirements.

7. How would you design a system to handle a million requests per day?
- Load balancing involves distributing incoming network traffic across multiple servers.
Load balancers can operate at different layers (e.g., application layer, network layer) and use various algorithms (round-robin, least connections, IP hash) to distribute requests.
Load balancing Enhances fault tolerance, enables horizontal scaling by adding more servers, and improves performance by optimizing resource utilization.

- Caching involves storing frequently accessed data temporarily in a cache (memory or disk) to reduce access latency and improve performance. Use caching mechanisms like in-memory caches (Redis, Memcached) or content delivery networks (CDNs) to cache static assets and frequently accessed data. Reduces database load, speeds up access times, and improves overall system responsiveness. 

- Horizontal scaling (scaling out) involves adding more machines or instances to distribute load across multiple resources.

- Database optimization involves improving the performance and efficiency of database operations to handle large volumes of data and concurrent requests. Use indexes and proper query optimization techniques.
Normalize or denormalize database schema based on access patterns.
Implement caching mechanisms for frequently accessed data.
Use partitioning to distribute data across multiple servers.

- CDNs are networks of servers spread globally to deliver web content quickly. They store copies of content like images and videos on servers closer to users, reducing the distance and speeding up access. When a user requests content, the CDN routes the request to the nearest server. If the content isn't already there, the server fetches it from the main server. CDNs cache static content for a period or based on rules set by the website. This caching reduces load on the main server and improves response times, especially for users far away. CDNs handle high traffic by distributing it across servers and offer security features like DDoS protection and encryption.


8. Describe different types of caching strategies you would use in your system design.

Strategies include client-side caching, server-side caching (e.g., using Redis or Memcached), and CDN caching.

9. Explain the role of a reverse proxy in system architecture.

A reverse proxy sits between clients and servers, forwarding client requests to the appropriate backend server. It can handle tasks like load balancing, SSL termination, and caching.

10. How would you design a microservices architecture?

Discuss the benefits (scalability, independent deployments) and challenges (complexity, network latency). Use service discovery, API gateways, and messaging queues.

11. What are the advantages of using RESTful APIs in microservices architecture?

RESTful APIs promote loose coupling between services, scalability, and ease of integration.
RESTful APIs are inherently flexible and support various data formats (like JSON, XML) and HTTP methods (GET, POST, PUT, DELETE). This flexibility allows microservices to use different technologies, programming languages, and databases as long as they adhere to the API contract. Each API endpoint represents a specific functionality or resource, making it easier to understand and maintain compared to monolithic systems where all functionality is tightly coupled. Testing microservices with RESTful APIs is straightforward because APIs are stateless and interactions can be simulated or mocked easily. 

12. Explain how you would secure a microservices architecture.
OAuth is a standard that lets users give websites or apps permission to access their info on other sites without giving away their passwords. It uses tokens to grant limited access, issued by a server, allowing apps to act on behalf of users securely. It's widely adopted for delegated authorization scenarios where a user (resource owner) allows a third-party application (client) to access their resources stored on another service (resource server) through an API. 

JWT (JSON Web Token) is a compact way to transfer claims securely between parties. It's often signed and optionally encrypted to ensure data integrity and confidentiality. JWTs are used as access tokens in OAuth and for secure authentication, being efficient for transmitting data securely as JSON objects.

RBAC (Role-Based Access Control) restricts network access based on user roles within an organization. Permissions are assigned to roles, simplifying management and ensuring consistent access control.

OAuth is a standard that lets users give websites or apps permission to access their info on other sites without giving away their passwords. It uses tokens to grant limited access, issued by a server, allowing apps to act on behalf of users securely.

JWT (JSON Web Token) is a compact way to transfer claims securely between parties. It's often signed and optionally encrypted to ensure data integrity and confidentiality. JWTs are used as access tokens in OAuth and for secure authentication, being efficient for transmitting data securely as JSON objects.

RBAC (Role-Based Access Control) restricts network access based on user roles within an organization. Permissions are assigned to roles, simplifying management and ensuring consistent access control.

ABAC (Attribute-Based Access Control) evaluates attributes like user roles, resource details, and environmental conditions to decide access. It offers precise control by defining policies based on a wide range of attributes, adapting to changing circumstances.

TLS (Transport Layer Security) is a protocol that secures communication over networks by ensuring privacy, integrity, and data protection. It's commonly used to secure HTTP connections (HTTPS), using cryptography for key exchange, data encryption, and integrity verification.


13. Describe the differences between SQL and NoSQL databases. When would you choose one over the other?

SQL databases are relational and suitable for structured data and complex queries. NoSQL databases are non-relational, suitable for unstructured or semi-structured data, and provide high availability and scalability.

14. How would you design a database schema for a social networking application?

Basic user information like username, email, and password is stored in one table to avoid repetition.

Additional profile details such as bio and profile picture are stored directly in the Users table.

The Connections table ensures each user ID references a valid user, maintaining relational integrity.

A status field tracks friend requests (pending, accepted, blocked).

Posts are linked to users via a UserID, ensuring each post is associated with a valid user.

Indexing UserID helps quickly retrieve posts for specific users.

Comments are linked to both users (UserID) and posts (PostID), ensuring data integrity and enabling efficient retrieval.

Indexing UserID and PostID speeds up comment queries related to users or posts.

Likes are linked to users (UserID) and posts (PostID), ensuring integrity and optimizing queries.

Indexing UserID and PostID helps with quick like queries.

Frequently queried fields like UserID, PostID, User1ID, and User2ID are indexed for faster retrieval.

Data types (e.g., VARCHAR, TEXT, TIMESTAMP) are chosen based on data size and usage patterns.

Large tables like Posts and Comments may be partitioned by time or other criteria to enhance performance.


15. What is ACID in database transactions? When would you use it?

ACID (Atomicity, Consistency, Isolation, Durability) ensures transactions are reliable. Use it when data integrity is critical, such as in financial applications.

16. How would you handle data consistency in a distributed system?

Two-Phase Commit (2PC): Protocol ensuring transaction atomicity and consistency across distributed participants.

    Phase 1 (Prepare): Coordinator asks participants to prepare.
    Phase 2 (Commit or Abort): Coordinator commits if all are prepared; otherwise, aborts.

Quorum-Based Systems: Ensure success of read/write operations by requiring a minimum replica acknowledgment.

    Read Quorum: Ensures reading the latest value.
    Write Quorum: Ensures successful write acknowledgment.

CRDTs (Conflict-free Replicated Data Types): Distributed data structures for conflict-free updates.

    Mergeable: Operations merge without conflicts.
    Conflict-free: No need for explicit coordination.

Comparison:

    2PC ensures atomicity but may have coordination and availability issues.
    Quorum systems balance consistency and availability.
    CRDTs allow independent updates across distributed nodes without synchronization delays.

17. Explain the concept of message queues and how they are used in system design.

Message queues decouple components by enabling asynchronous communication. They help manage workload spikes, provide fault tolerance, and support integration between distributed systems.

18. How do you design a system that can handle real-time data processing?

Before diving into the technical details, it's crucial to understand the specific requirements and use cases of your real-time data processing system. Consider aspects such as:

Data Sources: Where is the data coming from? (e.g., IoT devices, web applications, logs)
Processing Needs: What types of processing and analytics are required in real-time?
Scalability: How much data will the system handle? What are the expected growth rates?
Latency Requirements: What are the acceptable response times for data processing and analytics?

Example Architecture:

    Data Ingestion:
        Use Amazon Kinesis Data Streams to ingest data streams from IoT devices or applications. Kinesis Scales elastically to handle any amount of streaming data.
        Optionally, use Amazon Kinesis Data Firehose to load data directly into AWS data stores. Firehose Automatically scales to handle spikes in data volume. Can directly load data into S3, Redshift, Elasticsearch, and other AWS services.

    Real-Time Processing:
        Implement AWS Lambda functions triggered by Kinesis Data Streams events for real-time data transformations and enrichments. Lambda Automatically scales based on the number of incoming events.

    Data Storage:
        Store raw data in Amazon S3 for durable and scalable storage.
        Use Amazon DynamoDB for fast, low-latency access to processed data or metadata. 

    Analytics and Visualization:
        Elasticsearch Store and analyze log data, metrics, and other time-series data in real-time.
        Use Amazon Kinesis Data Analytics for real-time SQL-based analytics on streaming data.
        Store and analyze data in Amazon Elasticsearch Service for real-time querying and visualization using Kibana.

    Monitoring and Alerts:
        Set up Amazon CloudWatch for monitoring metrics and logs.
        Configure alerts based on thresholds for monitoring system health and performance.


19. Describe the process of designing a RESTful API.

Discuss resource identification, HTTP methods (GET, POST, PUT, DELETE), status codes, versioning, pagination, and HATEOAS (Hypermedia as the Engine of Application State).

20. How do you ensure data privacy and security in a distributed system?

Discuss encryption (at rest and in transit), access controls, secure coding practices, regular audits, and compliance with data protection regulations (e.g., GDPR).

21. How would you handle database migrations in a production environment?

Handling database migrations in production requires careful planning to ensure minimal downtime, data integrity, and smooth transitions between versions.

Blue-Green Deployment:

    Maintain two identical environments (blue and green), with only one serving live traffic at any time.
    Deploy database schema changes (and corresponding application updates) to the inactive environment (e.g., green).
    Perform necessary data migrations or updates in the inactive environment.
    Once migration is complete and validated, switch traffic to the updated environment (e.g., green). If issues arise, quickly switch back to the previous environment (e.g., blue).

Canary Releases:

    Gradually roll out updates to a small subset of users or traffic.
    Deploy database schema changes to a limited number of servers or databases initially.
    Monitor performance and stability of the updated subset closely.
    If successful, expand the update gradually to more servers or databases.
    Continuously monitor and rollback if issues are detected during the rollout.

Backward-Compatible Schema Changes:

    Introduce new database elements (columns, tables, constraints) that do not disrupt existing functionality.
    Ensure old and new versions of the application can coexist temporarily without errors or data loss.
    Update the application gradually to utilize new schema elements.
    Migrate existing data to fit the new schema incrementally or during off-peak hours to minimize disruption.

Flyway:

    Open-source database migration tool focused on simplicity and ease of use.
    Supports SQL-based migrations and integrates well with CI/CD pipelines.
    Manages database schema changes using versioned SQL scripts.
    Handles migrations automatically based on version control, supporting major databases like MySQL, PostgreSQL, Oracle, SQL Server, etc.

These strategies and tools are crucial for managing database schema changes in production environments effectively, ensuring continuous deployment with minimal disruptions and maintaining data integrity throughout the migration process.

22. Explain the use of containers (e.g., Docker) in system design.

Containers provide lightweight, isolated environments for applications. They promote consistency across environments, scalability, and efficient resource utilization.

23. How do you ensure monitoring and observability in a distributed system?

Logging, metrics collection, distributed tracing, and monitoring tools are essential for maintaining and optimizing system performance and reliability.

Metrics Collection:

    Metrics provide quantitative measurements of system performance, resource utilization, and other indicators.
    They support monitoring, capacity planning, and trend analysis.
    Tools like Prometheus are used to scrape metrics from monitored targets, store them locally, and offer a powerful query language (PromQL) for analysis.

Distributed Tracing:

    Distributed tracing, exemplified by tools like Jaeger, tracks and monitors transactions across distributed systems.
    It helps in identifying latency issues, debugging performance bottlenecks, and understanding system dependencies.

Monitoring Tools (e.g., Grafana or ELK Stack):

    Grafana is an open-source platform for monitoring and observability.
    It supports visualization of metrics, logs, and traces from various data sources including Prometheus and Elasticsearch.
    The ELK stack (Elasticsearch, Logstash, Kibana) is another widely used combination:
        Elasticsearch: Stores and indexes logs for fast retrieval.
        Logstash: Collects, processes, and enriches log data before sending it to Elasticsearch.
        Kibana: Provides visualization and querying capabilities for log data stored in Elasticsearch.

In summary, logging captures detailed records of system activities, metrics provide quantitative measurements for performance analysis, distributed tracing helps in understanding transaction flows across distributed systems, and monitoring tools like Grafana and ELK stack facilitate visualization and analysis of these data points to ensure optimal system operation and reliability.

24. How would you design a system for data analytics and reporting?
Designing a system for data analytics and reporting involves several essential components to ensure effective data collection, processing, analysis, and presentation:

Data Pipelines:

    Automated structures that manage the flow of data from various sources to a destination for storage and analysis.
    Stages include ingestion, processing, transformation, and loading of data.

Example Workflow:

    Data Ingestion: Extract data from CRM systems, databases, and APIs.
    Data Preparation: Clean and transform data using ETL tools for structured formats.
    Data Storage: Store processed data in cloud-based data warehouses like AWS Redshift.
    Analysis: Perform exploratory data analysis (EDA) and statistical analysis using tools like Python (Pandas, NumPy).
    Visualization: Create interactive dashboards with tools such as Tableau for business insights.
    Reporting: Automate generation and distribution of performance reports.

ETL Process:

    Extract: Gather data from various sources including databases, APIs, and files.
    Transform: Clean, validate, and structure data to ensure consistency and readiness for analysis.
    Load: Store transformed data in a data warehouse or data mart for querying and reporting.

Data Warehouse (e.g., AWS Redshift):

    Centralized repository optimized for storing structured, historical data from multiple sources.
    Supports efficient querying and analysis, integral to business intelligence (BI) and reporting.

AWS Redshift Features:

    Stores data in columnar format for enhanced query performance.
    Distributes and processes queries across multiple nodes for scalability with large datasets.
    Integrates seamlessly with BI tools like Tableau, Power BI, and Looker.
    Employs compression techniques for storage efficiency and automatic optimizations for query performance.

In summary, designing a data analytics and reporting system involves building robust data pipelines, leveraging ETL processes for data transformation, storing data in optimized warehouses like AWS Redshift, and utilizing BI tools for visualization and reporting to empower informed decision-making.

25. Describe the concept of load balancing and different algorithms used.

Load balancing distributes incoming network traffic across multiple servers to optimize resource utilization, maximize throughput, and minimize response time. Algorithms include round-robin, least connections, and IP hash.

26. How would you design a system to handle streaming video content?

Designing a system to handle streaming video content involves several key components and considerations to ensure scalability, reliability, and performance. Here’s a detailed breakdown of how such a system could be structured:

    1. Client-side Application:

        User Interface (UI): Includes playback controls, user preferences, and access to content.
        Media Player: Responsible for decoding and rendering video/audio streams.
        Streaming Protocol Support: Typically HTTP-based protocols like HLS (HTTP Live Streaming), MPEG-DASH (Dynamic Adaptive Streaming over HTTP), or proprietary protocols.

    2. Content Ingestion:

        Encoder/Transcoder: Converts raw video/audio into suitable formats and bitrates for streaming.
        Packager: Segments encoded media into small chunks (e.g., 2-10 seconds) and creates manifest files (e.g., .m3u8 for HLS, .mpd for MPEG-DASH).

    3. Content Storage:

        Origin Servers: Store and serve the original, encoded, and segmented media files.
        Content Delivery Network (CDN): Distributes content closer to end-users for faster delivery and reduced server load.

    4. Content Delivery:

        Edge Servers (CDN Edge): Cache and deliver media segments to users based on geographical proximity.
        Load Balancers: Distribute incoming requests across multiple servers to optimize performance and reliability.
        Quality of Service (QoS): Ensures reliable delivery with adaptive bitrate streaming based on network conditions.

    5. Streaming Protocols and Formats:

        HTTP Live Streaming (HLS): Apple’s adaptive streaming protocol widely supported on iOS and browsers.
        MPEG-DASH: Industry-standard for adaptive streaming, supported by a wide range of devices.
        RTMP (Real-Time Messaging Protocol): Older protocol, often used for live streaming.

    6. Security:

        Digital Rights Management (DRM): Protects content from unauthorized access and piracy.
        Encryption: Secures content during transmission (e.g., HTTPS, DRM-specific encryption).

    7. Analytics and Monitoring:

        Playback Analytics: Track user engagement, viewing habits, and quality of experience (QoE).
        Server Monitoring: Monitor server health, bandwidth usage, and CDN performance.

    8. Global Scalability:

        Multi-Region Deployment: Distribute servers across different regions to reduce latency and increase reliability.
        Auto-scaling: Automatically adjust server capacity based on traffic demands.

    9. Resilience and Fault Tolerance:

        Redundancy: Duplicate critical components to ensure uninterrupted service in case of failures.
        Backup and Recovery: Regular backups of content and configuration settings.

    10. Content Management:

        Metadata Management: Store and manage metadata related to videos (e.g., title, description, tags).
        Content Versioning: Manage different versions of videos, updates, and archival.

Example Workflow:

    Content Ingestion: Raw video is encoded into multiple bitrates and segmented.
    Content Storage: Segmented videos and manifests are stored on origin servers and replicated to CDN edge servers.
    Content Delivery: User requests are routed to the nearest edge server, which delivers the appropriate video segments based on available bandwidth and device capabilities.
    Security: Encrypted communication (HTTPS) and DRM protect content during transmission and playback.
    Analytics: Monitor user interactions and server performance for insights and optimizations.

Considerations:

    Bandwidth Optimization: Adapt streaming quality based on network conditions.
    Device Compatibility: Support for various devices and platforms (e.g., mobile, desktop, smart TVs).
    Latency: Minimize delay between live events and user playback.
    Regulatory Compliance: Adhere to local regulations regarding content delivery and storage.

Discuss content delivery networks (CDNs), video encoding/transcoding, adaptive bitrate streaming (ABR), and scalable storage solutions.

27. Explain the concept of sharding in database design.

Sharding involves horizontal partitioning of data across multiple databases to improve scalability and performance. Discuss sharding strategies and challenges (e.g., distributed joins).

28. How do you handle versioning in APIs?

Discuss URL versioning, header versioning, and best practices for maintaining backward compatibility while introducing new features.

29. Explain the concept of CDN caching and cache eviction policies.

Discuss cache eviction policies like Least Recently Used (LRU), Least Frequently Used (LFU), and Time-To-Live (TTL) for maintaining cache freshness.

30. Describe the use of proxies in system design.

Proxies intercept requests from clients and forward them to servers. Discuss types (forward, reverse, load balancing), benefits, and considerations.

31. How would you design a system to handle concurrent users?

Discuss techniques like connection pooling, thread pooling, rate limiting, and distributed locking mechanisms.

32. Explain the role of a message broker in system architecture.

A message broker facilitates communication between applications by enabling asynchronous message queuing and routing. Discuss brokers like RabbitMQ or ActiveMQ.

33. How do you design a system for logging and monitoring?

Discuss centralized logging (e.g., using ELK stack), metrics collection, alerting (e.g., using Prometheus and Grafana), and log aggregation.


34. Describe the use of API gateways in microservices architecture.

API gateways provide a single entry point for clients to access multiple microservices. They handle authentication, routing, load balancing, and protocol translation.
