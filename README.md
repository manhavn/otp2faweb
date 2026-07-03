# OTP 2FA Web

A Svelte/Vite web app for generating 2FA OTP codes, reading QR codes, and creating downloadable content files from tag lists.

## Features

- Enter an `otpauth://` URI or a Base32 secret to generate a 6-digit OTP code.
- Supports time-based TOTP and counter-based HOTP.
- TOTP codes update automatically on a 30-second period.
- HOTP supports manual counter input or quick counter increments with the `+1` button.
- Shows a countdown progress bar for the current TOTP period.
- Upload a QR code image and decode its content locally.
- Scan QR codes with the device camera.
- Shows the extracted key and supports copying it.
- Shows issuer and account metadata, with copy buttons for each field when available.
- Copies the current OTP code.
- Add, remove, and drag to reorder tags.
- Preview tag content in the `tag1|tag2|tag3` format.
- Generate tags and a filename from issuer, account, and extracted key with the `Generate content` button.
- Enter a filename and download a file containing the preview content.
- Sanitizes filenames automatically, for example `otp|GoogleLogin|user@gmail.com.txt` becomes `otp-GoogleLogin-user-gmail.com.txt`.
- English is the default UI language, with an EN/VI switcher available in the interface.

## Installation

```bash
npm install
```

## Development

```bash
npm run dev
```

Then open the Vite URL shown in the terminal, usually:

```text
http://localhost:5173
```

## Check

```bash
npm run check
```

## Production Build

```bash
npm run build
```

## Preview Build

```bash
npm run preview
```

## Usage

1. Paste a Base32 secret or an `otpauth://` URI into the `Content` field.
2. Or choose a QR code image with the image picker.
3. Or press `Camera` to scan a QR code directly from the device.
4. Choose the OTP type: `Time-based` for TOTP or `Counter-based` for HOTP.
5. If using HOTP, enter the `Counter` value or press `+1` to increment it.
6. View the extracted key, issuer, account, and current OTP code.
7. In the file builder, enter a tag and press `Add` or `Enter`.
8. Or press `Generate content` to create tags in the order `Issuer|Account|Extracted key`.
9. Drag tags to reorder them. The preview content updates automatically.
10. Enter a filename in the `Filename` field and press `Download`.

## Generate Content

The `Generate content` button creates downloadable content from the current OTP data:

- Tag 1: `Issuer`, if available.
- Tag 2: `Account`, if available.
- Tag 3: `Extracted key`.

The filename is generated with this pattern:

```text
otp-tags[-issuer][-account].txt
```

Example:

```text
otp-tags-Google-user-gmail.com.txt
```

## Notes

- All OTP, QR, and file-generation processing runs in the browser.
- Secrets and keys are not sent to a server.
- Camera scanning requires `localhost` or HTTPS so the browser can grant camera access.
- Web Crypto API requires a modern browser environment.

## Technology

- Svelte 5
- Vite
- TypeScript
- Web Crypto API
- jsQR
