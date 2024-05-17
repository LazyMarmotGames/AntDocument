# Ant - tips and tricks (C++, blueprint)
### 1- Re path-find by velocity timeout event.
![repathfind](https://github.com/LazyMarmotGames/AntDocument/blob/main/Assets/repathfind.jpg)

When we move a large number of agents together in a small area with many obstacles, some agents may unintentionally go out of their way and may not be able to return to the previous correct path. In these cases, the agent gets stuck and cannot reach the desired point. If we have already defined a certain value for `VelocityTimeout` for this agent, we will see that the related event will be fired by `OnMovementMissingVelocity`. now we know that some agents have moving issue and need a new and correct path according to their current location.

We can handle it like this:
```cpp
void OnVelocityTimeout(float Delta, const TArray<FAntHandle>& Agents)
{
	auto *ant = GetWorld()->GetSubsystem<UAntSubsystem>();
	for (const auto &agent : Agents)
	{
		// UE defaults navigation system
		auto *navSys = UNavigationSystemV1::GetCurrent(GetWorld());

		// get agent destination location
		const FVector destLocation(FVector2D(ant->GetAgentMovement(agent).GetTargetLocation()), 0);

		// find a fresh new path to that location
		const auto *path = navSys->FindPathToLocationSynchronously(GetWorld(), FVector(ant->GetAgentData(agent).GetLocationLerped()), destLocation);

		// move the agent through new path
		if (path && path->IsValid())
			ant->MoveAgentTo(agent, path->PathPoints, 15, 0, ant->GetAgentData(agent).GetRadius() * 1.5f, 0, 8000);
	}
}
```

### 2- Auto generate obstacles from navigation mesh.
![navmeshobs](https://github.com/LazyMarmotGames/AntDocument/blob/main/Assets/auto-gen-obs.jpg)

Making obstacles with code or blueprint is a time consuming task. Ant in its new version can build obstacles directly from navigation mesh! in the process of adding this feature to Ant, we encountered a challenge that caused agents to behave as if they had an unwanted extra radius.
A quick and dirty solution could be that we define the radius of the agents as 0 when generating the navigation mesh. But this solution would cause problems in the long run. another better but more complicated solution was to define obstacles that would ignore the radius of the agents and only count their center. We did this and the output was great.

![navmeshobs](https://github.com/LazyMarmotGames/AntDocument/blob/main/Assets/extra-radius.jpg)

C++ sample code to generate this type of obstacle from navigation mesh:
```cpp
// generate segments list from navigation mesh
TArray<TPair<FVector, FVector>> outSegments;
UAntUtil::GetNavMeshSegments(GetWorld(), outSegments);

// add segments list as obstacles to Ant
GetWorld()->GetSubsystem<UAntSubsystem>()->AddObstacleList(outSegments, ObstacleFlag, true);
```

There is also a BP node `AddObstaclesFromNavmesh` that do the same through blueprint.
