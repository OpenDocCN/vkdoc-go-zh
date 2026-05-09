# 3. 用户定义类型与并发

类型系统是编程语言最重要的特性之一；类型允许您组织并存储应用程序数据。在选择编程语言时，必须考虑其类型系统。`Go` 在整个语言设计中贯彻了简洁性原则。

`Go` 是一种静态类型语言，您可以使用内置类型和用户定义类型来存储应用程序数据。`Go` 提供了多种内置类型，例如 `int`、`float64`、`string` 和 `bool`。第 2 章 讨论了数组、切片和映射，这些是由其他类型（包括内置类型和用户定义类型）组成的复合类型。例如，复合类型 `map[string]float64` 表示一个 `float64` 值的集合，其中每个 `float64` 类型的值都可以通过一个 `string` 类型的键添加到集合中，并可通过相应的键值来检索值。除了内置类型之外，您还可以通过组合一个或多个内置类型或用户定义类型来创建自己的类型。`Go` 在其类型系统中提供了一种接口类型，使您能够开发出具有更高可扩展性的程序。

### 使用结构体定义用户定义类型

`Go` 的类型系统在设计时考虑了简洁性和实用性，这避免了在设计应用程序数据结构时出现许多复杂性。`Go` 的面向对象方法与其他编程语言完全不同。如果您来自 `C++`、`Java` 和 `C#` 等编程语言，您会发现 `Go` 的面向对象方法是不同且独特的，尽管您可能会怀念这些语言的某些特性。当您审视 `Go` 的类型系统及其用户定义类型时，您应该以全新的眼光看待这门语言，从而能够欣赏并利用其简洁性和强大功能来解决实际问题。

`Go` 的类型系统中没有提供类（classes）；它有结构体（structs），它类似于类。结构体可以被视为类的轻量级版本，但其设计是独特的，因为它专注于实际应用。`Go` 中的结构体让您可以创建用户定义的具体类型；它们是字段或属性的集合，可用于存储复杂数据。您可以使用结构体来存储应用程序领域模型。

#### 创建结构体类型

让我们创建一个用于表示人员信息的结构体类型（参见清单 3-1）。

**清单 3-1. 声明一个带有字段集合的结构体**

```
type Person struct {
        FirstName, LastName string
        Dob                 time.Time
        Email, Location     string
}
```

使用关键字 `struct` 创建一个用户定义类型：

```
type Person struct
```

因为类型标识符以大写字母开头，所以 `Person` 类型将被导出到其他包中。您在 `Person` 类型的主体中指定结构体字段。由于 `FirstName` 和 `LastName` 字段使用相同的数据类型，您可以在一条语句中声明这两个变量：

```
FirstName, LastName string
```

#### 创建结构体类型的实例

我们已经定义了一个名为 `Person` 的结构体类型。让我们创建一个 `Person` 类型的实例，并为字段赋值（参见清单 3-2）。

**清单 3-2. 创建结构体实例**

```
var p Person
p.FirstName="Shiju"
p.LastName="Varghese"
p.Dob= time.Date(1979, time.February, 17, 0, 0, 0, 0, time.UTC)
p.Email= "shiju@email.com"
p.Location="Kochi"
```

创建了一个 `Person` 类型的实例，并逐个为字段赋值。您也可以使用结构体字面量来创建结构体类型实例（参见清单 3-3）。

**清单 3-3. 使用结构体字面量创建结构体实例**

```
p:= Person {
    FirstName : "Shiju",
    LastName : "Varghese",
    Dob : time.Date(1979, time.February, 17, 0, 0, 0, 0, time.UTC),
    Email : "shiju@email.com",
    Location : "Kochi",
}
```

您可以通过使用结构体字面量来创建 `Person` 类型实例。当使用结构体字面量创建结构体类型实例时，您可以将结构体字段的赋值拆分到多行，这增强了代码块的可读性。当您以这种方式创建结构体实例时，必须在最后一个赋值语句末尾加上额外的逗号，这使得您可以轻松地重新排列结构体字段的赋值顺序，而无需担心移除最后一个赋值语句的逗号，并为所有其他字段添加逗号。现在，您必须为每个结构体类型字段插入逗号，无论它是否是最后一个字段。

如果您知道结构体字段的顺序，您可以用更高效的方式创建结构体实例（参见清单 3-4）。

**清单 3-4. 使用结构体字面量创建结构体实例**

```
p:= Person {
    "Shiju",
    "Varghese",
     time.Date(1979, time.February, 17, 0, 0, 0, 0, time.UTC),
    "shiju@email.com",
    "Kochi",
}
```

如果您知道结构体类型字段的顺序，这是一种非常方便的创建结构体类型实例的方式。您还可以使用单行语句创建结构体实例。

清单 3-5 展示了一个示例，通过使用结构体字面量指定部分字段，并在之后分配其余字段来创建结构体实例。

**清单 3-5. 通过指定部分字段创建结构体实例**

```
p:= Person { FirstName :"Shiju", LastName : "Varghese" }
p.Dob= time.Date(1979, time.February, 17, 0, 0, 0, 0, time.UTC)
p.Email= "shiju@email.com"
p.Location="Kochi"
```

#### 向结构体类型添加行为

`Go` 的类型系统允许您向结构体类型添加行为。与其他面向对象语言的类一样，`Go` 中的结构体允许您定义字段以及操作。让我们向 `Person` 类型添加几个行为（参见清单 3-6）。

**清单 3-6. 带有行为的结构体类型**

