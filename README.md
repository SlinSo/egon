Egon
===

## Not longer maintained
* the implemented features didnt' find their way into ego or egon :(
* for my projects I switched to [quicktemplate](https://github.com/valyala/quicktemplate)

## Added in this fork
* Added flags to the egon command
* Added view geneartion optional, because it is only needed if your web framework supports late rendering
* Added type safe generation like in [ftmpl](https://github.com/tkrajina/ftmpl) - use go vet
* Added Debug mode, otherwise dont print comments inside generated functions
* Added string optimisation, removed Sprintf for strings
* Added examples
* Added minify, currently only tested with HTML code
* Reverted fmt.Fprintf -> io.WriteString to avoid unnecessary allocations

## TODO
* XML Rendering (xml.Escape...)

===
**Note: This is a work in progress.**

Egon is a templating language for go, based on [Ego](https://github.com/benbjohnson/ego).
Egon parses .egon templates and converts them into Go source files.

## Differences from Ego

* Ego generates a single source file for every template in a package. Egon
  generates a source file per template.
* Ego includes options to name the output package and output files. Egon always
  determines these names based on the source structure.
* Ego generates a single function per template for rendering the template. Egon
  includes a second function, [Template]View that returns an egon.View struct
  and does not require a writer to be called (but the View does require a writer
  to render).
* Ego requires a full function definition, Egon only requires parameter declarations.

## Usage

To install egon:

```sh
$ go get github.com/commondream/egon/cmd/egon
```

Running the `egon` command will process all templates for path.

```sh
$ egon ./templates/
```

All egon files found in the given path are converted to .egon.go files.
Each .egon.go file defines two functions:

1. The Template function - a function with an io.Writer parameter followed by
   all parameters defined in the template, in the order in which they were
   defined.
2. The View function - a function with only the parameters defined in the
   template in the order that they were defined that returns an egon.View
   struct.


## Language Definition

An ego template is made up of several types of blocks:

* **Code Block** - These blocks execute raw Go code: `<% var foo = "bar" %>`

* **Print Block** - These blocks print a Go expression. They use `html.EscapeString` to escape it before outputting: `<%= myVar %>`

* **Raw Print Block** - These blocks print a Go expression raw into the HTML: `<%== "<script>" %>`

* **Header Block** - These blocks allow you to import packages: `<%% import "encoding/json" %%>`

* **Parameter Block** - This block defines the function signature for your template.

A single declaration block should exist at the top of your template and accept an `w io.Writer` and return an `error`. Other arguments can be added as needed. A function receiver can also be used.

```
<%! name string %>
```


## Example

Below is an example egon template for a web page:

```ego
// my_tmpl.egon
<%% import "strings" %%>
<%! u *User %>

<html>
  <body>
    <h1>Hello <%= strings.TrimSpace(u.FirstName) %>!</h1>

    <p>Here's a list of your favorite colors:</p>
    <ul>
      <% for _, colorName := range u.FavoriteColors { %>
        <li><%= colorName %></li>
      <% } %>
    </ul>
  </body>
</html>
```

Once this template is compiled you can call it using the parameters you specified:

```go
myUser := &User{
  FirstName: "Bob",
  FavoriteColors: []string{"blue", "green", "mauve"},
}
var buf bytes.Buffer
err := mypkg.MyTmplTemplate(&buf, myUser)
```

## Caveats

Unlike other runtime-based templating languages, Egon does not support ad hoc
templates. All templates must be generated before compile time.

Egon does not attempt to provide any security around the templates. Just like
regular Go code, the security model is up to you.
