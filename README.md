# TACHYON-LINK │ Zero-Latency Mesh Protocol

*"Latency is an architectural error. Decoupling transmission from request."*

---

## ⚡ Project Scope

TACHYON-LINK is a sovereign communication protocol and mesh networking layer designed for Perceived Zero-Latency operations in constrained environments (LoRa/ESP32).

Traditional mesh networks suffer from high latency due to route discovery and handshake overhead (RTT). TACHYON-LINK abandons the standard "Request-Response" model in favor of a Speculative Push model. It assumes bandwidth is cheap but time is expensive, utilizing probabilistic flooding and predictive caching to deliver data to edge nodes before it is explicitly requested.

**Operational Mandate**

- **Partition Tolerance:** Designed for "Disruption-Tolerant Networking" (DTN) where nodes frequently drop offline.  
- **Security:** All packets are encrypted and authenticated. No plaintext metadata.  
- **Efficiency:** Minimizes "Chattiness" (ACKs/NACKs) to preserve airtime in shared spectrums.  

---

## 🏎️ Core Mechanics (The Speed Layer)

The protocol achieves its performance targets through three specific engineering patterns:

### 1. Zero-RTT Establishment (Noise_IK)

- **Problem:** Standard TLS/TCP handshakes require multiple round-trips before data flows.  
- **Solution:** Implements the Noise Protocol Framework (IK Pattern).  
- **Mechanism:** The initiator encrypts the payload with the receiver's static public key in the very first packet. Authentication and Data Transmission happen simultaneously.  
- **Latency Reduction:** 50-75% vs Standard Handshakes.  

### 2. Forward Error Correction (Fountain Codes)

- **Problem:** Waiting for ACKs (acknowledgments) and retransmitting lost packets creates massive jitter in lossy networks (LoRa).  
- **Solution:** Implements RaptorQ (RFC 6330) erasure coding.  
- **Mechanism:** The transmitter generates an infinite stream of encoded shards derived from the original message. The receiver only needs to capture any subset of shards (K + ε) to reconstruct the message.  
- **Result:** Unidirectional, "Fire-and-Forget" reliability. No ACKs required.  

### 3. Speculative Prefetching

- **Problem:** Waiting for a user to request a resource (e.g., a map tile) introduces fetch latency.  
- **Solution:** Integration with Upstream Prediction Engines.  
- **Mechanism:** The protocol accepts "Probabilistic Push" commands. If an upstream agent predicts a 90% chance of a node needing Resource X, Tachyon pushes Resource X to the node's local cache immediately during idle airtime.  

---

## 🛠️ Tech Stack

- **Language:** Rust (no_std compatible for bare-metal execution).  
- **Hardware Targets:**  
  - Espressif: ESP32-S3, ESP32-C3 (WiFi/ESP-NOW)  
  - Semtech: SX1262, SX1276 (LoRa PHY)  
  - Nordic: nRF52840 (BLE/802.15.4)  
- **Cryptography:**  
  - Cipher: XChaCha20-Poly1305 (Extended nonce for stateless security)  
  - Key Exchange: X25519  

---

## 📦 Usage Example

```rust
use tachyon_link::{TachyonNode, LinkConfig, Priority};
use tachyon_crypto::NoiseConfig;

fn main() {
    // Configure the radio interface
    let config = LinkConfig {
        frequency_hz: 915_000_000,
        bandwidth_khz: 500,
        spreading_factor: 7, // Optimized for speed over range
    };

    // Initialize Node with static keys
    let node = TachyonNode::new(config, NoiseConfig::load_keys());

    // Broadcast High-Priority Telemetry
    // Uses Erasure Coding to ensure delivery without waiting for ACKs
    let payload = b"SYSTEM_STATUS::CRITICAL_HEAT";
    
    node.flood_gossip(
        payload, 
        target_group="ALL_NODES", 
        priority=Priority::RealTime
    );
}
```

---

## 🔒 Security Protocol

- Metadata Protection: Packet headers are minimized and encrypted to resist traffic analysis.

- Replay Protection: Uses a sliding window bloom filter to reject replayed packets without requiring synchronized clocks.

- Perfect Forward Secrecy: Sessions (when established) rotate keys automatically based on volume thresholds.

---

## ⚖️ License

### Apache 2.0

- Permissive: Allows integration into proprietary firmware without viral effects.

- Patent Grant: Includes an explicit patent grant, protecting users from litigation regarding the mesh routing algorithms.

---
