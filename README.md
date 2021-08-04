# TorrentClient for .NET Core

This is an implementation of a torrent peer using the BitTorrent Protocol 1.0 written in C#.

## Using the TorrentClient

 1. Download the source code.
 2. Include project TorrentClient.
 3. Use the following code:

```csharp
public void Run()
{
    int listeningPort = 4000; // must be available and between 1025 and 65535
    string baseDirectory = @".\Test"; // base torrent data directory
    string torrentInfoFilePath = @".\TorrentFile.torrent";
    TorrentClient torrentClient;

    if (TorrentInfo.TryLoad(torrentInfoFilePath, out TorrentInfo torrent))
    {
        torrentClient = new TorrentClient(listeningPort, baseDirectory);
        torrentClient.DownloadSpeedLimit = 100 * 1024; // 100 KB/s
        torrentClient.UploadSpeedLimit = 200 * 1024; // 200 KB/s
        torrentClient.TorrentHashing += this.TorrentClient_TorrentHashing;
        torrentClient.TorrentLeeching += this.TorrentClient_TorrentLeeching;
        torrentClient.TorrentSeeding += this.TorrentClient_TorrentSeeding;
        torrentClient.TorrentStarted += this.TorrentClient_TorrentStarted;
        torrentClient.TorrentStopped += this.TorrentClient_TorrentStopped;
        torrentClient.Start(); // start torrent client
        torrentClient.Start(torrent); // start torrent file
    }
}

private void TorrentClient_TorrentHashing(object sender, TorrentHashingEventArgs e)
{
    // occurs when a torrent's pieces are being hashed
    Console.WriteLine($"hashing {e.TorrentInfo.InfoHash}");
}

private void TorrentClient_TorrentLeeching(object sender, TorrentLeechingEventArgs e)
{
    // occurs when a torrent is being leeched (pieces are being downloaded)
    Console.WriteLine($"leeching {e.TorrentInfo.InfoHash}");
}

private void TorrentClient_TorrentSeeding(object sender, TorrentSeedingEventArgs e)
{
    // occurs when a torrent is being seeded (pieces are being uploaded)
    Console.WriteLine($"seeding {e.TorrentInfo.InfoHash}");
}

private void TorrentClient_TorrentStarted(object sender, TorrentStartedEventArgs e)
{
    // occurs when a torrent is started
    Console.WriteLine($"started {e.TorrentInfo.InfoHash}");
}

 private void TorrentClient_TorrentStopped(object sender, TorrentStoppedEventArgs e)
{
    // occurs when a torrent is stopped
    Console.WriteLine($"stopped {e.TorrentInfo.InfoHash}");
}
```

## Getting execution details

```csharp
TorrentProgressInfo info = torrentClient.GetProgressInfo(torrent.InfoHash);

Console.WriteLine($"duration: {info.Duration.ToString(@"hh\\:mm\\:ss", CultureInfo.InvariantCulture)}");
Console.WriteLine($"completed: {(int)Math.Round(info.CompletedPercentage * 100)}%");
Console.WriteLine($"download speed: {info.DownloadSpeed.ToBytes()}/s");
Console.WriteLine($"upload speed: {info.UploadSpeed.ToBytes()}/s");
Console.WriteLine($"downloaded: {info.Downloaded.ToBytes()}");
Console.WriteLine($"uploaded: {info.Uploaded.ToBytes()}");
Console.WriteLine($"seeders: {info.SeederCount}");
Console.WriteLine($"leechers: {info.LeecherCount}");
```

## Protocol execution

Example of the protocol being executed:

