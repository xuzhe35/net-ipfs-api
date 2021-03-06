# net-ipfs-api v0.4.10

> ![](https://ipfs.io/ipfs/QmQJ68PFMDdAsgCZvA1UVzzn18asVcf7HVvCDgpjiSCAse)

> The .NET portable class library (PCL) for the InterPlanetary File System (IPFS) API.

This library should be easy to use if you're familiar with the IPFS CLI. Top level commands (eg. `ipfs cat`, `ipfs add`) are methods in `IpfsClient`. Subcommands are methods in the IPFS client's properties named after the subcommands.

ie. `ipfs swarm peers` becomes `ipfs.Swarm.Peers()` and `ipfs add <file>` becomes `ipfs.Add(file)`. Easy!

The only exception is if a valid 'top level' command has subcommands, since C# can't have methods and properties with the same name. ie. `ipfs config` and `ipfs config edit` become `ipfs.ConfigCommand()` and `ipfs.Config.Edit()`.

CLI arguments are required method parameters and CLI options are optional method parameters.

## NuGet
This library is available on [NuGet](https://www.nuget.org/packages/net-ipfs-api).

You can install it with the NuGet package manager:

    PM> Install-Package net-ipfs-api
    

## Examples

### Adding & reading a file:

    using (var ipfs = new IpfsClient())
    {
      //Name of the file to add
      string fileName = "test.txt";

      //Wrap our stream in an IpfsStream, so it has a file name.
      IpfsStream inputStream = new IpfsStream(fileName, File.OpenRead(fileName));

      MerkleNode node = await ipfs.Add(inputStream);

      Stream outputStream = await ipfs.Cat(node.Hash);

      using(StreamReader sr = new StreamReader(outputStream))
      {
        string contents = sr.ReadToEnd();

        Console.WriteLine(contents); //Contents of test.txt are printed here!
      }
    }


### Disconnect from all peers:
    using (var ipfs = new IpfsClient())
    {
        var peers = await ipfs.Swarm.Peers();
        await ipfs.Swarm.Disconnect(peers);
    }
    
### Cancellation support

    try
    {
        var cts = new CancellationTokenSource(1000);
        
        using (var httpClient = new HttpClient() { Timeout = Timeout.InfiniteTimeSpan })
        using (var ipfsClient = new IpfsClient(new Uri("http://127.0.0.1:5001"), httpClient))
        {
            //The CancellationTokenSource will cancel the task after 1 second
            //Simulate a 2 second delay for this example to ensure the task gets cancelled
            await Task.Delay(2000);
            var result = await ipfsClient.Cat("QmPb8TjUbiRm1M9pvz8QW14PfBjkQyRitnJCZ95oRTRYBt", cts.Token);
        }
    }
    catch (TaskCanceledException)
    {
        Console.WriteLine("The request was canceled");                
    }

## License
MIT license, see LICENSE for details

## Thanks
Thanks to [@slothbag](https://github.com/slothbag) & [@timothyparez](https://github.com/timothyparez)
