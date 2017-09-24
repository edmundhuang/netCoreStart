# CRUD （创建，读取，更新和删除）

在上一章节中，您创建了一个使用 Entity Framework 和 SQL Server LocalDB 存储和显示数据的 MVC 应用程序。 在本章中，将复习并学习如何自定义 MVC 基架（脚手架）在控制器和视图中生成的 CRUD 代码。

> Repository （仓储）模式是一种常见的用于在控制器和数据访问层之间实现抽象的方法。为了保证本教程足够简单，并聚焦与学习如何使用 Entity Framework 技术， 我们将不使用仓储模式。 有关如何配合 EF 使用仓储模式， 请参见 [本教材最后一个章节](./advance.md)。

![student-details](./Images/student-details.png)

![student-create](./Images/student-create.png)

![student-edit-new](./Images/student-edit-new.png)

![student-delete](./Images/student-delete.png)

## 自定义 ```Detail``` 页

学生 Index 页面的脚手架代码没有使用到 ```Enrollment``` 属性，这个属性指向一个集合。  在 Detail 页面，您将在 HTML 表格中显示该集合的内容。

在 ```Controllers/StudentsController.cs``` 中， ```Detail``` 视图对应的 Action 方法使用 ```SingleOrDefaultAsync``` 方法来检索单个 ```Student``` 实体。 添加调用 Include, ThenInclude 和 AsNoTracking 方法的代码。 如下面的代码所示。
``` cs
public async Task<IActionResult> Details(int? id)
{
    if (id == null)
    {
        return NotFound();
    }

    var student = await _context.Students
        .Include(s => s.Enrollments)
            .ThenInclude(e => e.Course)
        .AsNoTracking()
        .SingleOrDefaultAsync(m => m.ID == id);

    if (student == null)
    {
        return NotFound();
    }

    return View(student);
}
```

```Include``` 和 ```ThenInclude``` 方法让 _context （数据库上下文） 同时加载```Student.Enrollments``` 导航属性，并对每个 ```Enrollment``` 加载对应的 ```Enrollment.Course``` 导航属性。 您将在 [六、读取相关数据](./chapters/relateData.md) 章节中了解有关这些方法的更多信息。

```AsNoTracking``` 方法在当前上下文生命周期中返回的实体不会更新的情况下有助于提高性能。 您将在本章节末尾详细了解 ```AsNoTracking```。

### MVC 路由

传递给 ```Details``` 方法的键值来自路由数据。 路由数据是模型绑定器在 URL 字符串中找到的数据。 例如，默认路由指定 controller，action 和 id ：
``` cs
app.UseMvc(routes =>
{
    routes.MapRoute(
        name: "default",
        template: "{controller=Home}/{action=Index}/{id?}");
});
```

在以下URL中，默认路由将 ```Instructor``` 作为控制器，将 ```Index``` 作为操作，将 ```1``` 作为 id ; 这些就是路由数据值。

```
http://localhost:1230/Instructor/Index/1?courseID=2021
```

URL 的最后一部分 ```"?courseID=2021"``` 是一个查询字符串值。 如果将其作为查询字符串值传递，模型绑定将会将 ID 值传递给 Details 方法的 id 参数：

```
http://localhost:1230/Instructor/Index?id=1&CourseID=2021
```

在 Index 页面中，Razor 引擎的 Tag 帮助辅助声明负责创建超链接。在以下 Razor 代码中，id 参数与默认路由匹配，因此 id 被添加到路由数据中。

``` html
<a asp-action="Edit" asp-route-id="@item.ID">Edit</a>
```

当上面代码中的 item.ID 值为 6，生成如下HTML：

``` html
<a href="/Students/Edit/6">Edit</a>
```

有关 Tag Helpers 的更多信息，请参阅 [Tag helpers in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro) 。

### 在 Detail 视图中添加 Enrollments 实体信息

