---
title: "Object Mapping Made Simple With AutoMapper"
date: 2023-05-28T00:00:00-07:00
draft: false
---

Converting one object to another can be a very tedious task. Normally this conversion takes place when converting a view model into an entity or vice versa.

For example, if we had a `BookViewModel`, we cannot simply store this object as is into our database.

```c#
public class BookViewModel
{
    public string Title { get; set; }
    public int pageCount { get; set; }
    public string ISBN { get; set; }
}
```

Most tables require some sort of key and this key is not included in most view models. The objective then becomes to convert this view model into an entity such as `Book`.

```c#
public class Book
{
    public long Id { get; set; }
    public string Title { get; set; }
    public int pageCount { get; set; }
    public string ISBN { get; set; }
}
```

As you can see, both objects are nearly identical. The significant difference is that the entity `Book` has a key property `Id`. This is where tools such as object to object mappers come into play.

## Object-Object Mapper: AutoMapper

[AutoMapper](https://github.com/AutoMapper/AutoMapper) was created to simplify this object mapping conversion process. The typical workflow for using AutoMapper is to have profile classes inherit from `Profile`, implement `CreateMap`, and have AutoMapper scan your assemblies for these profiles. The mapping process is then as simple as passing our objects into `Map` for conversion. 

To illustrate this process, we can use `BookViewModel` and `Book` from above with the following profile class, `BookProfile`, mentioned below. 

### Mapping Example

Let's start by adding AutoMapper to our project: 
```
dotnet add package AutoMapper
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```

Now, we create our profile class, `BookProfile`; notice how we inherit from `Profile`, the class responsible for defining our mappings.

```c#
public class BookProfile : Profile
{
    public BookProfile()
    {
        CreateMap<BookViewModel, Book>();
    }
}
```

Lastly, we inject this service into our project. 

```c#
builder.Services.AddAutoMapper(AppDomain.CurrentDomain.GetAssemblies());
```
This service will scan our assemblies for profiles and register them accordingly.

This mappings can now be accessed in our controllers via `Map`:
```c#
public class BooksController : Controller
{
    // Define the mapper
    public readonly IMapper _mapper;

    // Initialize the dependencies with constructor initialization
    public UsersController(IMapper mapper)
    {   
        _mapper = mapper;
    }

    [HttpPost]
    public async Task<IActionResult> MapBook(BookViewModel bookVm)
    {
        // Utilize the mapping :)
        var mappedBook = _mapper.Map<Book>(bookVm);

        return Ok(mappedBook);
    }
}
```

Now that we have our mapped entity object, we can easily inject them into our database using an ORM.


## Refining the Profile Creation Process

The above process produces a lot of unnecessary code. Our `BookProfile` class is not doing very much and it would be great if we could remove such profile classes. Luckily there is a way. 

To remove such classes, we introduce two interfaces: `IMapTo` and `IMapFrom`. The point of these interfaces is to remove the need of profile classes as they will be inferred by reflection.

`IMapTo` and `IMapFrom` will work by creating the mappings.

```c#
public interface IMapTo<T>
{
    void Mapping(Profile profile) => profile.CreateMap(GetType(), typeof(T));
}
```

```c#
public interface IMapFrom<T>
{
    void Mapping(Profile profile) => profile.CreateMap(typeof(T), GetType());
}
```

For instance, applying `IMapTo` to `BookViewModel`, we now have a mapping that maps `BookViewModel` to `Book`; the mapping is also explicit stated in the class declaration.

```c#
public class BookViewModel : IMapTo<Book>
{
    public string Title { get; set; }
    public int pageCount { get; set; }
    public string ISBN { get; set; }
}
```

> For more complicated mappings, we can easily override `Mapping` in `IMapTo`.

However, there is still one issue. We have the mappings but they are incomplete as they do not inherit from `Profile`. To accomplish this, we can use reflection. 

The following class, `MappingProfile`, inherits from `Profile` and calls `CreateMap` on each class that implements `IMapTo` or `IMapFrom`. This class will thereby register the mappings to a profile and thus setup our mappings accordingly. 

```c#
public class MappingProfile : Profile
{
    public MappingProfile()
    {
        AppDomain.CurrentDomain.GetAssemblies().ToList().ForEach(assembly =>
            ApplyMappingsToAssembly(ApplyMappingsFromAssembly(assembly)));
    }

    private Assembly ApplyMappingsFromAssembly(Assembly assembly)
    {
        var mapFromType = typeof(IMapFrom<>);
        
        var mappingMethodName = nameof(IMapFrom<object>.Mapping);

        bool HasInterface(Type t) => t.IsGenericType && t.GetGenericTypeDefinition() == mapFromType;
        
        var types = assembly.GetExportedTypes().Where(t => t.GetInterfaces().Any(HasInterface)).ToList();
        
        var argumentTypes = new Type[] { typeof(Profile) };

        foreach (var type in types)
        {
            var instance = Activator.CreateInstance(type);
            
            var methodInfo = type.GetMethod(mappingMethodName);

            if (methodInfo != null)
            {
                methodInfo.Invoke(instance, new object[] { this });
            }
            else
            {
                var interfaces = type.GetInterfaces().Where(HasInterface).ToList();

                if (interfaces.Count > 0)
                {
                    foreach (var @interface in interfaces)
                    {
                        var interfaceMethodInfo = @interface.GetMethod(mappingMethodName, argumentTypes);

                        interfaceMethodInfo?.Invoke(instance, new object[] { this });
                    }
                }
            }
        }

        return assembly;
    }

    private Assembly ApplyMappingsToAssembly(Assembly assembly)
    {
        var mapToType = typeof(IMapTo<>);

        var mappingMethodName = nameof(IMapTo<object>.Mapping);

        bool HasInterface(Type t) => t.IsGenericType && t.GetGenericTypeDefinition() == mapToType;

        var types = assembly.GetExportedTypes().Where(t => t.GetInterfaces().Any(HasInterface)).ToList();

        var argumentTypes = new Type[] { typeof(Profile) };

        foreach (var type in types)
        {
            var instance = Activator.CreateInstance(type);

            var methodInfo = type.GetMethod(mappingMethodName);

            if (methodInfo != null)
            {
                methodInfo.Invoke(instance, new object[] { this });
            }
            else
            {
                var interfaces = type.GetInterfaces().Where(HasInterface).ToList();

                if (interfaces.Count > 0)
                {
                    foreach (var @interface in interfaces)
                    {
                        var interfaceMethodInfo = @interface.GetMethod(mappingMethodName, argumentTypes);

                        interfaceMethodInfo?.Invoke(instance, new object[] { this });
                    }
                }
            }
        }

        return assembly;
    }
}
```

With these interfaces and classes in place, we no longer need profile classes, including `BookProfile`. In a sense, the profile classes have moved into the view model / entity classes, which arguably is a more suitable location.

As mentioned above, this approach has the added benefit of explicitly stating which view models map to which entities and vice versa. This is because implementing the interface clearly states which objects are convertible. 

With the use of profile classes this is not always clear. With profile classes, we must first verify if a class has an accompanying profile class. In a big project, searching for such profile classes can prove to be a real pain. However, if we see that a class implements `IMapTo` or `IMapFrom`, we can be certain that mappings exist for this class.

## Summary

Mapping objects should not be a tedious task. There are many well known libraries available for simplifying this task. In this post, we covered AutoMapper and how to use it in ASP.NET applications. We also introduced interfaces and reflection to simplify the code necessary to make AutoMapper work.

If you find yourself constantly casting between similar objects, a tool like [AutoMapper](https://github.com/AutoMapper/AutoMapper) may be exactly what you are looking for.

Thanks for reading!