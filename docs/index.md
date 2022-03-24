title: MkDocs

# **MkDocs**

For full documentation visit [mkdocs.org](https://www.mkdocs.org/user-guide/writing-your-docs/).

### Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve --dev-addr 172.17.0.5:80` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

Lists of extensions:

* [mkdocs.org](https://www.mkdocs.org/user-guide/configuration/#markdown_extensions)
* [mkdocs-material](https://squidfunk.github.io/mkdocs-material/reference/abbreviations/)
* [emojis](https://squidfunk.github.io/mkdocs-material/reference/icons-emojis/)

### Project layout
```
mkdocs.yml    # The configuration file.
docs/
    index.md  # The documentation homepage.
    ...       # Other markdown pages, images and other files.
```

### Dashes

* qwer
* asdf
* one-dash
* two -- dashes
* tree --- dashes



### :material-language-rust:{ .heart .twitter } Abbrs

The HTML specification is maintained by the W3C.

*[HTML]: Hyper Text Markup Language
*[W3C]: World Wide Web Consortium

### Math

$$
\operatorname{ker} f=\{g\in G:f(g)=e_{H}\}{\mbox{.}}
$$

The homomorphism $f$ is injective if and only if its kernel is only the 
singleton set $e_G$, because otherwise $\exists a,b\in G$ with $a\neq b$ such 
that $f(a)=f(b)$.

### Lists

`Lorem ipsum dolor sit amet`

:   Sed sagittis eleifend rutrum. Donec vitae suscipit est. Nullam tempus
    tellus non sem sollicitudin, quis rutrum leo facilisis.

`Cras arcu libero`

:   Aliquam metus eros, pretium sed nulla venenatis, faucibus auctor ex. Proin
    ut eros sed sapien ullamcorper consequat. Nunc ligula ante.

    Duis mollis est eget nibh volutpat, fermentum aliquet dui mollis.
    Nam vulputate tincidunt fringilla.
    Nullam dignissim ultrices urna non auctor.


- [x] Lorem ipsum dolor sit amet, consectetur adipiscing elit
- [ ] Vestibulum convallis sit amet nisi a tincidunt
    * [x] In hac habitasse platea dictumst
    * [x] In scelerisque nibh non dolor mollis congue sed et metus
    * [ ] Praesent sed risus massa
- [ ] Aenean pretium efficitur erat, donec pharetra, ligula non scelerisque

### Formatting

{==Highlighting==} is also possible {>>and comments can be added inline<<}.

~~This was deleted~~

H~2~0

A^T^A

++ctrl+alt+del++

### Tabs and code blocks


=== "Tab 1"

    ``` py title="bubble_sort.py" linenums="1" hl_lines="2 3"
    def bubble_sort(items):
        for i in range(len(items)):
            for j in range(len(items) - 1 - i):
                if items[j] > items[j + 1]:
                    items[j], items[j + 1] = items[j + 1], items[j]
    ```

=== "Tab 2"

    Phasellus posuere in sem ut cursus (1)


The `#!python range()` function is used to generate a sequence of numbers.



### Admonitions

!!! note "Phasellus posuere in sem ut cursus"

    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
    nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
    massa, nec semper lorem quam in massa.

??? abstract "collapsible abstract"

    Lorem

??? info "collapsible info"

    Lorem

??? tip "collapsible tip"

    asdf

??? success "collapsible success"

    asdf

??? question "collapsible question"

    asdf

??? warning "collapsible warning"

    asdf

??? failure "collapsible failure"

    asdf

??? danger "collapsible danger"

    asdf

??? bug "collapsible bug"

    asdf

??? example "collapsible example"

    asdf

??? quote "collapsible quote"

    asdf


### Inline

On the left:

!!! info inline end

    Lorem ipsum dolor sit amet, consectetur
    adipiscing elit. Nulla et euismod nulla.

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod nulla. Curabitur feugiat, tortor non consequat
Curabitur feugiat, tortor non consequat


On the right:

!!! info inline

    Lorem ipsum dolor sit amet, consectetur
    adipiscing elit. Nulla et euismod nulla.

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod nulla. Curabitur feugiat, tortor non consequat
Curabitur feugiat, tortor non consequat