打开 Views/Students/Details.cshtml 。 如下所示，可以看到，每个字段使用 ```DisplayNameFor``` 和 ```DisplayFor```  帮助方法来显示：

``` html
<dt>
    @Html.DisplayNameFor(model => model.LastName)
</dt>
<dd>
    @Html.DisplayFor(model => model.LastName)
</dd>
```

在最后一个字段之后，并在 ```</dl>``` 标记之前，添加以下代码以显示 Enrollment 列表：

``` html
<dt>
    @Html.DisplayNameFor(model => model.Enrollments)
</dt>
<dd>
    <table class="table">
        <tr>
            <th>Course Title</th>
            <th>Grade</th>
        </tr>
        @foreach (var item in Model.Enrollments)
        {
            <tr>
                <td>
                    @Html.DisplayFor(modelItem => item.Course.Title)
                </td>
                <td>
                    @Html.DisplayFor(modelItem => item.Grade)
                </td>
            </tr>
        }
    </table>
</dd>
```

粘贴代码后，如果代码缩进错误，请按 CTRL-K-D 进行更正。

这段代码循环读取 ```Enrollments``` 导航属性中的实体。 对于每个 Enrollment 实体，显示 ```Course Title``` （课程标题）和 ```Grade``` （成绩）。 课程标题从存储在 ```Enrollment``` 实体的 ```Course``` 导航属性中的 ```Course``` 实体检索。

运行应用程序，点击 Student 链接 ，然后单击任意学生的详细信息链接。 您会看到所选学生的课程和成绩列表：

![student-details](./Images/student-details.png)

### 更新 ```Create``` 页面

在 ```StudentsController.cs``` 中，修改 ```HttpPost Create``` 方法，添加一个try-catch 代码块，并从 ```Bind``` 特性中移除 ```ID``` 。

```cs
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Create(
    [Bind("EnrollmentDate,FirstMidName,LastName")] Student student)
{
    try
    {
        if (ModelState.IsValid)
        {
            _context.Add(student);
            await _context.SaveChangesAsync();
            return RedirectToAction(nameof(Index));
        }
    }
    catch (DbUpdateException /* ex */)
    {
        //Log the error (uncomment ex variable name and write a log.
        ModelState.AddModelError("", "Unable to save changes. " +
            "Try again, and if the problem persists " +
            "see your system administrator.");
    }
    return View(student);
}
```

这段代码将 ASP.NET MVC Model Binder （模型绑定器）创建的 ```Student``` 实体添加到 ```Students``` 实体集， 然后保存更改至数据库。模型绑定器是 ASP.NET MVC 功能，有助于更容易处理表单提交的数据;模型绑定器将发布的表单值转换为 CLR 类型，并将它们传递给参数中的action 方法，在这种情况下， 模型绑定器使用 Form 集合中的属性值为您实例化 ```Student``` 实体。

从 ```Bind``` 特性中删除了 ```ID``` ，因为 ```ID``` 是主键，SQL Server 会在添加数据行时自动设置。 来自用户的输入不设置 ```ID``` 值。

