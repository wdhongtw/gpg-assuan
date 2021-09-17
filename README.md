# JavaScript Assuan Protocol Library

Assuan protocol is designed and used in GnuPG toolchain, mainly for IPC between
`gpg` and `gpg-agent` processes.

This library provides the low level client side (and possibly server side) class
for easier communication with GnuPG tools.

## Related Links

- Official Assuan documentation: [Developing with Assuan](https://www.gnupg.org/documentation/manuals/assuan/index.html)
- Communicate with GPG agent through Assuan: [Agent Protocol](https://www.gnupg.org/documentation/manuals/gnupg/Agent-Protocol.html)


## Sample Usage

```javascript
const assuan = require('gpg-assuan')

async function main() {
    // Inject console for debugging log, mock it if no logging is required.
    const agent = new assuan.AssuanClient(console, process.env.HOME + '/.gnupg/S.gpg-agent')
    await agent.initialize()

    // GPG agent always response a OK once the connection is established.
    const greetingResponse = await agent.receiveResponse()
    greetingResponse.checkType(assuan.ResponseType.Ok)

    // Send command to check the version of tne GPG agent.
    await agent.sendRequest(assuan.Request.fromCommand(new assuan.RequestCommand('GETINFO', 'version')))
    const response = await agent.receiveResponse()
    response.checkType(assuan.ResponseType.RawData)
    const dataResponse = response.toRawData()
    const version = dataResponse.bytes.toString('utf8')
    console.info("Server version:", version)
    const okResponse = await agent.receiveResponse()
    okResponse.checkType(assuan.ResponseType.Ok)

    // Say BYE to close the session.
    await agent.sendRequest(assuan.Request.fromCommand(new assuan.RequestCommand('BYE')))
    const okClose = await agent.receiveResponse()
    okClose.checkType(assuan.ResponseType.Ok)
}

main()
```

Outputs:

```
Recv: OK Pleased to meet you, process 15843
Send: GETINFO version
Recv: D 2.2.19
Recv: OK
Server version: 2.2.19
Send: BYE
Recv: OK closing connection
```
