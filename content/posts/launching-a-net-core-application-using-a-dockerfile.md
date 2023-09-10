---
title: "Launching a .NET Core Application Using a DockerFile"
date: 2023-09-03T00:00:00-07:00
draft: false
---

Docker is a wonderful tool. Using it we can run sophisticated applications in a breeze. In this article, we will show how to take advantage of docker to launch a ASP.NET core application.

## What is Docker

Docker is a containerization tool that allows applications to run separately, yet alongside, our operating system. They share resources with the host OS, but are lightweight enough to not deprive the system of resources.

Docker really shines through when an application is going through a major upgrade. When performing a major upgrade, the underlying infrastructure may also need to migrate to newer or different technology.

For example, when moving from .NET 5 to .NET 6, instead of having to install the .NET 6 runtime on every server, we can package the runtime in the DockerFile.

In a sense, Docker serves as documentation for our applications infrastructure. It dictates the minimum requirements needed to run an application on a single machine.

## Inner Workings of DockerFile For .NET

Now that we know the problems Docker solves, let's move on to a creating our first DockerFile. A DockerFile is a special file understood by Docker. It contains the underlying steps needed to run our application in a container.

To begin our example, let's assume we have a project with the following structure:

```
AppProject
|_________src
    |________Web.csproj
    |________Domain.csproj
|_________tests
    |________Web.UnitTests.csproj
    |________Domain.UnitTests.csproj
```

Let's suppose our ASP.Net application is contained within the `Web.csproj` and our application is targeting .NET 7.

To create our image for this project, we will need a base image to serve as our starting point. Docker [provides a list](https://hub.docker.com/_/microsoft-dotnet-aspnet) of ASP.NET core runtimes. Since we are targeting .NET 7, we will utilize the .NET 7 runtime:
`mcr.microsoft.com/dotnet/aspnet:7.0`. Therefore, our first step will look like the following:

```bash
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443
```

We are exposing ports 80 and 443 so we can later attach to these ports.

Our next involves building out the application. We can naively perform this step using `dotnet build`. However, taking this naive approach will not take advantage of Docker's caching capabilities.

To take advantage of Docker's caching abilities, it is better to divide the build step into sub-components where less frequently changing instructions come earlier in the file, and more frequently changing instructions come later. To make this more concrete, let's show exactly what we mean.

```bash
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY ["src/**/*.csproj", "."]
RUN dotnet restore
COPY . .
WORKDIR "/src/Web"
RUN dotnet build "Web.csproj" -c Release -o /app/build
```

In this instruction set, we first copy the project files, restore the dependencies, copy the remaining files and finally, build our startup project. Notice how we put the restore step before the build step. By dividing the steps, if our dependencies do not change, we can take advantage of the cached NuGet packages stored by docker. In the long run, this will save us precious time when starting up our application.

The very last steps involve publish and running our application. These steps can be formed separately and can utilize previously declared steps:

```bash
FROM build AS publish
RUN dotnet publish "Web.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Web.dll"]
```
We prefer to keep the steps separate for readability. However, they can also be combined into a single step.

## Putting it all together

Putting all of our previous steps together, we end up with the following file:

```bash
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY ["src/**/*.csproj", "."]
RUN dotnet restore
COPY . .
WORKDIR "/src/Web"
RUN dotnet build "Web.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "Web.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Web.dll"]
```

We can test that it functioning by navigating to our DockerFile, building the image and running the container based on the image:

```bash
docker build -t my-asp-net-app .
docker run -d -p 5135:80 -p 7081:443 --name my-container-for-asp-net-app my-asp-net-app
```

Here we decided to give our container a name of `my-container-for-asp-net-app`, and our image a name of `my-asp-net-app`. The ports `5135` and `7081` are taken directly from the `launchSettings.json`.

```json
...
"http": {
  "commandName": "Project",
  "launchBrowser": true,
  "environmentVariables": {
    "ASPNETCORE_ENVIRONMENT": "Development"
  },
  "dotnetRunMessages": true,
  "applicationUrl": "http://localhost:5135"
},
"https": {
  "commandName": "Project",
  "launchBrowser": true,
  "environmentVariables": {
    "ASPNETCORE_ENVIRONMENT": "Development"
  },
  "dotnetRunMessages": true,
  "applicationUrl": "https://localhost:7081;http://localhost:5135"
}
...
```

If we navigate to these localhost ports, we should see our application running. To stop our container, remove it and the corresponding image, we can run the following:

```bash
docker stop my-container-for-asp-net-app
docker rm my-container-for-asp-net-app
docker my-asp-net-app
```

There we have it, a new way of launch our application.

Thanks for reading!