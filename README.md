

``csharp
static void Main(string[] args)
{
    CompletionPort completionPort = CompletionPort.Create();

    AutoResetEvent listenerEvent = new AutoResetEvent(false);
    AutoResetEvent clientEvent = new AutoResetEvent(false);
    AutoResetEvent serverEvent = new AutoResetEvent(false);

    AsyncSocket listener = AsyncSocket.Create(AddressFamily.InterNetwork, 
        SocketType.Stream, ProtocolType.Tcp);
    completionPort.AssociateSocket(listener, listenerEvent);

    AsyncSocket server = AsyncSocket.Create(AddressFamily.InterNetwork, 
        SocketType.Stream, ProtocolType.Tcp);
    completionPort.AssociateSocket(server, serverEvent);

    AsyncSocket client = AsyncSocket.Create(AddressFamily.InterNetwork, 
        SocketType.Stream, ProtocolType.Tcp);
    completionPort.AssociateSocket(client, clientEvent);

    Task.Factory.StartNew(() =>
    {
        CompletionStatus completionStatus;

        while (true)
        {
            var result = completionPort.GetQueuedCompletionStatus(-1, out completionStatus);

            if (result)
            {
                Console.WriteLine("{0} {1} {2}", completionStatus.SocketError, 
                    completionStatus.OperationType, completionStatus.BytesTransferred);

                if (completionStatus.State != null)
                {
                    AutoResetEvent resetEvent = (AutoResetEvent)completionStatus.State;
                    resetEvent.Set();
                }
            }
        }
    });

    listener.Bind(IPAddress.Any, 5555);
    listener.Listen(1);

    client.Connect("localhost", 5555);

    listener.Accept(server);


    listenerEvent.WaitOne();
    clientEvent.WaitOne();

    byte[] sendBuffer = new byte[1] { 2 };
    byte[] recvBuffer = new byte[1];

    client.Send(sendBuffer);
    server.Receive(recvBuffer);

    clientEvent.WaitOne();
    serverEvent.WaitOne();

    server.Dispose();
    client.Dispose();
}
```

## Compiling and Testing

To compile from source:

* Download [Visual Studio 2017](https://docs.microsoft.com/en-us/visualstudio/install/install-visual-studio). Ensure it is updated to v15.5.2.
* Ensure the component `.NET Core Runtime` is installed. This installs .NET Core v1.x.
* Download and install [.NET Core SDK v2.1.2](https://www.microsoft.com/net/download/windows).
