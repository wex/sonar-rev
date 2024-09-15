# C# solution for getting openapi configuration of sonar

## Launch:
* You need .net 8 SDK installed
* In SteelSeries.ApiGetter/SteelSeries.ApiGetter.csproj change SonarAssembliesPath.
It should be path for Sonar *.dll files, like "Sonar.Logging.dll", "SoundStage.Api.dll", "SoundStage.AudioRepository.dll", "SoundStage.IoC", "SoundStage.Models", "SoundStage.Services"
* run in terminal dotnet run
* Now you can use swagger UI at http://localhost:{pot-from-terminal}/swagger
* And json file generated at /openapi
