# ActionCableClient
### Swift Version 4.2

[![Version](https://img.shields.io/cocoapods/v/ActionCableClient.svg?style=flat)](http://cocoapods.org/pods/ActionCableClient)
[![License](https://img.shields.io/cocoapods/l/ActionCableClient.svg?style=flat)](http://cocoapods.org/pods/ActionCableClient)
[![Platform](https://img.shields.io/cocoapods/p/ActionCableClient.svg?style=flat)](http://cocoapods.org/pods/ActionCableClient)
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)

[ActionCable](https://github.com/rails/rails/tree/master/actioncable) is a new WebSockets server being released with Rails 5 which makes it easy to add real-time features to your app. This Swift client makes it dead-simple to connect with that server, abstracting away everything except what you need to get going.

## Installation

To install, simply:

#### Cocoapods

Add the following line to your `Podfile` and run `pod install`

```ruby
pod "ActionCableClient", :git => 'https://github.com/Kaakati/Swift-ActionCableClient.git', :branch => 'master'

```

#### Carthage

Add the following to your `Cartfile` and run `carthage update` as normal.
```ruby
github "danielrhodes/Swift-ActionCableClient"
```


## Usage

### Get Started & Connect

```swift
import ActionCableClient

self.client = ActionCableClient(url: URL(string: "ws://domain.tld/cable")!)

// Connect!
client.connect()

client.onConnected = {
    print("Connected!")
}

client.onDisconnected = {(error: Error?) in
    print("Disconnected!")
}
```

### Subscribe to a Channel

```swift
// Create the Room Channel
let roomChannel = client.create("RoomChannel") //The channel name must match the class name on the server

// More advanced usage
let room_identifier = ["room_id" : identifier]
let roomChannel = client.create("RoomChannel", identifier: room_identifier, autoSubscribe: true, bufferActions: true)

```

### Channel Callbacks

```swift

// Receive a message from the server. Typically a Dictionary.
roomChannel.onReceive = { (JSON : Any?, error : Error?) in
    print("Received", JSON, error)
}

// A channel has successfully been subscribed to.
roomChannel.onSubscribed = {
    print("Yay!")
}

// A channel was unsubscribed, either manually or from a client disconnect.
roomChannel.onUnsubscribed = {
    print("Unsubscribed")
}

// The attempt at subscribing to a channel was rejected by the server.
roomChannel.onRejected = {
    print("Rejected")
}

```

### Perform an Action on a Channel

```swift
// Send an action
roomChannel["speak"](["message": "Hello, World!"])

// Alternate less magical way:
roomChannel.action("speak", ["message": "Hello, World!"])

// Note: The `speak` action must be defined already on the server
```

### Authorization & Headers

```swift
// ActionCable can be picky about origins, so if you
// need it can be set here.
client.origin = "https://domain.tld/"

// If you need any sort of authentication, you 
// will not have cookies like you do in the browser,
// so set any headers here.
//
// These are available in the `Connection`
// on the server side.

client.headers = [
    "Authorization": "sometoken"
]
```

### Misc

```swift

client.onPing = {
    
}

```

### Codable

```swift
/**
 Returns Object from ActionCable Response

 - Parameter object: Codable object
 - Parameter JSON: ActionCable JSON response of type Any

 - Returns: Array in a Completion block.
 */
@available(iOS 11.0, *)
func serializeActionCableObject<T: Codable>(object: T.Type, JSON : Any? ,completion: ([T]) -> ()) {
    // Check if JSON with Single Object
    if let jsonResponse = JSON as? [String:Any] {
        do {
            let jsonData = try JSONSerialization.data(withJSONObject: jsonResponse, options: .sortedKeys)
            let decoder = JSONDecoder()
            let response = try decoder.decode(T.self, from: jsonData)
            completion([response])
            print("Recieved Single Object JSON")
        } catch let err {
            print("Single Object JSON Parsing Error", err)
        }
    } else if let jsonResponse = JSON as? [[String:Any]] { // JSON with Array of Objects
        do {
            let jsonData = try JSONSerialization.data(withJSONObject: jsonResponse, options: .sortedKeys)
            let decoder = JSONDecoder()
            let response = try decoder.decode([T].self, from: jsonData)
            completion(response)
            print("Recieved Multiple Objects JSON")
        } catch let err {
            print("Multiple Objects JSON Parsing Error", err)
        }
    }
}

// Useage:

// Receive a message from the server. Typically a Dictionary.
roomChannel.onReceive = { (JSON : Any?, error : Error?) in
    self.serializeActionCableObject(object: User.self, JSON: JSON, completion: { (users) in
        print(users.first?.name)
    })
}
```

For more documentation, see the [wiki](https://github.com/danielrhodes/Swift-ActionCableClient/wiki/Documentation)

## Requirements

[Starscream](https://github.com/daltoniam/Starscream): The underlying WebSocket library.

## Author

Daniel Rhodes, rhodes.daniel@gmail.com

## License

ActionCableClient is available under the MIT license. See the LICENSE file for more info.
