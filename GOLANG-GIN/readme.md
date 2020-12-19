## Why to Build a REST API with GO?

Go is the trending programming language currently. Go is very simple but offers similar performance to those low level languages like C++. Go is referred to be the fastest programming language that offers very high performance. 

## What’s GIN?

Gin is one of the incredible frameworks of Go, which is lightweight and massively fast. The interesting feature of Gin is that it provides a custom version of HttpRouter which makes the API routes extremely faster, than that of other Go frameworks. It is claimed to be 40 times faster than another Go framework called Martini and of course, it provides a lot more benefits compared to other frameworks and other programming languages.
Gin is just the microframework which does not provide you a lot of features as of other frameworks. It just provides you with necessary tools to build api, such as routing, validation, etc. Experts believe that Gin is best for long term investment, to grab proper advantage of its high performance and flexibility.

## We will be creating a very simple REST API using 
 - Go Programming Language
 - Gin Framework
 - Gorm Package
 - Mysql Database
 
We will be performing the CRUD operation for Book with few fields. 

## Prerequisites 
 - GO Installation 
 - Mysql Installation
 - You need to have clear knowledge of Go, to get started with the project that we are going to do.

### Let’s Go!

Wow it sounds cool with GO, isn’t it?

## Install Go

You will find a lot of materials teaching you how to install Go in your machine. I believe that you have already installed Go in your machine because we are going to write the intermediate code for Go. Even if you have not installed it yet, you can install it now. I have a very helpful article for you to install Go on your environment.
Whenever you have completed installation of Go or you have Go on your machine, please test whether it is working or not, by simply checking the version.

```
go version
```

## Create New Project

Create project folder GO-GIN, then download the script from Github and place into your local project folder then run below command

```
go mod init
```

This command creates the go.mod file while holding our external packages, if you open it up later on.

## Install Packages

These are the external packages for the project: sql driver for 
- mysql
- gin
- gorm

Enter the following command to install it in the project folder ( GO-GIN ).

```
go get github.com/go-sql-driver/mysql
go get github.com/gin-gonic/gin
go get github.com/jinzhu/gorm
```

## Project Structure
Certainly, we follow the MVC pattern for creating the API that is Model, View and Controller. But ‘V’ remains silent here because we are creating an API now. We have database configuration, model, routes, controller and of course our all time favorite Main file which is the starter of our project. I have the snapshot for creating the structure of the project.


## Database Setup
First of all, make sure you have mysql installed on the machine and some tools like phpmyadmin, mysql workbench or any other tools to use your database efficiently. 

Create a new database in mysql, then we add the database configuration in the Database.go file inside the Config directory. 
I have added the configuration details of mine and you have to add your own.

```
//Config/Database.go package Config

package Config

import (
	"fmt"

	"github.com/jinzhu/gorm"
)

var DB *gorm.DB

// DBConfig represents db configuration
type DBConfig struct {
	Host     string
	Port     int
	User     string
	DBName   string
	Password string
}

func BuildDBConfig() *DBConfig {
	dbConfig := DBConfig{
		Host:     "localhost",
		Port:     3306,
		User:     "root",
		Password: "",
		DBName:   "book-store",
	}
	return &dbConfig
}

func DbURL(dbConfig *DBConfig) string {
	return fmt.Sprintf(
		"%s:%s@tcp(%s:%d)/%s?charset=utf8&parseTime=True&loc=Local",
		dbConfig.User,
		dbConfig.Password,
		dbConfig.Host,
		dbConfig.Port,
		dbConfig.DBName,
	)
}

```

## Create Model
Now, let’s create a model for Book that consists following fields:
Title
Author
Description
Price
Isbn
Release_date
We create a BookModel.go file inside our Models directory.

```
//Models/BookModel.gopackage Models

type Book struct {
	Id           uint      `json:"id"`
	Title        string    `json:"title"`
	Author       string    `json:"author"`
	Description  string    `json:"description"`
	Price        float64   `json:"price"`
	Isbn         string    `json:"isbn"`	
}

func (b *Book) TableName() string {
	return "book"
}
```

Configure Routing

Similarly, we create the routing for our project. We create a group to adjust our routing related to books and we have to take care of the group while consuming the api.
So, let’s create Routes.go file inside the Routes folder. We will see the implementation of the controller functions below.

```
//Routes/Routes.gopackage Routes

package Routes

import (
	"first-api/Controllers"

	"github.com/gin-gonic/gin"
)

//SetupRouter ... Configure routes
func SetupRouter() *gin.Engine {
	r := gin.Default()
	grp1 := r.Group("/book-store")
	{
		grp1.GET("book", Controllers.GetBooks)
		grp1.POST("book", Controllers.CreateBook)
		grp1.GET("book/:id", Controllers.GetBookByID)
		grp1.PUT("book/:id", Controllers.UpdateBook)
		grp1.DELETE("book/:id", Controllers.DeleteBook)
	}
	return r
}

```


## Create Controller
We handle http requests coming from the front end in the controller. We create different functions that handle our specific requests routed to the controller by our router. 
We have Book.go inside Models to interact with the database. We respond to the book according to the data we get from our database. If we get no error, we supply a response as StatusOK and if we get the error, we supply error status. We create a Book.go file inside the Controllers folder.

