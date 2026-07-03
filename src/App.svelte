<script lang="ts">
  import jsQR from 'jsqr'

  const STEP_SECONDS = 30
  const DIGITS = 6

  let secretInput = $state('')
  let code = $state('------')
  let error = $state('')
  let qrError = $state('')
  let qrPreview = $state('')
  let copied = $state(false)
  let copiedKey = $state(false)
  let now = $state(Date.now())

  const secondsRemaining = $derived(STEP_SECONDS - (Math.floor(now / 1000) % STEP_SECONDS))
  const progress = $derived((secondsRemaining / STEP_SECONDS) * 100)
  const parsedSecret = $derived(parseSecret(secretInput))
  const formattedCode = $derived(code === '------' ? code : `${code.slice(0, 3)} ${code.slice(3)}`)

  $effect(() => {
    const timer = window.setInterval(() => {
      now = Date.now()
    }, 1000)

    return () => window.clearInterval(timer)
  })

  $effect(() => {
    secretInput
    now
    copied = false
    copiedKey = false
    void generateCode()
  })

  function parseSecret(value: string) {
    const trimmed = value.trim()

    if (!trimmed) return { secret: '', issuer: '', account: '' }

    if (!trimmed.toLowerCase().startsWith('otpauth://')) {
      return { secret: trimmed.replace(/\s|-/g, '').toUpperCase(), issuer: '', account: '' }
    }

    try {
      const uri = new URL(trimmed)
      const label = decodeURIComponent(uri.pathname.replace(/^\//, ''))
      return {
        secret: (uri.searchParams.get('secret') ?? '').replace(/\s|-/g, '').toUpperCase(),
        issuer: uri.searchParams.get('issuer') ?? '',
        account: label.includes(':') ? label.split(':').slice(1).join(':') : label,
      }
    } catch {
      return { secret: '', issuer: '', account: '' }
    }
  }

  function base32ToBytes(base32: string) {
    const alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ234567'
    const clean = base32.replace(/=+$/g, '')
    const bytes: number[] = []
    let bits = 0
    let value = 0

    for (const char of clean) {
      const index = alphabet.indexOf(char)
      if (index === -1) throw new Error('Khóa chỉ được chứa ký tự Base32 A-Z và 2-7.')

      value = (value << 5) | index
      bits += 5

      if (bits >= 8) {
        bytes.push((value >>> (bits - 8)) & 255)
        bits -= 8
      }
    }

    return new Uint8Array(bytes)
  }

  function counterToBytes(counter: number) {
    const bytes = new ArrayBuffer(8)
    const view = new DataView(bytes)
    view.setUint32(4, counter)
    return bytes
  }

  async function generateCode() {
    if (!secretInput.trim()) {
      code = '------'
      error = ''
      return
    }

    try {
      const keyBytes = base32ToBytes(parsedSecret.secret)
      if (keyBytes.length === 0) throw new Error('Không tìm thấy khóa bí mật hợp lệ.')

      const key = await crypto.subtle.importKey(
        'raw',
        keyBytes,
        { name: 'HMAC', hash: 'SHA-1' },
        false,
        ['sign'],
      )
      const counter = Math.floor(now / 1000 / STEP_SECONDS)
      const hmac = new Uint8Array(await crypto.subtle.sign('HMAC', key, counterToBytes(counter)))
      const offset = hmac[hmac.length - 1] & 0x0f
      const binary =
        ((hmac[offset] & 0x7f) << 24) |
        ((hmac[offset + 1] & 0xff) << 16) |
        ((hmac[offset + 2] & 0xff) << 8) |
        (hmac[offset + 3] & 0xff)

      code = String(binary % 10 ** DIGITS).padStart(DIGITS, '0')
      error = ''
    } catch (err) {
      code = '------'
      error = err instanceof Error ? err.message : 'Không thể tạo mã OTP từ khóa này.'
    }
  }

  async function copyCode() {
    if (code === '------') return

    await navigator.clipboard.writeText(code)
    copied = true
  }

  async function copyKey() {
    if (!parsedSecret.secret) return

    await navigator.clipboard.writeText(parsedSecret.secret)
    copiedKey = true
  }

  async function decodeQrImage(event: Event) {
    const input = event.currentTarget as HTMLInputElement
    const file = input.files?.[0]

    qrError = ''
    if (!file) return

    if (qrPreview) URL.revokeObjectURL(qrPreview)
    qrPreview = URL.createObjectURL(file)

    try {
      const bitmap = await createImageBitmap(file)
      const canvas = document.createElement('canvas')
      const context = canvas.getContext('2d', { willReadFrequently: true })

      if (!context) throw new Error('Trình duyệt không hỗ trợ đọc ảnh QR.')

      canvas.width = bitmap.width
      canvas.height = bitmap.height
      context.drawImage(bitmap, 0, 0)
      bitmap.close()

      const imageData = context.getImageData(0, 0, canvas.width, canvas.height)
      const result = jsQR(imageData.data, imageData.width, imageData.height)

      if (!result?.data) throw new Error('Không tìm thấy mã QR trong ảnh này.')

      secretInput = result.data
    } catch (err) {
      qrError = err instanceof Error ? err.message : 'Không thể đọc mã QR từ ảnh này.'
    } finally {
      input.value = ''
    }
  }
</script>

<main class="shell">
  <aside class="side-column">
    <section class="hero-card" aria-labelledby="title">
      <p class="eyebrow">TOTP 2FA Generator</p>
      <h1 id="title">Nhập khóa 2FA để lấy mã OTP theo thời gian</h1>
      <p class="intro">
        Dán secret Base32, URI <code>otpauth://</code>, hoặc tải ảnh QR 2FA. Mã được tính trực
        tiếp trong trình duyệt, không gửi khóa ra ngoài.
      </p>
    </section>

    <section class="qr-card" aria-label="Nhập ảnh QR code">
      <label for="qr-image">Ảnh QR code</label>
      <div class="upload-box">
        <input id="qr-image" type="file" accept="image/*" onchange={decodeQrImage} />
        <div>
          <strong>Chọn ảnh QR 2FA</strong>
          <span>Hỗ trợ PNG, JPG, WebP hoặc ảnh chụp màn hình.</span>
        </div>
      </div>

      {#if qrPreview}
        <img class="qr-preview" src={qrPreview} alt="Ảnh QR đã chọn" />
      {:else}
        <div class="qr-placeholder" aria-hidden="true">QR</div>
      {/if}

      {#if qrError}
        <p class="error">{qrError}</p>
      {/if}
    </section>
  </aside>

  <section class="panel" aria-label="Tạo mã OTP">
    <label for="secret">Khóa bí mật</label>
    <textarea
      id="secret"
      bind:value={secretInput}
      autocomplete="off"
      spellcheck="false"
      placeholder="Ví dụ: JBSWY3DPEHPK3PXP hoặc otpauth://totp/..."
    ></textarea>

    {#if error}
      <p class="error">{error}</p>
    {/if}

    <div class="key-card">
      <div>
        <span class="muted">Key trích xuất</span>
        <p>{parsedSecret.secret || 'Chưa có key'}</p>
      </div>
      <button type="button" onclick={copyKey} disabled={!parsedSecret.secret}>
        {copiedKey ? 'Đã copy' : 'Copy key'}
      </button>
    </div>

    <div class="otp-card">
      <div>
        <span class="muted">Mã hiện tại</span>
        <p class="otp-code" aria-live="polite">{formattedCode}</p>
      </div>
      <button type="button" onclick={copyCode} disabled={code === '------'}>
        {copied ? 'Đã copy' : 'Copy'}
      </button>
    </div>

    <div class="time-row">
      <span>Còn {secondsRemaining}s</span>
      <span>{STEP_SECONDS}s / chu kỳ</span>
    </div>
    <div class="progress" aria-hidden="true">
      <span style={`width: ${progress}%`}></span>
    </div>

    {#if parsedSecret.issuer || parsedSecret.account}
      <dl class="meta">
        {#if parsedSecret.issuer}
          <div>
            <dt>Nhà cung cấp</dt>
            <dd>{parsedSecret.issuer}</dd>
          </div>
        {/if}
        {#if parsedSecret.account}
          <div>
            <dt>Tài khoản</dt>
            <dd>{parsedSecret.account}</dd>
          </div>
        {/if}
      </dl>
    {/if}
  </section>
</main>