```
type Person struct {
    FirstName, LastName string
    Dob                 time.Time
    Email, Location     string
}

//一个人物方法
func (p Person) PrintName() {
    fmt.Printf("\n%s %s\n", p.FirstName, p.LastName)
}

//一个人物方法
func (p Person) PrintDetails() {
    fmt.Printf("[Date of Birth: %s, Email: %s, Location: %s ]\n", p.Dob.String(), p.Email, p.Location)
}
```

已经为 `Person` 类型添加了两个方法：`PrintName` 和 `PrintDetails`。`Go` 中的方法是一个带有接收器的函数。尽管方法看起来像普通函数，但它有一个接收器。您可以在 `func` 关键字和函数名之间指定方法接收器。在清单 3-6 中，`Person` 结构体作为接收器被添加到了函数 `PrintName` 和 `PrintDetails` 中。完成此操作后，函数就附加到了 `Person` 类型上，可以作为方法来调用。这使得您可以从结构体类型实例中使用点（`.`）运算符来调用这些函数，就像在传统面向对象语言中调用类的实例方法一样。

这里声明了带有 `Person` 接收器类型的方法：

```
func (p Person) PrintName() {
    fmt.Printf("\n%s %s\n", p.FirstName, p.LastName)
}

//一个人物方法
func (p Person) PrintDetails() {
    fmt.Printf("[Date of Birth: %s, Email: %s, Location: %s ]\n", p.Dob.String(), p.Email, p.Location)
}
```



#### 调用结构体方法

让我们创建一个 `Person` 类型的实例，并调用该类型提供的方法（见代码清单 3-7）。

```
代码清单 3-7.  调用结构体方法

p := Person{
        "Shiju",
        "Varghese",
        time.Date(1979, time.February, 17, 0, 0, 0, 0, time.UTC),
        "shiju@email.com",
        "Kochi",
}

p.PrintName()
p.PrintDetails()
```

你将看到以下输出：

```
Shiju Varghese
[Date of Birth: 1979-02-17 00:00:00 +0000 UTC, Email: shiju@email.com, Location: Kochi ]
```

#### 指针方法接收器

在 Go 语言中，你可以使用指针和非指针方法接收器来定义方法。代码清单 3-7 中的 `PrintName` 和 `PrintDetails` 方法使用了非指针接收器。如果你想在方法内部修改接收器的数据，那么接收器必须是指针类型，如代码清单 3-8 所示。

```
代码清单 3-8.  使用指针的接收器方法

//一个使用指针接收器的 person 方法
func (p *Person) ChangeLocation(newLocation string) {
    p.Location = newLocation
}
```

`ChangeLocation` 函数绑定到了一个指针接收器上。这里直接修改了方法内的 Location 字段。假设你将这个新方法添加到代码清单 3-7 中定义的 `Person` 结构体中。让我们创建一个 `Person` 类型实例并调用其方法（见代码清单 3-9）。

```
代码清单 3-9.  调用指针接收器的方法

p := &Person{
                "Shiju",
                "Varghese",
                time.Date(1979, time.February, 17, 0, 0, 0, 0, time.UTC),
                "shiju@email.com",
                "Kochi",
        }

        p.ChangeLocation("Santa Clara")
        p.PrintName()
        p.PrintDetails()
```

你将看到以下输出：

```
Shiju Varghese
[Date of Birth: 1979-02-17 00:00:00 +0000 UTC, Email: shiju@email.com, Location: Santa Clara ]
```

因为需要调用指针接收器方法，所以创建 `Person` 类型实例时使用了取地址符 (`&`)。调用 `ChangeLocation` 方法会修改 `Location` 字段的值，因此当调用 `PrintDetails` 方法时，你将得到修改后的值。如果 `ChangeLocation` 方法是非指针方法，`Location` 字段的值就不会被改变。

当你调用指针接收器方法时，传递的是引用，而非值传递（非指针接收器使用值传递）。当你想在方法内部修改接收器的状态（字段的值）时，这非常合理。如果结构体的某些方法使用了指针接收器，那么最好在其所有方法中都使用指针接收器，因为这能使结构体的行为具有更好的一致性和可预测性。

让我们修改程序，将 `Person` 结构体的所有方法都定义为使用指针接收器，以保持其所有方法的一致性（见代码清单 3-10）。

```
代码清单 3-10.  使用指针接收器方法的 Person 结构体

package main

import (
        "fmt"
        "time"
)

func main() {
        p := &Person{
                "Shiju",
                "Varghese",
                time.Date(1979, time.February, 17, 0, 0, 0, 0, time.UTC),
                "shiju@email.com",
                "Kochi",
        }

        p.ChangeLocation("Santa Clara")
        p.PrintName()
        p.PrintDetails()
}

type Person struct {
        FirstName, LastName string
        Dob                 time.Time
        Email, Location     string
}

//一个使用指针接收器的 person 方法
func (p *Person) PrintName() {
        fmt.Printf("\n%s %s\n", p.FirstName, p.LastName)
}

//一个使用指针接收器的 person 方法
func (p *Person) PrintDetails() {
        fmt.Printf("[Date of Birth: %s, Email: %s, Location: %s ]\n", p.Dob.String(), p.Email, p.Location)
}

//一个使用指针接收器的 person 方法
func (p *Person) ChangeLocation(newLocation string) {
        p.Location = newLocation
}
```



#### 类型组合

Go 的设计理念是成为一门专注于实际应用的简单语言；为了保持其极简特性，它忽略了许多学术观点。你在前面的章节中已经看到了 Go 类型系统的简洁性。其类型系统的一个主要决策是，虽然它不支持继承，但通过类型嵌入（type embedding）支持组合。Go 鼓励你使用组合而非继承。

