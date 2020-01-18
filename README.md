# MemosForUE4Network (Reminders).

### This repo does not contain the program code of any project, only hints and examples of individual parts of the code.

## An important memo about network class relationship:
![](https://sun9-29.userapi.com/c205524/v205524269/3e2a9/X-QWilMuijc.jpg)

## C++ code to work with replication (template):
##### Replicate variables:
Firstly create variable and then mark it as "replicated" (Dont forget to set "SetReplicates = true" and "ReplicateMovement = true"):
```
UPROPERTY(Replicated)
AMyClass* Variable;
```

##### Then add it in the DOREPLIFETIME.
Here you actually define the rules of replicating your variables:
```
void AYourCharacter::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const {
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);
    // Here we list the variables we want to replicate + a condition if wanted
    DOREPLIFETIME(AYourCharacter, Variable);
}
```
Or with condition:
```
// Replicates the Variable only to the Owner of this Object/Class
    DOREPLIFETIME_CONDITION(AYourCharacter, Variable, COND_OwnerOnly);
```
## About RepNotify
#### This section is about functions calling during any variable replicated
Create variable and function:
```
UPROPERTY(ReplicatedUsing=OnRep_Variable)
float Variable;

UFUNCTION()
void OnRep_Variable();
```
And then implement this function as usual.
## Remote Procedural Calls (RPC)
##### Is is used to call something on another instance:
* Client to Server
* Server to Client
* Server to the group
##### There are a few requirements that need to be met for RPCs to be completely functional:
1. They must be called from Actors
2. The Actor must be replicated
3. If the RPC is being called from Server to be executed on a Client,
only the Client who actually owns that Actor will execute the function
4. If the RPC is being called from Client to be executed on the Server,
the Client must own the Actor that the RPC is being called on
5. Multicast RPCs are an exception:
    * If they are called from the Server, the Server will execute them locally,
    as well as execute them on all currently connected Clients
    * If they are called from Clients, they will only execute locally,
    and will not execute on the Server
    * A Multicast function will not replicate more than twice in a given
    Actor's network update period.
##### C++ code example
Add it in .h file:
```
UFUNCTION(Server, unreliable)
void Server_DoSomething();
```
And the following code in the .cpp file ([function_name]_implementation):
```
void AYourCharacter::Server_DoSomething_Implementation() {
// DO STUFF!;
}
```
##### Also you could add validation in the UPROPERTY and add "_Validation" in the function. It works like "Should we execute current function or not?":
The .h file:
```
UFUNCTION(Server, unreliable, WithValidation)
void Server_DoSomething();
```

And in the .cpp file:
```
bool AYourCharacter::Server_DoSomething_Validate() {
return true;
}
```
