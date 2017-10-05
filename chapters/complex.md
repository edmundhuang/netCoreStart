# 创建复杂的数据模型

Contoso 大学示例 Web 应用程序演示如何使用实体框架（EF）Core 2.0 和 Visual Studio 2017 创建 ASP.NET Core 2.0 MVC Web 应用程序。 如欲了解更多本教程相关信息，请参阅 [一、入门](./chapters/start.md)

在前面的教程中，您使用由三个实体组成的简单数据模型。 在本章中， 您将添加多个实体和关系，并将通过指定格式化、验证以及数据库映射规则来自定义数据模型。

完成时，实体类构成的完整数据模型如下图所示：

![diagram.png](./Images/diagram.png)

## 使用特性自定义数据模型

在本节中，你将了解如何通过使用指定格式设置，验证和数据库的映射规则的属性来自定义数据模型。 然后在接下来的几节中，通过添加特性到类中，以及添加新的类来完成完整的 School 数据模型。

### ```DataType``` 特性（attribute）

对于学生注册日期，目前所有的网页同时显示日期和时间，虽然您关注的只是日期。通过使用数据注解特性，您可以只在一个地方进行代码更改，然后所有显示注册日期的格式将得到修正。 为了演示如何达成此目的，您将在 ```Student``` 类的 ```EnrollementDate``` 属性上添加一个 ```Attribute``` （特性）。

在 ```Models/Student.cs``` 中，添加 ```System.ComponentModel.DataAnnotations``` 命名空间的引用，并在 ```EnrollmentDate``` 属性上添加 ```DataType``` 和 ```DisplayFormat``` 特性，如下代码所示：

<pre>
using System;
using System.Collections.Generic;
<font color="yellow">using System.ComponentModel.DataAnnotations;</font>

namespace ContosoUniversity.Models
{
    public class Student
    {
        public int ID { get; set; }
        public string LastName { get; set; }
        public string FirstMidName { get; set; }

        <font color="yellow">[DataType(DataType.Date)]
        [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]</font>
        public DateTime EnrollmentDate { get; set; }

        public ICollection<Enrollment> Enrollments { get; set; }
    }
}
</pre>

```DataType``` 特性用于指定更为准确的数据库类型。在这个例子中，我们只想跟踪日期，而不是日期和时间。 DataType 枚举中包含许多数据类型，例如 Date, Time, PhoneNumber, Currency, EmailAddress, 等等。应用程序还可通过 DataType 特性自动提供类型特定的功能。例如，可以为 DataType.EmailAddress 创建 mailto: 链接，并且可以在支持 HTML5 的浏览器中为 DataType.Date 提供日期选择器。 DataType 特性生成 HTML5 浏览器可以理解的 HTML 5 data- (发音为 data dash) 。 DataType 特性不提供任何验证。

DataType.Date does not specify the format of the date that is displayed. By default, the data field is displayed according to the default formats based on the server's CultureInfo.

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

### ```StringLength``` 特性

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

```StringLength``` 特性不能阻止用户
The StringLength attribute won't prevent a user from entering white space for a name. You can use the RegularExpression attribute to apply restrictions to the input. For example, the following code requires the first character to be upper case and the remaining characters to be alphabetical:

StringLength属性不会阻止的用户的空白区域输入一个名称。 你可以使用RegularExpression要将限制应用到的输入属性。 例如，下面的代码要求的第一个字符是大写且其余的字符是按字母顺序排列：

