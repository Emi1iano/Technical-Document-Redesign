# Creating Commands
Creating commands can allow a mod developer to add functionality that can be used  
through a command. This document will teach you how to register commands and the general  
command structure of Brigadier.  

Brigadier is a command parser and dispatcher written by Mojang for Minecraft. It is a tree-based  
command library where you build a tree of commands and arguments.

Brigadier is open-source: https://github.com/Mojang/brigadier

# Client side way of adding commands
The Fabric API has a `ClientCommandManager` in `net.fabricmc.fabric.api.client.command.v2`  
package that can be used to register client-side commands.  

Adding client side commands means that only the client is the one that will know the command  
the server will not have any indication of the commands you register to the client.  
So you are limited to actions only the client can do, also only you would have the command and  
not others that are playing with you.  

This is useful for example if you want to make toggles to turn on and off bits of code.

__Adding commands for the client should only be done on the client side code.__
#
### Types you are passed
`CommandDispatcher<FabricClientCommandSource> dispatcher` - Used to register, parse and execute commands.  
FabricClientCommandSource is the type of command source the command dispatcher supports.  

`CommandRegistryAccess registryAccess` - Provides an abstraction to registries  
that may be passed to certain command argument methods
#
Have this where your mod initializer is at, since you need to register commands when the mod  
is being loaded up.
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
`coordinates` will have a following branch `detailed` so that you can type `/coordinates detailed`  
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
The Fabric API has a `CommandRegistrationCallback` in `net.fabricmc.fabric.api.client.command.v2`  
package that can be used to register client-side commands.  

Registering commands for a server side mod is very similiar to client side commmands.  
When you register server side commands everyone playing with you will have to have the mod  
in order to first join the server and then execute the command.

__Adding commands for the server should only be done on the server side code.__

#
### Types you are passed
`CommandDispatcher<ServerCommandSource> dispatcher` - Used to register, parse and  
execute commands. ServerCommandSource is the type of command source the command dispatcher supports.  

`CommandRegistryAccess registryAccess` - Provides an abstraction to registries that may be  
passed to certain command argument methods  

`CommandManager.RegistrationEnvironment environment` - Identifies the type of server the  
commands are being registered on.  
#
```Java
CommandRegistrationCallback.EVENT.register((dispatcher, registryAccess, environment) -> {
	dispatcher.register(Commands.literal("test_command")
        .executes(commandContext -> {
		    commandContext.getSource().sendSuccess(() -> Component.literal("Called /test_command."), false);
		    return 1;
	}));
});
```
The Fabric API provides us with `CommandRegistrationCallback` which will handle all the commands  
you will be adding. You subscribe to the `EVENT` so that when minecraft is loaded your commands  
are registered. You are passed a dispatcher, registryAccess, and an environment.  

Register the command by calling `register` on the `dispatcher` then begin building the command tree.  
`Commands.literal` starts the tree with `/test_command`. `executes` passes you the `commandContext`  
which you use to create the action you want to be executed. `sendSuccess` will simply send the player  
that executed the command the message you want, if the command executed successfully. The false  
represents that only the person who executed the command will recieve the message, where as true  
would send it to all players. Important if you make a command that you want to let others know about  
when it is executed.
#
## Command Requirements
```Java
CommandRegistrationCallback.EVENT.register((dispatcher, registryAccess, environment) -> {
	dispatcher.register(Commands.literal("required_command")
			.requires(source -> source.hasPermission(1))
			.executes(ExampleModCommands::executeRequiredCommand));
});
```
```Java
private static int executeRequiredCommand(CommandContext<CommandSourceStack> CommandContext) {
	CommandContext.getSource().sendSuccess(() -> Component.literal("Called /required_command."), false);
	return 1;
}
```
Using `requires` allows us to make it so only players at certain permissions can execute the command.  
`requires` passes a source which we can use to determine if the thing executing the command  
is at the right permission. This also has the benefit of not showing players who don't have high  
enough permissions the command when they are typing in commands.  

With the `executes` since it gives a commandContext we pass it immediately to a function so that we  
are able to make the execute block look much cleaner.
#

### FAQ
- A command should return an integer. The `executes` method recieves a commandContext where  
the expected return type should be an integer.

- For the most stable results all commands should be registered when minecraft is first being  
loaded up. Although registering commands while minecraft is running is possible it is not  
recommended. You get the `CommandManager` from the server and your commands to the `CommandDispatcher`  
and then resend the command tree to every player via `CommandManager.sendCommandTree(ServerPlayerEntity)`.  
You can also unregister commands during runtime but is unstable. You do this by removing the commands  
from the `CommandDispatcher` then sending the new command tree to every player like in the previous point.