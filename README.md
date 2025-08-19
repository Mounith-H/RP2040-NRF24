# RP2040-NRF24 Library

An advanced, full-featured NRF24L01/NRF24L01+ C++ library for Raspberry Pi Pico (RP2040) microcontroller with comprehensive features, excellent performance, and robust error handling.

## 🚀 Features

- **High Performance**: SPI communication up to 8MHz
- **Multi-pipe Support**: Up to 6 receiving pipes with individual configuration
- **Variable Data Rates**: 250kbps, 1Mbps, and 2Mbps
- **Flexible Power Control**: -18dBm to 0dBm power levels
- **Dynamic Payload**: 1-32 bytes with dynamic length support
- **Auto-Acknowledgment**: Configurable with retry mechanism
- **CRC Protection**: Disabled, 8-bit, or 16-bit CRC options
- **Advanced Diagnostics**: Packet statistics, carrier detection, signal monitoring
- **Comprehensive Error Handling**: Timeouts, connection verification, status monitoring
- **Backward Compatibility**: Works with existing NRF24 code
- **Test Modes**: Carrier wave generation and diagnostic modes

## 📋 Requirements

- **Hardware**: Raspberry Pi Pico (RP2040)
- **SDK**: Pico SDK
- **Module**: NRF24L01 or NRF24L01+ transceiver
- **Compiler**: GCC with C++11 support

## 🔌 Hardware Connections

| NRF24L01 Pin | RP2040 Pin | Description |
|--------------|------------|-------------|
| VCC | 3.3V | Power supply |
| GND | GND | Ground |
| CE | GPIO 6 | Chip Enable |
| CSN | GPIO 5 | SPI Chip Select |
| SCK | GPIO 2 | SPI Clock |
| MOSI | GPIO 3 | SPI Master Out |
| MISO | GPIO 4 | SPI Master In |
| IRQ | GPIO 7 | Interrupt (optional) |

## 🚀 Quick Start

### Basic Transmitter
```cpp
#include "pico/stdlib.h"
#include "NRF24.h"

int main() {
    stdio_init_all();
    
    // Initialize NRF24 (spi, sck, mosi, miso, csn, ce, irq)
    NRF24 nrf(spi0, 2, 3, 4, 5, 6, 7);
    
    if (!nrf.begin()) {
        printf("NRF24 initialization failed!\n");
        return -1;
    }
    
    // Configure for transmission
    uint8_t tx_address[] = {0xE7, 0xE7, 0xE7, 0xE7, 0xE7};
    nrf.openWritingPipe(tx_address);
    nrf.setChannel(76);
    nrf.setDataRate(NRF24_DATA_RATE_1MBPS);
    nrf.setPowerLevel(NRF24_POWER_LEVEL_0DBM);
    
    while (true) {
        uint8_t data[] = "Hello World!";
        if (nrf.write(data, sizeof(data))) {
            printf("Message sent successfully\n");
        } else {
            printf("Message failed to send\n");
        }
        sleep_ms(1000);
    }
}
```

### Basic Receiver
```cpp
#include "pico/stdlib.h"
#include "NRF24.h"

int main() {
    stdio_init_all();
    
    // Initialize NRF24
    NRF24 nrf(spi0, 2, 3, 4, 5, 6, 7);
    
    if (!nrf.begin()) {
        printf("NRF24 initialization failed!\n");
        return -1;
    }
    
    // Configure for reception
    uint8_t rx_address[] = {0xE7, 0xE7, 0xE7, 0xE7, 0xE7};
    nrf.openReadingPipe(0, rx_address);
    nrf.setChannel(76);
    nrf.setDataRate(NRF24_DATA_RATE_1MBPS);
    nrf.startListening();
    
    while (true) {
        if (nrf.available()) {
            uint8_t buffer[32];
            uint8_t len = nrf.read(buffer, sizeof(buffer));
            buffer[len] = '\0';
            printf("Received: %s\n", buffer);
        }
        sleep_ms(10);
    }
}
```

## 📚 Advanced Usage

### Multi-pipe Reception
```cpp
// Open multiple pipes with different addresses
uint8_t pipe0_addr[] = {0xE7, 0xE7, 0xE7, 0xE7, 0xE7};
uint8_t pipe1_addr[] = {0xC2, 0xC2, 0xC2, 0xC2, 0xC2};

nrf.openReadingPipe(0, pipe0_addr);
nrf.openReadingPipe(1, pipe1_addr);

// Check which pipe received data
uint8_t pipe_num;
if (nrf.available(&pipe_num)) {
    printf("Data received on pipe %d\n", pipe_num);
}
```

