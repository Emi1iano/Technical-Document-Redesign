# Technical-Document-Redesign
Technical Document Redesign

```Java
ClientCommandRegistrationCallback.EVENT.register(((dispatcher, registryAccess) -> {
            dispatcher.register(ClientCommandManager.literal("coordinates")
                    .executes(context -> printCoordinates()));
        }));
```