**注意**：组合是一种设计理念，即将较小的组件组合成较大的组件。

前一节定义了 `Person` 类型。你可以通过嵌入 `Person` 类型来创建更大、更具体的类型。在代码清单 3-11 中，通过嵌入 `Person` 类型创建了另外两种类型。

**代码清单 3-11. 用于组合的类型嵌入**

```
type Admin struct {
    Person //用于组合的类型嵌入
    Roles  []string
}

type Member struct {
    Person //用于组合的类型嵌入
    Skills []string
}
```

`Person` 类型被嵌入到 `Admin` 和 `Member` 类型中，因此 `Person` 的所有字段和方法都将在这两种新类型中可用。

让我们创建一个示例程序来理解类型嵌入的功能（参见代码清单 3-12）。

**代码清单 3-12. 类型组合的示例程序**

```
package main

import (
    "fmt"
    "time"
)

type Person struct {
    FirstName, LastName string
    Dob                 time.Time
    Email, Location     string
}

//Person 的一个方法
func (p Person) PrintName() {
    fmt.Printf("\n%s %s\n", p.FirstName, p.LastName)
}

//Person 的一个方法
func (p Person) PrintDetails() {
    fmt.Printf("[Date of Birth: %s, Email: %s, Location: %s ]\n", p.Dob.String(), p.Email, p.Location)
}

type Admin struct {
    Person //用于组合的类型嵌入
    Roles  []string
}

type Member struct {
    Person //用于组合的类型嵌入
    Skills []string
}

func main() {
    alex := Admin{
        Person{
            "Alex",
            "John",
            time.Date(1970, time.January, 10, 0, 0, 0, 0, time.UTC),
            "alex@email.com",
            "New York"},
        []string{"Manage Team", "Manage Tasks"},
    }

    shiju := Member{
        Person{
            "Shiju",
            "Varghese",
            time.Date(1979, time.February, 17, 0, 0, 0, 0, time.UTC),
            "shiju@email.com",
            "Kochi"},
        []string{"Go", "Docker", "Kubernetes"},
    }

    //为 alex 调用方法
    alex.PrintName()
    alex.PrintDetails()

    //为 shiju 调用方法
    shiju.PrintName()
    shiju.PrintDetails()
}
```

创建了一个带有几个字段和两个方法的 `Person` 类型。然后通过嵌入 `Person` 类型创建了另外两种类型：`Admin` 和 `Member`。这些新类型拥有 `Person` 类型提供的所有属性和方法。可以像这样通过简单地嵌入 `Person` 类型来创建 `Admin` 和 `Member` 类型的实例：

```
alex := Admin{
    Person{
        "Alex",
        "John",
        time.Date(1970, time.January, 10, 0, 0, 0, 0, time.UTC),
        "alex@email.com",
        "New York"},
    []string{"Manage Team", "Manage Tasks"},
}

shiju := Member{
    Person{
        "Shiju",
        "Varghese",
        time.Date(1979, time.February, 17, 0, 0, 0, 0, time.UTC),
        "shiju@email.com",
        "Kochi"},
    []string{"Go", "Docker", "Kubernetes"},
}
```

因为 `Person` 类型的所有方法对 `Admin` 和 `Member` 类型都是可用的，你可以像如下代码所示调用这些方法：

```
//为 alex 调用方法
alex.PrintName()
alex.PrintDetails()

//为 shiju 调用方法
shiju.PrintName()
shiju.PrintDetails()
```

你应该会看到以下输出：

```
Alex John
[Date of Birth: 1970-01-10 00:00:00 +0000 UTC, Email: alex@email.com, Location: New York ]
Shiju Varghese
[Date of Birth: 1979-02-17 00:00:00 +0000 UTC, Email: shiju@email.com, Location: Kochi ]
```

通过类型嵌入实现的类型组合既提供了使用继承的实际好处，又具有更好的可维护性。

### 覆盖嵌入类型的方法

在代码清单 3-11 中，`Person` 类型被嵌入到 `Admin` 和 `Member` 类型中，并调用了 `Person` 类型提供的方法。假设你想在 `Admin` 和 `Member` 中覆盖 `PrintDetails` 方法，因为这些类型有额外的字段：`Roles` 和 `Skills`。嵌入类型的方法可以被覆盖，如代码清单 3-13 所示。

**代码清单 3-13. 覆盖嵌入类型的方法**

```
type Admin struct {
    Person //用于组合的类型嵌入
    Roles  []string
}

//覆盖 PrintDetails
func (a Admin) PrintDetails() {
    //调用 Person 的 PrintDetails
    a.Person.PrintDetails()
    fmt.Println("Admin Roles:")
    for _, v := range a.Roles {
        fmt.Println(v)
    }
}

type Member struct {
    Person //用于组合的类型嵌入
    Skills []string
}

//覆盖 PrintDetails
func (m Member) PrintDetails() {
    //调用 Person 的 PrintDetails
    m.Person.PrintDetails()
    fmt.Println("Skills:")
    for _, v := range m.Skills {
        fmt.Println(v)
    }
}
```

`PrintDetails` 方法被覆盖以包含每个类型持有的额外信息。当此方法被覆盖时，你可以调用嵌入类型的 `PrintDetails` 方法来获取嵌入类型提供的基本信息，然后提供这些类型持有的额外信息：

```
func (m Member) PrintDetails() {
    //调用 Person 的 PrintDetails
    m.Person.PrintDetails()
    fmt.Println("Skills:")
    for _, v := range m.Skills {
        fmt.Println(v)
    }
}
```

