Continuous delivery has become an essential part of modern software development. It’s a methodology that automates the entire software delivery pipeline, from code changes to deployment. With continuous delivery, teams can release software faster and with more confidence, all while reducing the risk of errors and failures.

I’ve already talked about how Go has become one of the most popular languages for developing cloud-native applications in a previous article. As such, when you combine it with a continuous delivery platform, you have a powerful combination at your disposal. One such platform that caught my eye is GoCD.

GoCD is a popular open-source continuous delivery tool that provides developers with a powerful platform to deliver quality software quickly and reliably, all the while focusing on helping them visualize the entire process. **This makes it an excellent tool for beginners.**

Before we move on to the more practical side of things, let’s explore some of GoCD’s main features.

## What makes GoCD stand out?

GoCD is designed to be highly configurable and supports a wide range of use cases, from simple to complex. It provides a **visual** pipeline designer, which makes it easy to create and modify pipelines using a drag-and-drop interface. You can easily integrate GoCD with popular development tools such as GitHub, Bitbucket, and Jenkins.

One of GoCD’s standout features is its ability to manage complex deployments with ease. It supports deployment of multiple applications with multiple versions, and allows for customized deployment strategies, such as [blue-green](https://docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/bluegreen-deployments.html) and [canary](https://semaphoreci.com/blog/what-is-canary-deployment) deployments. GoCD also includes an advanced artifact management system, which helps to track and manage artifacts across pipelines and environments.

GoCD also supports test automation by providing a suite of testing tools and frameworks, such as JUnit, NUnit, and Selenium. It allows for parallel execution of tests, which can speed up the testing process, and integrates with popular testing frameworks like Cucumber and Spock.

In terms of security, GoCD offers a variety of authentication and authorization options, including LDAP, OAuth, and SAML. It also supports role-based access control, which enables fine-grained control over user permissions.

When compared to its competitors, such as Jenkins and CircleCI, GoCD stands out for its flexibility and configurability. While Jenkins has a vast library of plugins, GoCD has a more intuitive and user-friendly interface. CircleCI, on the other hand, is a cloud-based solution that limits customization options.

Aditionally, GoCD runs **natively** on Kubernetes, which means that it can provision and scale build infrastructure dynamically.

Now that we have an understanding of what GoCD is and its main features, let’s dive in and create a simple pipeline for a Go application.

# Creating a pipeline with GoCD

This guide assumes that you have a basic idea of how Go applications are created. As such, you’ll need to have a recent version of Go installed on your system.

## Installation

The first step is to [install](https://www.gocd.org/download/) GoCD on your system. GoCD consists of two components:

1. The GoCD Server, which configures and manages pipelines.
2. GoCD Agents, which execute tasks sent by the GoCD server. Agents can be installed on multiple systems.

In order to get started, you’ll need to install both the GoCD Server and GoCD Agent on your system.

## Verifying the installation

Once you’ve installed both components, you should be able to access the dashboard on `http://localhost:8153` and see something like this:

![](EyC0VGmIacAHAL-dlgKc9Q.webp)

You can verify that the GoCD Agent was installed successfully by clicking the ‘Agents’ tab at the top. You should you see your current system listed in the table.

![](rybFgI4KPIMY6YUVUwXCug.webp)
GoCD Agents that have been added

## Creating a simple application

As always, I’ll create a simple web server. I’ll start by intializing a new Git repository and creating a new Go module inside:

```sh
git init  
go mod init go-gocd-demo
```

Next, I’ll add the code for the web server inside my `main.go` file:

```go
package main  
  
import (  
 "fmt"  
 "net/http"  
)  
  
func main() {  
 http.HandleFunc("/", HelloServer)  
 fmt.Println("The server is listening...")  
 http.ListenAndServe(":8080", nil)  
}  
  
func HelloServer(writer http.ResponseWriter, request *http.Request) {  
 fmt.Fprintf(writer, "Hello, World!")  
}
```

I can test my application by the running the `go run` command:

`go run main.go`

If all goes well, I should be able to see a “Hello, World!” message pop up on `http://localhost:8080`.

## Creating the test file

I’ll be creating a simple test function to test the web server inside the `main_test.go` file:

```go
package main  
  
import (  
 "net/http"  
 "net/http/httptest"  
 "testing"  
)  
  
func TestHelloServer(t *testing.T) {  
 req, err := http.NewRequest("GET", "/", nil)  
 if err != nil {  
  t.Fatal(err)  
 }  
  
 rr := httptest.NewRecorder()  
 handler := http.HandlerFunc(HelloServer)  
  
 handler.ServeHTTP(rr, req)  
  
 if status := rr.Code; status != http.StatusOK {  
  t.Errorf("handler returned wrong status code: got %v want %v",  
   status, http.StatusOK)  
 }  
  
 expected := "Hello, World!"  
 if rr.Body.String() != expected {  
  t.Errorf("handler returned unexpected body: got %v want %v",  
   rr.Body.String(), expected)  
 }  
}
```

This test function sends a mock HTTP request to the server and checks if the response status code and body match the expected values. We can run this test using the `go test` command:

`go test -v`

The output should look something like this:

![](eBA8KBtt6mOrbZ2toeulRQ.webp)

Test output

I’ll now push all the files to my GitHub repository:

```sh
git add .  
git commit -m Add files  
git remote add origin https://github.com/username/repo-name.git  
git push -u origin main
```

Make sure to replace `username` and `repo-name` with your GitHub username and repository name respectively.

## Creating the pipeline

Let’s head over to the GoCD dashboard now. The first step is to add the URL of the GitHub repository we just pushed our files to:

![](UaNkvwOSF8K8WruHDx6PMQ.webp)

Next, we’ll give give our pipeline a name. I’m going to go with `go-gocd-demo` to keep things consistent.

After that, we’re going to go add a name for our first stage. Since we’re simply going to be testing our application, I’m going to name my stage `test`.

![](vMS6n-T9V8Iu-6T6CZQZEw.webp)

Adding the pipeline and stage names

Let’s now add a job. I’m going to name the job `run-test` and add the `go test` command in the script:

![](g_raq64lOlgN2-ocxQ-6A.webp)
Adding a job

Once you hit the “Save + Run This Pipeline” button, you’ll be brought back to the dashboard and should see the pipeline that we just created. After a couple of seconds, it should have green check mark at the bottom, indicating that the test stage has run successfully.

![](RcLV1rrFNTWMV24JdBIdwg.webp)

Let’s now break things by modify the code in our `main.go` file. I’ll change the output from “Hello, World!” to just “Hello!” as so:

```
...  
func HelloServer(writer http.ResponseWriter, request *http.Request) {  
 fmt.Fprintf(writer, "Hello!")  
}
```

As soon as I push my changes, the GoCD server detects that the code inside the repository has been modified and triggers the pipeline. This time, however, you’ll notice that the pipeline no longer has the green check mark at the bottom:

![](Qtyx-9nyjis9t6o99NYgrw.webp)

As expected, we can see that the test stage has failed.

You should now have a good idea of how simple it is to create pipelines with GoCD.