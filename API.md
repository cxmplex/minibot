# Custom API

- Common
  - [Arguments](#common-arguments)
  - [Constants](#common-constants)
  - [File](#common-file)
  - [Maths](#common-maths)
  - [Memory](#common-memory)
  - [Miscellanea](#common-miscellanea)
  - [Network](#common-network)
- In-World
  - [Action](#in-world-action)
  - [Aura](#in-world-aura)
  - [BlackTech](#in-world-blacktech)
  - [Missile](#in-world-missile)
  - [Navigation](#in-world-navigation)
  - [Spell](#in-world-spell)
  - [State](#in-world-state)
  - Object
    - [Constants](#in-world-object-constants)
    - [Descriptor](#in-world-object-descriptor)
    - [Field](#in-world-object-field)
    - [General](#in-world-object-general)
    - [Miscellanea](#in-world-object-miscellanea)
  - [Object Manager](#in-world-object-manager)
  - [Vision](#in-world-vision)

This is the up-to-date list of `MiniBot` custom API, classified into 3 categories according to their lifecycles:

1. Common<br />
The functions that always exist in both GLUE and World.
2. In-World<br />
The functions that only exist in-world.
3. GLUE<br />
The functions that only exist in GLUE screens.

All of the custom API below are registered in a specific global table `wmbapi` if the client is not OEM. Otherwise, the table name becomes `[MACRO]api`, such as "abcapi" if your OEM short name is "abc". Remember to refer back to this from time to time as the API may change in future.

Before diving into the list, last but not least, we **DO NOT** provide intensive support for the API set. Questions such as "hey, what does API XXX mean?" will be ignored. Please help yourself if you decide to utilize them for your own good.

[Back to Top](#custom-api)

## Common (Arguments)

- `path = GetAppStorageDirectory()`

Gets the base directory path of app storage.

- `path = GetAppDirectory()`

Gets the app base directory path.

- `username = GetAppUsername()`

Gets the app username.

- `count = GetAppSessionCount()`

Gets the total session count of the current launch.

- `index = GetAppSessionIndex()`

Gets the index of the current launched session, starting from 1.

- `token = GetAppSessionToken()`

Gets the unique token of the current launched session. This is a randomly generated unique string.

- `path = GetWoWDirectory()`

Gets the WoW base directory path.

- `value = GetSystemVar(name)`

Gets the value of the system variable previously set by `SetSystemVar`. **nil** if it does not exist.

- `value = SetSystemVar(name, value)`

Sets a system variable.

The standard Lua environment gets reset after screen switch including **/reload**. All Lua variables get lost because of this. In this way, we provide "System Variables" that allow you to persist information across different screens in the current game session. This is similar as the game "Console Variable", consisting of a name-value pair.

```lua
-- name (string): The name of the system variable.
-- value (string): The value of the system variable.
```

[Back to Top](#custom-api)

## Common (Constants)

- `encodings = GetStringEncodingsTable()`

Gets the table of supported string encodings.

```lua
encodings = {
  Base64 = 1,
  Hex = 2
}
```

Notice that "Hex" encoded strings are always generated in lower case, in order to match the global convention.

- `types = GetValueTypesTable()`

Gets the table that contains all value types to read from memory, which looks like:

```lua
types = {
  Bool = 1,
  Char = 2,
  Byte = 3,
  Short = 4,
  UShort = 5,
  Int = 6,
  UInt = 7,
  Long = 8,
  ULong = 9,
  Float = 10,
  Double = 11,
  String = 12,
  IntPtr = 13,
  UIntPtr = 14,
  GUID = 15,
}
```

## Common (Encryption)

- `output = Sha256(input[, encoding])`

Computes the hash digest of a plain string by SHA-256.

```lua
-- input (string): The plain string.
-- encoding (number): One of the encodings to generate the result, from GetStringEncodingsTable(). nil for default encodings.Base64.
-- output (string): The digest string.
```

In order to make sure your own SHA256 digest matches the API above, the test vector is given below:

```lua
local test = "The quick brown fox jumps over the lazy dog";
local encodings = GetStringEncodingsTable();
local digest = Sha256(test, encodings.Hex);
print("result:", digest);
-- result: d7a8fbb307d7809469ca9abcb0082e4f8d5651e46d3cdb762d02d0bf37c9e592
```

- `output = AesEncrypt(input, key, iv[, encoding])`

Encrypts a plain string by AES-256 CBC with PKCS#5 padding.

```lua
-- input (string): The plain string.
-- key (string): The key for encryption (must be 32 letters).
-- iv (string): The IV for encryption (must be 16 letters).
-- encoding (number): One of the encodings to encrypt the result, from GetStringEncodingsTable(). nil for default encodings.Base64.
-- output (string): The encrypted string.
```

- `output = AesDecrypt(input, key, iv[, encoding])`

Decrypts an encrypted string by AES-256 CBC with PKCS#5 padding.

```lua
-- input (string): The encrypted string.
-- key (string): The key for encryption (must be 32 letters).
-- iv (string): The IV for encryption (must be 16 letters).
-- encoding (number): One of the encodings into which input is encrypted, from GetStringEncodingsTable(). nil for default encodings.Base64.
-- output (string): The plain string. nil if decryption fails.
```

In order to make sure your own AES encryption matches the API above, the test vector is given below:

```lua
local test = "test";
local encodings = GetStringEncodingsTable();
local encrypted_string = AesEncrypt(test, "01234567890123456789012345678901", "0123456789012345", encodings.Hex);
local decrypted_string = AesDecrypt(encrypted_string, "01234567890123456789012345678901", "0123456789012345", encodings.Hex);
print("result:", encrypted_string, decrypted_string, test == decrypted_string);
-- result: a68c23c38693a8ebd1d6276fbb90e5e1 test true
```

- `output = RsaEncrypt(input, publicKey[, encoding])`

Encrypts a plain string by RSA OAEP with SHA padding.

```lua
-- input (string): The plain string.
-- publicKey (string): The PEM text of the public key.
-- output (string): The encrypted string. nil if encryption fails.
```

- `output = RsaDecrypt(input, privateKey[, encoding])`

Decrypts an encrypted string by RSA OAEP with SHA padding.

```lua
-- input (string): The encrypted string.
-- privateKey (string): The PEM text of the private key.
-- encoding (number): One of the encodings into which input is encrypted, from GetStringEncodingsTable(). nil for default encodings.Base64.
-- output (string): The plain string. nil if decryption fails.
```

In order to make sure your own RSA encryption matches the API above, the test vector is given below:

```lua
local test = "test";
local encodings = GetStringEncodingsTable();
local encrypted_string = RsaEncrypt(test, [[-----BEGIN PUBLIC KEY-----
MIIBojANBgkqhkiG9w0BAQEFAAOCAY8AMIIBigKCAYEA1MSdsaPH2ShtjOo4c02+
DbYcTdwUBLY+vNSXr2tV8/jGU059Jak9CA7VSlKR/fik18D7Lq1beLjW56kV4Xvm
3qmpxOc3eNGmj8dqtO0G3Lp1FAZzxlu2SZsHmmVq9isZcN70apkwlDgIZ11NVIq/
1iXzr0pIRMKkMNHTGBGBkYOrIcgdH2elvIqfiit6Gts/zho4YCjgyn/r3Vgy/jCu
6VbfwE9xVY/DB4srD5LrZMabRzN2YwSTI+sRqpbt7I7nZ6o8CuyqHDLjbO9VzE0p
ovBshTfoyog9XGcQHwTmWn4bdnsh2I1x3gQpaqxdRs4vnKmXJ9GvC/sYla0GYXyD
ecpgjITqx3QA6aKx9+EVh/o6owYTHXaToVkP7U5m8cqaloQFfA8HLsGDg9A0QaMt
ixnX7KtT/ZvKFMcazRJ1GX42UaeuO1opZKtjBHLtmaPadNeZdD77VytwY2UHeW5Q
Snfpos7IxUTATpd6KTWUV3snVQnyiltCI1BHJC01sWePAgMBAAE=
-----END PUBLIC KEY-----]], encodings.Base64);
-- encrypted_string = e5fVDawpzGV9SrPLmljQBSrk13P2EaH0vReA+pD6I4bMoe9BHBZS92eTSoqNn42QXzWFMwz+xuPKzTs0gcgr8Y3U9Wwsqs+uibFgBFyiCf9HrzqyftViWEUV+jzrFmvr88uOX0kRP+u2jkSWangc5b4S/IcnqrsHmqy8+HYT18Pv6/XZaV0f2u8lmPIzygFrFBGO/TeczW9bgRu7ZGAMpge8iI0pyXh9f41Ax9r3COtXpDSHF06Vno9hvPRIg8zbhWaqwjjH3SZ/vjytZCS3qxeLDzBPL36MURwRHJ7yFqBABZ0M8RecEYbDPp0i3T1xmjc72A5OqgxUCA/x1WU4dftTtSfG9+wQlWiEu7YhkPIntROXdtQ7WQccdV3RsH3XvsmoyWGqh99+IOthMOSLyvOH0YjfYqmmM8tVDcpLH2xyirmJ7aBR83uFmttEumbY/nZoR/9L1KznrgaKqmanza7WhhG7Yk3FvwzwjPfFD/hyviv0E4gJJsiKephZ/ufM
local decrypted_string = RsaDecrypt(encrypted_string, [[-----BEGIN RSA PRIVATE KEY-----
MIIG4wIBAAKCAYEA1MSdsaPH2ShtjOo4c02+DbYcTdwUBLY+vNSXr2tV8/jGU059
Jak9CA7VSlKR/fik18D7Lq1beLjW56kV4Xvm3qmpxOc3eNGmj8dqtO0G3Lp1FAZz
xlu2SZsHmmVq9isZcN70apkwlDgIZ11NVIq/1iXzr0pIRMKkMNHTGBGBkYOrIcgd
H2elvIqfiit6Gts/zho4YCjgyn/r3Vgy/jCu6VbfwE9xVY/DB4srD5LrZMabRzN2
YwSTI+sRqpbt7I7nZ6o8CuyqHDLjbO9VzE0povBshTfoyog9XGcQHwTmWn4bdnsh
2I1x3gQpaqxdRs4vnKmXJ9GvC/sYla0GYXyDecpgjITqx3QA6aKx9+EVh/o6owYT
HXaToVkP7U5m8cqaloQFfA8HLsGDg9A0QaMtixnX7KtT/ZvKFMcazRJ1GX42Uaeu
O1opZKtjBHLtmaPadNeZdD77VytwY2UHeW5QSnfpos7IxUTATpd6KTWUV3snVQny
iltCI1BHJC01sWePAgMBAAECggGAEG1tz31ZvMaGTs72tNBX0C8zWD+ZvBNmHKY9
X+nlpQScK2pv9yxt7eVXSnm9k+JSt+XKfvwbh+KdlR1U9yfd12s6FF3VxppJReib
sIRsdzZeO8GTxsjl9iDmIWGbNI53VGOic2iIe6kn3PMzOUfNL/eWLP6LPePZUXuh
1MXlPxrvZ5hPx1D1Vu1NDBn3P4OWFY+osqP1Vy0xRNG+fim8F4ABnpODqJuE71wr
YvRxAELlUkYC6fo8chWAM6+bhxwxVaGiIKluikmVJtt0/aAcKR6fUogGfcumRGPp
HzFRDZBVdLmVwbpVrfCbULP7wYk2A5QMu2skAlZSYtyWJbBRXvgweEXepJaXC6FW
atD5ypi1kSX9K71BRM7DKrmY2/RsyR6Y8a2PdiOHB5MNYKoeH5o2k0htsV2zUspo
4nER4AB5a5fEysGg3yCST+m2q7UOBvcB0LblE/0sNuOGtCNPmtChdZxspsVRm2ID
XkKrljy+cdOsxZ0iVcvGhyJRhlCBAoHBAOn6KMfbB11uliVyouFfV5ZoiWPeIXbF
wkAnev+8kF/GmYU7bAFAhRg2qzwqTVlC2eeG+dHKgr9+xHjsTOIoLB/5jPgcIfY9
l0lZ9LmNwwvI3wg6XWnwQf9X97YZ1E1A3TpBU5XNzTo7hVtZgHDIf4ufB5sDhZ1S
nXf/+uBe7gJMMnizpq/tqr+0oPJd4uac1rTp2wsFx6MJjOR8kijZOnr3SdKNU3xo
shZWlRHy9qCjftxTIuOFSxdEZhJUm87w8QKBwQDoy2hYI0hMn3+lwu30lk4+LGSW
9ij7AzyTVcRR9FbYciTMQ24IrK020A9rDXkVkJ6FeTbCtT3UkFOlz3JZkEpvY/qd
Mf8hfd5IO68R1Z5lZpLCFAqcIRUE9l7En9nMiuqdDPZJfhUjhlajzhQotYEv1Fqq
WDmK0IaklSfGJt0LVsZSuINErHaC5HjJocL86Cqao9a1rxgJA7maCfirwABAafHc
6OhFuW5Pi6IXj9QbM7PgbGjIIXPDFfs7FkqF4H8CgcEAu0MACJSAXIL5oJcTTZVl
IHgiHc/WsJyuT3JJuwxL8Juem0dntcjRvQNkIQ8qQNqEVA1vPDz8UA9BaBaXohnM
1vp/nMPHWrEIuChK+YdAJ9poxskPoo4sBBV/qDsb84iKhulp4GeKbaTdorMLXTja
/AAXsjUrZzKL3VL+kzzm+OfLLVd7fSqWkkAa4F/MDg5QuRLBwRyrHw2xud0Jja/u
YiQw7Vc3Dkcs4TwCqw7t3Lt9+RCAx+ASrViM6PbWjNXBAoHAa0fiDEwmM3mFn+RX
ONJTuH9I0/EZLaRuNA/ga0xJAXKI1sF0YfcB1DLKCDGrTW7aPvR/cfeISP9CLTWO
owvF4dOXWP4Db3HMEEnBAl0Jo/1DQMFvqkfsod7QCZkJDCQwvrOMhI3gPADayJ5d
1+zdXidkqQADdJ9ojUxXig+66lDREKoLhIheDTAxIeq0K0zq5Vz/w7avQug+jmht
+uh+tTCdz4peEFPGLE5TIrybqPWIvbH4D9KqwIrOvoolSdENAoHAaa+n0ZXGovFy
Hjk02KSinY80b0VzOKKXCh3vc5+2WAS9Ar4no7Cobt5QhKA0GtYpLSCmUFRvsZ1P
Gemb/FH+yC5nLvKaDOpHktZONIARP8e9R1ku9o+9lOFAIU0MYHx0Ep0y4XWgMrTp
UuP3ai7zn++ag7Lu1QEm5pQAd2n+zMuKZbBISVA9fPbC9RkJX66E4zVbsEUnDDBD
9Rlu+3Dc0LwSjtAxXPDInmEh2mp3O/aZtMPVUPgDA4Ig7GbQC6W/
-----END RSA PRIVATE KEY-----]], encodings.Base64);
print(test == decrypted_string);
-- true
```

## Common (File)

- `isOrNot = FileExists(path)`

Checks if a file exists.

- `content = ReadFile(path[, encoding])`

Reads all content from a file. If you specify encoding, binary content is read. Otherwise, UTF-8 text is read.

```lua
-- path (string): The file path.
-- encoding (number): One of the encodings for the binary read result, from GetStringEncodingsTable(). nil for text read.
-- content (string): The file content.
```

- `success = WriteFile(path, content[, append[, encoding]])`

Writes certain content to a file. If you specify encoding, binary content is written (append is not supported in this case). Otherwise, UTF-8 text is written.

```lua
-- path (string): The file path.
-- content (string): The file content.
-- append (boolean): Whether to append the text content or overwrite the file.
-- encoding (number): The encoding of the file binary content, from GetStringEncodingsTable(). nil for text write.
-- success (boolean): Whether the content is successfully written to the file.
```

- `isOrNot = DirectoryExists(path)`

Checks if a directory exists.

- `success = CreateDirectory(path)`

Creates a directory.

- `fileNames = GetDirectoryFiles(path)`

Gets all file names in a specific directory. Remind the path must end with wildcards. e.g. ```C:\Windows\*.lua```

- `folderNames = GetDirectoryFolders(path)`

Gets all sub folder names in a specific directory.

[Back to Top](#custom-api)

## Common (Maths)

- `centers = GetAllSpanningCircles(radius, minWeight, points)`

Gets all spanning circles of a specific radius over certain weighted points.

[Back to Top](#custom-api)

## Common (Memory)

- `value = ReadMemory(module, offset, type)`

Reads a value at a specific memory offset/rva in a specific memory module.

```lua
-- module (string): The name of the memory module. nil for "Wow.exe" aka the main module.
-- offset (number): The offset/rva in the memory module to read.
-- type (number): The type of the value. Check GetValueTypesTable().
-- value (number): The result value. nil if the memory address is not found.
```

- `offset = GetMemoryOffset(module, address)`

Gets the offset of a memory address in a specific module.

```lua
-- module (string): The name of the memory module. nil for "Wow.exe" aka the main module.
-- address (number): The absolute memory address.
-- offset (number): The result offset. nil if the memory address is not found.
```

## Common (Miscellanea)

- `down, toggled = GetKeyState(key)`

Gets the pressed state of a specific key.

- `success = PlaySoundFile(path)`

Plays a specific sound WAV/MP3 file once.

- `scriptFunction | error = LoadScript(name, script)`

Loads a Lua script as a function.

```lua
-- name (string): The script name, shown in any Lua error message.
-- script (string): The script text.
-- scriptFunction (function): The loaded function if load succeeds.
-- error (string): The Lua syntax error message if load fails.
```

- `scriptFunction | error = LoadScriptAesEncrypted(name, script, key, iv[, encoding])`

Loads a Lua script encrypted with AES-256 CBC with PKCS#5 padding as a function. Check `AesDecrypt()` for more details.

```lua
-- name (string): The script name, shown in any Lua error message.
-- script (string): The encrypted script text.
-- key (string): The key for encryption (must be 32 letters).
-- iv (string): The IV for encryption (must be 16 letters).
-- encoding (number): One of the encodings into which input is encrypted, from GetStringEncodingsTable(). nil for default encodings.Base64.
-- scriptFunction (function): The loaded function if load succeeds.
-- error (string): The Lua syntax error message if load fails.
```

- `RunScript(name, script)`

Runs a Lua script by engine. Notice that any syntax/runtime error will pop up an error message box.

```lua
-- name (string): The script name, shown in any Lua error message.
-- script (string): The script text.
```

- `RunScriptAesEncrypted(name, script, key, iv[, encoding])`

Runs a Lua script encrypted with AES by engine. Notice that any decryption failures will cause the API to silently return and any syntax/runtime error will pop up an error message box. Check `AesDecrypt()` for more details.

```lua
-- name (string): The script name, shown in any Lua error message.
-- script (string): The encrypted script text.
-- key (string): The key for encryption (must be 32 letters).
-- iv (string): The IV for encryption (must be 16 letters).
-- encoding (number): One of the encodings into which input is encrypted, from GetStringEncodingsTable(). nil for default encodings.Base64.
-- scriptFunction (function): The loaded function if load succeeds.
-- error (string): The Lua syntax error message if load fails.
```

- `SetCustomScript(name, script)`

Adds a custom script (indexed by name) that gets loaded side by side with the engine modules (Primary and Secondary). Notice that such script also gets loaded in GLUE screen.

[Back to Top](#custom-api)

## Common (Network)

- `request = SendHttpRequest(info)`

Sends a HTTP request asynchronously. The API will return immediately after the request is sent without waiting for the response.

```lua
-- The request info.
info = {
  -- string: The request URL.
  Url = "https://www.microsoft.com/",
  -- [OPTIONAL] string: The request method, can be "GET", "POST", "PUT" or "DELETE".
  Method = "POST",
  -- [OPTIONAL] string: The additional request headers.
  Headers = "Content-Type: application/json\r\nX-Custom: test",
  -- [OPTIONAL] string: The request body, only used Method is "POST" or "PUT".
  Body = "{\"test\": 123}",
  -- [OPTIONAL] string: The pinned HTTPs server certificate as a protection for packet sniffing. If provided, the server certificate must match it or the HTTP request would fail with status "INVALID_CERTIFICATE".
  Certificate = "PINNED CERTIFICATE",
  -- [OPTIONAL] function: The callback function when the status of the request is updated.
  Callback = function(request, status) ... end,
}
-- The HTTP request ID if sent successfully, for querying HTTP response later.
request = "abc123"
```

- `status[, response] = ReceiveHttpResponse(request)`

Checks the status and receives the response of an existing HTTP request. The request will be cleared from memory after the API call if the request is done successfully or unsuccessfully.

```lua
-- The HTTP request ID previously sent.
request = "abc123"
-- The current status of the HTTP request, can be:
-- "REQUESTING": The request is still on the way.
-- "REQUEST_FAILED": The request is terminated due to failures.
-- "INVALID_CERTIFICATE": The request is terminated due to invalid HTTPs certificate.
-- "RESPONDING": Downloading response after the request is sent.
-- "RESPONSE_HEADERS_FAILED": The response download is terminated while fetching response headers.
-- "RESPONSE_BODY_FAILED": The response download is terminated while fetching response body.
-- "SUCCESS": The response is received and everything about the HTTP request is done.
status = "CONNECTING"
-- The response data, available if status is "SUCCESS"
response = {
  -- number: The HTTP response status code.
  Code = 200,
  -- string: The HTTP response headers.
  Headers = "HTTP/1.1 200 OK...",
  -- string: The HTTP response body.
  Body = "...",
  -- string: The actual server certificate if the request is HTTPs, which can be used for pinning.
  Certificate = "SERVER CERTIFICATE",
}
```

Given the nature of async programming model, a simple example to send and receive HTTP is given as below, leveraging the callback function.

```lua
wmbapi.SendHttpRequest({
  Url = "https://www.microsoft.com",
  ...
  Callback = function(request, status)
    -- Deal with the current status and response of the HTTP request here.
    if (status == "SUCCESS") then
      local _, response = wmbapi.ReceiveHttpResponse(request);
      print("response body:", response.Body);
    end
  end
});
```

Another example to send and receive HTTP is given as below, leveraging frame updates.

```lua
local http_frame = CreateFrame("FRAME");
local http_request;
http_frame:SetScript("OnUpdate", function()
  if (not http_request) then
    local info = {...}; -- Fill your actual info here.
    http_request = wmbapi.SendHttpRequest(info);
  else
    local http_status, http_response = wmbapi.ReceiveHttpResponse(http_request)
    if (not http_status) then
      -- The HTTP request has been fulfiled.
      http_frame:SetScript("OnUpdate", nil);
      http_frame = nil;
      return;
    end
    -- Deal with the current status and response of the HTTP request here.
    if (http_status == "SUCCESS") then
      print("response body:", http_response.Body);
    end
  end
end);
```

- `connection = ConnectWebsocket(info)`

Connects to a remote websocket server asynchronously. The API will return immediately after the connection is initiated.

```lua
-- The websocket info.
info = {
  -- string: The websocket URL. (Use http(s) instead of ws(s))
  Url = "https://echo.websocket.org",
  -- [OPTIONAL] string: The additional request headers.
  Headers = "Content-Type: application/json\r\nX-Custom: test",
  -- [OPTIONAL] string: The pinned HTTPs server certificate as a protection for packet sniffing. If provided, the server certificate must match it or the websocket connection would fail with status "INVALID_CERTIFICATE".
  Certificate = "PINNED CERTIFICATE",
  -- [OPTIONAL] function: The callback function when the status of the connection is updated, which can be any one of "CONNECTING", "CONNECTION_FAILED", "INVALID_CERTIFICATE", "CONNECTED", "CLOSING", "CLOSED".
  ConnectCallback = function(connection, status) ... end,
  -- [OPTIONAL] function: The callback function when a piece of data is sent over the connection. (Only string data is supported)
  SendCallback = function(connection, data) ... end,
  -- [OPTIONAL] function: The callback function when a piece of data is received over the connection. (Only string data is supported)
  ReceiveCallback = function(connection, data) ... end
}
-- The websocket connection ID if sent successfully.
connection = "abc123"
```

- `success = CloseWebsocket(connection)`

Closes an existing websocket connection.

- `success = SendWebsocketData(connection, data)`

Sends a piece of string data over an existing websocket connection.

Given the nature of async programming model, a simple example to establish a websocket connection, send data, and disconnect it after data is received is given as below.

```lua
wmbapi.ConnectWebsocket({
  Url = "https://echo.websocket.org",
  ...
  ConnectCallback = function(connection, status)
    -- Deal with the current connection status here.
    if (status == "CONNECTED") then
      print("connected!");
      wmbapi.SendWebsocketData(connection, "test data!");
    elseif (status == "CLOSED") then
      print("disconnected!");
    end
  end,
  SendCallback = function(connection, data)
    print("data sent:", data);
  end,
  ReceiveCallback = function(connection, data)
    print("data received:", data);
    wmbapi.CloseWebsocket(connection);
  end
});
```

- `success = DisconnectFromServer(reason)`

Disconnects from the connected realm server.

```lua
-- reason (number): The system reason of the disconnection, in integer.
-- success (boolean): Whether the disconnection succeeds.
```

## In-World (Action)

- `ClickPosition(x, y, z[, right])`

Clicks a world position.

- `FaceDirection(angle[, update])`

Faces a horizontal direction, in radian.

- `SetPitch(angle)`

Sets the player vertical pitch, in radian.

- `MoveTo(x, y, z[, instantTurn])`

Moves the player to a specific position, using CTM.

[Back to Top](#custom-api)

## In-World (Aura)

- `count = GetAuraCount(unit, [spellId])`

Gets the count of auras on a specific unit, optionally filtered by spell ID. You must call this API first to actually search auras to be saved in memory. This API returns all auras including hidden ones, more than you can get normally by BLZ API [UnitAura](https://wow.gamepedia.com/API_UnitAura).

```lua
-- unit (string): The unit to check the auras.
-- spellId (number): The spell ID of the auras to filter. nil for all.
-- count (number): The actual aura count. nil if the unit does not exist.
```

- `spellId | ... = GetAuraWithIndex(index[, detailed = false])`

Gets the info of a specific aura, saved by the most recent call to `GetAuraCount()`. If "detailed" is true, the return values will be the same as the BLZ API [UnitAura](https://wow.gamepedia.com/API_UnitAura). If "detailed" is false, the return value will be only the aura spell ID with a bit better performance.

Due to technical difficulties, we only make partial results available in Classic version, which include `_, _, count, _, duration, expirationTime, source, _, _, spellId, ...` ONLY. But in Retail, all are just fine so don't worry.

```lua
-- index (number): The index of the aura.
-- detailed (boolean): Whether to get the details for the aura.
-- spellId (number): The spell ID of the aura.
```

A code example to show all auras of the current target is given below.

```lua
if (UnitExists("target")) then
  local aura_count = GetAuraCount("target");
  print(">>>>" .. UnitName("target") .. "<<<<");
  for i = 1, aura_count do
    print(GetAuraWithIndex(i, true));
  end
end
```

[Back to Top](#custom-api)

## In-World (BlackTech)

- `SetCameraDistanceMax([distance])`

Sets the current camera distance maximum. If nil, restore original setting.

- `SetClimbAngle([angle])`

Sets the engine allowed climb angle, in radian. If nil, restore original setting.

- `success = SetCVarEx(name, value)`

Sets a CVar without system limitation.

- `SetNameplateDistanceMax([distance])`

Sets the current nameplate visible distance maximum. If nil, restore original setting.

- `StopFalling()`

Stops the current falling of the character right now.

- `EnableFlyingMode(enabled)`

Enable/Disable flying mode of the character. (HINT: DO NOT enable flying while falling or you would get disconnected)

Due to 9.0 changes, this API stops working in Retail. We are working on alternative solutions, but please sit tight.

- `isFlying = IsFlyingModeEnabled()`

Gets whether flying mode is enabled for the character.

Due to 9.0 changes, this API stops working in Retail. We are working on alternative solutions, but please sit tight.

- `modes = GetNoClipModes()`

Gets the current no-clip mode flags, which is a sum of:
0: none
1: building
2: static object
4: dynamic object
8: terrain (be careful!)

- `SetNoClipModes(modes)`

Sets the current no-clip mode flags. Check the enum above.

[Back to Top](#custom-api)

## In-World (Missile)

- `count = GetMissileCount()`

Gets the count of the flying missiles.

- `spellId, spellVisualId, x, y, z, sourceObject, sourceX, sourceY, sourceZ, targetObject, targetX, targetY, targetZ = GetMissileWithIndex(index)`

Gets the info of a specific missile.

[Back to Top](#custom-api)

## In-World (Navigation)

- `mapId, zoneId = GetCurrentMapInfo()`

Gets the map information about the current location.

- `isOrNot = MapExists(mapId)`

Checks whether the navigation files for a specific map exists.

- `success = LoadMap(mapId)`

Loads a navigation map. Notice that map files must be placed correctly before loading.

- `success = UnloadMap(mapId)`

Unloads a navigation map.

- `isOrNot = IsMapLoaded(mapId)`

Checks if a navigation map is loaded.

- `points, polygons = FindPath(mapId, x1, y1, z1, x2, y2, z2[, ignoreWater = true|ignoreFlags = 8(NAV_WATER), searchCapacity = 1024, agentRadius = 0])`

Calculates a path to navigate from one position to another. Notice that the mapId must be loaded beforehand for the calculation to work properly.

```lua
-- mapId (number): The map ID.
-- x1, y1, z1 (number): The start position.
-- x2, y2, z2 (number): The end position.
-- ignoreWater (boolean): Whether to avoid walking into water terrain.
-- ignoreFlags (number): Alternatively, if you pass a number in place of ignoreWater, you can specify navmesh polygon flags to be ignored. ignoreWater = true is equal to ignoreFlags = 8 which is NAV_WATER flags on Classic or ignoreFlags = 2 which is NAV_WATER flags on Retail. This is useful if your map pack has different poly flags or if you wish to ignore arbitrary polygons.
-- searchCapacity (number): This is a combined value to specify the maximum count of waypoints and the size of search node pool. Increase the number if you wish to find longer path, but meanwhile it will also take longer time due to more navmesh polygons to traverse.
-- agentRadius (number): The radius of the navigation agent (your character), in yards. To avoid wall collision, you should set the value accordingly. This value can only take 5 in maximum, and any number > 5 will lead to ignorance.
-- points (table): The array of navigation points of the found path. NIL if path is not found.
-- polygons (table): The array of navigation mesh polygons of the found path. NIL if path is not found.
```

- `resultX, resultY, resultZ, distance = GetClosestPositionOnMesh(mapId, x, y, z[, ignoreWater = true])`

(OBSOLETE, Use `GetClosestMeshPolygon()` instead)

Projects a specific position onto a navigation mesh map. Notice that if the position is too far away from any mesh polygons, the API will return nil.

```lua
-- mapId (number): The map ID.
-- x, y, z (number): The position to be projected.
-- ignoreWater (boolean): Whether water map areas are to be ignored/excluded.
-- resultX, resultY, resultZ (number): The result position projected on mesh. nil if the position is too far away.
-- distance (number): The distance between {x, y, z} and {resultX, resultY, resultZ}.
```

- `polygon[, polygonX, polygonY, polygonZ, polygonDistance] = GetClosestMeshPolygon(mapId, x, y, z, dX, dY, dZ[, ignoreWater = true|ignoreFlags = NAV_WATER])`

Searches for the closest polygon on mesh in a certain cube with a specific center position.

```lua
-- mapId (number): The map ID.
-- x, y, z (number): The center position.
-- dX, dY, dZ (number): The cube dimensions.
-- ignoreWater (boolean): Whether to ignore water areas.
-- ignoreFlags (number): Alternatively, if you pass a number in place of ignoreWater, you can specify navmesh polygon flags to be ignored. ignoreWater = true is equal to ignoreFlags = 8 which is NAV_WATER flags on Classic or ignoreFlags = 2 which is NAV_WATER flags on Retail. This is useful if your map pack has different poly flags or if you wish to ignore arbitrary polygons.
-- polygon (string): The closest polygon found. NIL if none.
-- polygonX, polygonY, polygonZ (number): The closest position on polygon. NIL if none.
-- polygonDistance (number): The distance between {x, y, z} and {polygonX, polygonY, polygonZ}.
```

- `polygons = GetMeshPolygons(mapId, startPolygon, x, y, z, radius[, ignoreWater = true|ignoreFlags = NAV_WATER, resultCountMax = 1024])`

Searches for adjacent mesh polygons starting from a certain polygon, within a certain sphere. Notice that only adjacent polygons that interset with the sphere will be included.

```lua
-- mapId (number): The map ID.
-- startPolygon (string): The navigation mesh polygon to start with.
-- x, y, z (number): The center position.
-- radius (number): The search range, in yards.
-- ignoreWater (boolean): Whether to ignore water areas.
-- ignoreFlags (number): Alternatively, if you pass a number in place of ignoreWater, you can specify navmesh polygon flags to be ignored. ignoreWater = true is equal to ignoreFlags = 8 which is NAV_WATER flags on Classic or ignoreFlags = 2 which is NAV_WATER flags on Retail. This is useful if your map pack has different poly flags or if you wish to ignore arbitrary polygons.
-- resultCountMax (number): The maximum count of polygons to search.
-- polygons (table): The found polygons.
```

- `flags = GetMeshPolygonFlags(mapId, polygon)`

Gets the flags of a certain mesh polygon. Polygon flags are used in `FindPath()` to mark certain polygon areas as unpassable.

```lua
-- mapId (number): The map ID.
-- polygon (string): The certain navigation mesh polygon.
-- flags (number): The flags of the polygon.
```

- `success = SetMeshPolygonFlags(mapId, polygon, flags)`

Sets the flags of a certain mesh polygon. Polygon flags are used in `FindPath()` to mark certain polygon areas as unpassable.

```lua
-- mapId (number): The map ID.
-- polygon (string): The certain navigation mesh polygon.
-- flags (number): The flags of the polygon.
-- success (boolean): Whether the polygon flags are updated.
```

- `vertices = GetMeshPolygonVertices(mapId, polygon)`

Gets the vertices of a certain mesh polygon, of 6 in maximum. You can use this API to render the mesh polygons in world.

```lua
-- mapId (number): The map ID.
-- polygon (string): The certain navigation mesh polygon.
-- vertices (table): The array of vertex positions of the polygon. NIL if the polygon does not exist.
```

[Back to Top](#custom-api)

## In-World (Object-Constants)

All of the constants are contained in different tables as below. You should print the most up-to-date actual constants in-game using a simple Lua "for-pairs" loop for 100% accuracy.

- `flags = GetObjectTypeFlagsTable()`

Gets the table that contains all type flags, which looks like:

```lua
flags = {
  Object = 1,
  Item = 2,
  Container = 4,
  AzeriteEmpoweredItem = 8,
  AzeriteItem = 16,
  ...
}
```

- `fields = GetObjectFieldsTable()`

Gets the table that contains all object field offsets, which looks like:

```lua
fields = {
  ["AnimationState"] = 123,
  ...
}
```

- `descriptors = GetObjectDescriptorsTable()`

Gets the table that contains all object descriptor offsets, which looks like:

```lua
descriptors = {
  ["CGObjectData__m_guid"] = 0,
  ["CGObjectData__m_entryID"] = 8,
  ...
}
```

Also notice that there are some descriptor names ending with "_union". This indicates that there might be one or several child values mixed together in the descriptor.

- `movementFlags = GetUnitMovementFlagsTable()`

Gets the table that contains all unit movement flags, which looks like:

```lua
movementFlags = {
  Forward = 1,
  Backward = 2,
  StrafeLeft = 4,
  StrafeRight = 8,
  TurnLeft = 16,
  ...  
}
```

[Back to Top](#custom-api)

## In-World (Object-Descriptor)

- `value = ObjectDescriptor(object, offset, type)`

Gets a descriptor value of an object.

- `scale = ObjectScale(object)`

Gets the scale of an object.

- `dynamicFlags = ObjectDynamicFlags(object)`

Gets the dynamic flags of an object.

- `creator = UnitCreator(object)`

Gets the creator object of an object.

- `radius = UnitBoundingRadius(object)`

Gets the bounding radius of an unit.

- `reach = UnitCombatReach(object)`

Gets the combat reach of an unit.

- `target = UnitTarget(object)`

Gets the target object of an unit.

- `flags = UnitFlags(object)`

Gets the flags of an unit.

[Back to Top](#custom-api)

## In-World (Object-Field)

- `value = ObjectField(object, offset, type)`

Gets a field value of an object.

- `spellId, target, startTime, endTime = UnitCasting(unit)`

Gets the casting info of a unit, an enhanced version for the BLZ API ```UnitCastingInfo```.

Notice in Classic, the API only works for "player" unit due to game data restrictions. If you need further info, [LibClassicCasterino](https://github.com/rgd87/LibClassicCasterino) based on combat event estimation is recommended.

```
spellId (number): The ID of the spell casting.
target (string): The casting target object.
startTime (number): The casting start timestamp, in milliseconds.
endTime (number): The casting end timestamp, in milliseconds.
```

- `spellId, target, startTime, endTime = UnitChannel(unit)`

Gets the channel info of a unit, an enhanced version for the BLZ API ```UnitChannelInfo```.

Notice in Classic, the API only works for "player" unit while the target is nil due to game data restrictions. If you need further info, [LibClassicCasterino](https://github.com/rgd87/LibClassicCasterino) based on combat event estimation is recommended.

```
spellId (number): The ID of the spell channelling.
target (string): The channelling target object.
startTime (number): The channelling start timestamp, in milliseconds.
endTime (number): The channelling end timestamp, in milliseconds.
```

- `castingTarget = UnitCastingTarget(unit)`

Gets the casting target object of a unit.

- `transport = UnitTransport(unit)`

Gets the transport object of a unit.

- `pitch = UnitPitch(unit)`

Gets the vertical pitch of a unit, in radian.

- `flags = UnitMovementFlags(unit)`

Gets the movement flags of a unit, indicating its moving status.

- `value = UnitMovementField(unit, offset, type)`

Gets the field value of a unit's movement struct.

- `typeId = UnitCreatureTypeId(unit)`

Gets the ID of a unit's creature type.

- `familyId = UnitCreatureFamilyId(unit)`

Gets the ID of a unit's creature family. nil if the unit does not have one.

- `value = UnitCreatureField(object, offset, type)`

Gets the field value of a unit's creature cache struct.

- `typeId, typeName = GameObjectType(object)`

Gets the type info of a game object.

```lua
-- object (string): The game object.
-- typeId (number): The type ID of the game object, starting from 1.
-- typeName (string): The type name of the game object.
```

[Back to Top](#custom-api)

## In-World (Object-General)

- `object = GetObject(object | UnitID)`

Gets the object by its ID. nil if the object does not exist.

- `object = GetObjectWithGUID(guid)`

Gets the object by its GUID.

- `flags = ObjectTypeFlags(object)`

Gets the type flags of an object.

- `isOrNot = ObjectIsType(object, type)`

Checks if an object is of specific type.

- `isOrNot = ObjectExists(object)`

Checks whether an object exists in memory.

- `id = ObjectId(object)`

Gets the ID of an object.

```lua
-- id (number): The object ID. nil if the object does not exist anymore.
```

- `x, y, z = ObjectPosition(object)`

Gets the world position of an object.

- `facing = ObjectFacing(object)`

Gets the horizontal rotation of an object, in radian.

- `distance = GetDistanceBetweenPositions(x1, y1, z1, x2, y2, z2)`

Gets the distance between two positions in 3D.

- `distance = GetDistanceBetweenObjects(object1, object2)`

Gets the distance between two objects in 3D.

- `x, y, z = GetPositionBetweenObjects(object1, object2, distance)`

Gets the position from object 1 to object 2.

- `x, y, z = GetPositionBetweenPositions(x1, y1, z1, x2, y2, z2, distance)`

Gets the position from position 1 to position 2.

- `x, y, z = GetPositionFromPosition(x1, y1, z1, distance, facing, pitch)`

Gets the position relative to a specific position.

- `facing, pitch = GetAnglesBetweenObjects(object1, object2)`

Gets the angles (facing & pitch) between two objects.

- `isOrNot = ObjectIsFacing(object1, object2[, tolerance = PI/2])`

Checks if an object is facing another object.

- `isOrNot = ObjectIsBehind(object1, object2)`

Checks if an object is behind another object.

[Back to Top](#custom-api)

## In-World (Object-Miscellanea)

- `x, y, z = GetCorpsePosition()`

Gets the position of the corpse.

- `isOrNot = UnitIsLootable(unit)`

Gets whether a unit is lootable.

- `isOrNot = UnitIsSkinnable(unit)`

Gets whether a unit is skinnable.

- `isOrNot = UnitIsMounted(unit)`

Gets whether a unit is mounted.

[Back to Top](#custom-api)

## In-World (Object Manager)

- `count, isUpdated, addedObjects, removedObjects = GetObjectCount()`

Gets the count of all world objects, as well as the changes compared to the last call. This must be called first to search and save the memory objects. (ideally once for every frame)

```lua
-- count (number): The current total count.
-- isUpdated (boolean): Whether the current world objects are changed compared to the last call.
-- addedObjects (table): The added world objects array compared to the last call.
-- removedObjects (table): The removed world objects array compared to the last call.
```

- `object = GetObjectWithIndex(index)`

Gets a specific world object by its index in the saved memory objects from the previous call to `GetObjectCount`. Notice this must be called within the same frame as the previous call to `GetObjectCount`.

- `count = GetNpcCount([center | x, y, z][, range][, rangeOption])`

Gets the count of specific NPCs, similar as `GetObjectCount` except that it is restricted to NPCs (non-player units) only.

- `npc = GetNpcWithIndex(index)`

Gets a specific NPC by its index in the saved memory objects from the previous call to `GetNpcCount`. Notice this must be called within the same frame as the previous call to `GetNpcCount`.

- `count = GetPlayerCount([center | x, y, z][, range][, rangeOption])`

Gets the count of specific players, similar as `GetObjectCount` except that it is restricted to players only.

- `player = GetPlayerWithIndex(index)`

Gets a specific player by its index in the saved memory objects from the previous call to `GetPlayerCount`. Notice this must be called within the same frame as the previous call to `GetPlayerCount`.

```lua
-- rangeOption can optionally specify the meaning of range, which is one of the several values below.
direct = 0; -- raw distance between myself and the unit
target_reach_only = 1; -- consider only the unit reach on top of the range
both_reaches = 2; -- consider both reaches of myself and the unit on top of the range
both_melee_reaches = 3; -- consider the unit as a melee target on top of the range
both_melee_reaches_with_movement = 4; -- consider the unit as a moving melee target on top of the range
```

- `count = GetGameObjectCount([center | x, y, z][, range])`

Gets the count of specific game objects, similar as `GetNpcCount` except that it is restricted to game objects only.

- `gameObject = GetGameObjectWithIndex(index)`

Gets a specific game object by its index in the saved memory objects from the previous call to `GetGameObjectCount`. Notice this must be called within the same frame as the previous call to `GetGameObjectCount`.

- `count = GetDynamicObjectCount([center | x, y, z][, range])`

Gets the count of specific dynamic objects, similar as `GetNpcCount` except that it is restricted to dynamic objects only.

- `dynamicObject = GetDynamicObjectWithIndex(index)`

Gets a specific dynamic object by its index in the saved memory objects from the previous call to `GetDynamicObjectCount`. Notice this must be called within the same frame as the previous call to `GetDynamicObjectCount`.

- `count = GetAreaTriggerCount([center | x, y, z][, range])`

Gets the count of specific area triggers, similar as `GetNpcCount` except that it is restricted to area triggers only.

- `areaTrigger = GetAreaTriggerWithIndex(index)`

Gets a specific area trigger by its index in the saved memory objects from the previous call to `GetAreaTriggerCount`. Notice this must be called within the same frame as the previous call to `GetAreaTriggerCount`.

[Back to Top](#custom-api)

## In-World (Spell)

- `false | spellId = IsAoEPending()`

Checks if there is a pending spell on the cursor.

```lua
false (boolean): There is no cursor spell pending.
spellId (number): The ID of the spell pending on cursor.
```

- `CancelPendingSpell()`

Cancels the pending spell on the cursor.

[Back to Top](#custom-api)

## In-World (State)

- `ResetAfk()`

Resets the timer for AFK.

- `account = GetCurrentAccount()`

Gets the name of the current WoW account. (same as the WTF subfolder)

- `minute, hour, weekday, day, month, year = GetCurrentTime()`

Gets the current game server time at minute precision.

[Back to Top](#custom-api)

## In-World (Vision)

- `x, y, z = TraceLine(x1, y1, z1, x2, y2, z2, flags)`

Checks ray cast intersection from position 1 to position 2.

- `x, y, z = GetCameraPosition()`

Gets the position of the camera.

- `isOnScreen, x, y = WorldToScreen(object | x, y, z)`

Projects a world position to the screen NDC position, ranging between 0.0 to 1.0 with coord origin at the bottom-left of the window client area.

This is useful to display some "overlay" WoW frame objects for the world objects. Considering the UI scaling, an example to create a text label for an world object is given below.

```lua
function create_object_label(object)
  local is_on_screen, x, y = WorldToScreen(object);
  if (is_on_screen) then
    local screen_physical_width, screen_physical_height = GetPhysicalScreenSize();
    local scale_x = 768 / GetScreenHeight() * GetScreenWidth() / (screen_physical_width * UIParent:GetEffectiveScale());
    local scale_y = 768 / (screen_physical_height * UIParent:GetEffectiveScale());
    label = CreateFrame("Frame", nil, esp_background_frame);
    label_font_string = label:CreateFontString(nil, "BORDER");
    label_font_string:SetPoint("TOPLEFT");
    label_font_string:SetJustifyH("CENTER");
    label_font_string:SetText("some words");
    label:SetWidth(label_font_string:GetStringWidth());
    label:SetHeight(label_font_string:GetStringHeight());
    label:SetPoint("BOTTOMLEFT", x * screen_physical_width * scale_x, y * screen_physical_height * scale_y);
  end
end
```

- `x, y, z[, object] = ScreenToWorld(x, y)`

Projects a screen NDC position (ranging between 0.0 to 1.0 with coord origin at the bottom-left of the window client area) to a world object or a terrain position.

This is useful to calculate a world position for the cursor position. Combined with the BLZ API "GetCursorPosition()", an example to do this is given below with UI scaling involved.

```lua
function get_cursor_world_position()
  local scale, cursor_x, cursor_y = UIParent:GetEffectiveScale(), GetCursorPosition();
  local screen_width = GetScreenWidth() * scale;
  local screen_height = GetScreenHeight() * scale;
  x, y, z = ScreenToWorld(cursor_x / screen_width, cursor_y / screen_height);
  if (x) then
      return {x, y, z};
  else
      return nil;
  end
end
```

[Back to Top](#custom-api)
