# 创建复杂的数据模型

Contoso 大学示例 Web 应用程序演示如何使用实体框架（EF）Core 2.0 和 Visual Studio 2017 创建 ASP.NET Core 2.0 MVC Web 应用程序。 如欲了解更多本教程相关信息，请参阅 [一、入门](./chapters/start.md)

在前面的教程中，您使用由三个实体组成的简单数据模型。 在本章中， 您将添加多个实体和关系，并将通过指定格式化、验证以及数据库映射规则来自定义数据模型。

完成时，实体类构成的完整数据模型如下图所示：

![diagram.png](./Images/diagram.png)

## 使用特性自定义数据模型

在本节中，你将了解如何通过使用指定格式设置，验证和数据库的映射规则的属性来自定义数据模型。 然后在接下来的几节中，通过添加特性到类中，以及添加新的类来完成完整的 School 数据模型。

#### ```DataType``` 特性（attribute）

对于学生注册日期，目前所有的网页同时显示日期和时间，虽然您关注的只是日期。通过使用数据注解特性，您可以只在一个地方进行代码更改，然后所有显示注册日期的格式将得到修正。 为了演示如何达成此目的，您将在 ```Student``` 类的 ```EnrollementDate``` 属性上添加一个 ```Attribute``` （特性）。

在 ```Models/Student.cs``` 中，添加 ```System.ComponentModel.DataAnnotations``` 命名空间的引用，并在 ```EnrollmentDate``` 属性上添加 ```DataType``` 和 ```DisplayFormat``` 特性，如下代码所示：

<pre>
using System;
using System.Collections.Generic;
<span style="background-color: #FF0;">using System.ComponentModel.DataAnnotations;</span>

namespace ContosoUniversity.Models
{
    public class Student
    {
        public int ID { get; set; }
        public string LastName { get; set; }
        public string FirstMidName { get; set; }

        <span style="background-color: #FF0;">[DataType(DataType.Date)]
        [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]</span>
        public DateTime EnrollmentDate { get; set; }

        public ICollection<Enrollment> Enrollments { get; set; }
    }
}
</pre>

```DataType``` 特性用于指定更为准确的数据库类型。在这个例子中，我们只想跟踪日期，而不是日期和时间。 DataType 枚举中包含许多数据类型，例如 Date, Time, PhoneNumber, Currency, EmailAddress, 等等。应用程序还可通过 DataType 特性自动提供类型特定的功能。例如，可以为 DataType.EmailAddress 创建 mailto: 链接，并且可以在支持 HTML5 的浏览器中为 DataType.Date 提供日期选择器。 DataType 特性生成 HTML5 浏览器可以理解的 HTML 5 data- (发音为 data dash) 。 DataType 特性不提供任何验证。

DataType.Date 不指定显示日期的格式。 默认情况下，数据字段基于服务器 CultureInfo 的默认格式显示。

DisplayFormat 特性用于显式指定日期格式：

``` cs
[DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
```

```ApplyFormatInEditMode``` 设置在文本框编辑模式下也应用数据格式。（有时候您可能不愿意在某些字段上实现 -- 例如，对于货币值，您可能不想在文本框编辑模式下出现货币符号。)

您可以单独使用 ```DisplayFormat``` 特性， 但通常情况下，同时使用 ```DataType``` 比较好。 ```DataType``` 特性传达的是数据的语义而不是如何在屏幕上显示，并提供了您无法从使用 ```Displayformat``` 中得到的好处：

* 浏览器可以启用 HTML5 功能 （例如显示一个日历控件、 区域设置对应的货币符号、 电子邮件链接，某些客户端输入验证，等等。）。

* 默认情况下，浏览器将根据区域设置采用正确的格式呈现数据。

