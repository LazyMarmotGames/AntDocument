# Ant - tips and tricks (C++, blueprint)
### 1- Re path-find by velocity timeout event.
![repathfind](https://github.com/LazyMarmotGames/AntDocument/blob/main/Assets/repathfind.jpg)

When you move a large number of agents together and in a small environment with many obstacles, some agents may unintentionally go out of their way and may not be able to return to the previous correct path. In these cases, the agent gets stuck and cannot reach the desired point. If we have already defined a certain value for `VelocityTimeout` for this agent, we will see that the related event will be fired by `OnMovementMissingVelocity`. now we know that some agents have moving issue and need a new and correct path according to their current location.
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
