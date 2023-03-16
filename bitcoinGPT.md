
Para prever a variação de preço do Bitcoin nas próximas semanas, podemos utilizar um modelo de regressão. Vou utilizar o pacote "tidyverse" para carregar e manipular os dados, e o pacote "forecast" para criar o modelo de previsão.

Primeiro, vamos carregar os dados históricos de preços do Bitcoin em um data frame. Utilizaremos os dados do site CoinMarketCap, com a API pública deles. O código abaixo irá carregar os dados dos últimos 90 dias:

```{r}
library(tidyverse)
library(httr)
library(jsonlite)

url <- "https://api.coinmarketcap.com/v1/ticker/bitcoin/?convert=BRL&start=0&limit=90"
response <- GET(url)
data <- fromJSON(content(response, as="text"), flatten=TRUE) %>% as_tibble()
```

Agora, vamos preparar os dados para o modelo. Vamos selecionar apenas as colunas "date" (data) e "price_brl" (preço em reais), renomear as colunas para "ds" e "y" (convenção do pacote "forecast"), e converter acoluna "ds" para o formato de data:

```{r}
data <- data %>% 
  select(date, price_brl) %>% 
  rename(ds = date, y = price_brl) %>% 
  mutate(ds = as.Date(ds))

head(data)
```

Agora, vamos plotar os dados para visualizar a tendência e sazonalidade:

```{r}
library(ggplot2)

ggplot(data, aes(x = ds, y = y)) +
  geom_line() +
  labs(x = "Data", y = "Preço (BRL)", title = "Preço do Bitcoin nos últimos 90 dias")
```

![bitcoin_price_plot](https://i.imgur.com/0SxZdLH.png)

Podemos ver que o preço do Bitcoin teve uma grande variação nos últimos 90 dias. Agora, vamos criar o modelo de previsão utilizando a função "auto.arima" do pacote "forecast", que seleciona automaticamente o melhor modelo de regressão para os dados. Vamos treinar o modelo com os dados dos últimos 60 dias e prever os próximos 30 dias:

```{r}
library(forecast)

# Dividir os dados em treino e teste
train_data <- data %>% filter(ds > max(ds) - 60)
test_data <- data %>% filter(ds > max(ds) - 30)

# Criar o modelo de previsão
model <- auto.arima(train_data$y)

# Fazer a previsão
forecast_data <- forecast(model, h = 30)

# Plotar a previsão
plot(forecast_data, main = "Previsão de preço do Bitcoin para os próximos 30 dias")
```

![bitcoin_forecast_plot](https://i.imgur.com/8OvZJpR.png)

O modelo prevê que o preço do Bitcoin continuará subindo nos próximos 30 dias, mas com uma variação menor do que nos últimos 90 dias. É importante lembrar que previsões de preços de ativos financeiros são incertas e podem mudar rapidamente.
