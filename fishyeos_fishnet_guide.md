# Complete Guide: FishyEOS + FishNet Integration

## Overview

FishNet is an original free versatile networking solution for Unity, built from the ground up, offering more features than any other free solution. FishyEOS is a transport library that provides Epic Online Services (EOS) implementation for Fish-Networking.

This guide will walk you through the complete setup and implementation of FishyEOS with FishNet for Unity multiplayer games using Epic Online Services as the backend.

## What You'll Need

### Prerequisites
- Unity 2020.1.11f1 or newer
- Epic Games Developer Account
- EOS Application configured in Epic Developer Portal

### Required Packages
1. **FishNet** - Core networking solution
2. **EOS Plugin for Unity** - Epic Online Services integration
3. **FishyEOS** - Transport layer connecting FishNet with EOS
4. **Unity Editor Coroutines** - Required dependency

## Part 1: Installing Dependencies

### Step 1: Install FishNet

#### Method 1: Unity Package Manager (Recommended)
1. Open Unity Package Manager: `Window → Package Manager`
2. Click the `+` button → `Add package from git URL`
3. Enter: `https://github.com/FirstGearGames/FishNet.git?path=Assets/FishNet`
4. Click Add

#### Method 2: Unity Asset Store
1. Visit the [FishNet Asset Store page](https://assetstore.unity.com/packages/tools/network/fishnet-networking-evolved-207815)
2. Download and import into your project

### Step 2: Install EOS Plugin for Unity

⚠️ **Important**: Do not download as ZIP. Use git clone to avoid DLL errors.

1. Open terminal/command prompt
2. Navigate to your project's root directory
3. Run: `git clone https://github.com/PlayEveryWare/eos_plugin_for_unity`
4. Copy `Assets/Plugins` from the cloned repository to your project's `Assets/Plugins` folder

### Step 3: Install Unity Editor Coroutines
1. Open Package Manager: `Window → Package Manager`
2. Search for "Editor Coroutines"
3. Install `com.unity.editorcoroutines`

### Step 4: Install FishyEOS

#### Method 1: Package Manager (Recommended)
1. Open Unity Package Manager
2. Click `+` → `Add package from git URL`
3. Enter: `https://github.com/ETdoFresh/FishyEOS.git?path=/FishNet/Plugins/FishyEOS`

#### Method 2: Manual Installation
1. Download the FishyEOS repository
2. Copy `FishNet/Plugins/FishyEOS` folder to your project's `Assets` or `Packages` folder

## Part 2: EOS Configuration

### Step 1: Configure EOS Plugin

1. Follow the [EOS Plugin configuration guide](https://github.com/PlayEveryWare/eos_plugin_for_unity#steps)
2. Set up your Epic Online Services project information in the Unity editor
3. This will create a configuration file at: `Assets/StreamingAssets/EOS/EpicOnlineServicesConfig.json`

### Step 2: Platform-Specific Setup

#### For Non-Windows Platforms (Android, iOS, macOS)
1. Go to `Edit → Project Settings → Player Settings → Other Settings`
2. Add `EOS_PREVIEW_PLATFORM` to Scripting Define Symbols
3. Click Apply

#### For Android Specifically
1. **Player Settings**:
   - Other Settings → Identification → Minimum API Level: 23 or higher
   - Publishing Settings → Uncheck "Custom Main Gradle Template"
   - If prompted to update gradleTemplate.properties, click Yes

2. **Configuration Files**:
   - Duplicate `Assets/StreamingAssets/EOS/EpicOnlineServicesConfig.json`
   - Rename the copy to `eos_android_config.json`

3. **Build Tools**: Ensure Android Build Tools 30.0.3 are installed
   ```cmd
   # Run in administrator command prompt
   cd \Program Files\Unity\Hub\Editor\2020.3.33f1\Editor\Data\PlaybackEngines\AndroidPlayer\SDK\tools\bin
   sdkmanager.bat --install build-tools;30.0.3
   ```

#### For macOS
1. Duplicate `Assets/StreamingAssets/EOS/EpicOnlineServicesConfig.json`
2. Rename the copy to `eos_macos_config.json`

## Part 3: Setting Up Your Project

### Step 1: Create Network Manager

1. Create a new GameObject in your scene
2. Name it "NetworkManager"
3. Add the following components:
   - `NetworkManager` (from FishNet)
   - `TransportManager` (added automatically)
   - `FishyEOS Transport` (from FishyEOS)

### Step 2: Configure Transport Manager

1. Select your NetworkManager GameObject
2. In the TransportManager component, set the Transport field to your FishyEOS Transport component

### Step 3: Configure FishyEOS Transport

In the FishyEOS Transport component, configure:
- **EOS Socket Name**: A unique identifier for your socket (e.g., "ExampleFishyEOS")
- **Server Product User ID**: The EOS Product User ID for the server (when acting as client)

## Part 4: Implementation

### Server Implementation

```csharp
using FishNet.Managing;
using UnityEngine;

public class GameServer : MonoBehaviour
{
    [SerializeField] private NetworkManager networkManager;
    [SerializeField] private string socketName = "ExampleFishyEOS";
    
    private void Start()
    {
        // Configure the socket name
        var fishyTransport = FindObjectOfType<FishyEOS.FishyEOSTransport>();
        if (fishyTransport != null)
        {
            fishyTransport.SetSocketName(socketName);
        }
        
        // Start server
        StartServer();
    }
    
    public void StartServer()
    {
        if (networkManager != null)
        {
            networkManager.ServerManager.StartConnection();
            Debug.Log("Server started successfully!");
        }
    }
    
    public void StopServer()
    {
        if (networkManager != null)
        {
            networkManager.ServerManager.StopConnection(true);
            Debug.Log("Server stopped.");
        }
    }
}
```

### Client Implementation

```csharp
using FishNet.Managing;
using UnityEngine;

public class GameClient : MonoBehaviour
{
    [SerializeField] private NetworkManager networkManager;
    [SerializeField] private string socketName = "ExampleFishyEOS";
    [SerializeField] private string serverProductUserId = "0002780586644887316944b9a41246b3";
    
    public void ConnectToServer()
    {
        // Configure connection settings
        var fishyTransport = FindObjectOfType<FishyEOS.FishyEOSTransport>();
        if (fishyTransport != null)
        {
            fishyTransport.SetSocketName(socketName);
            fishyTransport.SetServerProductUserId(serverProductUserId);
        }
        
        // Start client connection
        if (networkManager != null)
        {
            networkManager.ClientManager.StartConnection();
            Debug.Log("Attempting to connect to server...");
        }
    }
    
    public void Disconnect()
    {
        if (networkManager != null)
        {
            networkManager.ClientManager.StopConnection();
            Debug.Log("Disconnected from server.");
        }
    }
}
```

### Host Implementation (Server + Client)

```csharp
using FishNet.Managing;
using UnityEngine;

public class GameHost : MonoBehaviour
{
    [SerializeField] private NetworkManager networkManager;
    [SerializeField] private string socketName = "ExampleFishyEOS";
    
    public void StartHost()
    {
        // Configure the socket name
        var fishyTransport = FindObjectOfType<FishyEOS.FishyEOSTransport>();
        if (fishyTransport != null)
        {
            fishyTransport.SetSocketName(socketName);
        }
        
        // Start server first
        networkManager.ServerManager.StartConnection();
        
        // Then start client (creates a special "clientHost")
        networkManager.ClientManager.StartConnection();
        
        Debug.Log("Host started successfully!");
    }
    
    public void StopHost()
    {
        networkManager.ClientManager.StopConnection();
        networkManager.ServerManager.StopConnection(true);
        Debug.Log("Host stopped.");
    }
}
```

## Part 5: Creating Networked Objects

### Basic NetworkObject Example

```csharp
using FishNet.Object;
using UnityEngine;

public class Player : NetworkBehaviour
{
    [SerializeField] private float speed = 5f;
    
    private void Update()
    {
        // Only allow movement on client that owns this object
        if (!base.IsOwner) return;
        
        HandleMovement();
    }
    
    private void HandleMovement()
    {
        Vector3 movement = Vector3.zero;
        
        if (Input.GetKey(KeyCode.W)) movement += Vector3.forward;
        if (Input.GetKey(KeyCode.S)) movement += Vector3.back;
        if (Input.GetKey(KeyCode.A)) movement += Vector3.left;
        if (Input.GetKey(KeyCode.D)) movement += Vector3.right;
        
        if (movement != Vector3.zero)
        {
            transform.Translate(movement * speed * Time.deltaTime);
            
            // Send movement to server for validation
            ServerMovePlayer(transform.position);
        }
    }
    
    [ServerRpc]
    private void ServerMovePlayer(Vector3 position)
    {
        // Validate movement on server
        transform.position = position;
        
        // Update all clients
        ObserversUpdatePosition(position);
    }
    
    [ObserversRpc(ExcludeOwner = true)]
    private void ObserversUpdatePosition(Vector3 position)
    {
        transform.position = position;
    }
}
```

### NetworkTransform Usage

For simpler synchronization, use NetworkTransform:

```csharp
using FishNet.Object;
using FishNet.Component.Transforming;
using UnityEngine;

public class SimplePlayer : NetworkBehaviour
{
    [SerializeField] private float speed = 5f;
    private NetworkTransform networkTransform;
    
    private void Awake()
    {
        networkTransform = GetComponent<NetworkTransform>();
    }
    
    private void Update()
    {
        if (!base.IsOwner) return;
        
        Vector3 movement = Vector3.zero;
        if (Input.GetKey(KeyCode.W)) movement += Vector3.forward;
        if (Input.GetKey(KeyCode.S)) movement += Vector3.back;
        if (Input.GetKey(KeyCode.A)) movement += Vector3.left;
        if (Input.GetKey(KeyCode.D)) movement += Vector3.right;
        
        if (movement != Vector3.zero)
        {
            transform.Translate(movement * speed * Time.deltaTime);
        }
    }
}
```

## Part 6: EOS Authentication Integration

### EOS Login Manager

```csharp
using Epic.OnlineServices;
using Epic.OnlineServices.Auth;
using Epic.OnlineServices.Connect;
using UnityEngine;

public class EOSLoginManager : MonoBehaviour
{
    private void Start()
    {
        // Initialize EOS
        var eosManager = FishyEOS.EOS.GetManager();
        if (eosManager == null)
        {
            Debug.LogError("EOS Manager not found!");
            return;
        }
        
        StartDeviceIdLogin();
    }
    
    public void StartDeviceIdLogin()
    {
        var connectInterface = FishyEOS.EOS.GetPlatformInterface().GetConnectInterface();
        
        var loginOptions = new Epic.OnlineServices.Connect.LoginOptions()
        {
            Credentials = new Epic.OnlineServices.Connect.Credentials()
            {
                Type = ExternalCredentialType.DeviceidAccessToken,
                Token = null
            }
        };
        
        connectInterface.Login(ref loginOptions, null, OnConnectLoginComplete);
    }
    
    private void OnConnectLoginComplete(ref Epic.OnlineServices.Connect.LoginCallbackInfo data)
    {
        if (data.ResultCode == Result.Success)
        {
            Debug.Log($"EOS Connect Login successful! User ID: {data.LocalUserId}");
            // Now you can start your network connection
        }
        else if (data.ResultCode == Result.InvalidUser)
        {
            // Create user account
            CreateDeviceUser();
        }
        else
        {
            Debug.LogError($"EOS Connect Login failed: {data.ResultCode}");
        }
    }
    
    private void CreateDeviceUser()
    {
        var connectInterface = FishyEOS.EOS.GetPlatformInterface().GetConnectInterface();
        
        var createUserOptions = new Epic.OnlineServices.Connect.CreateUserOptions()
        {
            ContinuanceToken = null
        };
        
        connectInterface.CreateUser(ref createUserOptions, null, OnCreateUserComplete);
    }
    
    private void OnCreateUserComplete(ref Epic.OnlineServices.Connect.CreateUserCallbackInfo data)
    {
        if (data.ResultCode == Result.Success)
        {
            Debug.Log($"EOS User created successfully! User ID: {data.LocalUserId}");
            // Retry login
            StartDeviceIdLogin();
        }
        else
        {
            Debug.LogError($"EOS Create User failed: {data.ResultCode}");
        }
    }
}
```

## Part 7: Troubleshooting

### Common Issues and Solutions

1. **"EOSManager not found" Error**
   - Don't add EOSManager to your scene manually
   - Use `FishyEOS.EOS.GetManager()` and `FishyEOS.EOS.GetPlatformInterface()` instead

2. **DLL Errors on Import**
   - Make sure you cloned the EOS plugin repository instead of downloading as ZIP
   - Verify all DLL files are properly imported

3. **Connection Failures**
   - Ensure both server and client have the same Socket Name
   - Verify EOS authentication is successful before attempting connection
   - Check that Product User IDs are valid

4. **Self-Connection Issues**
   - EOS prevents connecting to yourself with the same ID
   - Use two different Epic Connect Product User IDs for testing
   - Consider using different sign-in providers (Developer and DeviceCode)

5. **Platform-Specific Issues**
   - Ensure `EOS_PREVIEW_PLATFORM` is defined for non-Windows platforms
   - Verify platform-specific configuration files are created
   - Check minimum API level requirements for Android

### Debug Tips

1. **Enable Verbose Logging**:
   ```csharp
   // Add to your initialization code
   Epic.OnlineServices.Logging.SetLogLevel(LogCategory.AllCategories, LogLevel.Verbose);
   Epic.OnlineServices.Logging.SetCallback((ref LogMessage message) => {
       Debug.Log($"EOS: {message.Message}");
   });
   ```

2. **Check Network Stats**:
   ```csharp
   // Monitor connection stats
   private void Update()
   {
       if (NetworkManager.ServerManager.Started)
       {
           Debug.Log($"Connected Clients: {NetworkManager.ServerManager.Clients.Count}");
       }
   }
   ```

3. **Validate EOS Connection**:
   ```csharp
   private bool IsEOSInitialized()
   {
       var platform = FishyEOS.EOS.GetPlatformInterface();
       return platform != null;
   }
   ```

## Part 8: Best Practices

### Performance Optimization

1. **Use NetworkTransform wisely**
   - Only sync transforms that need real-time updates
   - Consider using lower update rates for non-critical objects

2. **Batch RPCs when possible**
   - Group multiple actions into single RPC calls
   - Use [ObserversRpc] for events that affect all players

3. **Implement proper prediction**
   - Use client-side prediction for responsive gameplay
   - Validate actions on the server

### Security Considerations

1. **Always validate on server**
   - Never trust client data completely
   - Implement server-side validation for all critical actions

2. **Use proper RPC permissions**
   - Restrict sensitive RPCs to server only
   - Use appropriate RPC targets (Server, Observers, Target)

3. **Rate limiting**
   - Implement rate limiting for frequent actions
   - Prevent spam attacks through proper validation

### Code Organization

1. **Separate networking logic**
   - Keep network code separate from game logic where possible
   - Use interfaces to decouple systems

2. **Use FishNet's built-in features**
   - Leverage NetworkBehaviour lifecycle methods
   - Use SyncTypes for automatic data synchronization

3. **Handle disconnections gracefully**
   - Implement proper cleanup on disconnect
   - Save critical game state when appropriate

## Conclusion

This guide covers the complete setup and implementation of FishyEOS with FishNet. The combination provides a powerful, scalable networking solution with Epic Online Services integration, suitable for both small indie games and larger commercial projects.

Key benefits of this setup:
- **Free and Open Source**: No CCU limits or licensing fees
- **Epic Games Integration**: Leverage EOS features like friends, achievements, and cross-platform play
- **High Performance**: Optimized for bandwidth and CPU efficiency
- **Cross-Platform**: Support for multiple platforms including mobile

For additional help and community support, join the [FirstGearGames Discord](https://discord.gg/Ta9HgDh4Hj) where you can get real-time assistance from the FishNet community.

## Additional Resources

- [FishNet Documentation](https://fish-networking.gitbook.io/docs/)
- [FishNet GitHub Repository](https://github.com/FirstGearGames/FishNet)
- [FishyEOS GitHub Repository](https://github.com/ETdoFresh/FishyEOS)
- [EOS Plugin for Unity](https://github.com/PlayEveryWare/eos_plugin_for_unity)
- [Epic Online Services Documentation](https://dev.epicgames.com/docs/epic-online-services)
- [FirstGearGames Discord Community](https://discord.gg/Ta9HgDh4Hj)