语句 `m.Person.PrintDetails()` 调用了 `Person` 类型的 `PrintDetails` 方法。

让我们运行这个修改后的程序，其代码如代码清单 3-14 所示。

**代码清单 3-14. 运行包含方法覆盖的程序**

```
alex := Admin{
    Person{
        "Alex",
        "John",
        time.Date(1970, time.January, 10, 0, 0, 0, 0, time.UTC),
        "alex@email.com",
        "New York"},
    []string{"Manage Team", "Manage Tasks"},
}

shiju := Member{
    Person{
        "Shiju",
        "Varghese",
        time.Date(1979, time.February, 17, 0, 0, 0, 0, time.UTC),
        "shiju@email.com",
        "Kochi"},
    []string{"Go", "Docker", "Kubernetes"},
}

//为 alex 调用方法
alex.PrintName()
alex.PrintDetails()

//为 shiju 调用方法
shiju.PrintName()
shiju.PrintDetails()
```

你应该会看到以下输出：

```
Alex John
[Date of Birth: 1970-01-10 00:00:00 +0000 UTC, Email: alex@email.com, Location: New York ]
Admin Roles:
Manage Team
Manage Tasks
Shiju Varghese
[Date of Birth: 1979-02-17 00:00:00 +0000 UTC, Email: shiju@email.com, Location: Kochi ]
Skills:
Go
Docker
Kubernetes
```



### 使用接口

在 Go 语言的类型系统中，你可以创建具体类型和接口类型。（具体类型已在本章前几节讨论过。）接口是 Go 语言最强大的特性之一，因为它为用户自定义的具体类型提供了契约，从而允许你为对象定义行为。接下来，让我们创建一个接口类型来为 `Person` 对象指定行为（参见清单 3-15）。

**清单 3-15.** 定义接口类型

```go
type User interface {
    PrintName()
    PrintDetails()
}
```

上述代码定义了一个名为 `User` 的接口类型，包含两个行为：`PrintName` 和 `PrintDetails`。现在来看看 `Person` 类型：

```go
type Person struct {
    FirstName, LastName string
    Dob                 time.Time
    Email, Location     string
}

// Person 的方法
func (p Person) PrintName() {
    fmt.Printf("\n%s %s\n", p.FirstName, p.LastName)
}

// Person 的方法
func (p Person) PrintDetails() {
    fmt.Printf("[Date of Birth: %s, Email: %s, Location: %s ]\n", p.Dob.String(), p.Email, p.Location)
}
```

关于接口最令人惊讶的一点是：你无需在具体类型中显式实现接口，只需根据接口类型的规范在具体类型中定义方法即可。`Person` 类型已经实现了 `User` 接口。它实现了在 `User` 接口类型中定义的 `PrintName` 和 `PrintDetails` 方法。在 C# 和 Java 等编程语言中，你必须显式地将接口类型实现到具体类型中。Go 语言在保持自身为静态类型语言的同时，提供了很高的开发效率。

**清单 3-16** 展示了一个示例程序，描述了接口及其具体类型：

**清单 3-16.** 包含接口的示例程序

```go
package main

import (
    "fmt"
    "time"
)

type User interface {
    PrintName()
    PrintDetails()
}

type Person struct {
    FirstName, LastName string
    Dob                 time.Time
    Email, Location     string
}

// Person 的方法
func (p Person) PrintName() {
    fmt.Printf("\n%s %s\n", p.FirstName, p.LastName)
}

// Person 的方法
func (p Person) PrintDetails() {
    fmt.Printf("[Date of Birth: %s, Email: %s, Location: %s ]\n", p.Dob.String(), p.Email, p.Location)
}

func main() {
    alex := Person{
        "Alex",
        "John",
        time.Date(1970, time.January, 10, 0, 0, 0, 0, time.UTC),
        "alex@email.com",
        "New York",
    }

    shiju := Person{
        "Shiju",
        "Varghese",
        time.Date(1979, time.February, 17, 0, 0, 0, 0, time.UTC),
        "shiju@email.com",
        "Kochi",
    }

    users := []User{alex, shiju}

    for _, v := range users {
        v.printName()
        v.PrintDetails()
    }
}
```

基于接口类型定义的行为，创建了一个接口类型和一个具体类型。在 `main` 函数中，创建了两个 `Person` 对象，然后创建了一个包含这两个 `Person` 对象的用户接口类型切片。最后，遍历该集合并调用了接口类型中定义的方法：`PrintName` 和 `PrintDetails`。

你应该会看到以下输出：

```
Alex John
[Date of Birth: 1970-01-10 00:00:00 +0000 UTC, Email: alex@email.com, Location: New York ]
Shiju Varghese
[Date of Birth: 1979-02-17 00:00:00 +0000 UTC, Email: shiju@email.com, Location: Kochi ]
```

接下来，通过编写 **清单 3-17** 中的示例程序，我们将本章讨论的用户自定义类型的不同特性结合起来。

**清单 3-17.** 包含接口、组合和方法重写的示例程序

```go
package main

import (
    "fmt"
    "time"
)

type User interface {
    PrintName()
    PrintDetails()
}

type Person struct {
    FirstName, LastName string
    Dob                 time.Time
    Email, Location     string
}

// Person 的方法
func (p Person) PrintName() {
    fmt.Printf("\n%s %s\n", p.FirstName, p.LastName)
}

// Person 的方法
func (p Person) PrintDetails() {
```


```go
fmt.Printf("[出生日期: %s, 邮箱: %s, 位置: %s ]\n", p.Dob.String(), p.Email, p.Location)
```

```go
}
```

