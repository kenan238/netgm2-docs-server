# NetGM2: IPlugin Interface
Every plugin must inherit from this interface, events are optional methods that can be overriden.
```c#
public interface IPlugin
  {
    // Run anytime NetGM2 needs to get the Plugin-Info, run more than once
    public PluginInfo GetInfo();

    // When the assembly is loaded
    public void Loaded();
    // When the server finishes initializing
    public void Initialize();
    // Never called, as unloading plugins is not supported yet, they stay loaded until the server shuts down
    public void Shutdown();

#region Events
/* Player events */
    public void PlayerJoin(NetGM2PluginAPI.Player player)
    { }
    public void PlayerLeave(NetGM2PluginAPI.Player player)
    { }

/* SyncObject events */
    public void SyncObjectCreated(NetGM2PluginAPI.SyncObject @object)
    { }
    public void SyncObjectDestroyed(NetGM2PluginAPI.SyncObject @object)
    { }
    public void SyncObjectSignal(NetGM2PluginAPI.SyncObject @object, NetGM2PluginAPI.Player by)
    { }

/* Spawn basic event */
    public void BasicObjectCreated(NetGM2PluginAPI.Player player, string objectAsset)
    { }

/* Stream events */
    public void StreamCreated(NetGM2PluginAPI.Stream stream)
    { }
    public void StreamDestroyed(NetGM2PluginAPI.Stream stream)
    { }

/* Sound event */
    public void SoundPlayed(Dictionary<string, object> audio_play_sound_ext_options)
    { }

/* GameState event */
    public void GameStateUpdate()
    { }

/* Player props update event */
    public void PlayerPropsUpdate(NetGM2PluginAPI.Player player)
    { }

/* P2P event */
    public void P2PSend(NetGM2PluginAPI.Player from, NetGM2PluginAPI.Player? to, Dictionary<string, object> data)
    { }

/* RPC event */
    public void RpcCalled(NetGM2PluginAPI.Player by, string rpcName)
    { }

/* Chat event */
    public void ChatMessageSent(int senderId, string message)
    { }
#endregion
  }
```

# The PluginInfo struct
```c#
public struct PluginInfo
{ 
  public string Name { get; set; }
  public string Description { get; set; }
  public string Author { get; set; }
  public string? Repository { get; set; } // Optional github repository, leave as null if not present
}
```

# NetGMS2PluginAPI
```c#
  public static class NetGM2PluginAPI
  {
    // This is only meant to be used if you REALLY have to implement a custom packet type.
    public static void AddPacketHandler(Packet.PacketType type, Action<Packet> handler);
    public static void ClearHandlers(Packet.PacketType type);
    public static ServerConfig Config;

    public static class Logger
    {
      public static void Log(Terminal.Logger.Levels level, string message);
    }

    public static class Rpc
    {
      public static void Add(string name, RpcManager.RpcAction action);
      public static void Remove(string name);

      public static void Call(string name, object[] args, object caller);
    }

    public static class Chat
    {
      public static void Send(string msg, int playerId = -1);
    }

    public static class Gamestate
    {
      /**
        * Make sure to call BeginChanges() first
        * Then, modify everything you need to in `data`
        * and then Commit() your changes to every client
        */

      public static Dictionary<string, object> data;

      public static void BeginChanges();

      public static void Commit();
    }

    public sealed record SyncObject
    {
      public int Id { get; private set; }
      public int Room { get; private set; }
      public string Object { get; private set; }
      public Dictionary<string, dynamic> Variables { get; private set; }
      public Player Owner;

      // You may not call the constructor, call Create(...) instead.
      private SyncObject() 
      { }

      private static SyncObject Create(string obj, int initialRoom, Dictionary<string, dynamic> initialVariables);

      public static SyncObject? ById(int id);
      public void Destroy();
    }

    public sealed record Stream
    {
      public int OwnerId { get; set; }
      public int StreamId { get; set; }
    }

    public class Player
    {
      public string Name { get; private set; }
      public int Id { get; private set; }
      public int AccountId { get; private set; }

      public List<SyncObject> SyncedObjects;
      public List<Stream> Streams;
      public AccountServer.Model.Account.PowerTypes Power { get; private set; }

      // Is true if the powertype is equal to MODERATOR or above
      public bool IsModerator;

      public static List<Player> Players;
      public static uint Count;

      public static Player? ById(int id);
      // Annotation is either a number (player ID) or the player name
      public static Player? ByAnnotation(string annotation);
      // This is only meant to be used if you REALLY have to implement a custom packet type.
      public void Send(dynamic data) => NetGms2Base.Player.FromId(Id)!.SendJson(data);
    
      public void Kick(string reason = "Kicked by plugin");
    }
  }
  ```

# AccountServer.Model.Account.PowerTypes
```c#
public enum PowerTypes
{
  BANNED,
  NORMAL,
  MODERATOR,
  ADMIN,
  OWNER,
  KENAN // ;)
}
```