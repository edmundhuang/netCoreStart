# 更新相关数据

Contoso 大学示例 Web 应用程序演示如何使用实体框架（EF）Core 2.0 和 Visual Studio 2017 创建 ASP.NET Core 2.0 MVC Web 应用程序。 如欲了解更多本教程相关信息，请参阅 [一、入门](./chapters/start.md)

在上一个教程中，您学习了显示相关数据。本教程中， 您将通过更新外键字段和导航属性来更新相关数据。
以下图片显示了您将用到的一些页面。

![course-edit.png](./Images/course-edit.png)

![instructor-edit-courses.png](./Images/instructor-edit-courses.png)

### 自定义课程的 "创建" 和 "编辑" 页面

创建新的课程实体时，必须关联到一个现有的部门。为了方便起见，脚手架代码包括控制器方法和创建和编辑视图，其中包含用于选择部门的下拉列表。 下拉列表设置 ```Course.DepartmentID``` 外键属性，而这正是 Entity Framework 需要以便加载 ```Department``` 导航属性及其对应的 ```Department``` 实体。您将使用脚手架代码，但稍稍更改以添加错误处理并对下拉列表进行排序。

在 ```CoursesController.cs``` 中，删除四个 ```Create``` 和 ```Edit``` 方法，并使用以下代码替换它们：

``` cs
public IActionResult Create()
{
    PopulateDepartmentsDropDownList();
    return View();
}

[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Create([Bind("CourseID,Credits,DepartmentID,Title")] Course course)
{
    if (ModelState.IsValid)
    {
        _context.Add(course);
        await _context.SaveChangesAsync();
        return RedirectToAction(nameof(Index));
    }
    PopulateDepartmentsDropDownList(course.DepartmentID);
    return View(course);
}


public async Task<IActionResult> Edit(int? id)
{
    if (id == null)
    {
        return NotFound();
    }

    var course = await _context.Courses
        .AsNoTracking()
        .SingleOrDefaultAsync(m => m.CourseID == id);
    if (course == null)
    {
        return NotFound();
    }
    PopulateDepartmentsDropDownList(course.DepartmentID);
    return View(course);
}

[HttpPost, ActionName("Edit")]
[ValidateAntiForgeryToken]
public async Task<IActionResult> EditPost(int? id)
{
    if (id == null)
    {
        return NotFound();
    }

    var courseToUpdate = await _context.Courses
        .SingleOrDefaultAsync(c => c.CourseID == id);

    if (await TryUpdateModelAsync<Course>(courseToUpdate,
        "",
        c => c.Credits, c => c.DepartmentID, c => c.Title))
    {
        try
        {
            await _context.SaveChangesAsync();
        }
        catch (DbUpdateException /* ex */)
        {
            //Log the error (uncomment ex variable name and write a log.)
            ModelState.AddModelError("", "Unable to save changes. " +
                "Try again, and if the problem persists, " +
                "see your system administrator.");
        }
        return RedirectToAction(nameof(Index));
    }
    PopulateDepartmentsDropDownList(courseToUpdate.DepartmentID);
    return View(courseToUpdate);
}
```

在 ```Edit``` (HttpPost) 方法后， 创建一个新方法用于加载下拉列表的部门信息。

```cs 
private void PopulateDepartmentsDropDownList(object selectedDepartment = null)
{
    var departmentsQuery = from d in _context.Departments
                           orderby d.Name
                           select d;
    ViewBag.DepartmentID = new SelectList(departmentsQuery.AsNoTracking(), "DepartmentID", "Name", selectedDepartment);
}
```


