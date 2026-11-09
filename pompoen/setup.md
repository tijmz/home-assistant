# Introduction

A talking, light-up Halloween pumpkin. When someone walks up to it, a presence radar triggers Home Assistant (HA), which plays a random grumpy line in an ElevenLabs voice and switches on a flickering candle effect on an LED strip inside the pumpkin. The pumpkin complains about being woken up, with a curse as the opening move.

It runs on an M5Stack Atom Echo with ESPHome. The Echo's built-in speaker is too quiet, so for now the voice plays on another speaker in the house. The lines are fixed (pregenerated and cached by HA's TTS), which avoids the delay of calling a language model on every visit. A plain ESP32 board would work as well, since the Atom Echo's speaker and microphone are not used.

Everything is built without soldering (Wago connectors and jumper wires).

# Preparation

## Hardware
- M5Stack Atom Echo
- HLK-LD2410C presence radar
- WS2812B LED strip, 5 V (55 pixels in my case)
- 330 ohm resistor
- Wago 221 connectors, jumper wires (male-to-female and female-to-female)
- USB-C cable and a 5 V / 2 A charger (55 LEDs at about 30% orange draw roughly 0.6 to 0.8 A)
- Pumpkin, cardboard and baking paper

## Software
- HA with the ESPHome integration and the ESPHome add-on
- ElevenLabs integration in HA (set your voice as the default voice of the integration)
- A media player in HA that can play TTS, in my case a Music Assistant speaker
- Two `input_datetime` helpers (time only): `pompoen_actief_vanaf` and `pompoen_actief_tot`, which define the hours in which the pumpkin is allowed to speak

# Building

## Wiring

5 V and GND from the Echo go to two Wago connectors, which feed the radar and the LED strip.

| From | To | Note |
| :-- | :-- | :-- |
| Echo 5V | Wago A (5 V) | Radar and strip hang on this |
| Echo GND | Wago B (GND) | All grounds together |
| LD2410C VCC | Wago A (5 V) | |
| LD2410C GND | Wago B (GND) | |
| LD2410C OUT | Echo G25 | High means someone is present |
| LED strip 5 V (red) | Wago A (5 V) | |
| LED strip GND (white) | Wago B (GND) | |
| LED strip DIN | 330 ohm resistor, then Echo G21 | Resistor close to the strip |

The arrows on the strip point in the direction of the data. Connect the Echo to the end the arrows point away from (DIN).

A loose OUT jumper made the radar look permanently occupied. Press it in firmly and tape it.

## Flashing
Add the contents of `pompoen-echo.yaml` to ESPHome. Add `wifi_ssid`, `wifi_password`, `wifi_ssid_2` and `wifi_password_2` to `secrets.yaml` (remove the second network if you only have one). Flash the first time over USB using ESPHome Web, after that wirelessly. Set `num_leds` to the number of pixels you use.

## Home Assistant
Create the two helpers, then add the two automations:

- `automation_pompoen_greeting.yaml`: plays a random line and switches the strip on `light_delay` seconds later, in parallel with the audio so the light matches the first audible word. Replace `YOUR_ELEVENLABS_VOICE_ID`, and change the entity names to yours.
- `automation_pompoen_strip_off.yaml`: switches the strip off when the radar has been clear for 10 seconds.

Notes from building this:

- Use `media_player.play_media` with a `media-source://tts/...` URL rather than `tts.speak`, because `tts.speak` plays an announcement chime on some speakers.
- A Music Assistant speaker took about 5 seconds to start playing, which is why the light is delayed with `light_delay`.
- The voice only changed reliably after I set it as the default voice of the ElevenLabs integration. Passing it only in the URL did not always work.
- Some words are pronounced poorly by the TTS (I had to drop the word "beven" and replace "lauw"). Listen to every line.
- After saving an automation through the API it was sometimes left disabled, so check that it is on.

## Finishing the pumpkin
- Keep the LED strip on cardboard, away from the wet flesh. Baking paper makes a good diffuser, and a layer directly against the pumpkin keeps moisture off.
- Keep the Echo outside the pumpkin. Wrap the radar in plastic film, it still sees through it.
- A hollowed-out pumpkin starts to collapse after a few days. Take the electronics out in time and dry them.

# Not done (yet)
- Speaker on the Echo itself with a MAX98357A amplifier and a 4 ohm speaker. Pins would be G19 (BCLK), G33 (LRC) and G22 (DIN), with 5 V and GND from the Wago connectors. Untested.
- Wake word and conversation: the Echo cannot do on-device wake word detection (that needs an ESP32-S3), and I decided against a question-and-answer setup.