```csharp
creating torrent client
listening port: 4000
base directory: .\Test\
setting download speed limit to 102400B/s
starting torrent client
starting torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0
creating persistence manager for .\Test\
creating torrent manager for torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0
local peer id -AB1100-99D4EC68117C1DBAEA960B3E
starting torrent manager for torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0
torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0 hashing
verifying file .\Test\Torrent\a
starting tracking udp://192.168.0.10:4009/announce for torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0
torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0 started
udp://192.168.0.10:4009/announce -> UdpTrackerAnnounceMessage
udp://192.168.0.10:4009/announce <- UdpTrackerAnnounceResponseMessage
adding seeding peer 192.168.0.10:4000 to torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0
192.168.0.10:4000 -> HandshakeMessage: PeerID = -AB1100-99D4EC68117C1DBAEA960B3E, InfoHash = C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0, FastPeer = False, ExtendedMessaging = False
adding leeching peer 192.168.0.10:54977 to torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0
192.168.0.10:54977 -> HandshakeMessage: PeerID = -AB1100-99D4EC68117C1DBAEA960B3E, InfoHash = C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0, FastPeer = False, ExtendedMessaging = False
192.168.0.10:4000 <- HandshakeMessage: PeerID = -AB1100-99D4EC68117C1DBAEA960B3E, InfoHash = C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0, FastPeer = False, ExtendedMessaging = False
fatal communication error occurred for peer 192.168.0.10:4000 on torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0: Invalid handshake message.
disposing peer 192.168.0.10:4000
disposing peer communicator for 192.168.0.10:4000
received no data from 192.168.0.10:54977
adding leeching peer 192.168.0.10:54982 to torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0
192.168.0.10:54982 -> HandshakeMessage: PeerID = -AB1100-99D4EC68117C1DBAEA960B3E, InfoHash = C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0, FastPeer = False, ExtendedMessaging = False
192.168.0.10:54982 <- BitfieldMessage: Bitfield = 11111111111111111111111111111111111111111111101111101111111111111111111111111111111111101111111111111011111111111111111111111111111111111111111111111111111111111111111111110111111111111111111110111111111111111111111111101111110111011111111111111111111111111111111111101111111111111111111111111111111111111111101111111111111111111111111111111111011111111111111111111111011111111111011111111111111111111111011111111111111111111111111111111111110111111110111111111111111101111111111111111111111111111111111111011111111111111111111110111111011111111111111111111111111111111111111111111011111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111011101111111111111100001111
....
192.168.0.10:54982 <- HaveMessage: Index = 306
192.168.0.10:54982 <- HaveMessage: Index = 42
192.168.0.10:54977 -> InterestedMessage
192.168.0.10:54982 -> InterestedMessage
192.168.0.10:54982 <- UnChokeMessage
192.168.0.10:54977 -> InterestedMessage
192.168.0.10:54982 -> RequestMessage: PieceIndex = 1, BlockOffset = 0, BlockLength = 16384
fatal communication error occurred for peer 192.168.0.10:54977 on torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0: Unable to write data to the transport connection: An established connection was aborted by the software in your host machine.
disposing peer 192.168.0.10:54977
disposing peer communicator for 192.168.0.10:54977
192.168.0.10:54982 -> RequestMessage: PieceIndex = 1, BlockOffset = 16384, BlockLength = 16384
192.168.0.10:54982 -> RequestMessage: PieceIndex = 1, BlockOffset = 32768, BlockLength = 16384
...
192.168.0.10:54982 <- PieceMessage: PieceIndex = 1, BlockOffset = 1015808, BlockData = byte[1048576]
192.168.0.10:54982 <- PieceMessage: PieceIndex = 1, BlockOffset = 1032192, BlockData = byte[1048576]
piece 1 completed for torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0
torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0 leeching
192.168.0.10:54982 -> HaveMessage: Index = 1
192.168.0.10:54982 -> RequestMessage: PieceIndex = 2, BlockOffset = 0, BlockLength = 16384
...
192.168.0.10:54982 -> RequestMessage: PieceIndex = 2, BlockOffset = 1032192, BlockLength = 16384
192.168.0.10:54982 <- PieceMessage: PieceIndex = 2, BlockOffset = 0, BlockData = byte[1048576]
...
192.168.0.10:54982 <- PieceMessage: PieceIndex = 2, BlockOffset = 999424, BlockData = byte[1048576]
piece 2 completed for torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0
torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0 leeching
192.168.0.10:54982 -> HaveMessage: Index = 2
192.168.0.10:54982 -> RequestMessage: PieceIndex = 3, BlockOffset = 0, BlockLength = 16384
...
192.168.0.10:54982 -> RequestMessage: PieceIndex = 3, BlockOffset = 1032192, BlockLength = 16384
192.168.0.10:54982 <- PieceMessage: PieceIndex = 3, BlockOffset = 0, BlockData = byte[1048576]
...
192.168.0.10:54982 <- PieceMessage: PieceIndex = 3, BlockOffset = 999424, BlockData = byte[1048576]
piece 3 completed for torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0
torrent C4BB6FEC9FE8E0C240302785AE1A3AD2964CEEB0 leeching
192.168.0.10:54982 -> HaveMessage: Index = 3
...
```


https://qawithexperts.com/questions/478/how-we-can-open-torrent-files-in-c

OR you can also try these Nuget package

https://www.nuget.org/packages/Leak.Core/

https://www.nuget.org/packages/MonoTorrent/