```go
type Admin struct {
    Person //通过类型嵌入实现组合
    Roles  []string
}
```

```go
//重写 PrintDetails 方法
func (a Admin) PrintDetails () {
    //调用 Person 的 PrintDetails 方法
    a.Person.PrintDetails()
    fmt.Println("管理员角色:")
    for _, v := range a.Roles {
        fmt.Println(v)
    }
}
```

```go
type Member struct {
    Person //通过类型嵌入实现组合
    Skills []string
}
```

```go
//重写 PrintDetails 方法
func (m Member) PrintDetails () {
    //调用 Person 的 PrintDetails 方法
    m.Person.PrintDetails()
    fmt.Println("技能:")
    for _, v := range m.Skills {
        fmt.Println(v)
    }
}
```

```go
type Team struct {
    Name, Description string
    Users             []User
}
```

```go
func (t Team) GetTeamDetails() {
    fmt.Printf("团队: %s  - %s\n", t.Name, t.Description)
    fmt.Println("团队成员详情:")
    for _, v := range t.Users {
        v.PrintName()
        v.PrintDetails()
    }
}
```

```go
func main() {
    alex := Admin{
        Person{
            "亚历克斯",
            "约翰",
            time.Date(1970, time.January, 10, 0, 0, 0, 0, time.UTC),
            "alex@email.com",
            "纽约"},
        []string{"管理团队", "管理任务"},
    }

    shiju := Member{
        Person{
            "希居",
            "瓦尔盖斯",
            time.Date(1979, time.February, 17, 0, 0, 0, 0, time.UTC),
            "shiju@email.com",
            "科钦"},
        []string{"Go", "Docker", "Kubernetes"},
    }

    chris := Member{
        Person{
            "克里斯",
            "马丁",
            time.Date(1978, time.March, 15, 0, 0, 0, 0, time.UTC),
            "chris@email.com",
            "圣克拉拉"},
        []string{"Go", "Docker"},
    }

    team := Team{
        "Go",
        "Golang 卓越中心",
        []User{alex, shiju, chris},
    }

    //获取团队详情
    team.GetTeamDetails()
}
```

您应该会看到以下输出：

```text
团队: Go  - Golang 卓越中心
```

以下是团队成员的详细信息：

```text
亚历克斯·约翰
[出生日期: 1970-01-10 00:00:00 +0000 UTC, 邮箱: alex@email.com, 位置: 纽约 ]
管理员角色:
管理团队
管理任务
希居·瓦尔盖斯
[出生日期: 1979-02-17 00:00:00 +0000 UTC, 邮箱: shiju@email.com, 位置: 科钦 ]
技能:
Go
Docker
Kubernetes
克里斯·马丁
[出生日期: 1978-03-15 00:00:00 +0000 UTC, 邮箱: chris@email.com, 位置: 圣克拉拉 ]
技能:
Go
Docker
```

除了代码清单 3-16 中定义的类型之外，还新增了接口类型的两个具体类型（`User` - `Admin` 和 `Member`），其中嵌入了 `Person` 类型。同时修改了 `Admin` 和 `Member` 类型的 `PrintName` 和 `PrintDetails` 方法。最后，创建了一个新的结构体类型 (`Team`)，它包含两个字符串字段和一个用户接口类型的切片，其中可以添加 `Person`、`Admin` 或 `Member` 类型的对象。在 `GetTeamDetails` 方法中，遍历由 `Admin` 和 `Member` 类型组成的 `Users` 集合，并调用 `PrintName` 和 `PrintDetails` 方法：

```go
type Team struct {
    Name, Description string
    Users             []User
}

func (t Team) GetTeamDetails() {
    fmt.Printf("团队: %s  - %s\n", t.Name, t.Description)
    fmt.Println("团队成员详情:")
    for _, v := range t.Users {
        v.PrintName()
        v.PrintDetails()
    }
}
```

这个程序展示了使用接口类型的强大之处。当创建 `Team` 类型对象时，提供了一个用户接口的切片，其中 `Users` 字段由两个 `Member` 类型对象和一个 `Admin` 类型对象组成。能够为 `Users` 字段提供不同类型的对象，是因为这些对象都实现了接口 `User`。

接口是一个契约，它为程序增加了可扩展性和可维护性，允许你根据抽象而非具体实现来编写程序。在示例程序中，`Users` 字段被定义为用户接口类型的切片，这样你就可以提供任何类型的对象来实现用户接口中定义的契约。如果你将 `Users` 属性的类型定义为 `Member` 类型的切片，那么为 `Users` 字段提供的值将仅限于 `Member` 类型的对象。

#### 并发

在开发大型应用程序时，可能需要完成多个任务才能执行程序。其他程序则由许多较小的子程序组成。当开发这类应用程序时，如果能够并发地执行这些任务和子程序，就可以提升性能。假设你正在开发一个基于 Web 的后端 API，许多并发用户正在访问该 API。如果你能在 Web 服务器上并发地处理这些并发的 Web 请求，就可以显著提高系统的性能和效率。

在开发 Web 应用程序和 Web API 时，管理大量并发用户确实是一个挑战。Go 语言旨在解决现代编程和大型系统面临的挑战。它在核心语言层面提供了对并发的支持，并将并发直接实现在其语言和运行时中。这有助于你轻松构建高性能系统。

许多编程环境通过额外的库来提供并发支持，但并发并非核心语言的内置特性。并发性是 Go 语言的主要卖点之一，与其简洁性和实用性齐名。在 Go 中，并发是通过使用两个独特的功能实现的：goroutine 和 channel。


### Goroutine（协程）

