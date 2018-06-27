# Movie Voting App (on .NET Core framework, and SQL Server 2017)
This project is build with `ASP.NET Core` framwork, conecting to a database in `SQL Server 2017 on Docker`. To setup a simple web app with `ASP.NET Core` framework, you can refer to [this tutorial](https://docs.microsoft.com/en-us/aspnet/core/tutorials/razor-pages-mac/?view=aspnetcore-2.1) on microsoft.com

To run this project on your local machine, follows the settings mentioned on the [Setup database connection on local dev environment](#setup-database-connection-on-local-dev-environment) section.

To deploy this project to Docker, follow steps `6, 7, 8, 9` on the [Setup project to deploy to Docker](#setup-project-to-deploy-to-docker) section.


## Setup SQL Server on Docker
1. Download SQL Server container
```shell
docker pull microsoft/mssql-server-linux:2017-latest
```
2. Install SQL Server container (replace all `<YourStrong!Passw0rd>` after this to your own password)
```shell
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=<YourStrong!Passw0rd>" -p 1433:1433 --name sql1 -d microsoft/mssql-server-linux:2017-latest
```
3. View docker containers to check if microsoft/mssql-server-linux is running
```shell
docker ps -a
```
4. Access your database from Database IDE with this configuration:
```shell
    Host: localhost
    Port: 1433
    User: SA
    Password: <YourStrong!Passw0rd>
```
5. On your Database IDE, create a database instance (the instance name will be `<YourDatabaseInstance>` below)


## Setup database connection on local dev environment
1. In `appsettings.json` file of your project, add a default connection string (replace all `<YourDatabaseInstance>` after this to your own instance name)
```shell
ConnectionStrings": {
  "DefaultConnection": "Data Source=127.0.0.1,1433;Initial Catalog=<YourDatabaseInstance>;User ID=sa;Password=<YourStrong!Passw0rd>"
}
```
2. In `Startup.cs` file of your project, add a SQLServer connection (or change from the original SQLite connection)
```shell
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
```


## Setup project to deploy to Docker
1. Right click project on Solution Explorer > Add > Add Docker Support
2. Change `Dockerfile` to (replace `ProjectName` to your own project name)
```shell
FROM microsoft/aspnetcore-build:lts
COPY . /app
WORKDIR /app
RUN dotnet restore ProjectName.csproj
RUN dotnet build ProjectName.csproj
EXPOSE 80/tcp
RUN chmod +x ./entrypoint.sh
CMD /bin/bash ./entrypoint.sh
```
3. Create `entrypoint.sh` file on your project directory (Same directory as `Dockerfile`), with the following content
```shell
#!/bin/bash

set -e
run_cmd="dotnet run --server.urls http://*:80"

until dotnet ef database update; do
>&2 echo "SQL Server is starting up"
sleep 1
done

>&2 echo "SQL Server is up - executing command"
exec $run_cmd
```
4. Update your `docker-compose.yml` file on your project directory with this content
```shell
version: "3"
services:
    web:
        build: .
        ports:
            - "8000:80"
        depends_on:
            - db
    db:
        image: "microsoft/mssql-server-linux"
        environment:
            SA_PASSWORD: "<YourStrong!Passw0rd>"
            ACCEPT_EULA: "Y"
```
5. In `Startup.cs` file of your project, change you SQLServer connection
```shell
// Database connection string.
// Make sure to update the Password value below from "<YourStrong!Passw0rd>" to your actual password.
var connection = @"Server=db;Database=<YourDatabaseInstance>;User=sa;Password=<YourStrong!Passw0rd>;";

// This line uses 'UseSqlServer' in the 'options' parameter
// with the connection string defined above.
services.AddDbContext<ApplicationDbContext>(
    options => options.UseSqlServer(connection));
```
6. On command prompt, navigate to project's directory
7. On command prompt, run `docker build -t image-name .` (replace `image-name` to your own preferred name)
8. On command prompt, run `docker run -d -p 8000:80 --name container-name image-name` (replace `container-name` and `image-name` to your own preferred name)
9. Visit http://localhost:8000


## References
1. Simple ASP.NET Core tutorial - https://docs.microsoft.com/en-us/aspnet/core/tutorials/razor-pages-mac/?view=aspnetcore-2.1
2. Run SQL Server 2017 container image on Docker - https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-linux-2017
3. Use SQL Server with Docker on local ASP.NET Core project - https://codebrains.io/how-to-use-sql-server-with-docker-and-asp-net-core-on-mac/ 
4. ASP.NET Core with SQL Server on Docker - https://docs.docker.com/compose/aspnet-mssql-compose/