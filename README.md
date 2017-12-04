# AzureFunctionsNodeLeaderboards-Cosmos

This project allows you to set up an Express Node.js app on an Azure Function that talks to a CosmosDB database via MongoDB API. The app is a RESTful API service that stores game leaderboards (scores) and exposes them via well-known HTTP methods. Azure Application Insights service is used to provide information and metrics regarding application performance. A Unity game engine client is also provided, with a minimal SDK to access the Azure Function.

## Deployment

Click the button below to deploy the project in your Azure subscription.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdgkanatsios%2FAzureFunctionsNodeLeaderboard%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>

The script will take some time to execute (due to resources creation and npm install execution), please be patient.
Be aware that AppService name, storage account name and database name must all be globally unique. If not, the script will fail to execute. 

## Architecture

The leaderboards API that's created is served by [Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/), a [serverless](https://azure.microsoft.com/en-us/overview/serverless-computing/) compute platform that enables execution of code without you having to worry about the underlying infrastructure. The code is written in [Node.js](https://nodejs.org/en/) whereas the database that backs our project is [Azure Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/introduction) using the [MongoDB protocol](https://docs.microsoft.com/en-us/azure/cosmos-db/mongodb-introduction). Moreover, [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview) service is used to track the application's performance.

![alt text](https://github.com/dgkanatsios/AzureFunctionsNodeScores-Cosmos/blob/master/media/functions.JPG?raw=true "Reference architecture")

On the software architecture side of things, [Mongoose](http://mongoosejs.com) is used to facilitate interactions with the database whereas the frontend API calls are served by [Express](https://expressjs.com/) web framework. Also, the [azure-functions-express](https://github.com/yvele/azure-function-express) package is used to facilitate the usage of Express within an Azure Function.

## Designing the leaderboard

Leaderboards within games can easily vary. In designing this library, we tried to satisfy the below requirements:

- store all scores (of course!)
- a score object is immutable (so you will not find any update methods here)
- we need to store all scores for each user
- we need to see the top scores of each user
- we need to see the top scores for today
- we need to see the top scores of all time, along with the users that got them

## Usage
After you deploy the script, you will have an Azure Resource Group will the following resources
- A CosmosDB database that uses the MongoDB API
- A Storage Account
- An App Service Name that hosts the Azure Function
- The Azure Function will pull the code from the GitHub repo you designate

Now you can call the available web service methods from your game. You can visit the Azure Portal to get the Azure Function URL, it will have the format https://**functionName**.azurewebsites.net

## Authentication
It is required that you set two headers on each request to the Function, their names are `x-ms-client-principal-id` and `x-ms-client-principal-name`. If values are missing, then you request will fail. The `x-ms-client-principal-id` should be unique for each user. That is, each time you use the same `x-ms-client-principal-id` for inserting a new score, this score will belong to the same user. The 'how' this values are filled is left to you as an implementation. App Service (the service on which Azure Functions is based on) supports various authentication methods, you can check them [here](https://docs.microsoft.com/en-us/azure/app-service/app-service-authentication-overview). It is worth mentioning that there is no applied authorization on the API calls. This means that all users can call all methods with any kind of parameters.

## Operations supported

Here you can see a short list of all the operations that are supported, check [here](README.technicalDetails.md) for full details.

| VERB | Method name | URL | Description | 
| --- | --- | --- | --- |
| POST | createScore | https://**functionURL**/api/scores | Creates a new score. Post body has the format { "value":Integer value of the score }. Returns the updated user details. |
| GET | listScoresForCurrentUser | https://**functionURL**/api/user/scores/:count | Gets the top 'count' scores for logged in user sorted by score value |
| GET | listTopScores | https://**functionURL**/api/scores/top/:count | Gets top scores achieved in the game by all users, in descending order. This can include more than one score per user |
| GET | listTopScorePerUser | https://**functionURL**/api/users/maxscore/:count | Gets all the max scores achieved in the game by all users, in descending order. Practically this includes the max score per user in descending order |
| GET | listTodayTopScores | https://**functionURL**/api/scores/today/top/:count | Gets the top 'count' scores for today|
| GET | listTopUsersTotalTimesPlayed | https://**functionURL**/api/users/toptotaltimesplayed/:count | Gets the top users for all time in regards to the times they have played (i.e. number of times they have posted a new score).|
| GET | listLatestScores | https://**functionURL**/api/scores/latest/:count | Gets the latest 'count' scores |
| GET | getUser | https://**functionURL**/api/users/:userId | Gets a specific user's details, including top score and latest scores | 
| GET | getScore | https://**functionURL**/api/scores/:scoreID | Gets a specific score |
| GET | checkDBhealth | https://**functionURL**/api/health | Gets the application's health |

## FAQ 
Check [here](README.faq.md) for answers to common questions you may have.

## Docker
You might notice that there is a Dockerfile inside the Azure Functions code. Check the [README.technicalDetails.md](README.technicalDetails.md) file for instructions on how to build and run the projet on a Docker container.