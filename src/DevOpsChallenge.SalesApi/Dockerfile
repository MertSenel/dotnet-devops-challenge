#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:5.0-buster-slim AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:5.0-buster-slim AS build
WORKDIR /src
COPY ["src/DevOpsChallenge.SalesApi/DevOpsChallenge.SalesApi.csproj", "src/DevOpsChallenge.SalesApi/"]
COPY ["src/DevOpsChallenge.SalesApi.Business/DevOpsChallenge.SalesApi.Business.csproj", "src/DevOpsChallenge.SalesApi.Business/"]
COPY ["src/DevOpsChallenge.SalesApi.Database/DevOpsChallenge.SalesApi.Database.csproj", "src/DevOpsChallenge.SalesApi.Database/"]
COPY ["tests/DevOpsChallenge.SalesApi.Business.UnitTests/DevOpsChallenge.SalesApi.Business.UnitTests.csproj", "Tests/DevOpsChallenge.SalesApi.Business.UnitTests/"]
RUN dotnet restore "src/DevOpsChallenge.SalesApi/DevOpsChallenge.SalesApi.csproj"
COPY . .
WORKDIR "/src/src/DevOpsChallenge.SalesApi"
RUN dotnet build "DevOpsChallenge.SalesApi.csproj" -c Release -o /app/build

FROM build as testrunner
WORKDIR /tests/DevOpsChallenge.SalesApi.Business.UnitTests
RUN dotnet tool install dotnet-reportgenerator-globaltool --tool-path /dotnetglobaltools
LABEL unittestlayer=true
RUN dotnet test --logger "trx;LogFileName=DevOpsChallenge.SalesApi.Business.UnitTests.xml" /p:CollectCoverage=true /p:CoverletOutputFormat=cobertura /p:CoverletOutput=/out/testresults/coverage/ /p:Exclude="[xunit.*]*" --results-directory /out/testresults
RUN /dotnetglobaltools/reportgenerator "-reports:/out/testresults/coverage/coverage.cobertura.xml" "-targetdir:/out/testresults/coverage/reports" "-reporttypes:HTMLInline;HTMLChart"
 
FROM build AS test
LABEL test=true
RUN dotnet test -c Release --results-directory /testresults --logger "trx;LogFileName=test_results.trx" tests/DevOpsChallenge.SalesApi.Business.UnitTests/DevOpsChallenge.SalesApi.Business.UnitTests.csproj

FROM build AS publish
RUN dotnet publish "DevOpsChallenge.SalesApi.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "DevOpsChallenge.SalesApi.dll"]