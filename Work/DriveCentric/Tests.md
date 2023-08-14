---
title: Tests
lastmod: 2023-05-30T14:06:27-05:00
---
# Tests
View readme [here](https://github.com/drivevelocity/LeadCrumb/tree/develop/DC.Atlas/DC.Database.Mock).
Should update star.bak periodically

Startup command:
```bash
docker compose -f tests.docker-compose.yml up -d
```

## Debugging Functional Test
Within the `CreateTestServer` function of `TestBase`, logging can be enabled using the following:
```csharp
	// more test server creation above
    .UseStartup<Startup>()
    .ConfigureLogging((_, loggingBuilder) =>
    {
        loggingBuilder.AddConsole();
    });
return new TestServer(hostBuilder)
{
    PreserveExecutionContext = true,
};
```
