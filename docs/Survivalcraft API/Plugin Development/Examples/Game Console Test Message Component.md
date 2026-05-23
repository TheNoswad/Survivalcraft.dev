# Create a component that sends messages to the Game's console

In this section you can see a simple example of a component that you can add to the player to send messages to the Survivalcraft
console (which you can find in the settings > Compatibility > View Game Log section)


# Step 1 Create a template
Copy the following code and paste it into your XML file:
```Xml
<Mod>
	<EntityTemplate Name="Player" Guid="4be6c1c5-d65d-4537-8a8b-a391969e6dc2">
		<MemberComponentTemplate Name="ExampleComponent" Guid="d0dc11c0-d2b8-4c1c-b9e9-2e4ef62081ff">
			<Parameter Name="Class" Guid="6382924c-918e-4845-8af1-fff3a7b14dfc" Value="Game.KeyBoardExample" Type="string"/>
		</MemberComponentTemplate>
	</EntityTemplate>
</Mod>
```
This is one way to register the C# class that will be shown later. Save the XML file in your scmod folder with any name you like, for example, ```mod.xml```, and then rename the ```.xml``` extension to ```.xdb.```

# Step 2 Create a C# class
Within the code you will see comments about the functions that each call performs; copy this code and paste it into the class you created.
```CSharp
using Engine;
using Engine.Input;
using GameEntitySystem;
using TemplatesDatabase;

namespace Game
{
    internal class KeyBoardExample : Component, IUpdateable
    {
        public UpdateOrder UpdateOrder => UpdateOrder.Default;

        // The Load method is called once when loading a world or every time the entity appears, in this case the player.
        public override void Load(ValuesDictionary valuesDictionary, IdToEntityMap idToEntityMap)
        {
            base.Load(valuesDictionary, idToEntityMap);
            Log.Information("Hello World!");
        }

        // The update method is called on every frame all the time
        public void Update(float dt)
        {
            // You can choose the indicated key to send the message to the game's console.
            if (Keyboard.IsKeyDown(Key.Q)) Log.Information("Hello World!");
            /*
             * If you hold down the assigned key, this message will appear repeatedly in the console.
             * You can use IsKeyDownOnce to make the message appear only once, even if you hold down the key:
             * Keyboard.IsKeyDownOnce(Key.Q)
             */
        }
    }
}
```
Finally, compile the DLL file and move it to your scmod folder, compress the folder into a zip file, and rename the extension from ```.zip``` to ```.scmod```.
Move the scmod file to your mods folder, open the SC API, start any world, and press the key you assigned in the code. For example, you would press the Q key if that's the default.
