
## Macros in Rust

### Introduction

Rust is a powerful language, which combines performance and pleasant coding
experience.

Macros make Rust even more powerful. They are a way of 
[metaprogramming](https://en.wikipedia.org/wiki/Metaprogramming) - basically,
writing code that writes other code. Let's explore the types of macros present in
Rust.

### Declarative macros

Declarative macros is the most fun-to-write (at least, from my perspective)
type of macros. They are defined using `macro_rules!` construct, after which
we write the definition. An example of a declarative macro would be the `println!` macro.

A little more complex example:
```rust
macro_rules! data_endpoint {
    ($endp_name: expr, $mthd_name: ident, $typ: ty, $act: ident) => {
         #[post($endp_name)]
         async fn $mthd_name(req: HttpRequest, body: web::Json<$typ>, shop_repo: web::Data<Arc<SupermarketRepository>>
         ) -> impl Responder {
            let body = body.into_inner();
            let action_fut = Action::from_req(&req, body);
            utils::handle_query_and_auth(&req, action_fut.and_then(|act| shop_repo.$act(act)).await, "manager")
        }
    };
}
```

This macro creates an endpoint with given url path, method name,
input JSON model and handler method.

Generally, declarative macros can be thought of as a pattern-match on a piece of
input code.

The patterns are matched against their position and types. The types used in
declarative macros include:

* `item`: an [Item](https://doc.rust-lang.org/reference/items.html) (module, crate etc.)
* `block`: a [BlockExpression](https://doc.rust-lang.org/reference/expressions/block-expr.html)
  (`{ let x = 42; x}`)
* `stmt`: a statement without the trailing semicolon (except for item 
  statements that require semicolons)
* `pat`: [Pattern](https://doc.rust-lang.org/reference/patterns.html)
* `expr`: an [Expression](https://doc.rust-lang.org/reference/expressions.html)
  (`"I love Rust!"`, `42`)
* `ty`: a [Type](https://doc.rust-lang.org/reference/types.html#type-expressions)
  (`TokenStream`, `u8` etc.)
* `ident`: an identifier or a keyword (`let`, method/function names)
* `path`: a [TypePath](https://doc.rust-lang.org/reference/paths.html#paths-in-types)
  style path (e.g., `std::env::Var`)
* `tt`: a [TokenTree](https://doc.rust-lang.org/reference/macros.html#macro-invocation)
  (a single token or tokens in matching delimiters `()`, `[]`, or `{}`)
* `meta`: an attribute, the contents of an attribute
* `lifetime`: a lifetime annotation
* `vis`: a possibly empty visibility qualifier
* `literal`: matches literal expression

(see more in the [reference](https://doc.rust-lang.org/reference/macros-by-example.html))

After the patterns match, they are bound to the variables (denoted by
the `$` sign) and used in output code.

It is important to note that unlike C or C++ preprocessor macros Rust macros
make use of a concept called 
[macro hygiene](https://doc.rust-lang.org/reference/macros-by-example.html#hygiene).

In short, Rust macro hygiene imposes a set of rules that the macros have to follow.
Those may include using fully-qualified paths to identifiers and some other rules.

The compiler expands macros when compiling code and executes them as regular code
at the call-site.

It is possible to see how the macros are expanded into code with this command:

`cargo rustc --lib -- -Z unstable-options -Z macro-backtrace --pretty=expanded > compiled.rs`
(requires nightly toolchain).

This produces a valid rust file with expanded macros.

[A great tutorial on Rust macros](https://blog.logrocket.com/macros-in-rust-a-tutorial-with-examples/)

### Procedural macros

Another type of macros in Rust is procedural macros. They are vastly different
from their declarative counterparts in the definition, but similar in how they work.

Generally, a procedural macro is a function that takes a 
[TokenStream](https://doc.rust-lang.org/proc_macro/struct.TokenStream.html) as an
input and produces a TokenStream as an output.

A quote from the 
[book](https://doc.rust-lang.org/book/ch19-06-macros.html#procedural-macros-for-generating-code-from-attributes):

"The second form of macros is procedural macros, which act more like functions 
(and are a type of procedure). Procedural macros accept some code as an input,
operate on that code, and produce some code as an output rather than matching
against patterns and replacing the code with other code as declarative macros do."

They are divided into three subcategories: custom derive macros (like
`#[derive(Serialize, Deserialize)]`), attribute-like macros (`#[ink::contract]`)
and function-like (`sql!(SELECT * FROM posts WHERE id=1)`) macros.

**IMPORTANT TO NOTE**: procedural macros do not maintain macro hygiene!

#### Custom derive macros

Custom derive macros usually provide implementations of some traits that 
are trivial to implement.

Take the `serde::Serialize` trait: a `derive` macro can trivially implement
Serialize trait for a struct/enum, for fields/variants of which Serialize is 
implemented.

The structure of a macro goes like this:
```rust
#[proc_macro_derive(Serialize, attributes(serde))]
pub fn derive_serialize(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    ser::expand_derive_serialize(&input)
        .unwrap_or_else(to_compile_errors)
        .into()
}
```

The macro definition only contains the stages of parsing and processing the input,
where the logic is delegated to other functions/macros.

See the [chapter](https://doc.rust-lang.org/book/ch19-06-macros.html#how-to-write-a-custom-derive-macro)
on how to write custom derive macros.

#### Attribute-like macros

While derive macros typically implement some trait on a type, attribute-like macros
have wider range of usage. They still operate on TokenStreams, but may have custom names
(recall `#[ink::contract]` macro) and items inside them.

See the [reference](https://doc.rust-lang.org/reference/procedural-macros.html#attribute-macros)
for a more involved explanation.

#### Function-like macros

These are kind of a combination of procedural and declarative macros. They are
used just like declarative macros, but do not operate on patterns and types. They
accept TokenStream as an input and produce TokenStream as an output.

[Example](https://doc.rust-lang.org/reference/procedural-macros.html#function-like-procedural-macros)
from the reference

### Conclusion

There are two main types of macros: declarative and procedural.
Declarative macros match input code based on types and position of the values,
while procedural macros process and output an abstract stream of tokens.

A great 
[thread from reddit](https://www.reddit.com/r/rust/comments/cd9agr/elif_the_difference_between_declarative_and/ett8g5x?utm_source=share&utm_medium=web2x&context=3)
that summarizes macro usage:

*"Declarative macros are simpler to use for simple tasks once you're familiar with the syntax,
but less powerful. If you've ever written a web application, they're like HTML templates,
but for Rust code. (They're Rust code with extra markers that say "insert argument here" or
"repeat this part for each argument").
Procedural macros are more powerful, but there's a certain amount of minimum work that you have
to do, no matter how simple your task, because they're full-blown programs that run inside
the compiler. (A procedural macro is a program which will receive Rust code and must modify
it however it wants and then return it.)"*

*Varg Vikernes*