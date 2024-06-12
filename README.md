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

**Segundo código e teste**

Para iniciar é necessário substituir o conteúdo do program.cs pelo código a seguir: 

```
using Microsoft.AspNetCore.Http.Features;

var builder = WebApplication.CreateBuilder();
var app = builder.Build();

app.Use(async (context, next) =>
{
    var tagsFeature = context.Features.Get<IHttpMetricsTagsFeature>();
    if (tagsFeature != null)
    {
        var source = context.Request.Query["utm_medium"].ToString() switch
        {
            "" => "none",
            "social" => "social",
            "email" => "email",
            "organic" => "organic",
            _ => "other"
        };
        tagsFeature.Tags.Add(new KeyValuePair<string, object?>("mkt_medium", source));
    }

    await next.Invoke();
});

app.MapGet("/", () => "Hello World!");

app.Run();
```

Este código acima configura uma aplicação web ASP.NET Core que adiciona uma middleware para processar todas as requisições HTTP. O middleware verifica se a requisição contém um parâmetro de consulta utm_medium e, com base no valor desse parâmetro, define uma tag mkt_medium com valores como "social", "email", "organic", ou "other". Se o parâmetro estiver vazio, a tag é definida como "none". Essas tags são adicionadas ao recurso de métricas HTTP (IHttpMetricsTagsFeature) para serem usadas em monitoramento ou logging. Por fim, a aplicação mapeia uma rota GET para a raiz ("/") que retorna a mensagem "Hello World!"

E o resultado dele é a imagem a seguir

![image](https://github.com/mariana2903/ponderada-metricas/assets/99264876/9685f439-e198-47fb-9c96-2782f0b4d38e)

**Terceiro código e teste**

Depois trocamos novamente o conteúdo do program.cs, pelo bloco de código a seguir: 

```
public class ContosoMetrics
{
    private readonly Counter<int> _productSoldCounter;

    public ContosoMetrics(IMeterFactory meterFactory)
    {
        var meter = meterFactory.Create("Contoso.Web");
        _productSoldCounter = meter.CreateCounter<int>("contoso.product.sold");
    }

    public void ProductSold(string productName, int quantity)
    {
        _productSoldCounter.Add(quantity,
            new KeyValuePair<string, object?>("contoso.product.name", productName));
    }
}
```

E a seguir esse código em execução

![image](https://github.com/mariana2903/ponderada-metricas/assets/99264876/f0972853-ace9-4abe-bf49-7d14e739fbcf)

## Coleta das métricas com prometheus e grafana

Depois das execuções anteriores, é necessário voltar para a primeira versão do program.cs e confirmar as demais partes do projeto para que seja possível coletar as métricas com prometheus e criar os gráficos com grafana

Para iniciar instalei o prometheus que será exposto na porta local 9090, e grafana que ao rodar será exposto na porta 3000

E foi necessário configurar o prometheus.yaml para coletar as métricas da porta em que a aplicação está rodando "localhost:7027" com o endpoint /metrics, como na imagem a seguir 

![image](https://github.com/mariana2903/ponderada-metricas/assets/99264876/3eadb961-7ee7-407a-a104-936e5786d4f9)

Depois isso é coletado pelo prometeheus, como na image a seguir: 

![image](https://github.com/mariana2903/ponderada-metricas/assets/99264876/cf6a1ecd-16ea-4dc7-bdd5-64b06ac27d67)

Por fim os gráficos gerados com grafana a partir das métricas que estão sendo coletadas com prometheus e gerando gráficos como na imagem a seguir: 

![image](https://github.com/mariana2903/ponderada-metricas/assets/99264876/5fc977e8-c5d5-4a6f-9590-68332e07bb95)


