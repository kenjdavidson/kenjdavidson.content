---
type: Blog
title: For my first IOS project, why not start with Streams!
summary:
  While developting my companies mobile application I needed to write a custom library for IOS to allow Bluetooth Classic scanning.
  Why not jump directly into Swift streams as my first (ever) IOS project.  What could go wrong?
categories: [Blog]
tags: [Swift, React Native]
---

While working on the library [react-native-bluetooth-classic](https://github.com/kenjdavidson/react-native-bluetooth-classic) I got my first taste of Swift Streams through the [ExternalAccessory](https://developer.apple.com/documentation/externalaccessory) framework. Since I've worked with streams in Java a number of times I thought this would be as straight forward, although not as straight forward, once I grasped the concept it finished itself pretty quickly.

Following the documentation on the framework, there are generally two methods for communicating with streams - which are described in the document [Polling Versus Runloop](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Streams/Articles/PollingVersusRunloop.html), so I won't spend much time repeating the content:

## Polling

As per the documentation this is not the preferred way of communicating with streams. Effectively, you just setup a thread loop that will continually ask if there is space/room and then write/read appropriately. I decided against this as:

1. It isn't the preferred method
2. It would have required me learning the specifics of Thread programming in Swift

## Stream Delegate

The resources for Swift and [Stream Delegate](https://developer.apple.com/documentation/foundation/streamdelegate) are pretty light. The only real documentation is for [Objective C](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Streams/Streams.html#//apple_ref/doc/uid/10000188-SW1) which just needed a little massaging into Swift. While going through this process I started with Swift over Objective C, this was mainly due to that I was more comfortable with the Swift syntax -

<blockquote cite="Gob Bluth">
<p> I've made a huge mistake.</p>
</blockquote>

I wish I had taken the time to power through Objective C off the bat, after getting my feet a little more wet it's not that bad and it just has too many benefits so far with React Native.

Within the `StreamDelegate.stream` function we need to setup handling of all the available `Stream.Event` [values](https://developer.apple.com/documentation/foundation/stream/event):

- `openCompleted` notifies us when the Stream has completed opening and the Input/Output streams are available
- `hasBytesAvailable` there are bytes on the InputStream
- `hasSpaceAvailable` there is room on the OutputStream for bytes. This is where there are some issues as this is only called AFTER bytes have already been sent.
- `errorOccurred` if something goes down
- `endEncountered` the stream was closed on either end

## Writing to the Stream

Handling the `Stream.Event.hasSpaceAvailable`, when space is available we want to call the `writeData`:

```swift
case .hasSpaceAvailable:
  // As per the documents, this event occurs repeatedly as long as you're writing data
  // in the examples I've found they just assume the initial write will work (using
  // a smaller value) and then continues on doing so.
  NSLog("Stream %@ has space available", aStream)
  writeData()
  break;
```

Once the delgate is setup, you can send some data - in this case though, sending data isn't just sending data. If you attempt to loop through the data and write to the stream, you'll get a bunch of funky threading issues and data errors in the buffer. Essentially what you need to do is:

- Send some data to a buffer
- Then call `writeData` to process the information

#### Write Data

```swift
private func writeData() {
  if (outBuffer.isEmpty) {
      NSLog("(BluetoothDevice:writeData) No buffer data scheduled for deliver")
      return
  }

  let len:Int = (outBuffer.count > maxBytesPerSend) ? maxBytesPerSend : outBuffer.count
  NSLog("(BluetoothDevice:writeData) Attempting to send %d bytes to the device", len)

  let buffer:UnsafeMutablePointer<UInt8> = UnsafeMutablePointer.allocate(capacity: len)
  outBuffer.copyBytes(to: buffer, count: len)
  outBuffer.removeFirst(len)

  let bytesWritten = session?.outputStream!.write(buffer, maxLength: len) ?? 0
  NSLog("(BluetoothDevice:writeData) Sent %d bytes to the device", bytesWritten)
}
```

This is where things go shady (in my mind) in that you can't just write to the stream. You can't just write it to the stream then remove it from the buffer (as when the write happens the delegate will be called) so you need to:

- Removed the assumed amount of data from the start of the buffer
- `writeData` needs to pull out the data it's going to send and write it.
- End gracefully

When this happens you'll fall into a nice little loop of writing, ending gracefully and letting the stream delegate be called repeatedly until no more room is left. This works well for stopping as well, when the buffer is emptied and no data is written, it stops the `hasSpaceAvailable` looping:

```swift
func writeToDevice(_ message:String) {
  NSLog("(BluetoothDevice:writeToDevice) Writing %@ to device %@", message, accessory.serialNumber)
  if let sending = message.data(using: .utf8) {
      outBuffer.append(sending)
  }

  // If there is space available for writing then we want to kick off the process.
  // If all the data cannot be fully written, then the hasSpaceAvailable will be
  // fired and we can continue.  In most cases, we shouldn't be sending that much
  // data.
  writeData()
}
```

I still don't think I'm managing the threads properly, but when I attempt to change the thread on which the delegate is run no data comes back. I'd love to hear if I'm missing something, this works, but it just doesn't feel right.

## Reading from the Stream

In order to manage reading from the stream we need to handle `Stream.Event.hasBytesAvailable`

```swift
case .hasBytesAvailable:
  NSLog("Stream %@ has bytes available", aStream)
  readData()
  break;
```

where `readData` pulls as many bytes as possible from the `InputStream`. There aren't too many issues with this, as the Stream starts the process of reading (unlike the write method) and this will continue whenever new data is available.

```swift
private func readData() {
  // Create the buffer that we'll read into
  let buffer = UnsafeMutablePointer<UInt8>.allocate(capacity: maxBytesPerReceive)
  let numBytesRead = session?.inputStream!.read(buffer, maxLength: maxBytesPerReceive) ?? 0

  if (numBytesRead <= 0) {
      NSLog("(BluetoothDevice:readData) No buffer")
      return
  }

  // If there is a receiveDelegate then let them deal with the data and update with the remaining
  inBuffer.append(buffer, count: numBytesRead)
  if let bd = receivedDelegate {
      inBuffer = bd.onReceivedData(fromDevice: self, receivedData: inBuffer)
  }
}
```

All in all, this could probably be documented better - when I get a chance I'll come back and try to re-write it - but for now I'm hoping it will help someone (if not just me).
