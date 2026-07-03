<script lang="ts">
  const STEP_SECONDS = 30
  const DIGITS = 6

  let secretInput = $state('')
  let code = $state('------')
  let error = $state('')
  let copied = $state(false)
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
</script>

<main class="shell">
  <section class="hero-card" aria-labelledby="title">
    <p class="eyebrow">TOTP 2FA Generator</p>
    <h1 id="title">Nhập khóa 2FA để lấy mã OTP theo thời gian</h1>
    <p class="intro">
      Dán secret Base32 hoặc URI <code>otpauth://</code>. Mã được tính trực tiếp trong trình duyệt
      bằng Web Crypto, không gửi khóa ra ngoài.
    </p>
  </section>

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
