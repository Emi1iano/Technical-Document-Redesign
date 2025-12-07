# Creating Commands
Creating commands can allow a mod developer to add functionality that can be used  
through a command. This tutorial will teach you how to register commands and the general  
command structure of Brigadier.  

Brigadier is a command parser and dispatcher written by Mojang for Minecraft. It is a tree-based  
command library where you build a tree of commands and arguments.

Brigadier is open-source: https://github.com/Mojang/brigadier

# Client side way of adding commands
The Fabric API has a `ClientCommandManager` in `net.fabricmc.fabric.api.client.command.v2`  
package that can be used to register client-side commands.  

Adding client side commands means that only the client is the one that will know the command  
the server will not have any indication of the commands you register to the client.  
So you are limited to actions only the client can do.

__Adding commands for the client should only be done on the client side code.__
#
### Types you are passed
`CommandDispatcher<FabricClientCommandSource> dispatcher` - Used to register, parse and execute commands.  
S is the type of command source the command dispatcher supports.  
`CommandRegistryAccess registryAccess` - Provides an abstraction to registries  
that may be passed to certain command argument methods
#
```Java
ClientCommandRegistrationCallback.EVENT.register(((dispatcher, registryAccess) -> {
    dispatcher.register(ClientCommandManager.literal("coordinates")
        .executes(commandContext -> printCoordinates()));
}));
```
The Fabric API provides us with `ClientCommandRegistrationCallback` which takes care of managing  
all added commands. Subscribing to `EVENT` allows you to register commands that then will be  
loaded when minecraft is ran. You are passed the dispatcher and registryAccess.

You register your command tree to the dispatcher. You actually start the command tree by  
calling ClientCommandManager. You give the name of the command using `literal` then if you follow  
it by the `executes` method, then the commandContext will be executed. Here we pass the commandContext  
the function we want it to execute.
#
```Java
static int printCoordinates() {
    ClientPlayerEntity player = MinecraftClient.getInstance().player;
    if(player == null) return -1;
    player.networkHandler.sendChatMessage(String.format("X: %2d Y: %2d Z: %2d", player.getBlockX(), player.getBlockY(), player.getBlockZ()));
    return 0;
}
```
Sample function to send out the coordinates of the player to everyone else on the server.  
Returning a 0 or positive value implies the command executed successfully  
negative values imply failure of execution.
#
### For branching commands

```Java
ClientCommandRegistrationCallback.EVENT.register(((dispatcher, registryAccess) -> {
    dispatcher.register(ClientCommandManager.literal("coordinates")
        .executes(commandContext -> printCoordinates())
        .then(ClientCommandManager.literal("detailed")
            .executes(commandContext -> printDetailedCoordinates())));
}));
```
Using the `then` on the same level as the `executes` will mean that the first literal  
`coordinate` will have a following branch `detailed` so that you can type */coordinates detailed*  
and the `executes` following the `then` will be executed.
```Java
static int printDetailedCoordinates() {
    ClientPlayerEntity player = MinecraftClient.getInstance().player;
    if(player == null) return -1;
    player.networkHandler.sendChatMessage(String.format("X: %.2f Y: %.2f Z: %.2f",player.getX(), player.getY(), player.getZ()));
    return 0;
}
```

# Server side way of adding commands