在 Go 语言中，goroutine 是实现程序并发运行的主要机制。Goroutine 允许你将函数与其他函数并发执行，通过将函数作为 goroutine 调用即可访问 Go 的并发能力。当你将一个函数创建为 goroutine 时，它会作为一个独立的任务单元，与其他 goroutine 并发运行。简而言之，goroutine 是由 Go 运行时管理的轻量级线程。

Go 并发最强大的能力在于，所有与并发相关的内容完全由 Go 运行时管理，而非操作系统资源。Go 运行时拥有一个名为调度器的强大软件组件，它控制并管理所有与 goroutine 调度和运行相关的操作。由于 Go 运行时完全控制着使用 goroutine 运行的并发任务，因此当你利用 Go 的并发能力时，它可以实现高性能并对应用程序提供更好的控制。

要调用一个函数作为 goroutine，请使用关键字 `go`：

```
go function()
```

由于 goroutine 被视为独立并发运行的单元，请确保所有 goroutine 在主程序终止前都得到执行。你可以通过使用 `sync` 标准库包提供的 `WaitGroup` 类型来实现这一点。

清单 [3-18] 是一个示例程序，其中启动了两个 goroutine，并在程序终止前执行它们。

**清单 3-18.** 使用 Goroutine 实现并发

```
// 此示例程序演示如何创建 goroutine

package main

import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)

// wg 用于等待程序完成 goroutine 的执行。
var wg sync.WaitGroup

func main() {
    // 添加计数 2，每个 goroutine 对应一个。
    wg.Add(2)
    fmt.Println("启动 Goroutines")
    // 启动一个标签为 "A" 的 goroutine
    go printCounts("A")
    // 启动一个标签为 "B" 的 goroutine
    go printCounts("B")
    // 等待 goroutine 完成。
    fmt.Println("等待完成")
    wg.Wait()
    fmt.Println("\n 终止程序")
}

func printCounts(label string) {
    // 安排调用 WaitGroup 的 Done 方法以告知执行完毕。
    defer wg.Done()
    // 随机等待
    for count := 1; count <= 10; count++ {
        sleep := rand.Int63n(1000)
        time.Sleep(time.Duration(sleep) * time.Millisecond)
        fmt.Printf("计数: %d 来自 %s\n", count, label)
    }
}
```

运行程序时，你会看到以下输出。由于程序执行过程中的随机等待，每次输出都会有所不同：

```
启动 Goroutines
等待完成
计数: 1 来自 A
计数: 1 来自 B
计数: 2 来自 B
计数: 2 来自 A
计数: 3 来自 B
计数: 3 来自 A
计数: 4 来自 A
计数: 5 来自 A
计数: 4 来自 B
计数: 6 来自 A
计数: 5 来自 B
计数: 7 来自 A
计数: 6 来自 B
计数: 7 来自 B
计数: 8 来自 A
计数: 8 来自 B
计数: 9 来自 B
计数: 9 来自 A
计数: 10 来自 A
计数: 10 来自 B
终止程序
```

创建了一个名为 `printCounts` 的函数，该函数被作为 goroutine 调用两次：

```
// 启动一个标签为 "A" 的 goroutine
go printCounts("A")
// 启动一个标签为 "B" 的 goroutine
go printCounts("B")
```

为了确保所有 goroutine 在程序终止前都得到执行，使用了由 `sync` 包提供的 `WaitGroup`：

```
var wg sync.WaitGroup
```

在 `main` 函数中，为两个 goroutine 向 `WaitGroup` 添加计数 `2`：

```
wg.Add(2)
```

使用关键字 `go` 启动两个 goroutine：

```
// 启动一个标签为 "A" 的 goroutine
go printCounts("A")
// 启动一个标签为 "B" 的 goroutine
go printCounts("B")
```

在 `printCounts` 函数中，打印了从 1 到 10 的值。为了演示效果，执行过程被随机延迟。

```
func printCounts(label string) {
    // 安排调用 WaitGroup 的 Done 方法以告知执行完毕。
    defer wg.Done()
    // 随机等待
    for count := 1; count <= 10; count++ {
        sleep := rand.Int63n(1000)
        time.Sleep(time.Duration(sleep) * time.Millisecond)
        fmt.Printf("计数: %d 来自 %s\n", count, label)
    }
}
```

在 `printCounts` 函数的开头，安排了调用 `WaitGroup` 类型的 `Done` 方法，以告知 `main` 程序该 goroutine 已执行完毕。`WaitGroup` 类型的 `Done` 方法通过使用关键字 `defer`（如[第 2 章]所述）来安排调用。这个关键字允许你在函数返回时安排其他函数被调用。在此示例中，当 goroutine 执行完毕时会调用 `Done` 方法，这会确保 `WaitGroup` 的值递减，以便 `main` 函数可以检查是否有 goroutine 尚未执行。在 `main` 函数中，调用了 `WaitGroup` 的 `Wait` 方法，该方法会检查 `WaitGroup` 的计数，并阻塞程序直到计数变为零。当调用 `WaitGroup` 的 `Done` 方法时，计数会减一。在程序执行开始时，计数被设置为 2。当调用 `Wait` 时，它会等待计数变为零，从而确保两个 goroutine 在程序终止前都得到执行。当计数变为零时，程序终止：

```
wg.Wait()
```

### GOMAXPROCS 与并行性

