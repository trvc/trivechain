
## Controlling Trivechain

Your Trivechain node (trivecoind) can be controlled via [HTTP JSON-RPC](http://json-rpc.org/wiki/specification)  commands.

You must create a trivecoin.conf configuration file setting rpcuser and rpcpassword; for example:

```
server=1
daemon=1
rpcuser=user
rpcpassword=password
rpcallowip=127.0.0.1 
```

Now run:
```
  $ ./trivecoind
  trivecoin server starting
  $ ./trivecoin-cli help
  # shows the help text
```

An list of RPC calls will be shown.
```
  $ ./trivecoin-cli getbalance
  2000.00000
```

If you are learning the API, it is a very good idea to use the test network (add testnet=1 in your trivecoin.conf).

## JSON-RPC 

Running Bitcoin with the -server argument (or running trivecoind) tells it to function as a [HTTP JSON-RPC](http://json-rpc.org/wiki/specification) server, but 
[Basic access authentication](http://en.wikipedia.org/wiki/Basic_access_authentication) must be used when communicating with it, and, for security, by default, the server only accepts connections from other processes on the same machine.  If your HTTP or JSON library requires you to specify which 'realm' is authenticated, use 'jsonrpc'.

Allowing arbitrary machines to access the JSON-RPC port (using the rpcallowip [[Running_Bitcoin|configuration option]]) is dangerous and **strongly discouraged** -- access should be strictly limited to trusted machines.

To access the server you should find a [suitable library](http://json-rpc.org/wiki/implementations) for your language.


## Python 
[python-jsonrpc](http://json-rpc.org/wiki/python-json-rpc) is the official JSON-RPC implementation for Python.
It automatically generates Python methods for RPC calls.


```
from jsonrpc import ServiceProxy
  
trivechain = ServiceProxy("http://user:password@127.0.0.1:9998")
trivechain.getinfo()
trivechain.listreceivedbyaddress(6)
#trivechain.sendtoaddress("TRYM2o3M8jxXjYCVM2BSX3aE3R24PStBLU", 10)
```



## PHP 

The [JSON-RPC PHP](http://jsonrpcphp.org/) library also makes it very easy to connect to Trivechain.  For example:

```
  require_once 'jsonRPCClient.php';
  
  $trivechain = new jsonRPCClient('http://user:password@127.0.0.1:9998/');
   
  echo "<pre>\n";
  print_r($trivechain->getinfo()); echo "\n";
  echo "Received: ".$trivechain->getreceivedbylabel("Your Address")."\n";
  echo "</pre>";
```

'''Note:''' The jsonRPCClient library uses fopen() and will throw an exception saying "Unable to connect" if it receives a 404 or 500 error from bitcoind. This prevents you from being able to see error messages generated by bitcoind (as they are sent with status 404 or 500). The [https://github.com/aceat64/EasyBitcoin-PHP EasyBitcoin-PHP library] is similar in function to JSON-RPC PHP but does not have this issue.


## .NET (C#) 
The communication with the RPC service can be achieved using the standard http request/response objects.
A library for serializing and deserializing Json will make your life a lot easier:

Json.NET ( http://james.newtonking.com/json ) is a high performance JSON package for .NET.  It is also available via NuGet from the package manager console ( Install-Package Newtonsoft.Json ).

The following example uses Json.NET:

```
 HttpWebRequest webRequest = (HttpWebRequest)WebRequest.Create("http://localhost.:9998");
 webRequest.Credentials = new NetworkCredential("user", "pwd");
 /// important, otherwise the service can't desirialse your request properly
 webRequest.ContentType = "application/json-rpc";
 webRequest.Method = "POST";
  
 JObject joe = new JObject();
 joe.Add(new JProperty("jsonrpc", "1.0"));
 joe.Add(new JProperty("id", "1"));
 joe.Add(new JProperty("method", Method));
 // params is a collection values which the method requires..
 if (Params.Keys.Count == 0)
 {
  joe.Add(new JProperty("params", new JArray()));
 }
 else
 {
     JArray props = new JArray();
     // add the props in the reverse order!
     for (int i = Params.Keys.Count - 1; i >= 0; i--)
     {
        .... // add the params
     }
     joe.Add(new JProperty("params", props));
     }
  
     // serialize json for the request
     string s = JsonConvert.SerializeObject(joe);
     byte[] byteArray = Encoding.UTF8.GetBytes(s);
     webRequest.ContentLength = byteArray.Length;
     Stream dataStream = webRequest.GetRequestStream();
     dataStream.Write(byteArray, 0, byteArray.Length);
     dataStream.Close();
     
     
     WebResponse webResponse = webRequest.GetResponse();
     
     ... // deserialze the response
```

== Node.js ==

[jayson](https://github.com/tedeh/jayson) 

Example using jayson:

```
var jayson = require('jayson');

// create a client
var client = jayson.client.http({
	port: 9998,
	host: '127.0.0.1',
	auth: 'user:pass'
});

client.request('getinfo', [], function(err, response) {
  if(err) {
  	console.log(err);
  	return;
  }
  console.log(response.result);
});
```

## Command line (cURL)

You can also send commands and see results using [cURL](http://curl.haxx.se/) or some other command-line HTTP-fetching utility; for example:

```
  curl --user user --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "getinfo", "params": [] }' 
    -H 'content-type: text/plain;' http://127.0.0.1:9998/
```

You will be prompted for your rpcpassword, and then will see something like:

```
  {"result":{"version":1000001,"protocolversion":70208,"walletversion":61000,"balance":0.00000000,"exclusivesend_balance":0.00000000,"blocks":16438,"timeoffset":0,"connections":5,"proxy":"","difficulty":2573.697682180916,"testnet":false,"keypoololdest":1522680793,"keypoolsize":999,"paytxfee":0.00000000,"relayfee":0.00010000,"errors":""},"error":null,"id":"curltest"}
```

