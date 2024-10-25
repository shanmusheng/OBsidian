```
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Spotify OAuth Login")),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            if (_accessToken == null)
              ElevatedButton(
                onPressed: _connectAndGetToken,
                child: Text("Login with Spotify"),
              )
            else
              Column(
                children: [
                  Text("Access Token: $_accessToken"),
                  ElevatedButton(
                    onPressed: getCurrentTrackInfo,
                    child: Text("Login with Spotify"),
                  ),
                  Text('Current track: $trackName'),
                  Text('Artist: $artistName'),
                  Text('Album: $albumName'),
                ],
              ),
          ],
        ),
      ),
    );
  }
```