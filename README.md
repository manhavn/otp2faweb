# OTP 2FA Web

A Svelte/Vite web app for generating 2FA OTP codes, reading QR codes, creating editable `otpauth://` content, and exporting downloadable tag/QR files.

## Features

- Enter an `otpauth://` URI or a Base32 secret to generate a 6-digit OTP code.
- Supports time-based TOTP and counter-based HOTP.
- TOTP codes update automatically on a 30-second period.
- HOTP supports manual counter input or quick counter increments with the `+1` button.
- Shows a countdown progress bar for the current TOTP period.
- Upload a QR code image and decode its content locally, including SVG QR images.
- Scan QR codes with the device camera.
- Shows the extracted key and supports copying it.
- Lets you enter or edit issuer and account values even when the source content does not include them.
- Generates new `otpauth://totp/...` or `otpauth://hotp/...` content from the selected OTP type, extracted key, issuer, account, and HOTP counter.
- Copies the generated `otpauth://` content.
- Downloads the generated `otpauth://` content as a `.txt` file named from issuer and account.
- Exports the generated `otpauth://` content as QR code `.svg` or `.png` files named from issuer and account.
- Copies the current OTP code.
- Add, remove, and drag to reorder tags.
- Preview tag content in the `tag1|tag2|tag3` format.
- Generate tags and a filename from the editable issuer, editable account, and extracted key with the `Generate content` button.
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
6. View the extracted key and current OTP code.
7. Edit `Issuer` and `Account` in the `Generated otpauth content` section if needed.
8. Copy the generated `otpauth://` content, download it as TXT, or export it as QR SVG/PNG.
9. In the file builder, enter a tag and press `Add` or `Enter`.
10. Or press `Generate content` to create tags in the order `Issuer|Account|Extracted key` using the editable values from `Generated otpauth content`.
11. Drag tags to reorder them. The preview content updates automatically.
12. Enter a filename in the `Filename` field and press `Download`.

## Generated OTPAuth Content

The `Generated otpauth content` section builds a new OTPAuth URI from the current form state:

- OTP type: `totp` or `hotp`.
- Secret: the current extracted Base32 key.
- Label: editable `Issuer` and `Account` values.
- Query params: `secret`, `digits`, plus `period` for TOTP or `counter` for HOTP.

You can export this generated URI as:

- Plain text: `[issuer-account].txt`.
- QR SVG: `[issuer-account].svg`.
- QR PNG: `[issuer-account].png`.

If issuer and account are empty, the app falls back to `otp-auth` for generated export filenames.

## Generate Content

The `Generate content` button creates downloadable tag content from the editable OTPAuth data:

- Tag 1: editable `Issuer`, if available.
- Tag 2: editable `Account`, if available.
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
- qrcode-generator
