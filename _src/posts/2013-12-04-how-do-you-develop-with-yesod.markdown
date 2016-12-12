    Title: How do you develop with Yesod
    Date: 2013-12-04T14:39:03Z+0100
    Tags: haskell, web

Yesod tutorials and examples can be found on the [Yesod book](http://www.yesodweb.com/book),
which is freely available online.

However, many of the examples given are self-contained (which means that
handler functions, data models and templates are defined in a single module,
where a Warp web server is started as well). There is
[a chapter that describes the scaffolded template](http://www.yesodweb.com/book/scaffolding-and-the-site-template),
i.e.  what you get after you start with Yesod, but it is rather a description
of its advantages and structure, not so much a detailed how-to.

The purpose of this post is to explain to a new Yesod developer "What do I
do after ... ?" :

```console
$ yesod init
```
<!-- more -->

Briefly summarized, what you need after this command is:

1. Describe the **data models** your Web application is going to store persistently

2. Describe which **routes** are going to exist in your Web application

2. Write the **handler functions** that answer requests to those routes

2. Write the **Hamlet templates** that are filled out by the handler functions (and
   possibly Lucius and Julius templates)


The `yesod add-handler` command automates the process of adding **routes** and
**handler functions**.

### Adding data models

A new data model has to be described in the `config/models` text file. With
"text file" we mean that you write in "near free" syntax, but nevertheless many
of the keywords are Haskell. 

#### Basics

A new model consists of a name and a description of its fields. The description
of fields consists, in turn, of a name and a type for the field. The type can
be `Int`, `Text`, `UTCTime`, or other Haskell types. The type can also be
qualified with a Monad (usually `Maybe`) and have a defaul value. Let's see a
simple example:

```
Book
    title Text
    author Text
    publisher Text Maybe
    published UTCTime default=CURRENT_TIME
```

An identifier field is **automatically added** to each model. The name of that
field is always `<modelName>Id`, e.g. for the previous model it would be
`BookId`. This is a unique identifier stored in the database using the database
mechanisms if available, or the Persistent module's mechanisms otherwise. We
usually do not need to care about **writing** this ID, but we will be
interested in **reading** it from the database, as we will see later.

Another thing we get for free are **accessor functions** for each field. These
are called `bookTitle` to get the title, `bookAuthor` to get the author, and so
on...

With the mentioned, you can create your own models. Let's review other
features that are slightly more advanced but almost unavoidable in any app that
is not completely trivial.

#### Reference other models

You can **reference field in other models**. This creates *foreign keys* in
relational databases, or is managed by Persistent otherwise. You refer to the
other model with an uppercase and join the name of model and field in camel
case. An example is in the default models written by the scaffolding:

```
User
    ...
Email
    user UserId Maybe
```

This is also an example of using the automatically created field `UserId` for
the User.

#### Declare records to be unique

You can **declare records to be unique with respect to a field**. The unique ID
are handed out to anything new in the store otherwise. You can specify that a
record has to have a new value in one of its fields to be accepted. Again the
default models provide an example of this:

```
User
    ident Text
    UniqueUser ident
```

#### Derive typeclasses

And as a last note, you can directly **declare your models to derive Haskell
typeclasses** e.g. Eq, Ord, Show. Which means the model, when loaded from
database and available to be manipulated, will have the correct functions of
that typeclass already implemented. ( In the case of Eq, it will be equatable,
in the case of Ord, it will be comparable, and in the case of Show it will be
printable; this, of course, in the case the compiler can deduce out of the
fields composing the model how to implement such functions for you ).

An example of deriving typeclasses:

```
Person
    ...
    deriving (Show)
```

### Adding routes

You declare new routes to be answered to in the `config/routes` text file. This
can be automated with the `yesod add-handler` command. However, let's review it
to gain a bit of control on the underlying machinery.

A new route starts from the root `/` and lists a *resource name* and *query
parameters*. After that there is a space, and the name of the handler
function(s) that handle requests to that particular route. Again a space, and a
space-separated list of HTTP methods that are allowed on that route. An example
with all of these:

```
/book/#BookId BookR GET PUT DELETE
```

where `book` is the *resource name*, `#BookId` is the *query parameter*, BookR
is the shared part of the name of the *handler functions*, and the allowed
methods are GET, PUT and DELETE.

We say `BookR` is the shared part of the name because a single handler function
is expected **for each one of the allowed methods**. That is, in the above
example, `getBookR`, `putBookR` and `deleteBookR` are expected to exist
somewhere (probably in a module called `Book` within the `Handlers` directory).

Usually the convention for a REST API is that GET allows to see an instance of
a model, POST allows to create an instance of a model, PUT allows to modify an
instance of a model, and DELETE allows to, well, delete it.

After you have added a new route, the application will not compile until there
are handler functions for all of the new resources and allowed methods.

### Adding handler functions

You add new functions to handle requests in modules under the `Handler`
directory. This can be automated with the `yesod add-handler` command. However,
let's review it to gain a bit of control on the underlying machinery.

A handler function must exist **for each resource and for each allowed HTTP
method on that resource**. That means if we declared the resource `/donkey` to
accept requests with GET, POST and DELETE methods, then we have to implement
`getDonkeyR`, `postDonkeyR` and `deleteDonkeyR`. The `R` means "resource".

Let us see a stub example of one of those functions:

```haskell
getDonkeyR :: DonkeyId -> Handler Html
getDonkeyR donkeyId = undefined
```

#### Handler function signature

Being a Haskell function, the function signature is important to understand
what the function does.

In the case of Yesod, the **input to the function are
query parameters**, which are typed to correspond to model fields. That is, you
do not get an integer, but a `DonkeyId` -- if you specified in the route that the
`donkey` resource has a query parameter of that type. This helps ensure that
there are no typos in the `routes` and `models` text files that cause your app
to fail on deployment.

The **output of the function** is quite determined by Yesod, so it is easy to
read, but you have to accept that the framework "knows what it is doing". You
usually have a `Handler Html` result type; which means that the result of your
function is a function that knows how to answer a request with HTML. This
function is what is going to be called by the application when it is running.

*Do not let that make your head explode*: it is like when you compile a
function with gcc. It is not your source code that is run when the resulting
binary is executed: it is the object representation of it. This is not exactly
the same but for the moment, it is a good enough explanation.

Other output types for the function are `Handler RepJson` if the function
returns JSON, or even `Handler TypedContent` if the function returns HTML or
JSON depending on what the client has requested.

#### Handler function body

In the function itself, you may want to do many different things. Let us
discuss the most usual:

1. getting something out of the database and into a template for viewing
   ( **Database -> User** )

2. getting something out of a template form from the user, and inserting that
   into the database ( **User -> Database** )

##### Database -> User

Consider the following function:

```haskell
getBooksR :: Handler Html
getBooksR = do
        books <- runDB $ selectList [] [Asc BookTitle]
        (formWidget, enctype) <- generateFormPost bookForm
        defaultLayout $(widgetFile "books")
```

The first line after `do` allows us to get a Book model out of the database.
The `runDB` function runs any query on the database, while the `selectList`
function composes a query that returns its results as a list. The empty first
argument means we take **all fields** from each record, and the second argument
accepts a number of filters and modifyers.  The example shown is equivalent, in
SQL, to:

```sql
SELECT * FROM books ORDER BY "BookTitle" ASC;
```

Then we would go on to render the correct HTML template with the default
layout. The template will have direct access to the `books` variable because it
is in the same scope (the handler function). We will see how this happens when
we discuss templates. The rendering happens with the line:

```haskell
defaultLayout $(widgetFile "books")
```

The template rendering takes the `defaultLayout` function, applied on the
result of loading a widgetFile by name. The default layout renders the template
found under `templates/default-layout.*` and includes within it the result of
rendering whatever is on the `templates/books.*` files. This helps make the
look and feel uniform across the website.

(You may notice that we are skipping a line in the `getBooksR` function,
between reading from database and rendering the template. It is concerned with
**form rendering**, but for the sake of brevity we will not discuss it here)

##### User -> Database

Consider the following function:

```haskell
postBooksR :: Handler Html
postBooksR = do
    ((result, _), _) <- runFormPost bookForm
    case result of
        FormSuccess book -> do
            bookid <- runDB $ insert book
            setMessage "Book registered"
        _ -> do
            setMessage "Invalid data"
            redirect $ BooksR
```

The first line after `do` gets the result out of a form that is defined and
rendered elsewhere.  The result may be a success, because all inputs are filled
out and are of the expected types, or it may be anything else (an error), where
we print an error message and do no more. If we have a success, then we can
extract the specific `book` (of type `Book`) out of the `FormSuccess` monad via
pattern matching, and then insert it into the DB via the Persist `insert`
function:

```haskell
bookid <- runDB $ insert book
```

Notice this works without any more specifications because we know, from the
compile process, that the result of parsing the form contents must have
enough informations to build a Book model, and a Book model can be written to
database because we have already parsed it and created the necessary tables (in
the case of a relational DB engine) on app startup.

Notice also we receive a bookId as a result of inserting, but we can ignore it.

#### After writing your handlers

After writing your handlers, you must make the application aware that they
exist. Import the relevant module in in the `Application` module, and also
remember to list them in the `<application>.cabal` file, under
`exposed-modules`. This makes the `cabal` build system know that it must
include the new handler files in the compilation, and the `Application` module
aware that it has to import those modules if it wants to answer the calls to
some of the resources.

### Adding Hamlet templates

We said earlier that the variables extracted from storage are rendered in the
template. A template is a file that lives in the `templates` directory and is
called whatever we want the `widgetFile` function to find.

Hamlet is a template language that describes HTML to produce. A very short
summary of Hamlet: you do not need to close HTML tags, this will be done for
you. *With the important exception of links. Do not forget to close them with
`</a>`*. 

In the template, you can refer to any variable that is available in the handler
function (because the generated code is inserted in the function code directly
before the results are produced). See this example template to render a Book
model:

```
<h2> Book: #{bookTitle book}

<p> by #{bookAuthor book}
```

Within the template, we can use the **accessor functions** we got for free out
of the model definition. Remember that this is Haskell, so the function must go
before the arguments. We drop into Haskell from the template by using `#{ }`.

There is more syntax to iterate over a collection of `books` and control `if`
we render some thing or the other depending on the presence of a variable, but
we will skip this. There is Yesod documentation available.

Let us mention another important feature of these templates: **compiled in-site
links**. This means you refer to other pages within your website *through their
resource names*, not through a simple string. In the homepage, for instance,
you should include a link to the "books" page by using:

```
<a href=@{BooksR}>Add new book</a>
```

Notice you use an `@`, not the `#` symbol.

This makes it **impossible for you to have dead links within the site**. They
are checked by the compiler on each site update, and the compilation will fail
if there is something misspelled.

### Summing up

We have reviewed what are the minimal changes you have to do to a scaffolded
site in order to add a new feature to it, in some more depth than necessary
since you can use `yesod add-handler` to make many of the changes for you.
However, now you should understand better what is this command doing and why
could it fail. The changes are:

1. In `config/models`: describe **data models** stored

2. In `config/routes`: describe **routes** served, as resources

2. In `Handler/<name>.hs`: write **handler functions** that answer requests to
   those routes; include them in `Application.hs` and list them on
   `<application>.cabal`

2. In `templates/<name>.{hamlet,lucius,julius}`: write the **templates** that
   are filled out by the handler functions
