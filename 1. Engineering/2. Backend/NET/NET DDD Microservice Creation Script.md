-o 1. Initialize the solution folder
	```dotnet new sln -o ProjectName```

2. Create the first microservice projects:
	 2.1. Adding the WebApi project:
	 ```dotnet new webapi -o ProjectName.MicroserviceName.Api```
	 
	 2.2. Adding the Contracts project (Optional):
	 ```dotnet new classlib -o ProjectName.MicroserviceName.Contracts```
	 
	 2.3. Adding the Infrastructure Layer: 
	 ```dotnet new classlib ProjectName.MicroserviceName.Infrastruvture```
	 
	 2.4. Adding the Application Layer:
	 ```dotnet new classlib ProjectName.MicroserviceName.Application```
	 
	 2.5. Adding the Domain Layer:
	```dotnet new classlib ProjectName.MicroserviceName.Domain```  

3. Adding all created projects to the solution recursively:
	- Windows (cmd): 
	  ```for /r %i in (*csproj) do dotnet sln add "%i"```
	- PowerShell:
	  ```Get-ChildItem -Recurse -Filter '*.csproj' | ForEach-Object { dotnet sln add $_ }```
	- Linux:
	  ```dotnet sln add $(find . -name "*.csproj")$```
	
4. Build the solution to check if everything works fine:
	```dotnet build```
	
5. Adding references between the projects according the Clean Architecture Principles:
	5.1.  ```dotnet add .\ProjectName.MicroserviceName.Api\ reference .\ProjectName.MicroserviceName.Contracts\ .\ProjectName.MicroserviceName.Application\ .\ProjectName.MicroserviceName.Infrastructure\ ``` 
	
	5.2. ```dotnet add .\ProjectName.MicroserviceName.Infrastructure\ reference .\ProjectName.MicroserviceName.Application\``` 
	
	5.3. ```dotnet add .\ProjectName.MicroserviceName.Application\ reference .\ProjectName.MicroserviceName.Domain\``` 
	
	![[Onion Architecture Diagram.svg]]
	