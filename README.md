# go-gin-web
gin是golang官方支持的web框架，该项目翻译了golang官方的教程：https://go.dev/doc/tutorial/web-service-gin
上传了完整demo

<a name="d6t1c"></a>
# 官方文档
[https://go.dev/doc/tutorial/web-service-gin](https://go.dev/doc/tutorial/web-service-gin)<br />本文介绍了如何使用web框架Gin（[https://gin-gonic.com/docs/](https://gin-gonic.com/docs/)）实现RESTful API的基本知识
<a name="AeZpq"></a>
# 介绍
Gin简化了许多与构建web应用程序（包括web服务）相关的编码任务。在本教程中，您将使用Gin来实现对请求的路由，获取请求详细信息，并封装JSON数据返回给浏览器。<br />本教程的例子是一个关于老式爵士乐唱片的数据存储库。<br />本文使用的例子需要Go 1.16及以上版本支持。
<a name="p6pUE"></a>
# 开始
<a name="bFbPT"></a>
## 一、Design API endpoints(设计API接口)
我们的需求是提供一个老式爵士乐唱片的数据存储库服务，因此需要提供添加和获取唱片的API，api设计的原则应该保持简单易懂，这样更容易满足用户。<br />下面是我们要设计的两个api：
<a name="Q5MQb"></a>
### 1、/albums

- GET – 以JSON的方式返回所有的唱片数据.
- POST – 客户端已JSON的数据格式添加一个唱片.
<a name="akZBX"></a>
### 2、/albums/:id

- GET – 根据唱片id获取详细的唱片内容，以JSON方式返回.
<a name="MSCbC"></a>
## 二、创建项目
我们可以通过命令行工具进行创建，使用mkdir创建一个文件夹，然后在该文件夹下使用go mod init go-gin-web命令进行工程初始化即可。这里我使用的是Goland 编辑器进行创建。<br />本案例我们使用内存的方式存储数据，因此每次重启服务之后，唱片数据会被清掉。
<a name="FNyVX"></a>
## 三、创建数据
<a name="hcxna"></a>
### 1 创建主程序文件
在go-gin-web文件下创建文件main.go，注：package声明为main
<a name="cjjLv"></a>
### 2 创建唱片结构体
在main.go的包声明下创建一个唱片结构体，用于存储唱片信息，如下：
```go
// album represents data about a record album.
type album struct {
	ID     string  `json:"id"`
	Title  string  `json:"title"`
	Artist string  `json:"artist"`
	Price  float64 `json:"price"`
}
```
<a name="txUig"></a>
### 3 初始化唱片信息
在唱片结构体下面创建一个album结构体切片，并初始化三条唱片信息
```go
// albums slice to seed record album data.
var albums = []album{
	{ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
	{ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
	{ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
}
```
<a name="skFDM"></a>
## 四、唱片列表的处理器
当客户端通过http的get方法访问/albums路径时，我们需要一个处理该请求的数据，返回所有的唱片列表信息。
<a name="Re2pA"></a>
### 1、下载gin依赖包
在go.mod同级目录下，使用执行以下命令下载gin。命令执行完毕后，可以看到go.mod文件中增加了很多依赖。
```go
go get -u github.com/gin-gonic/gin
```
<a name="DBFcm"></a>
### 2、引入依赖包
在main.go 中添加gin和net/http依赖
```go
import (
    "github.com/gin-gonic/gin"
    "net/http"
)
```
<a name="ezYSS"></a>
### 3、编写代码
创建一下函数getAlbums函数，该函数将albums切片的数据封装为json数据，返回给respnse，并设置http状态码为StatusOK（200）。
```go
// getAlbums responds with the list of all albums as JSON.
func getAlbums(c *gin.Context) {
    c.IndentedJSON(http.StatusOK, albums)
}
```
�在main方法中编写以下代码：
```go
func main() {
    // 使用默认方法初始化一个路由器
    router := gin.Default()
    // 在路由中设置/albums 路径get方法的处理器：getAlbums
    router.GET("/albums", getAlbums)
    // 启动web服务
    router.Run("localhost:8080")
}
```
<a name="IrINA"></a>
### 4、运行代码
如果是命令行，则在main.go文件同级目录以下执行命令：
```go
go run .
// 或者 go run main.go
```
在浏览器中输入：[http://localhost:8080/albums](http://localhost:8080/albums)，将看到以下数据：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12382373/1679149646257-4c234b52-4b5f-49d1-aa1f-6221a2527724.png#averageHue=%23fdfdfd&clientId=u46fa518a-d8ae-4&from=paste&height=430&id=u867ee055&name=image.png&originHeight=860&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=96803&status=done&style=none&taskId=uf053e9f9-3030-4ca6-a595-4a753ba3f3c&title=&width=540)
<a name="hDj5K"></a>
## 五、创建唱片的处理器
<a name="eDML5"></a>
### 1、编写代码
编写postAlbums函数，作为处理器处理添加唱片的请求。<br />c.BindJSON(&newAlbum)将收到的JSON数据写入newAlbum对象
```go
// postAlbums adds an album from JSON received in the request body.
func postAlbums(c *gin.Context) {
    var newAlbum album

    // Call BindJSON to bind the received JSON to newAlbum.
    if err := c.BindJSON(&newAlbum); err != nil {
        return
    }

    // Add the new album to the slice.
    albums = append(albums, newAlbum)
    c.IndentedJSON(http.StatusCreated, newAlbum)
}
```
在main方法中添加以下代码，设置路由：
```go
router.POST("/albums", postAlbums)
```
<a name="Csjhf"></a>
### 2、运行代码
运行代码，启动服务器之后，使用以下命令添加一个唱片（浏览器不支持直接访问post请求）：
```shell
curl http://localhost:8080/albums \
--include \
--header "Content-Type: application/json" \
--request "POST" \
--data '{"id": "4","title": "The Modern Sound of Betty Carter","artist": "Betty Carter","price": 49.99}'
```
请求成功的话，会看到以下结果：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12382373/1679150192907-99dfe147-eadb-45e3-a55d-602bd1250166.png#averageHue=%23292625&clientId=u46fa518a-d8ae-4&from=paste&height=235&id=u791436fc&name=image.png&originHeight=470&originWidth=1724&originalType=binary&ratio=2&rotation=0&showTitle=false&size=91052&status=done&style=none&taskId=u3ef8220e-c909-4320-8d9f-f788ce2b470&title=&width=862)<br />此时访问[http://localhost:8080/albums](http://localhost:8080/albums)，回返回4条数据。
<a name="WZCUh"></a>
## 六、唱片详情处理器
<a name="rHqXt"></a>
### 1、编码
同上，创建getAlbumByID函数，用以处理详情查询请求：
```go
// getAlbumByID locates the album whose ID value matches the id
// parameter sent by the client, then returns that album as a response.
func getAlbumByID(c *gin.Context) {
    id := c.Param("id")

    // Loop over the list of albums, looking for
    // an album whose ID value matches the parameter.
    for _, a := range albums {
        if a.ID == id {
            c.IndentedJSON(http.StatusOK, a)
            return
        }
    }
    c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
}
```
在main方法中添加以下代码，将getAlbumByID设置为/albums/:id路径的处理器
```go
router.GET("/albums/:id", getAlbumByID)
```
<a name="Xs3Pl"></a>
### 2、运行代码
运行代码，在浏览器中输入：[http://localhost:8080/albums/2](http://localhost:8080/albums/2)，将返回以下结果：![image.png](https://cdn.nlark.com/yuque/0/2023/png/12382373/1679150587974-7fdbbc07-fb8f-490f-88b8-8083c3ba44a5.png#averageHue=%23fcfcfc&clientId=u46fa518a-d8ae-4&from=paste&height=173&id=u48f0b32c&name=image.png&originHeight=346&originWidth=558&originalType=binary&ratio=2&rotation=0&showTitle=false&size=27445&status=done&style=none&taskId=uf3102984-15fe-41af-8b87-d6775099f56&title=&width=279)
<a name="V3A46"></a>
# 总结
以上便完成了一个支持查询和添加唱片信息的最基础的RESTful api风格的web服务。<br />想要跟深入了解Gin，可以访问：[https://gin-gonic.com/docs/quickstart/](https://gin-gonic.com/docs/quickstart/)
<a name="F2lUR"></a>
## 和net/http框架对比：
```go
// net/http
func main() {
	// 1、设置对应路径的处理器
	http.HandleFunc("/albums/list/", editHandler)
	http.HandleFunc("/albums/add/", editHandler)
	http.HandleFunc("/albums/detail", editHandler)
	// 2、启动服务器
	http.ListenAndServe(":8080", nil)
}

// gin
func main() {
	router := gin.Default()
	router.GET("/albums", getAlbums)
	router.GET("/albums/:id", getAlbumByID)
	router.POST("/albums", postAlbums)
	router.Run("localhost:8080")
}
```
可以看到，在最基本的demo中，gin的路由器支持率对同一路径（如：/albums）不同方法（GET/POST）的处理器设置。net/http则不支持，需要自行实现对http方法的解析处理。
<a name="VVEzL"></a>
# 完整代码
```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

// album represents data about a record album.
type album struct {
	ID     string  `json:"id"`
	Title  string  `json:"title"`
	Artist string  `json:"artist"`
	Price  float64 `json:"price"`
}

// albums slice to seed record album data.
var albums = []album{
	{ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
	{ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
	{ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
}

func main() {
	router := gin.Default()
	router.GET("/albums", getAlbums)
	router.GET("/albums/:id", getAlbumByID)
	router.POST("/albums", postAlbums)
	router.Run("localhost:8080")
}

// getAlbums responds with the list of all albums as JSON.
func getAlbums(c *gin.Context) {
	c.IndentedJSON(http.StatusOK, albums)
}

// postAlbums adds an album from JSON received in the request body.
func postAlbums(c *gin.Context) {
	var newAlbum album

	// Call BindJSON to bind the received JSON to newAlbum.
	if err := c.BindJSON(&newAlbum); err != nil {
		return
	}

	// Add the new album to the slice.
	albums = append(albums, newAlbum)
	c.IndentedJSON(http.StatusCreated, newAlbum)
}

// getAlbumByID locates the album whose ID value matches the id
// parameter sent by the client, then returns that album as a response.
func getAlbumByID(c *gin.Context) {
	id := c.Param("id")

	// Loop over the list of albums, looking for
	// an album whose ID value matches the parameter.
	for _, a := range albums {
		if a.ID == id {
			c.IndentedJSON(http.StatusOK, a)
			return
		}
	}
	c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
}

```
