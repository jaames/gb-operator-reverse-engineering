Reverse engineering notes for the [GB Operator](https://www.epilogue.co/product/gb-operator) by Epilogue. I've only spent a couple of hours poking at it so far, so this is far from complete.

## USB

The GB Operator's USB vendor ID is `7504`, and its product is `24600`. It communicates with the Operator app using USB bulk IN and OUT transfers. 

A 64-byte command is sent to the device, and then the device responds with one or more transfers which have a maximum size of 64 bytes each. The first byte of the command packet indicates the command type:

### Type `0x00` - Read Game

The device will respond with the cart's ROM data spread over a series of bulk IN transfers. Not entirely sure how it knows how long the game data is. The command packet seems to contain other parameters, but I haven't been able to figure out what these are yet.

### Type `0x01` - Write Game

I don't have a writable cart to test this yet.

### Type `0x02` - Read Save

The device will respond with the cart's save data spread over a series of bulk IN transfers. Not entirely sure how it knows how long the save data is. The command packet seems to contain other parameters, but I haven't been able to figure out what these are yet.

### Type `0x03` - Write Save

Sends new save data to the cart, spread over a series of bulk OUT transfers. The command packet seems to contain other parameters, but I haven't been able to figure out what these are yet.

### Type `0x04` - Read Cart Signature

The device will respond with details about the currently inserted cartridge. When idle, the Operator app sends this command regularly to check which cart is currently inserted. The command packet doesn't contain any other parameters.

I haven't worked out the response yet, however I'm fairly sure the 3 bytes at 0x10 represent the cart's game code, and the byte at 0x3C might be the cart's country/language code.

### Type `0x05` - Enter DFU Mode

Puts the device's STM32 into DFU mode, for firmware updates.

## Server API

The Operator app communicates with a minimal API to check for app and firmware updates, the base path for this API is `https://www.epilogue.co/api`.

### GET `/update`

#### Query Params

| Param | Value |
|:-|:-|
| `v` | Operator app version |
| `fv` | Device firmware version |
| `o` | Operating System; `osx`, `windows` or `linux` |
| `s` | Device serial number |

#### Response

Responds with a JSON containing the latest available app versions for each OS, and the latest available firmware version for each device.

### GET `/firmware/operator/gb/download`

#### Query Params

| Param | Value |
|:-|:-|
| `v` | Operator app version |
| `fv` | Device firmware version |
| `o` | Operating System; `osx`, `windows` or `linux` |
| `s` | Device serial number |

#### Response

Responds with the most recent firmware data for the device.
