# Swagger Client Provider

<Note type="warning">

This page is outdated and will be updated soon.

</Note>

Start by loading the swagger provider.

```fsharp
#load @"packages/SwaggerProvider/SwaggerProvider.fsx"
open SwaggerProvider

let [<Literal>]schema = "http://petstore.swagger.io/v2/swagger.json"
type PetStore = SwaggerClientProvider<schema> // Provided Types
let petStore = PetStore() // Instance for communication
```

## SwaggerProvider

When you use TP you can specify following parameters

| Parameter | Description |
|-----------|-------------|
| `Schema` | Url or Path to Swagger schema file. By default relative file paths will based off the IDE execution directory. `__SOURCE_DIRECTORY__` can be used to make the path relative to the location of the source file. For example `let [<Literal>]schema = __SOURCE_DIRECTORY__ + "./../APIDefs/swagger.json"`will navigate up one directory and down into APIDefs.|
| `Headers` | Headers that will be used to access the schema. |
| `IgnoreOperationId` | `IgnoreOperationId` tells SwaggerProvider not to use `operationsId` and generate method names using `path` only. Default value `false`. |
| `IgnoreControllerPrefix` | Instead of generating one client per controller, generate one client and attach all operations to it. Default value `false`. |
| `PreferNullable` | Generate `Nullable<_>` properties instead of `Option<_>`. Default value `false`. |
| `PreferAsync` | Generate return types as `Async<_>` instead of `Task<_>`. Default value `false` |

## Generated constructor

SwaggerProvider generates `.ctor` that have to be used to create instance of
generated type for communication with a server. The following parameter may be specified

| Parameter | Description |
|-----------|-------------|
| `host` | Server Url, if you want call server that differs from one specified in the schema |
| `CustomizeHttpRequest` | Function that is called for each `System.Net.HttpWebRequest` created by Type Provider. Here you can apply any transformation to the request (add credentials, headers, cookies and etc.) |

[Read more about available configuration options.](http://stackoverflow.com/questions/37566751/what-should-i-do-to-prevent-a-401-unauthorised-when-using-the-swagger-type-provi/37628857#37628857)

## Usage sample

Instantiate the types provided by the SwaggerProvider.

```fsharp
let tag = PetStore.Tag()
tag.Id <- Some 1337L
tag.Name <- "foobar"

let category = new PetStore.Category()
category.Id <- Some 1337L
category.Name <- "dog"

let pet = new PetStore.Pet (Name = "foo", Id = Some 1337L)
pet.Name <- "bar" // Overwrites "foo"
pet.Category <- category
pet.Status <- "sold"
pet.Tags <- [|tag|]

let user = new PetStore.User()
user.Id <- Some 1337L
user.FirstName <- "Firstname"
user.LastName <- "Lastname"
user.Email <- "e-mail"
user.Password <- "password"
user.Phone <- "12345678"
user.Username <- "user_name"
```

Invoke the Swagger operations using `petStore` instance.

```fsharp
async {
    let! f = petStore.GetPetById(6L)
    f.Category <- PetStore.Category(Id = Some 1337L, Name = "dog")
    f.Name <- "Hans"
    f.Tags <- Array.append f.Tags [|tag|]

    let! pet = petStore.AddPet(pet)
    let! x = petStore.FindPetsByTags([|"tag1"|])
    Array.length x

    do! petStore.UpdatePetWithForm(-1L, "name", "sold")
    let! leetPet = petStore.GetPetById(1337L)

    let! h = petStore.FindPetsByStatus([|"pending";"sold"|])
    h.ToString()
    let! i = petStore.FindPetsByTags([|"tag2"|])
    i.ToString()

    let! inventory = petStore.GetInventory()
    inventory.ToString()
    let! order = petStore.GetOrderById(3L)
    order.ToString()

    let! ``14`` = petStore.GetPetById(14L)
    ``14``.ToString()
    do! petStore.DeletePet(14L, "no-key")
    // throws, the pet no longer exists!
    let! pet = petStore.GetPetById(14L)

    let! ``14`` = PetStore.Pet.GetPetById(14L)
    ``14``.ToString()
    // throws, the pet no longer exists!
    do! PetStore.Pet.DeletePet(14L, "no-key")
    let! ``14`` = PetStore.Pet.GetPetById(14L)
    ``14``.ToString()
}
```

Enjoy!

> Beware that the PetStore is publicly available and may be changed by anyone.