```
//Controllers/Book.go package Controllers

package Controllers

import (
	"first-api/Models"
	"fmt"
	"net/http"

	"github.com/gin-gonic/gin"
)

//GetBooks ... Get all books
func GetBooks(c *gin.Context) {
	var book []Models.Book
	err := Models.GetAllBooks(&book)
	if err != nil {
		c.AbortWithStatus(http.StatusNotFound)
	} else {
		c.JSON(http.StatusOK, book)
	}
}

//CreateBook ... Create Book
func CreateBook(c *gin.Context) {
	var book Models.Book
	c.BindJSON(&book)
	err := Models.CreateBook(&book)
	if err != nil {
		fmt.Println(err.Error())
		c.AbortWithStatus(http.StatusNotFound)
	} else {
		c.JSON(http.StatusOK, book)
	}
}

//GetBookByID ... Get the book by id
func GetBookByID(c *gin.Context) {
	id := c.Params.ByName("id")
	var book Models.Book
	err := Models.GetBookByID(&book, id)
	if err != nil {
		c.AbortWithStatus(http.StatusNotFound)
	} else {
		c.JSON(http.StatusOK, book)
	}
}

//UpdateBook ... Update the book information
func UpdateBook(c *gin.Context) {
	var book Models.Book
	id := c.Params.ByName("id")
	err := Models.GetBookByID(&book, id)
	if err != nil {
		c.JSON(http.StatusNotFound, book)
	}
	c.BindJSON(&book)
	err = Models.UpdateBook(&book, id)
	if err != nil {
		c.AbortWithStatus(http.StatusNotFound)
	} else {
		c.JSON(http.StatusOK, book)
	}
}

//DeleteBook ... Delete the book
func DeleteBook(c *gin.Context) {
	var book Models.Book
	id := c.Params.ByName("id")
	err := Models.DeleteBook(&book, id)
	if err != nil {
		c.AbortWithStatus(http.StatusNotFound)
	} else {
		c.JSON(http.StatusOK, gin.H{"id" + id: "is deleted"})
	}
}

```
## Handle Requests
This is the crucial file which fetch data and interacts directly with our database. We create a Book.go file inside the Models folder to handle the database requests.

```
//Models/Book.go package Models

package Models

import (
	"first-api/Config"
	"fmt"

	_ "github.com/go-sql-driver/mysql"
)

//GetAllBooks Fetch all book data
func GetAllBooks(book *[]Book) (err error) {
	if err = Config.DB.Find(book).Error; err != nil {
		return err
	}
	return nil
}

//CreateBook ... Insert New data
func CreateBook(book *Book) (err error) {
	if err = Config.DB.Create(book).Error; err != nil {
		return err
	}
	return nil
}

//GetBookByID ... Fetch only one book by Id
func GetBookByID(book *Book, id string) (err error) {
	if err = Config.DB.Where("id = ?", id).First(book).Error; err != nil {
		return err
	}
	return nil
}

//UpdateBook ... Update book
func UpdateBook(book *Book, id string) (err error) {
	fmt.Println(book)
	Config.DB.Save(book)
	return nil
}

//DeleteBook ... Delete book
func DeleteBook(book *Book, id string) (err error) {
	Config.DB.Where("id = ?", id).Delete(book)
	return nil
}

```
## Setting UP Server

This is the starter function of our project. We connect mysql, auto migrate our modal and setup router from here. We have to create main.go in the root of the project.

```
//main.gopackage main

import (
"first-api/Config"
"first-api/Models"
"first-api/Routes"
"fmt""github.com/jinzhu/gorm"
)var err error

func main() {
Config.DB, err = gorm.Open("mysql", Config.DbURL(Config.BuildDBConfig()))if err != nil {
 fmt.Println("Status:", err)
}defer Config.DB.Close()
Config.DB.AutoMigrate(&Models.Book{})r := Routes.SetupRouter()
//running
r.Run()
}
```
Now, we are all set to test our first api using Go and Gin!
## Start Your Server

Run the following command for starting your server.
```
go run main.go
```
If you encountered any of the problems during serving, first fix them because you may have missed something in between.

## Endpoints

### These are the endpoints we will use to create, update, read and delete the book data.
- GET book-store/book → Retrieves all the Book data
- POST book-store/book → Add new Book data
- GET book-store/book/{id} → Retrieve the single Book data
- PUT book-store/book/{id} → Update the Book data
- DELETE book-store/book/{id} → Delete the Book data

Now, Open up the endpoint testing tools you have; we will be using Postman which is simply awesome. Please make sure that data is inserted in JSON format through Postman because we are not working with form-data in this module.

## Create New Book
Here, we will create two books simultaneously.




## Get ALL Books
Get all the available books in our database.


## Get Book By ID
Get the book information for the book having Id=1.


## Update The Book
Update the book information for the book having Id=1.


## Delete The Book
Delete the book having Id=2.


This is the very very simple rest api that we have built using Go, Gin, Gorm and Mysql. There is a lot more things you can do with Go. I will be in touch with you with some more articles related to Go in the coming days. If you have any confusions related to the article, please let me know in the comments below or if you are having trouble with covering it all, then I have the full version of code also in my Github repository. See you in the next article!
Stay tuned!
Github Repository: https://github.com/TravelXML/REST-API-WITH-PYTHON-PHP-NODEJS-GO-DJANGO-LARAVEL-LUMEN-Examples/tree/main/GOLANG-GIN
