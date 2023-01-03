## Package Design
Write shy code - modules that don’t reveal anything unnecessary to other modules and that don’t rely on other module's implementations.

- **Name your package for what it provides, not what it contains.**

- **Within your project, each package name should be unique.**

- **Avoid package names like `base`, `common` or `util`**

My recommendation to improve the name of `utils` or `helpers` packages is to analyse where they are called and if possible move the relevant functions into their caller’s package.
Even if this involves duplicating some helper code this is better than introducing an import dependency between two packages.

**A little duplication is far cheaper than the wrong abstraction.**

In the case where utility functions are used in many places prefer multiple packages, each focused on a single aspect, to a single monolithic package.

An identifier’s name includes its package name.
It’s important to remember that the name of an identifier includes the name of its package.

## Package Group
You can follow these rules and of course adapt depending on your individual case:

1. **Group subpackages by dependency**
2. **Use shared mock subpackage**
Create `mock` package and create different files with the needed mocks as well as initialization function which can model the mock differently

```go
package mock

// UserService represents a mock implementation of UserService interface.
type UserService struct {
        // additional fields for modeling different responses
}

// User invokes the mock implementation and marks the function as invoked.
func (s *UserService) InitUser(inputData) (*ServiceUser, error) {
        // model the UserService with the different input data
}
//implementing methods
```

## Project Structure

**Consider fewer larger packages**

If you’re arranging your packages by what they provide to callers, should you do the same for files within a Go package? How do you know when you should break up a `.go` file into multiple ones? How do you know when you’ve gone to far and should consider consolidating `.go` file?
1. Start each package with one `.go` file. Give that file the same name as the name of the folder. eg. package `http` should be placed in a file called `http.go` in a directory named `http`.
2. As your package grows you may decide to split apart the various responsibilities into different files. eg, `messages.go` contains the `Request` and `Response` types, `client.go` contains the `Client` type, `server.go` contains the Server type.
3. If you find your files have similar import declarations, consider combining them. Alternatively, identify the differences between the import sets and move those.
4. Different files should be responsible for different areas of the package. `messages.go` may be responsible for marshalling of HTTP requests and responses on and off the network.
`http.go` may contain the low level network handling logic, `client.go` and `server.go` implement the HTTP business logic of request construction or routing, and so on.

Your main function, and `main` package should do as little as possible. This is because `main.main` acts as a singleton; there can only be one `main` function in a program, including tests.
Because `main.main` is a singleton there are a lot of assumptions built into the things that `main.main` will call that they will only be called during `main.main` or `main.init`, and only called once. This makes it hard to write tests for code written in `main.main`, thus you should aim to move as much of your business logic out of your `main` function and ideally out of your `main` package.

 



