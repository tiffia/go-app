# go-app
Creating a simple go-app and integrating with k3s

## Step By Step Guide

We’ll go step by step, starting with writing a simple Go app in GitHub Codespaces. Once that’s working, we’ll move on to containerization and deploying it with K3s.

### Step 1: Setting Up GitHub Codespaces for Go Development
Before we write any code, let’s set up Codespaces to support Go development.
1. Open Your GitHub Repository in Codespaces
• Go to your GitHub repository.
• Click the “Code” button.
• Select “Codespaces” and click “Create codespace on main”.
2. Install Go (if not pre-installed)
Once inside Codespaces, check if Go is installed by running:
    ```bash 
    go version 
    ```
    If it’s not installed, install it with:

    ```bash
    sudo apt update && sudo apt install -y golang
    # sudo apt update: 
    # sudo: Runs the command with superuser (root) privileges.
    # apt update: Updates the package lists for the APT package manager. This ensures that you have the latest information about available packages and their versions.
    # apt install: Installs the specified package(s).
    # -y: Automatically answers "yes" to any prompts during the installation process.
    # golang: The package name for the Go programming language.
    ```

3. Create a New Go Project
Run the following commands to initialize a new Go project:
    ```bash
    mkdir go-app && cd go-app

    go mod init github.com/your-username/go-app
    # go mod init: This is the Go command to initialize a new module.
    # github.com/tiffia/go-app: This is the module path, which typically corresponds to the repository URL where the module will be hosted. It is a unique identifier for your module.
    ```
Replace your-username with your GitHub username.

4. Write a Simple Go Web App
Create a new file called main.go:
    ```bash
    touch main.go
    # touch: This is a standard Unix/Linux command used to create an empty file or update the timestamp of an existing file.
    code main.go
    ```
    Copy the following code into main.go:
    ```go
        package main
        import (
        "fmt"
        "net/http"
        )
        func handler(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, K3s!")
        }
        func main() {
        http.HandleFunc("/", handler)
        fmt.Println("Starting server on :8080")
        http.ListenAndServe(":8080", nil)
        }
    ```
    #### Code Explanation:
    * package main: This declares that the file belongs to the main package, which is the entry point for a Go executable program.
    * import: This statement imports the fmt and net/http packages. The fmt package is used for formatted I/O operations, and the net/http package provides HTTP client and server implementations.
    * Handler Function:
        * func handler(w http.ResponseWriter, r *http.Request): This defines a handler function named handler that takes two parameters:
        * w http.ResponseWriter: Used to write the HTTP response.
        * r *http.Request: Represents the incoming HTTP request.
        * fmt.Fprintf(w, "Hello, K3s!"): This writes the string "Hello, K3s!" to the HTTP response.
    func main(): This is the main function, which is the entry point of the program.
        * http.HandleFunc("/", handler): This registers the handler function to handle requests to the root URL path ("/").
        *fmt.Println("Starting server on :8080"): This prints a message to the console indicating that the server is starting on port 8080.
        * http.ListenAndServe(":8080", nil): This starts the HTTP server on port 8080. The second parameter is nil, which means the default ServeMux is used to handle incoming requests.
Summary:
When you run this Go application, it starts an HTTP server on port 8080. Any HTTP request made to http://localhost:8080/ will be handled by the handler function, which responds with "Hello, K3s!". The server will continue running and listening for incoming requests until it is stopped.
5. Run the Go App
    
    To run the app, use:
    ```go
    go run main.go
    ```
Then, open the preview URL in Codespaces to see your app in action.

### Step 2: Containerizing the Go App with Docker
Before deploying to K3s, we need to containerize the app.
1. Create a Dockerfile
Inside your go-app directory, create a new file named Dockerfile:
    ```bash
        touch Dockerfile
        code Dockerfile
    ```
2. Write the Dockerfile
Copy and paste the following into Dockerfile:
    ```docker
    # Use the official Golang image as a builder
    FROM golang:1.23.4 AS builder
    # Set the working directory inside the container
    WORKDIR /app
    # Copy the Go module files and download dependencies
    COPY go.mod go.sum ./
    RUN go mod download
    # Copy the rest of the application code
    COPY . .
    # Build the Go application
    RUN go build -o go-app
    # Use a smaller base image for the final container
    FROM debian:bookworm-slim
    # Set the working directory in the new container
    WORKDIR /app
    # Copy the compiled binary from the builder
    COPY --from=builder /app/go-app .
    # Expose the port the app runs on
    EXPOSE 8080
    # Run the application
    CMD ["./go-app"]
    ```

    In the provided Dockerfile, the WORKDIR /app command sets the working directory inside the container to /app. This means that any subsequent COPY, RUN, or other commands will be executed relative to this directory.

#### Breakdown:
* Set the Working Directory: `WORKDIR /app`
    
    This command sets the working directory inside the container to /app. If the directory does not exist, it will be created.

* Copy Go Module Files: `COPY go.mod go.sum ./`

    This command copies the go.mod and go.sum files from your local machine (the build context) to the /app directory inside the container. The gh-terra-goapp at the end of the command specifies the current working directory inside the container, which is /app due to the previous WORKDIR command.

* Download Dependencies: `RUN go mod download`

    This command runs go mod download inside the container, which downloads the dependencies specified in the go.mod file to the module cache inside the container.

* Copy the Rest of the Application Code: `COPY . .`

This command copies all the remaining files and directories from your local machine (the build context) to the /app directory inside the container. The first . refers to the current directory on your local machine, and the second . refers to the current working directory inside the container (/app).

Local Files:
go.mod and go.sum: These files should be located in the root directory of your Go project on your local machine. When you run the docker build command, the current directory (where the Dockerfile is located) is used as the build context, and these files are copied from there into the container.
Example Directory Structure:
```
/workspaces/gh-terra-goapp
├── Dockerfile
├── go.mod
├── go.sum
├── main.go
└── other_files_and_directories
```
Assuming your project directory on your local machine looks like this:

When you run the `docker build` command, the `go.mod` and `go.sum` files will be copied from gh-terra-goapp on your local machine to `/app` inside the container.

Summary:
The `WORKDIR /app` command sets the working directory inside the container to `/app`.
The `COPY go.mod go.sum ./` command copies the go.mod and go.sum files from your local machine to the /app directory inside the container.
The `COPY . .` command copies the rest of the application code from your local machine to the /app directory inside the container.
This setup ensures that the Go module files and the rest of your application code are available in the /app directory inside the container, allowing the build process to proceed correctly.

### Step 3: Build and Run the Docker Container
1. Build the Docker image:
    `docker build -t go-app .`
    
    If this throws error that there is no file as `go.sum`, ensure that the external package is added: 
    `go get github.com/gorilla/mux`
    `go mod tidy`

    then rebuild the app.

2. Run the container:
`docker run -p 8080:8080 go-app`

3. Access the app:
• Like before, forward port 8080 in Codespaces.
• Open the browser and verify that the app still displays “Hello, K3s!”.