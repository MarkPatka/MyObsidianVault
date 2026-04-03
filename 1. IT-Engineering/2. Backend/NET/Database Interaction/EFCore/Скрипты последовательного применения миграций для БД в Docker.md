## Создать миграцию средствами Entity Framework Core
```powershell
dotnet ef migrations add InitialCreate -p EventService.Infrastructure -s
EventService.Api
```

## Создать SQL скрипт миграции
```powershell
 dotnet ef migrations script `
>>   -p EventService.Infrastructure `
>>   -s EventService.Api `
>>   -o migration.sql                    
```

## Выполнить SQL-скрипт, подключившись к docker-контейнеру с БД 
```bash
docker exec -i mockevent_postgres psql -U postgres -d mockeventdb < 
"D:/Repositories/EventService/migration.sql"
```

## Подключиться к БД в контейнере
```powershell
docker exec -it mockevent_postgres psql -U postgres -d mockeventdb
```

## Выполнить тестовый скрипт
```powershell
mockeventdb=# SELECT * FROM "EventStatuses";
```