有关详细信息，请参阅 [\<input> tag helper documentation](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms#the-input-tag-helper) 。

运行应用程序，转到 Student Index 页，可以看到，Enrollment Date 字段不再显示时间。 其他使用 Student 模型的视图也一样。

![dates-no-times.png](./Images/dates-no-times.png)

#### ```StringLength``` 特性

您还可以通过使用特性来指定数据验证规则及验证错误消息。 ```StringLength``` 特性设置在数据库中保存的最大长度，并为 ASP.NET MVC 应用提供客户端和服务端验证。 您还可以在这个特性中指定最小字符串长度，但最小值对数据库结构没有任何影响。

现在假设您想确保用户不能在名字中输入超过50个字符。 为了添加此限制， 在 ```LastName``` 和 ```FirstMidName``` 属性上添加 ```StringLength``` 特性，如下所示：

<pre>
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;

namespace ContosoUniversity.Models
{
    public class Student
    {
        public int ID { get; set; }

        <span style="background-color: #FF0;">[StringLength(50)]</span>
        public string LastName { get; set; }
        <span style="background-color: #FF0;">[StringLength(50, ErrorMessage = "First name cannot be longer than 50 characters.")]</span>
        public string FirstMidName { get; set; }
        

        [DataType(DataType.Date)]
        [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
        public DateTime EnrollmentDate { get; set; }

        public ICollection<Enrollment> Enrollments { get; set; }
    }
}
</pre>

```StringLength``` 特性并无法阻止用户在名字中输入空格。可以使用 ```RegularExpress``` （正则表达式）特性来限制输入。例如，下面的代码要求第一个字符是大写且其余的字符都是字母：

``` cs
[RegularExpression(@"^[A-Z]+[a-zA-Z''-'\s]*$")]
```

```MaxLength``` 特性提供了类似于 ```StringLength``` 特性的功能，但也没有提供客户端验证功能。

现在，数据模型已更改，并需要将此更改反映到数据库结构中。 您将使用迁移来更新数据库结构，同时不会丢失在使用应用程序过程中已经输入数据库的数据。

保存所做的更改并生成项目。 然后在项目文件夹中打开命令窗口并输入以下命令：

``` console
dotnet ef migrations add MaxLengthOnNames
```

``` console
dotnet ef database update
```

```migrations add``` 命令警告说可能发生数据丢失，因为更改导致两个数据列的最大长度变短。 迁移功能创建了一个名为 ```<时间戳>_MaxLengthOnNames.cs``` 的文件。 此文件中的 ```Up``` 方法将会更新数据库结构，以匹配当前数据模型。 ```database update``` 命令负责执行此代码。

Entity Framework 使用迁移文件名前的时间戳对迁移进行排序。 你可以在运行 update-database 命令前, 创建多个迁移，这样所有的迁移将按照创建顺序依次应用。

运行应用，转至 Student 页面，点击 Create New ， 并在 First Name 、Last Name 中输入超过 50 个字符。 当你单击创建，客户端验证显示一条错误消息。

![string-length-errors.png](./Images/string-length-errors.png)

#### ```Column``` 特性

特性还可用于控制如何类和属性映射到数据库。假设你已将 FirstMidName 用于 first-name 字段（因为字段中还包含 middle name）。但同时你又希望数据库中的列名为 FirstName， 因为编写针对数据库查询的用户习惯于该名称。 使用 ```Column``` 特性可以达成此映射要求。

```Column``` 特性指定当创建数据库时，映射到 Student表的 FirstMidName 属性将被命名为 FirstName。 换句话说， 当你的代码引用 Student.FirstMidName时， 数据读和写都是发生在 Student 表的 FirstName 列。 如果未指定列名称，系统使用属性名作为列名。

在 Student.cs 文件中， 添加 System.ComponentModel.DataAnnotations.Schema 命名空间引用，并在 FirstMidName 属性添加 ```column``` 特性，如下代码中高亮所示：

<pre>
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
<span style="background-color: #FF0;">using System.ComponentModel.DataAnnotations.Schema;</span>

namespace ContosoUniversity.Models
{
    public class Student
    {
        public int ID { get; set; }
        [StringLength(50)]
        public string LastName { get; set; }
        [StringLength(50, ErrorMessage = "First name cannot be longer than 50 characters.")]
        <span style="background-color: #FF0;">[Column("FirstName")]</span>
        public string FirstMidName { get; set; }
        [DataType(DataType.Date)]
        [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
        public DateTime EnrollmentDate { get; set; }

        public ICollection<Enrollment> Enrollments { get; set; }
    }
}
</pre>

```Column``` 特性的添加改变了 SchoolContext 的模型，导致与数据库不再匹配。

保存所做的更改并生成项目。 然后在项目文件夹中打开命令窗口并输入以下命令以创建另一个迁移：

``` console
dotnet ef migrations add ColumnFirstName
```

``` console
dotnet ef database update
```

在 SQL Server 对象资源管理器，通过双击 Student 表打开 Student 表设计视图。

![ssox-after-migration.png](./Images/ssox-after-migration.png)

应用前两个迁移之前，FirstName 和 LastName 列为 nvarchar (max) 类型。 而现在是 nvarchar(50) 类型，并且相应的列从 FirstMidName 更改为 FirstName。

> ### 备注
> 如果您完成下列部分中创建的所有实体类之前尝试编译，可能会收到编译器错误。

## 最终的 Student 实体更改 

![student-entity.png](./Images/student-entity.png)

在 Models/Student.cs 文件中，替换早期代码为如下代码。其中有修改的部分以高亮显示。

<pre>
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace ContosoUniversity.Models
{
    public class Student
    {
        public int ID { get; set; }
        <span style="background-color: #FF0;">[Required]</span>
        [StringLength(50)]
        <span style="background-color: #FF0;">[Display(Name = "Last Name")]</span>
        public string LastName { get; set; }
        <span style="background-color: #FF0;">[Required]</span>
        [StringLength(50, ErrorMessage = "First name cannot be longer than 50 characters.")]
        [Column("FirstName")]
        <span style="background-color: #FF0;">[Display(Name = "First Name")]</span>
        public string FirstMidName { get; set; }
        [DataType(DataType.Date)]
        [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
        <span style="background-color: #FF0;">[Display(Name = "Enrollment Date")]</span>
        public DateTime EnrollmentDate { get; set; }
        <span style="background-color: #FF0;">[Display(Name = "Full Name")]
        public string FullName
        {
            get
            {
                return LastName + ", " + FirstMidName;
            }
        }</span>

        public ICollection<Enrollment> Enrollments { get; set; }
    }
}
</pre>

#### ```Required``` 特性

```Required``` 特性使得名字属性成为必填字段。 无需为不可为空类型指定 ```Required``` 特性，如值类型（Datetime, int, double, float 等。）。 不可为空类型自动被视为必填字段。

您也可以移除 ```Require``` 特性，并代之以 ```StringLenght``` 特性中的一个最小长度参数。

``` cs
[Display(Name = "Last Name")]
[StringLength(50, MinimumLength=1)]
public string LastName { get; set; }
```

```Display``` 特性

```Display``` 特性指定文本框的标题应为 "First Name", "Last Name", "Full Name" 及 "Enrollment Date" 而不是使用属性名。（属性名单词中没有空格）。

#### FullName 计算属性

FullName 是一个计算属性， 由两个其他属性合并返回一个值。 因此它仅有一个 get 访问器，数据库中不会有 FullName 列生成。

## 创建 Instructor 实体

![instructor-entity.png](./Images/instructor-entity.png)

创建Models/Instructor.cs，并替换为以下代码：

```cs
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace ContosoUniversity.Models
{
    public class Instructor
    {
        public int ID { get; set; }

        [Required]
        [Display(Name = "Last Name")]
        [StringLength(50)]
        public string LastName { get; set; }

        [Required]
        [Column("FirstName")]
        [Display(Name = "First Name")]
        [StringLength(50)]
        public string FirstMidName { get; set; }

        [DataType(DataType.Date)]
        [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
        [Display(Name = "Hire Date")]
        public DateTime HireDate { get; set; }

        [Display(Name = "Full Name")]
        public string FullName
        {
            get { return LastName + ", " + FirstMidName; }
        }

        public ICollection<CourseAssignment> CourseAssignments { get; set; }
        public OfficeAssignment OfficeAssignment { get; set; }
    }
}
```

注意到在 Student 和 Instructor 实体中有多个相同的属性。 在本系列教程的 [Implementing Inheritance] 中，您将重构此代码以消除冗余。

多个特性可以放在一行上， 因此你可以像下面这样写 HireDate 的特性。

``` cs
[DataType(DataType.Date),Display(Name = "Hire Date"),DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]    
```

### CourseAssignments 和 OfficeAssignment 导航属性

```CourseAssignments``` 和 ```OfficeAssignment``` 属性是导航属性。

一个 Instructor （教师）可以教授任意数量的 Course （课程），因此 ```CourseAssignments``` 定义为集合。

``` cs
public ICollection<CourseAssignment> CourseAssignments { get; set; }
```

如果导航属性可以包含多个实体，其类型必须是一个可以添加、删除和更新的实体列表。 您可以指定为 ICollection<T> 、List<T> 或者 HashSet<T> 类型。 如果您指定为 ICollection<T> 类型， EF 默认创建为一个 HashSet<T> 集合类型。

关于为何这是 CouseAssignment 实体集合，将在下面的多对多关系一节中详细阐述。

Contoso 大学的业务规则指出，一个 Instructor （教师）只能有最多一个 office （办公室），因此 OfficeAssignment 属性只包含一个 OfficeAssignment 实体 （在未分配办公室的情况下也可能为空）。

``` cs 
public OfficeAssignment OfficeAssignment { get; set; }
```

## 创建 OfficeAssignment 实体

![officeassignment-entity.png](./Images/officeassignment-entity.png)

创建 Models/OfficeAssignment.cs 替换为以下代码：

```
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace ContosoUniversity.Models
{
    public class OfficeAssignment
    {
        [Key]
        public int InstructorID { get; set; }
        [StringLength(50)]
        [Display(Name = "Office Location")]
        public string Location { get; set; }

        public Instructor Instructor { get; set; }
    }
}
```

#### ```Key``` 特性

在 Instructor 和 OfficeAssignment 实体间有一个 “一 对 零或一” 的关系。 一个 OfficeAssignment 实体只与分配给的 Instructor 有关系，因此其主键也同时是连接到 Instructor 实体的外键。 但是， Entity Framework 无法自动识别 InstructorID 作为此实体的主键，因为其名称未遵循 ID 或 <类名>ID 的命名约定。 因此，我们使用 ```Key``` 特性来标识 InstructorID 作为主键。

``` cs
[Key]
public int InstructorID { get; set; }
```

如果实体已经有主键，但你想要用不同于 <类名>ID 或 ID 的属性名的时候，你也可以使用 ```Key``` 特性。

默认情况下， EF 将 ```Key``` 处理为 non-database-generated （非数据库生成），因为此列是用于识别关系。

#### ```Instructor``` 导航属性

```Instructor``` 实体有一个可为空的 OfficeAssignment 导航属性（因为教师可能没有分配一个办公室）， ```OfficeAssignment``` 实体有一个不可为空的 ```Instructor``` 导航属性（当没有教师的时候，一个 OfficeAssignment - 办公室分配不可能存在 -- InstructorID 不可为空）。 当 Instructor 实体具有相关的 OfficeAssignment 实体时，两个实体都有一个指向对方的导航属性。

您可以在 ```Instructor``` 导航属性前加一个 ```Required``` 特性以指定必须有相关的 Instructor ， 但实际无需这样做，因为 InstructorID 外键 （同时也是此表的主键）本就是不可为空的。

## 修改 Course 实体

![course-entity1.png](./Images/course-entity1.png)

在 Models/Course.cs 文件中，使用如下代码替换之前的代码。 更改位置采用高亮显示。

<pre>
using System.Collections.Generic;
<span style="background-color: #FF0;">using System.ComponentModel.DataAnnotations;</span>
using System.ComponentModel.DataAnnotations.Schema;

namespace ContosoUniversity.Models
{
    public class Course
    {
        [DatabaseGenerated(DatabaseGeneratedOption.None)]
        <span style="background-color: #FF0;">[Display(Name = "Number")]</span>
        public int CourseID { get; set; }

        <span style="background-color: #FF0;">[StringLength(50, MinimumLength = 3)]</span>
        public string Title { get; set; }

        <span style="background-color: #FF0;">[Range(0, 5)]</span>
        public int Credits { get; set; }

        <span style="background-color: #FF0;">public int DepartmentID { get; set; }</span>

        <span style="background-color: #FF0;">public Department Department { get; set; }</span>
        public ICollection<Enrollment> Enrollments { get; set; }
        <span style="background-color: #FF0;">public ICollection<CourseAssignment> CourseAssignments { get; set; }</span>
    }
}
</pre>

```Course``` 实体有一个 DepartmentID 外键属性和一个 Department 导航属性用于指向相关的 Department 实体。

当您已经拥有相关实体的导航属性时， Entity Framework 不强求你一定要添加一个外键属性到数据模型中。 EF 可以在需要的时候自动在数据库中创建外键及外键的影子属性。 但是，数据模型中具有外键可以使更新更简单、 更高效。例如，当您提取一个 Course 实体用于编辑时， 如果你没有声明加载的话，Department 实体是空的，当您要更新 Course 实体时，您将需要先读取 Department 实体。 如果外键属性 DepartmentID 包含在数据模型中，则您无需在更新前读取 Department 实体。

#### ```DatabaseGenerated``` 特性

在 CourseID 属性上标注的 ```DatabaseGenerated``` 特性与 ```None``` 参数指定主键值有用户而不是数据库生成。

```cs 
[DatabaseGenerated(DatabaseGeneratedOption.None)]
[Display(Name = "Number")]
public int CourseID { get; set; }
```

默认情况下，实体框架假定主键值都由数据库生成。 这是大部分情况下您希望的。但是，对于 Course 实体， 您将使用一个用户指定的课程号码，比如 1000 系列用于一个部门，2000 系列用于另外一个部门，依此类推。

```DatabaseGenerated``` 特性也可用于生成默认值， 比如在数据库中用于记录数据行创建或更新日期的列。有关详细信息，请参阅  [Generated Properties] 。

#### 外键和导航属性

Course 实体中的外键属性和导航属性反映了以下关系：

一个课程将分配到一个部门，如上面提到的一样，这儿有一个 DepartmentID 外键和一个 Department 导航属性。

``` cs
public int DepartmentID { get; set; }
public Department Department { get; set; }
```

一个课程可以有任意多的学生注册， 因此 Enrollments 导航属性是个集合：

``` cs
public ICollection<Enrollment> Enrollments { get; set; }
```

一个课程可能由多个教师执教，因此 CourseAssignments 导航属性是个集合（CourseAssignment 类型稍后解释）。

``` cs
public ICollection<CourseAssignment> CourseAssignments { get; set; }
```

## 创建 Department 实体

![department-entity.png](./Images/department-entity.png)

创建 Models/Department.cs 并替换为以下代码：

``` cs
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace ContosoUniversity.Models
{
    public class Department
    {
        public int DepartmentID { get; set; }

        [StringLength(50, MinimumLength = 3)]
        public string Name { get; set; }

        [DataType(DataType.Currency)]
        [Column(TypeName = "money")]
        public decimal Budget { get; set; }

        [DataType(DataType.Date)]
        [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
        [Display(Name = "Start Date")]
        public DateTime StartDate { get; set; }

        public int? InstructorID { get; set; }

        public Instructor Administrator { get; set; }
        public ICollection<Course> Courses { get; set; }
    }
}
```

#### ```Column``` 特性

早前您使用 Column 特性更改列名映射。在 Department 实体代码中， Column 特性用于更改 SQL 数据类型映射，以便此列在数据库中使用 SQL Server 的 money 类型。

``` cs
[Column(TypeName="money")]
public decimal Budget { get; set; }
```

列映射通常并无必要， 因为 Entity Framework 会基于您给属性定义的 CLR 类型，为您选择适合的 SQL Server 数据类型。 CLR decimal 类型映射到 SQL Server 的 deccimal 类型。 不过，本例中，您指定此列将用于保存货币金额，money 数据类型更为合适。

#### 外键和导航属性

外键和导航属性反映了以下关系：

一个部门可以有或者没有管理员，一个管理员总是一个教师。因此 InstructorID 属性用于指向 Instructor 实体的外键，```int``` 后面的 ```?``` 表示这个属性是可为空属性。 导航属性命名为 Administrator 但实际上包含的是 Instructor 实体。

``` cs
public int? InstructorID { get; set; }
public Instructor Administrator { get; set; }
```

一个部门可能有多个课程，因此有一个 Course 导航属性：
``` cs
public ICollection<Course> Courses { get; set; }
```

> ### 备注
> 按照约定，Entity Framework 对不为空外键和多对多关系实施级联删除。 这可能导致循环的级联删除规则，当您添加一个迁移时，会引发异常。 例如，如果您定义 Department.InstructorID 属性可为空， EF 会配置一个级联删除规则，用于删除部门， 而这并不是您所希望发生的。 如果您的业务规则要求 InstructorID 属性不可为空，那么您必须使用如下的 fluent API 语句来禁用关系上的级联删除规则。
> ``` cs
> modelBuilder.Entity<Department>()
>   .HasOne(d => d.Administrator)
>   .WithMany()
>   .OnDelete(DeleteBehavior.Restrict)
> ```

## 修改 Enrollment 实体

![enrollment-entity1.png](./Images/enrollment-entity1.png)

在 Models/Enrollment.cs 文件中，使用以下代码替换之前的代码：

<pre>
<span style="background-color: #FF0;">using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;</span>

namespace ContosoUniversity.Models
{
    public enum Grade
    {
        A, B, C, D, F
    }

    public class Enrollment
    {
        public int EnrollmentID { get; set; }
        public int CourseID { get; set; }
        public int StudentID { get; set; }
        <span style="background-color: #FF0;">[DisplayFormat(NullDisplayText = "No grade")]</span>
        public Grade? Grade { get; set; }

        public Course Course { get; set; }
        public Student Student { get; set; }
    }
}
</pre>

#### 外键和导航属性

外键属性和导航属性反映了以下关系：

An enrollment record is for a single course, so there's a CourseID foreign key property and a Course navigation property:

注册记录已经是单个的课程，因此CourseID外键属性和Course导航属性：