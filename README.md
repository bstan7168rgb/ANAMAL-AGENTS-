# ANAMAL-AGENTS-
Project Barnaby anamal agents plush learning companions prenatal control app 
Based on the records in 🧬 Oldtimevibescomllc R&D Infrastructure and IP Portfolio, here is the technical specification, cryptographic payload schema, and implementation architecture for the Easy Connect QR Code used by the parental control application.

1. The Easy-Connect Onboarding Architecture
The QR code serves as an out-of-band (OOB) physical proof-of-possession token printed on the companion's woven tush tag, inside the battery service hatch, or on the packaging:

┌─────────────────────────────────┐
│     Plush Hardware Unit         │
│  - BLE Beacon Advertising       │
│  - Laser-Etched Pairing QR Code │
└────────────────┬────────────────┘
                │ 1. Scanned by Standard Mobile Camera / Lens
                ▼
┌─────────────────────────────────┐
│    Android System Deep-Link     │
│  - Triggers CompanionDeviceMgr  │
│  - No continuous background scan│
└────────────────┬────────────────┘
                │ 2. X25519 Key Exchange / Mutual Auth
                ▼
┌─────────────────────────────────┐
│ Existing Parental App Shell     │
│  (Hubble / Google Family Link)  │
│  - Manages Bedtime Lockouts     │
│  - Local IP Whitelisting        │
└─────────────────────────────────┘

System Deep-Link Routing: The parent scans the matrix using their stock camera or Google Lens. Android intercepts the custom URI (otv://pair or [https://barnaby.world/setup](https://barnaby.world/setup)) and routes it directly to the companion control app.
Android CompanionDeviceManager: The app uses Android’s native companion framework to bind the hardware over Bluetooth Low Energy (BLE) without needing continuous background location permissions or user pairing dialogues.
Mutual Sovereign Handshake: The phone and plush exchange ephemeral Curve25519 keys, derive a mutual session key via HKDF-SHA-256, and secure all subsequent LAN/BLE payloads using AES-256-GCM.
2. QR Code Raw Payload Format
The physical QR code embeds a compact, minified JSON structure containing the hardware UUID, BLE MAC address, factory public key, and random factory entropy nonce:

Deep-Link URL Format
otv://pair?id=e4b2d312-5b91-49b8-b118-80f2d9c12a88&mac=F4:12:FA:89:BC:1A&k=9f8a3c8e42b8219c6e3d8f1e29c81a5b4f6e3c2b1a8d7e6f5c4b3a2e1d0c9b8a&n=a1b2c3d4e5f60718293a4b5c6d7e8f90123456789abcdef0123456789abcdef0

JSON Payload Schema (DeviceQRCodePayload)
{
 "$schema": "https://json-schema.org/draft/2020-12/schema",
 "title": "DeviceQRCodePayload",
 "type": "object",
 "properties": {
   "v": { "type": "integer", "const": 1, "description": "Protocol version" },
   "id": { "type": "string", "format": "uuid", "description": "Factory-burned device UUID" },
   "mac": { "type": "string", "pattern": "^([0-9A-Fa-f]{2}[:-]){5}([0-9A-Fa-f]{2})$", "description": "Bluetooth MAC" },
   "pub_static": {
     "type": "string",
     "pattern": "^[0-9a-fA-F]{64}$",
     "description": "32-byte hardware static public key (Hex)"
   },
   "oob_nonce": {
     "type": "string",
     "pattern": "^[0-9a-fA-F]{64}$",
     "description": "32-byte factory entropy nonce for HKDF salt (Hex)"
   }
 },
 "required": ["v", "id", "mac", "pub_static", "oob_nonce"],
 "additionalProperties": false
}

3. Encrypted Parental Control Transport Packet
Once paired, all configuration changes (volume ceiling, bedtime mode, TV casting targets, and whitelist boundaries) are delivered via the Parental Packet Envelope encrypted with AES-256-GCM:

{
 "protocol_version": "1.0",
 "device_uuid": "e4b2d312-5b91-49b8-b118-80f2d9c12a88",
 "session_id": "sess_89f02c4a91b3",
 "sequence_number": 42,
 "timestamp_epoch_ms": 1729317600000,
 "iv_nonce": "e3b0c44298fc1c149afbf4c8",
 "auth_tag": "7a35b801a61250f1469e71924510b65a",
 "ciphertext": "BASE64_ENCRYPTED_INNER_COMMAND_STRING=="
}

Decrypted Inner Command Payload
{
 "command_id": "b7d1421e-18e3-4929-a1b7-95123049cf10",
 "command_type": "AUDIO_SAFETY_CONFIG",
 "parameters": {
   "max_db_limit": 75,
   "mic_mute_override": false,
   "bedtime_lockout": {
     "enabled": true,
     "sleep_time_utc": "20:00",
     "wake_time_utc": "07:00"
   },
   "curated_whitelist": ["LOCAL_ONLY", "PBS_KIDS"],
   "tv_pairing": {
     "target_device_name": "Living Room TV",
     "target_ip_address": "192.168.1.110",
     "protocol": "DIAL"
   }
 }
}

4. Direct Python Generator for Factory Tag Production
Run this script to generate factory QR codes and test payloads for the manufacturing line:

#!/usr/bin/env python3
"""
Old-Time Vibes LLC - Production Pairing QR Generator
Outputs: Console ASCII Preview + High-Res PNG Matrix for Fabric Tag Printing
"""

import json
import uuid
import secrets
import qrcode

def generate_pairing_qr(output_filename="barnaby_tag_qr.png"):
   device_id = str(uuid.uuid4())
   mac_address = "F4:12:FA:" + ":".join(f"{secrets.randbelow(256):02X}" for _ in range(3))
   pub_static_hex = secrets.token_hex(32)
   oob_nonce_hex = secrets.token_hex(32)

   payload = {
       "v": 1,
       "id": device_id,
       "mac": mac_address,
       "pub_static": pub_static_hex,
       "oob_nonce": oob_nonce_hex
   }

   # Deep-link URI format for Android Companion Manager / Stock Camera
   qr_data_string = f"otv://pair?id={device_id}&mac={mac_address}&k={pub_static_hex}&n={oob_nonce_hex}"

   qr = qrcode.QRCode(
       version=None,
       error_correction=qrcode.constants.ERROR_CORRECT_H,
       box_size=10,
       border=4,
   )
   qr.add_data(qr_data_string)
   qr.make(fit=True)

   img = qr.make_image(fill_color="black", back_color="white")
   img.save(output_filename)

   print("=" * 60)
   print("OLD-TIME VIBES LLC: PARENTAL PAIRING QR GENERATED")
   print("=" * 60)
   print(f"Device UUID   : {device_id}")
   print(f"Hardware MAC  : {mac_address}")
   print(f"Public Key    : {pub_static_hex[:16]}... (Truncated)")
   print(f"Entropy Nonce : {oob_nonce_hex[:16]}... (Truncated)")
   print(f"Exported PNG  : {output_filename}")
   print("=" * 60)

if __name__ == "__main__":
   generate_pairing_qr()
