# Stocks app - microservices & docker

University project - Service Oriented Architecture

## System diagram

```mermaid
graph TD
    User([User]) -. Interacts with .-> StockSystem(SOA-Stocks system)
    StockSystem -. Authenticates requests using .-> Auth0(Auth0 System)
		StockSystem -. Requests stock data from .-> Finnhub(Finnhub System)
```

## Container diagram (of SOA-Stocks system)

```mermaid
graph TD
		subgraph SOA-Stocks system
			WebApp(Web App - Single page application)
			API(Api Gateway)
			Auth(Authentication Microservice)
			Stocks(Stocks Microservice)
			Redis(Redis message broker)
		end

		Finnhub(Finnhub system)
		Auth0(Auth0 system)

		WebApp -. "[HTTP] Makes calls to" .-> API
		WebApp -. "Authenticates user" .-> Auth0
		API -. "[TCP] Authenticates user requests" .-> Auth
		API -. "[TCP] Requests stock data" .-> Stocks
		API -. "[WebSocket] Sends real-time stock prices" .-> WebApp
		Redis -. "[TCP] Sends stock prices" .-> API

		Auth -. "Verifies user tokens" ..-> Auth0

		Stocks -. "[HTTP] Makes calls to" ..-> Finnhub
		Stocks -. "[TCP] Publishes stock prices" .-> Redis
		Finnhub -. "[WebSocket] Sends real-time stock prices" ..-> Stocks
```
## Component diagram (of Web App - Single page application)

```mermaid
graph TD
		subgraph Web App - Single page application
			AppComponent --> NotAuthGuard --> LandingComponent 
			AppComponent --> AuthGuard --> DashboardComponent
			
			DashboardComponent --> StockChartComponent
			DashboardComponent --> StockSelectComponent
			DashboardComponent -. Uses .-> StockService

			StockChartComponent -. Uses .-> StockService

			StockSelectComponent -. Uses .-> StockService

		end

		AuthService("AuthService (Auth0 System)")
		AppComponent -. Uses ...-> AuthService
		LandingComponent -. Uses ...-> AuthService
		DashboardComponent -. Uses ...-> AuthService

		StockService -. "Makes calls to" .-> Api("Api Gateway")
```
## Component diagram (of Api Gateway)

```mermaid
graph TD
	subgraph Api Gateway
		AppModule --> SharedModule & StocksModule & RedisModule
		SharedModule ---> AuthGuard & WsAuthGuard
		RedisModule ---> RedisService
		StocksModule --> StocksController & StocksGateway
		StocksGateway -. uses .-> RedisService & WsAuthGuard
		StocksController -. uses .-> AuthGuard
	end

	AuthGuard & WsAuthGuard -. Makes calls to .-> AuthMicroservice(Authentication Microservice)
	RedisService -. Subscribes to messages from .-> Redis(Redis message broker)
	StocksController & StocksGateway -. Makes calls to ..-> StocksMicroservice(Stocks Microservice)
```

## Component diagram (of Auth Microservice)

```mermaid
graph TD
	subgraph Auth Microservice
		AppModule --> AuthModule
		AuthModule --> AuthController & AuthService
		AuthController -. uses .-> AuthService
	end

	AuthService -. verifies tokens using .-> Auth0(Auth0 System)
```

## Component diagram (of Stocks Microservice)
