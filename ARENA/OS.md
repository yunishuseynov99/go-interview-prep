# Go's `os` package

---

### 1. Interacting with the Program's Environment

These functions let you get information passed to your program when it starts.

#### `os.Args`
*   **What it is:** A slice of strings containing the command-line arguments your program was run with. `os.Args[0]` is always the name of the program itself.
*   **When to use it:** When you want your program to take input directly from the command line, like a filename or a configuration flag.
*   **Example:**
    ```go
    // Run this program with: go run main.go hello world
    package main

    import (
        "fmt"
        "os"
    )
    
    func main() {
        programName := os.Args[0]
        firstArg := os.Args[1]
        secondArg := os.Args[2]
        
        fmt.Println("Program Name:", programName)
        fmt.Println("First Argument:", firstArg)
        fmt.Println("Second Argument:", secondArg)
    }
    // Output:
    // Program Name: /var/folders/..../main
    // First Argument: hello
    // Second Argument: world
    ```

#### `os.Getenv` & `os.Setenv`
*   **What they do:** Get and set environment variables. These are variables set in your operating system, outside your Go program.
*   **When to use them:** This is the standard way to configure an application without hard-coding secrets. Use it for API keys, database connection strings, or port numbers.
*   **Example:**
    ```go
    // In your terminal before running: export API_KEY="abc-123"
    // Then run: go run main.go

    apiKey := os.Getenv("API_KEY")
    if apiKey == "" {
        fmt.Println("API_KEY environment variable not set!")
    } else {
        fmt.Println("API Key found:", apiKey)
    }
    // Output:
    // API Key found: abc-123
    ```

---

### 2. Working with Files

This is the most common use of the `os` package.

#### The `Open -> Defer Close` Pattern
A core concept: whenever you open a file, you **must** close it. The `defer file.Close()` pattern ensures the file is closed right after the function finishes, even if an error occurs.

#### `os.ReadFile` & `os.WriteFile` (The Easy Way)
*   **What they do:** These are modern, simple functions to read or write an *entire* file in one command.
*   **When to use them:** For small to medium-sized files. This is the **preferred method** if you don't need to process a huge file piece by piece.
*   **Example:**
    ```go
    // Write data to a file
    data := []byte("Hello, from Go!")
    err := os.WriteFile("output.txt", data, 0644) // 0644 is standard file permission
    if err != nil {
        panic(err)
    }

    // Read the entire file back
    readData, err := os.ReadFile("output.txt")
    if err != nil {
        panic(err)
    }
    fmt.Println(string(readData))
    // Output:
    // Hello, from Go!
    ```

#### `os.Create` (The Manual Way)
*   **What it does:** Creates a new file. If the file already exists, its contents are erased (truncated). It returns a `*os.File` object that you can write to.
*   **When to use it:** When you need to create a new file and write to it, possibly in chunks.
*   **Example:**
    ```go
    file, err := os.Create("manual.txt")
    if err != nil {
        panic(err)
    }
    defer file.Close() // Crucial: ensure the file is closed when we're done

    file.WriteString("This is a manually created file.")
    ```

#### `os.Stat`
*   **What it does:** Gets metadata (information) about a file or directory, such as its size, modification time, and permissions.
*   **When to use it:** To check if a file exists before trying to open it, or to get its size.
*   **Example:**
    ```go
    info, err := os.Stat("output.txt")
    if os.IsNotExist(err) {
        fmt.Println("File does not exist.")
    } else {
        fmt.Printf("File '%s' is %d bytes long.\n", info.Name(), info.Size())
    }
    // Output:
    // File 'output.txt' is 15 bytes long.
    ```

#### `os.Remove`
*   **What it does:** Deletes a file.
*   **When to use it:** To clean up temporary files or any file you no longer need.
*   **Example:**
    ```go
    err := os.Remove("manual.txt")
    if err != nil {
        fmt.Println("Error deleting file:", err)
    }
    ```

---

### 3. Working with Directories

#### `os.MkdirAll`
*   **What it does:** Creates a directory, including any necessary parent directories. For example, `os.MkdirAll("temp/logs", 0755)` will create `temp` if it doesn't exist, and then `logs` inside it.
*   **When to use it:** This is almost always what you want when creating a directory. The simpler `os.Mkdir` will fail if a parent directory is missing.
*   **Example:**
    ```go
    // Create a nested directory structure
    err := os.MkdirAll("config/production", 0755) // 0755 is standard dir permission
    if err != nil {
        panic(err)
    }
    fmt.Println("Directories created.")
    ```

#### `os.ReadDir`
*   **What it does:** Reads the contents of a directory and returns a list of its entries (files and subdirectories).
*   **When to use it:** When you need to list or iterate over the files in a folder.
*   **Example:**
    ```go
    entries, err := os.ReadDir(".") // "." means the current directory
    if err != nil {
        panic(err)
    }

    fmt.Println("Files in current directory:")
    for _, e := range entries {
        fmt.Println("-", e.Name())
    }
    ```

#### `os.RemoveAll`
*   **What it does:** Deletes a path and anything inside it (it's a recursive delete).
*   **When to use it:** To clean up an entire directory structure. **Warning: This is a destructive command. Use it with extreme care.**
*   **Example:**
    ```go
    // This will delete the 'config' directory and everything inside it.
    err := os.RemoveAll("config")
    if err != nil {
        panic(err)
    }
    fmt.Println("Directory 'config' removed.")
    ```

---

### 4. Exiting the Program

#### `os.Exit`
*   **What it does:** Immediately terminates the program. The number you give it is the "exit code." By convention, `0` means success and any non-zero number (usually `1`) means an error occurred.
*   **When to use it:** When a fatal, unrecoverable error happens and you need to stop everything immediately. **Note:** `defer` statements will *not* be executed when you call `os.Exit`.
*   **Example:**
    ```go
    _, err := os.Open("non_existent_file.txt")
    if err != nil {
        fmt.Println("Fatal Error: Could not open essential file.", err)
        os.Exit(1) // Exit with an error code
    }
    ```

### Simple Summary

| Function/Variable | What It Does                                        | Main Use Case                                                      |
| :---------------- | :-------------------------------------------------- | :----------------------------------------------------------------- |
| **`os.Args`**       | Slice of command-line arguments.                    | Getting input from the user when they run the program.             |
| **`os.Getenv`**     | Gets an environment variable.                       | Reading configuration like API keys or database URLs.              |
| **`os.WriteFile`**  | Writes a slice of bytes to a file in one step.      | The easiest way to write a new file.                               |
| **`os.ReadFile`**   | Reads an entire file into a slice of bytes.         | The easiest way to read a whole file into memory.                  |
| **`os.Stat`**       | Gets info about a file (size, etc.).                | Checking if a file exists.                                         |
| **`os.MkdirAll`**   | Creates a directory and all its parents.            | The standard way to create directories.                            |
| **`os.RemoveAll`**  | Deletes a path and everything inside it.            | Cleaning up a directory structure. **(Use with care!)**            |
| **`os.Exit`**       | Immediately stops the program with a status code.   | Halting the program on a fatal, unrecoverable error.               |