Go 运行时调度器通过利用操作系统线程的数量来尝试同时执行 goroutine，从而管理 goroutine 的执行。操作系统线程的值取自 `GOMAXPROCS` 环境变量。在 Go 1.5 之前，`GOMAXPROCS` 的默认设置为 1，这意味着 goroutine 默认在单个操作系统线程上运行。（请记住，Go 运行时可以在单个操作系统线程上执行数千个 goroutine。）从 Go 1.5 开始，`GOMAXPROCS` 的默认设置改为可用 CPU 的数量，由 `runtime` 包提供的 `NumCPU` 函数确定。此默认行为让 Go 程序能够利用所有可用的 CPU 核心来并行运行 goroutine。`GOMAXPROCS` 的值可以通过显式设置 `GOMAXPROCS` 环境变量或从程序中调用 `runtime.GOMAXPROCS` 来设定。

通过提高 `GOMAXPROCS` 设置可以实现 goroutine 的同时执行。这种行为就是并行性。并发不是并行性；Go 中的并发是指通过将程序分解为可以独立执行（利用可用资源）的 goroutine（作为子程序或任务）来设计程序。并行性则是关于一次完成大量计算。在并行性中，会利用更多的 CPU 核心来尽可能实现 goroutine 的同时执行。

在许多情况下，并行性可以提供更好的性能，因为 Go 运行时可以利用计算机所有可用的计算资源来并行运行 goroutine。请记住，并行运行 goroutine 并不总能带来更好的性能，因为这取决于你的程序上下文。有时并发性可以胜过并行性，因为并行性可能会给操作系统资源带来更大压力。

你可以根据应用程序的上下文在程序中控制 `GOMAXPROCS` 设置。清单 [3-19] 是一个将 `GOMAXPROCS` 从其默认设置修改为 1 的代码块：

**清单 3-19.** 显式设置 GOMAXPROCS 设置

```
import "runtime"

// 设置 GOMAXPROCS 的值。
runtime.GOMAXPROCS(1)
```