除了 ```Bind``` 特性外，```try-catch``` 代码块是您对脚手架代码进行的唯一更改。 如果在保存更改时捕获来自 ```DbUpdateException``` 的异常，则会显示通用错误消息。 ```DbUpdateException``` 异常有时由应用程序外部的东西引起，而不是编程错误，因此建议用户再次尝试。 虽然在此示例中未实现，但生产质量应用程序将记录异常。 有关更多信息，请参阅 [Monitoring and Telemetry (Building Real-World Cloud Apps with Azure)](https://docs.microsoft.com/aspnet/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry) 的 Log for insight 部分。

```ValidateAntiForgeryToken``` 特性用于防止跨域请求伪造（CSRF）攻击。 令牌由 ```FormTagHelper``` 自动注入视图，并在用户提交表单时包含该令牌。 该令牌由 ```ValidateAntiForgeryToken``` 特性验证。 有关 CSRF 的更多信息，请参阅  [Anti-Request Forgery](https://docs.microsoft.com/en-us/aspnet/core/security/anti-request-forgery) 。

### Over-Posting （过多提交） 安全提示
脚手架代码在 ```Create``` 方法中包含的 ```Bind``` 特性是在创建场景中防止出现 ```Over-Posting``` （过多提交）的一种方法。 例如，假设学生实体中包含您不希望此网页可以设置的 ```Secret``` 属性。
``` cs
public class Student
{
    public int ID { get; set; }
    public string LastName { get; set; }
    public string FirstMidName { get; set; }
    public DateTime EnrollmentDate { get; set; }
    public string Secret { get; set; }
}
```

即使您没有网页上的 ```Secret``` 字段，黑客也可以使用诸如 Fiddler 之类的工具，或者写一些 JavaScript 来发布一个 ```Secret``` 表单值。 如果没有 Binder 特性限制模型绑定器创建```Student``` 实例时可以使用的字段，模型绑定器将取得该 ```Secret``` 表单值，并用它来创建 ```Student``` 实体实例。 那么无论什么值，黑客为秘密表单字段指定的数据将在数据库中更新。 以下图像显示了 Fiddler 工具将 Secret 字段（值为 ```“OverPost”```）添加到发布的表单值。

![fiddler](./Images/fiddler.png)

然后，```OverPost``` 值将成功添加到插入行的 ```Secret``` 属性，尽管您从未打算让网页能够设置该属性。

在编辑场景，您可以通过先从数据库读取实体，然后调用 ```TryUpdateModel```，传递一个明确允许的属性列表，从而防止编辑场景中的 Over-Posting 攻击。 这也是本教程中使用的方法。

开发人员喜欢使用的另外一个防止 Over-Posting 攻击的方法是使用 ```ViewModel``` （视图模型），而不是直接绑定数据实体。 在 ```ViewModel``` 中仅包含需要更新的属性。 一旦 MVC 模型绑定完成，复制 ```ViewModel``` 属性到数据实体，可选择使用 ```AutoMapper``` 工具。 在实体实例上使用 ```_context.Entry``` 将其状态设置为 ```Unchanged``` ，然后在 ```ViewModel``` 中包含的每个实体属性上设置 ```Property("PropertyName")``` 的 ```IsModified``` 为 ```true``` 。 此方法可用于编辑和创建场景。

### 测试 ```Create``` 页面 

```Views/Students/Create.cshtml``` 的代码中，字段使用了 ```label```, ```Input```, ```span``` （用于展示验证消息） 等标签创建。

运行应用程序，点击 ```Student``` 链接，然后点击 ```Create```。

输入名称和日期。 尝试输入无效的日期，如果您的浏览器允许您这样做。 （某些浏览器强制您使用日期选择器。）然后单击 ```Create``` 以查看错误消息。

![date-error.png](./Images/date-error.png)

This is server-side validation that you get by default; in a later tutorial you'll see how to add attributes that will generate code for client-side validation also. The following highlighted code shows the model validation check in the Create method.
这是您默认获得的服务器端验证; 在后面的教程中，您将看到如何添加可以生成客户端验证代码的特性。 以下代码显示了 ```Create``` 方法中的模型验证检查。
``` cs
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Create(
    [Bind("EnrollmentDate,FirstMidName,LastName")] Student student)
{
    try
    {
        if (ModelState.IsValid)
        {
            _context.Add(student);
            await _context.SaveChangesAsync();
            return RedirectToAction(nameof(Index));
        }
    }
    catch (DbUpdateException /* ex */)
    {
        //Log the error (uncomment ex variable name and write a log.
        ModelState.AddModelError("", "Unable to save changes. " +
            "Try again, and if the problem persists " +
            "see your system administrator.");
    }
    return View(student);
}
```

将日期更改为有效值，然后单击 ```Create```，将会看到这个新学生出现在 ```Index``` 页面中。

### 更新 ```Edit``` 页面 