### Dynamic Payload
```cpp
// Enable dynamic payload
nrf.enableDynamicPayloads();
nrf.enableDynamicPayload(0);

// Send variable length data
uint8_t short_msg[] = "Hi";
uint8_t long_msg[] = "This is a longer message";

nrf.write(short_msg, strlen((char*)short_msg));
nrf.write(long_msg, strlen((char*)long_msg));
```

### Power Management
```cpp
// Set different power levels
nrf.setPowerLevel(NRF24_POWER_LEVEL_NEG18DBM);  // Minimum power
nrf.setPowerLevel(NRF24_POWER_LEVEL_0DBM);      // Maximum power

// Power down when not in use
nrf.powerDown();
sleep_ms(1000);
nrf.powerUp();
```

## 📊 Diagnostics and Monitoring

```cpp
// Get transmission statistics
printf("Packets sent: %d\n", nrf.getPacketsSent());
printf("Packets lost: %d\n", nrf.getPacketsLost());
printf("Retransmit count: %d\n", nrf.getRetransmitCount());

// Check connection status
if (nrf.isConnected()) {
    printf("NRF24 module is connected\n");
}

// Monitor signal quality
if (nrf.testCarrier()) {
    printf("Carrier detected\n");
}

// Print detailed module information
nrf.printDetails();
```

## 🛠️ API Reference

### Initialization
- `NRF24(spi_inst_t *spi, uint16_t sck, uint16_t mosi, uint16_t miso, uint16_t csn, uint16_t ce, uint16_t irq)`
- `bool begin()`
- `bool isConnected()`
- `void reset()`

### Configuration
- `void setChannel(uint8_t channel)` - Set RF channel (0-125)
- `void setDataRate(NRF24_DataRate rate)` - Set data rate
- `void setPowerLevel(NRF24_PowerLevel level)` - Set transmission power
- `void setAddressWidth(NRF24_AddressWidth width)` - Set address width
- `void setCRCLength(NRF24_CRCLength length)` - Set CRC length

### Pipe Management
- `void openReadingPipe(uint8_t pipe, uint8_t *address)` - Open receiving pipe
- `void openWritingPipe(uint8_t *address)` - Set transmission address
- `void setPayloadSize(uint8_t size)` - Set payload size
- `void setAutoAck(bool enable)` - Enable/disable auto-acknowledgment

### Data Transmission
- `bool write(uint8_t *data, uint8_t len)` - Send data
- `bool writeBlocking(uint8_t *data, uint8_t len, uint32_t timeout_ms)` - Send with timeout
- `void startListening()` - Enter receive mode
- `void stopListening()` - Exit receive mode

### Data Reception
- `bool available()` - Check if data is available
- `uint8_t read(uint8_t *data, uint8_t len)` - Read received data
- `uint8_t getDynamicPayloadSize()` - Get dynamic payload size

## 🔧 Configuration Options

### Data Rates
- `NRF24_DATA_RATE_250KBPS` - 250 kbps
- `NRF24_DATA_RATE_1MBPS` - 1 Mbps  
- `NRF24_DATA_RATE_2MBPS` - 2 Mbps

### Power Levels
- `NRF24_POWER_LEVEL_NEG18DBM` - -18 dBm
- `NRF24_POWER_LEVEL_NEG12DBM` - -12 dBm
- `NRF24_POWER_LEVEL_NEG6DBM` - -6 dBm
- `NRF24_POWER_LEVEL_0DBM` - 0 dBm

### CRC Options
- `NRF24_CRC_DISABLED` - No CRC
- `NRF24_CRC_8BIT` - 8-bit CRC
- `NRF24_CRC_16BIT` - 16-bit CRC

## 🐛 Troubleshooting

### Common Issues

1. **Module not detected**: Check wiring and power supply
2. **Poor range**: Increase power level, check antenna
3. **Data corruption**: Enable CRC, check for interference
4. **High packet loss**: Adjust retry settings, check channel

### Debug Information
```cpp
// Print detailed module status
nrf.printDetails();

// Check specific registers
uint8_t status = nrf.getStatus();
printf("Status register: 0x%02X\n", status);
```

## 📁 File Structure

```
RP2040-NRF24/
├── NRF24.h              # Library header file
├── NRF24.cpp            # Library implementation
├── README.md            # This file
├── IMPLEMENTATION_SUMMARY.md  # Implementation details
├── CONFIG_MATCH.md      # Configuration guide
└── TROUBLESHOOTING.md   # Troubleshooting guide
```

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Nordic Semiconductor for the NRF24L01 documentation
- Raspberry Pi Foundation for the excellent Pico SDK
- Community contributors and testers

## 📞 Support

If you encounter any issues or have questions:
1. Check the [TROUBLESHOOTING.md](TROUBLESHOOTING.md) file
2. Review the [CONFIG_MATCH.md](CONFIG_MATCH.md) for configuration help
3. Open an issue on GitHub

---

**Made with ❤️ for the RP2040 community**