**注：** 请查阅 Rob Pike 的精彩演讲 “Concurrency Is Not Parallelism” 以了解并发与并行的区别：[`www.youtube.com/watch?v=cN_DpYBzKso`](http://www.youtube.com/watch?v=cN_DpYBzKso)



### 通道

清单 `3-18` 创建了两个独立运行、无需相互通信的 goroutine。然而，有时 goroutine 之间需要通信来发送和接收数据，因此需要在 goroutine 之间进行同步。在许多编程环境中，并发程序之间的通信要么很复杂，要么功能有限。Go 允许你通过通道在 goroutine 之间进行通信，从而能够同步 goroutine 的执行。

使用内置的 `make` 函数并借助关键字 `chan` 来声明通道，其后跟随类型，用于指定你要交换的数据类型：

`count := make(chan int)`

这里声明了一个整数 `类型` 的通道，因此整数值将被传入通道。有两种类型的通道可用于 goroutine 的同步：

- 有缓冲通道
- 无缓冲通道

清单 `3-20` 是声明无缓冲通道和有缓冲通道的代码块。

**清单 3-20.** 声明无缓冲通道和有缓冲通道

```
// 无缓冲整数通道。
count := make(chan int)

// 有缓冲整数通道，最多缓冲 10 个值。
count := make(chan int, 10)
```

当声明有缓冲通道时，必须指定通道容纳数据的容量。如果你尝试发送超过其容量的数据，将会收到错误提示。

#### 无缓冲通道

无缓冲通道提供 goroutine 之间的同步通信，从而确保消息在它们之间传递。对于无缓冲通道，只有当存在相应的接收者准备好接收消息时，才允许发送消息。在这种情况下，通道的双方都必须等待，直到另一方准备好发送和接收消息。而对于有缓冲通道，即使没有对应的并发接收者来接收消息，也可以向通道中发送有限数量的消息。消息被发送到有缓冲通道后，再从通道中接收这些消息。与无缓冲通道不同，有缓冲通道无法保证消息的传递。

`<-` 运算符用于向通道发送值（见清单 `3-21`）。

**清单 3-21.** 向通道发送值

```
// 有缓冲字符串通道。
messages := make(chan string, 2)

// 向通道发送一条消息。
messages <- "Golang"
```

要从通道接收消息，`<-` 运算符被用作一元运算符（见清单 `3-22`）。

**清单 3-22.** 从通道接收值

```
// 从通道接收一个字符串值。
value := <-messages
```

清单 `3-23` 是一个示例程序，演示了如何使用无缓冲通道在 goroutine 之间通信和同步数据。

**清单 3-23.** 使用无缓冲通道的示例程序

```
package main

import (
	"fmt"
	"sync"
)

// wg 用于等待程序结束。
var wg sync.WaitGroup

func main() {
	count := make(chan int)
	// 添加计数 2，每个 goroutine 对应一个。
	wg.Add(2)
	fmt.Println("启动 Goroutines")
	//启动一个标签为 "A" 的 goroutine
	go printCounts("A", count)
	//启动一个标签为 "B" 的 goroutine
	go printCounts("B", count)
	fmt.Println("通道开始")
	count <- 1
	// 等待 goroutines 完成。
	fmt.Println("等待完成")
	wg.Wait()
	fmt.Println("\n 终止程序")
}

func printCounts(label string, count chan int) {
	// 安排调用 WaitGroup 的 Done 来告知我们已完成。
	defer wg.Done()
	for {
		//从通道接收消息
		val, ok := <-count
		if !ok {
			fmt.Println("通道已关闭")
			return
		}
		fmt.Printf("计数: %d 从 %s 接收 \n", val, label)
		if val == 10 {
			fmt.Printf("通道从 %s 关闭 \n", label)
			// 关闭通道
			close(count)
			return
		}
		val++
		// 将计数发回给另一个 goroutine。
		count <- val
	}
}
```

运行该程序时，输出可能会有所不同。你应该会看到类似于以下的输出：

```
启动 Goroutines
通道开始
计数: 1 从 A 接收
计数: 2 从 B 接收
等待完成
计数: 3 从 A 接收
计数: 4 从 B 接收
计数: 5 从 A 接收
计数: 6 从 B 接收
计数: 7 从 A 接收
计数: 8 从 B 接收
计数: 9 从 A 接收
计数: 10 从 B 接收
通道从 B 关闭
通道已关闭
终止程序
```

在这个程序中，两个 goroutine 使用无缓冲通道进行同步和通信。这里使用了一个整数类型的通道用于 goroutine 之间的数据交换。在 `main` 函数中，声明了无缓冲通道 `count` 用于两个 goroutine 之间的消息交换。为两个 goroutine 在 `WaitGroup` 中添加了一个计数 2。然后通过传递该通道启动了两个 goroutine：

```
count := make(chan int)

// 添加计数 2，每个 goroutine 对应一个。
wg.Add(2)

//启动一个标签为 "A" 的 goroutine
go printCounts("A", count)

//启动一个标签为 "B" 的 goroutine
go printCounts("B", count)
```

在 `printCounts` 函数内部，使用了一个无限 `for` 循环，当从通道接收到值 `10` 时，从函数中返回。启动两个 goroutine 后，通过发送值 `1` 来启动通道，该值将由某个 goroutine 接收。无缓冲通道会阻塞接收者，直到通道中有消息可用。

`printCounts` 函数中的以下代码块会阻塞，直到值被发送到通道中：

```
//从通道接收消息
val, ok := <-count
```

当从通道接收值时，可以使用赋值左侧的两个变量：一个是从通道接收的数据，另一个是表示通道是否可用的布尔值。如果通道已关闭，我们就从函数返回。

```
if !ok {
	fmt.Println("通道已关闭")
	return
}
```

当从通道接收到值 `10` 时，通道被关闭。如果接收到的通道值小于 `10`，则对该值进行自增，并将新值发送到通道中。这个过程会阻塞发送者，直到某个接收者从通道中接收到该值。

```
if val == 10 {
	fmt.Printf("通道从 %s 关闭 \n", label)
	// 关闭通道
	close(count)
	return
}
val++
// 将计数发回给另一个 goroutine。
count <- val
```

关于无缓冲通道最重要的是：对通道的接收操作会阻塞 goroutine，直到通道获取到数据；对通道的发送操作会阻塞 goroutine，直到有接收者接收到数据，从而确保了 goroutine 之间的消息传递。如果你查看程序输出，就能清楚地理解通道的功能。



#### 缓冲通道

无缓冲通道在 Goroutine 之间提供了一种同步的数据通信方式，能够确保消息的可靠投递。缓冲通道则与此不同。与无缓冲通道不同，缓冲通道在创建时会指定其可容纳的值的数量。缓冲通道在收到值之前，可以接受指定数量的值。

清单 3-24 是一个演示缓冲通道的基础示例程序。

**清单 3-24.** 使用缓冲通道的示例程序

```
package main

import "fmt"

func main() {

        messages := make(chan string, 2)

        messages <- "Golang"
        messages <- "Gopher"

        //从缓冲通道接收值
        fmt.Println(<-messages)
        fmt.Println(<-messages)

}
```

你应该会看到以下输出：

```
Golang
Gopher
```

清单 3-24 是一个简单的示例程序，演示了无需任何 Goroutine 就能使用的缓冲通道。它创建了一个 `messages` 通道，用于缓冲最多两个字符串值。值被发送到通道中，直到达到其容量（两个）。由于这个通道是缓冲的，值的发送无需依赖接收方来接收它们。如果你尝试添加超过其容量的值，将会遇到错误。一旦值被发送到缓冲通道中，就可以从通道中接收它们。

### 总结

Go 的类型系统包含两种基本类型：具体类型和接口类型。你可以使用内置类型（如 `bool`、`int`、`string` 和 `float64`）来创建具体类型。你还可以创建复合类型，如数组、切片、映射和通道，以及自定义的用户定义类型。

在 Go 中，结构体用于创建用户定义类型。结构体类似于经典面向对象语言中的类，但与其他语言相比，其设计是独特的。结构体是类的轻量级版本。Go 的类型系统不支持继承。相反，你可以通过类型嵌入来组合类型。

接口类型是一个强大的特性；它允许你在构建软件系统时提供极大的扩展性和组合性。接口为用户定义的具体类型提供契约。在 Go 中，你无需显式地将接口实现到具体类型中；你只需根据接口类型中定义的方法，在你的具体类型中提供方法的实现，即可将接口实现到具体类型中。

Go 中的并发性是函数与其他函数同时运行的能力。并发性是 Go 语言的内置特性，Go 运行时使用调度器来管理并发函数的执行。Go 中的并发性通过两个特性来实现：Goroutine 和通道。Goroutine 是可以与其他函数并发运行的函数，作为一个独立的单元工作。通道用于同步 Goroutine 以发送和接收消息。在 Go 中，你可以创建两种类型的通道：缓冲通道和无缓冲通道。无缓冲通道会阻塞 Goroutine 的接收者，直到通道上有可用的数据；同时也会阻塞 Goroutine 的发送者，直到有接收者可用。缓冲通道仅在缓冲区已满时阻塞发送者。

在第 2 章和第 3 章中，你已经学习了 Go 编程语言的基础知识。从第 4 章开始，你将学习 Go 中的 Web 编程以及如何开发 Web 应用和 RESTful 服务。

