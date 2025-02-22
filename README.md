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
