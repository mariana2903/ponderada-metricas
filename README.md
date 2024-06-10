# ponderada-metricas

## Tecnologias utilizadas

- ASP.NET Core 8.0
- Prometheus
- Grafana

## Coleta de métricas

**Criação projeto**

A criação do projeto e o início da instalação das dependências com os comandos a seguir:

```
dotnet new web -o WebMetric
cd WebMetric
dotnet add package OpenTelemetry.Exporter.Prometheus.AspNetCore --prerelease
dotnet add package OpenTelemetry.Extensions.Hosting
```

**Primeiro código e teste**

Depois de criação do projeto é necessário trocar o conteúdo do program.cs e inserir o conteúdo a seguir 

```
using OpenTelemetry.Metrics;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddOpenTelemetry()
    .WithMetrics(builder =>
    {
        builder.AddPrometheusExporter();

        builder.AddMeter("Microsoft.AspNetCore.Hosting",
                         "Microsoft.AspNetCore.Server.Kestrel");
        builder.AddView("http.server.request.duration",
            new ExplicitBucketHistogramConfiguration
            {
                Boundaries = new double[] { 0, 0.005, 0.01, 0.025, 0.05,
                       0.075, 0.1, 0.25, 0.5, 0.75, 1, 2.5, 5, 7.5, 10 }
            });
    });
var app = builder.Build();

app.MapPrometheusScrapingEndpoint();

app.MapGet("/", () => "Hello OpenTelemetry! ticks:"
                     + DateTime.Now.Ticks.ToString()[^3..]);

app.Run();
```

Para exibir as métricas é é necessário instalar o dotnet-counters com o comando a seguir pra instalar as dependências necessárias

```
dotnet tool update -g dotnet-counters
```

E depois rodar esse comando a seguir que funciona para aparecer as métricas e gera o que está na imagem em seguida

```
dotnet-counters monitor -n WebMetric --counters Microsoft.AspNetCore.Hosting
```

![image](https://github.com/mariana2903/ponderada-metricas/assets/99264876/d6e04bf4-f6a8-40b3-b7e5-8f7c13ce6778)
