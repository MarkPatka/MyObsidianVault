```powershell
# Initialize the solution folder
dotnet new sln -o ProjectName

# Create the first microservice projects
dotnet new webapi -o ProjectName.MicroserviceName.Api
dotnet new classlib -o ProjectName.MicroserviceName.Contracts
dotnet new classlib ProjectName.MicroserviceName.Infrastruvture
dotnet new classlib ProjectName.MicroserviceName.Application
dotnet new classlib ProjectName.MicroserviceName.Domain

# Adding all created projects to the solution recursively
dotnet sln add (ls -r **\*.csproj)
dotnet build
	
# Adding references between the projects following the Clean Architecture
dotnet add .\ProjectName.MicroserviceName.Api\ reference .\ProjectName.MicroserviceName.Contracts\ .\ProjectName.MicroserviceName.Application\ .\ProjectName.MicroserviceName.Infrastructure\
	
dotnet add .\ProjectName.MicroserviceName.Infrastructure\ reference .\ProjectName.MicroserviceName.Application\ 

dotnet add .\ProjectName.MicroserviceName.Application\ reference .\ProjectName.MicroserviceName.Domain\
```
