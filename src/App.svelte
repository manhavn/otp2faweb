<script lang="ts">
  import jsQR from 'jsqr'

  const STEP_SECONDS = 30
  const DIGITS = 6
  const translations = {
    en: {
      invalidBase32: 'Secret can only contain Base32 characters A-Z and 2-7.',
      missingSecret: 'No valid secret found.',
      invalidCounter: 'Counter must be an integer greater than or equal to 0.',
      codeError: 'Unable to generate an OTP from this secret.',
      qrUnsupported: 'This browser does not support QR image decoding.',
      qrNotFound: 'No QR code was found in this image.',
      qrReadError: 'Unable to read a QR code from this image.',
      fileOpenError: 'Unable to open this file.',
      cameraUnsupported: 'This browser does not support camera scanning.',
      cameraPreviewError: 'Unable to initialize the camera preview.',
      cameraOpenError: 'Unable to open the camera for QR scanning.',
      eyebrow: 'TOTP 2FA Generator',
      title: 'Enter a 2FA key to get time-based OTP codes',
      intro: 'Paste a Base32 secret, an otpauth:// URI, or upload a 2FA QR image. Codes are calculated directly in your browser without sending the key anywhere.',
      qrSectionLabel: 'QR code image input',
      qrImage: 'QR code image',
      chooseQr: 'Choose a 2FA QR image',
      qrFormats: 'PNG, JPG, WebP, or a screenshot.',
      stop: 'Stop',
      camera: 'Camera',
      cameraVideoLabel: 'QR scanning camera',
      cameraHint: 'Place the QR code inside the camera frame',
      qrPreviewAlt: 'Selected QR image',
      otpPanelLabel: 'Generate OTP code',
      content: 'Content',
      loadContent: 'Load content',
      contentPlaceholder: 'Enter any content, a Base32 secret, or otpauth://totp/...',
      otpType: 'OTP type',
      timeBased: 'Time-based',
      counterBased: 'Counter-based',
      counter: 'Counter',
      extractedKey: 'Extracted key',
      noKey: 'No key yet',
      copied: 'Copied',
      copyKey: 'Copy key',
      currentCode: 'Current code',
      copy: 'Copy',
      remaining: 'Remaining',
      period: 'period',
      currentCounter: 'Current counter',
      issuer: 'Issuer',
      account: 'Account',
      fileBuilder: 'File builder',
      fileTitle: 'Create a file from tags',
      generateContent: 'Generate content',
      addTag: 'Add tag',
      tagPlaceholder: 'Enter a tag, then press Enter or Add',
      add: 'Add',
      tagListLabel: 'Draggable tag list',
      dragTagLabel: 'Drag to reorder tag',
      removeTagLabel: 'Remove tag',
      emptyTags: 'No tags yet. Tags will appear in this box after you add them.',
      previewContent: 'Preview content',
      filename: 'Filename',
      download: 'Download',
      language: 'Language',
    },
    vi: {
      invalidBase32: 'Khóa chỉ được chứa ký tự Base32 A-Z và 2-7.',
      missingSecret: 'Không tìm thấy khóa bí mật hợp lệ.',
      invalidCounter: 'Bộ đếm phải là số nguyên từ 0 trở lên.',
      codeError: 'Không thể tạo mã OTP từ khóa này.',
      qrUnsupported: 'Trình duyệt không hỗ trợ đọc ảnh QR.',
      qrNotFound: 'Không tìm thấy mã QR trong ảnh này.',
      qrReadError: 'Không thể đọc mã QR từ ảnh này.',
      fileOpenError: 'Không thể mở file này.',
      cameraUnsupported: 'Trình duyệt không hỗ trợ camera scanner.',
      cameraPreviewError: 'Không thể khởi tạo camera preview.',
      cameraOpenError: 'Không thể mở camera để quét QR.',
      eyebrow: 'TOTP 2FA Generator',
      title: 'Nhập khóa 2FA để lấy mã OTP theo thời gian',
      intro: 'Dán secret Base32, URI otpauth://, hoặc tải ảnh QR 2FA. Mã được tính trực tiếp trong trình duyệt, không gửi khóa ra ngoài.',
      qrSectionLabel: 'Nhập ảnh QR code',
      qrImage: 'Ảnh QR code',
      chooseQr: 'Chọn ảnh QR 2FA',
      qrFormats: 'PNG, JPG, WebP hoặc ảnh chụp màn hình.',
      stop: 'Dừng',
      camera: 'Camera',
      cameraVideoLabel: 'Camera quét QR',
      cameraHint: 'Đưa QR vào khung camera',
      qrPreviewAlt: 'Ảnh QR đã chọn',
      otpPanelLabel: 'Tạo mã OTP',
      content: 'Nội dung',
      loadContent: 'Load content',
      contentPlaceholder: 'Nhập nội dung bất kỳ, secret Base32 hoặc otpauth://totp/...',
      otpType: 'Loại OTP',
      timeBased: 'Theo thời gian',
      counterBased: 'Theo bộ đếm',
      counter: 'Bộ đếm',
      extractedKey: 'Key trích xuất',
      noKey: 'Chưa có key',
      copied: 'Đã copy',
      copyKey: 'Copy key',
      currentCode: 'Mã hiện tại',
      copy: 'Copy',
      remaining: 'Còn',
      period: 'chu kỳ',
      currentCounter: 'Bộ đếm hiện tại',
      issuer: 'Nhà cung cấp',
      account: 'Tài khoản',
      fileBuilder: 'File builder',
      fileTitle: 'Tạo file từ tags',
      generateContent: 'Generate content',
      addTag: 'Thêm tag',
      tagPlaceholder: 'Nhập tag rồi Enter hoặc bấm Add',
      add: 'Add',
      tagListLabel: 'Danh sách tags có thể kéo thả',
      dragTagLabel: 'Kéo để đổi vị trí tag',
      removeTagLabel: 'Xóa tag',
      emptyTags: 'Chưa có tag nào. Tags sẽ nằm trong box này sau khi add.',
      previewContent: 'Preview content',
      filename: 'Filename',
      download: 'Download',
      language: 'Ngôn ngữ',
    },
  }

  type Language = keyof typeof translations

  let secretInput = $state('')
  let code = $state('------')
  let error = $state('')
  let qrError = $state('')
  let qrPreview = $state('')
  let scanning = $state(false)
  let copied = $state(false)
  let copiedKey = $state(false)
  let copiedIssuer = $state(false)
  let copiedAccount = $state(false)
  let now = $state(Date.now())
  let otpMode = $state<'totp' | 'hotp'>('totp')
  let hotpCounter = $state(0)
  let tagInput = $state('')
  let tags = $state<string[]>([])
  let draggedTagIndex = $state<number | null>(null)
  let filename = $state('otp-tags.txt')
  let language = $state<Language>('en')
  let videoElement = $state<HTMLVideoElement | null>(null)
  let scanStream: MediaStream | null = null
  let scanFrameId = 0

  const secondsRemaining = $derived(STEP_SECONDS - (Math.floor(now / 1000) % STEP_SECONDS))
  const progress = $derived((secondsRemaining / STEP_SECONDS) * 100)
  const parsedSecret = $derived(parseSecret(secretInput))
  const formattedCode = $derived(code === '------' ? code : `${code.slice(0, 3)} ${code.slice(3)}`)
  const filePreview = $derived(tags.join('|'))
  const t = $derived(translations[language])
  const base32Pattern = /^[A-Z2-7]+=*$/

  $effect(() => {
    const timer = window.setInterval(() => {
      now = Date.now()
    }, 1000)

    return () => window.clearInterval(timer)
  })

  $effect(() => {
    return () => stopCameraScan()
  })

  $effect(() => {
    secretInput
    otpMode
    hotpCounter
    now
    language
    copied = false
    copiedKey = false
    copiedIssuer = false
    copiedAccount = false
    void generateCode()
  })

  function parseSecret(value: string) {
    const trimmed = value.trim()

    if (!trimmed) return { secret: '', issuer: '', account: '' }

    const uriStart = trimmed.toLowerCase().indexOf('otpauth://')

    if (uriStart !== -1) {
      const uriText = trimmed.slice(uriStart).split(/\s+/)[0]

      try {
        const uri = new URL(uriText)
        const label = decodeURIComponent(uri.pathname.replace(/^\//, ''))
        return {
          secret: normalizeBase32(uri.searchParams.get('secret') ?? ''),
          issuer: uri.searchParams.get('issuer') ?? '',
          account: label.includes(':') ? label.split(':').slice(1).join(':') : label,
        }
      } catch {
        return { secret: '', issuer: '', account: '' }
      }
    }

    const tagParts = trimmed.split('|').map((tag) => tag.trim()).filter(Boolean)
    const secretFromTags = [...tagParts].reverse().find((tag) => isBase32Secret(normalizeBase32(tag)))

    if (secretFromTags) {
      return {
        secret: normalizeBase32(secretFromTags),
        issuer: tagParts.length > 1 ? tagParts[0] : '',
        account: tagParts.length > 2 ? tagParts[1] : '',
      }
    }

    return { secret: extractBase32Secret(trimmed), issuer: '', account: '' }
  }

  function normalizeBase32(value: string) {
    return value.replace(/[\s-]/g, '').toUpperCase()
  }

  function isBase32Secret(value: string) {
    return value.length >= 16 && base32Pattern.test(value)
  }

  function extractBase32Secret(value: string) {
    const candidates = value.match(/[A-Z2-7][A-Z2-7\s=-]{14,}[A-Z2-7=]/gi) ?? []
    return candidates.map(normalizeBase32).filter(isBase32Secret).sort((a, b) => b.length - a.length)[0] ?? ''
  }

  function base32ToBytes(base32: string) {
    const alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ234567'
    const clean = base32.replace(/=+$/g, '')
    const bytes: number[] = []
    let bits = 0
    let value = 0

    for (const char of clean) {
      const index = alphabet.indexOf(char)
      if (index === -1) throw new Error(t.invalidBase32)

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
      if (keyBytes.length === 0) throw new Error(t.missingSecret)

      const key = await crypto.subtle.importKey(
        'raw',
        keyBytes,
        { name: 'HMAC', hash: 'SHA-1' },
        false,
        ['sign'],
      )
      const counter = otpMode === 'totp' ? Math.floor(now / 1000 / STEP_SECONDS) : hotpCounter
      if (!Number.isSafeInteger(counter) || counter < 0) {
        throw new Error(t.invalidCounter)
      }

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
      error = err instanceof Error ? err.message : t.codeError
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

  async function copyIssuer() {
    if (!parsedSecret.issuer) return

    await navigator.clipboard.writeText(parsedSecret.issuer)
    copiedIssuer = true
  }

  async function copyAccount() {
    if (!parsedSecret.account) return

    await navigator.clipboard.writeText(parsedSecret.account)
    copiedAccount = true
  }

  function updateHotpCounter(event: Event) {
    const value = Number((event.currentTarget as HTMLInputElement).value)
    hotpCounter = Number.isFinite(value) ? Math.max(0, Math.floor(value)) : 0
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

      if (!context) throw new Error(t.qrUnsupported)

      canvas.width = bitmap.width
      canvas.height = bitmap.height
      context.drawImage(bitmap, 0, 0)
      bitmap.close()

      const imageData = context.getImageData(0, 0, canvas.width, canvas.height)
      const result = jsQR(imageData.data, imageData.width, imageData.height)

      if (!result?.data) throw new Error(t.qrNotFound)

      secretInput = result.data
    } catch (err) {
      qrError = err instanceof Error ? err.message : t.qrReadError
    } finally {
      input.value = ''
    }
  }

  async function openSecretFile(event: Event) {
    const input = event.currentTarget as HTMLInputElement
    const file = input.files?.[0]

    if (!file) return

    try {
      secretInput = await file.text()
    } catch {
      error = t.fileOpenError
    } finally {
      input.value = ''
    }
  }

  async function startCameraScan() {
    qrError = ''

    try {
      if (!navigator.mediaDevices?.getUserMedia) {
        throw new Error(t.cameraUnsupported)
      }

      stopCameraScan()
      scanning = true
      scanStream = await navigator.mediaDevices.getUserMedia({
        video: { facingMode: { ideal: 'environment' } },
        audio: false,
      })

      if (!videoElement) throw new Error(t.cameraPreviewError)

      videoElement.srcObject = scanStream
      await videoElement.play()
      scanCameraFrame()
    } catch (err) {
      stopCameraScan()
      qrError = err instanceof Error ? err.message : t.cameraOpenError
    }
  }

  function stopCameraScan() {
    if (scanFrameId) cancelAnimationFrame(scanFrameId)
    scanFrameId = 0

    scanStream?.getTracks().forEach((track) => track.stop())
    scanStream = null
    scanning = false

    if (videoElement) videoElement.srcObject = null
  }

  function scanCameraFrame() {
    if (!videoElement || !scanning) return

    if (videoElement.videoWidth > 0 && videoElement.videoHeight > 0) {
      const canvas = document.createElement('canvas')
      const context = canvas.getContext('2d', { willReadFrequently: true })

      if (context) {
        canvas.width = videoElement.videoWidth
        canvas.height = videoElement.videoHeight
        context.drawImage(videoElement, 0, 0, canvas.width, canvas.height)

        const imageData = context.getImageData(0, 0, canvas.width, canvas.height)
        const result = jsQR(imageData.data, imageData.width, imageData.height)

        if (result?.data) {
          secretInput = result.data
          stopCameraScan()
          return
        }
      }
    }

    scanFrameId = requestAnimationFrame(scanCameraFrame)
  }

  function addTag() {
    const newTags = tagInput
      .split(/[\n,]+/)
      .map((tag) => tag.trim())
      .filter(Boolean)

    if (newTags.length === 0) return

    tags = [...tags, ...newTags]
    tagInput = ''
  }

  function handleTagInputKeydown(event: KeyboardEvent) {
    if (event.key !== 'Enter') return

    event.preventDefault()
    addTag()
  }

  function removeTag(index: number) {
    tags = tags.filter((_, tagIndex) => tagIndex !== index)
  }

  function moveTag(fromIndex: number, toIndex: number) {
    if (fromIndex === toIndex) return

    const nextTags = [...tags]
    const [movedTag] = nextTags.splice(fromIndex, 1)
    nextTags.splice(toIndex, 0, movedTag)
    tags = nextTags
  }

  function getTagDropIndex(event: DragEvent) {
    const tagBox = event.currentTarget as HTMLElement
    const target = event.target as HTMLElement
    const hoveredTag = target.closest<HTMLElement>('.tag-chip')

    if (!hoveredTag || !tagBox.contains(hoveredTag)) return tags.length

    const tagElements = Array.from(tagBox.querySelectorAll<HTMLElement>('.tag-chip'))
    const hoveredIndex = tagElements.indexOf(hoveredTag)

    if (hoveredIndex === -1) return tags.length

    const hoveredRect = hoveredTag.getBoundingClientRect()
    const isAfterHoveredTag = event.clientY > hoveredRect.top + hoveredRect.height / 2

    return hoveredIndex + (isAfterHoveredTag ? 1 : 0)
  }

  function handleTagDragStart(event: DragEvent, index: number) {
    const draggedElement = event.currentTarget as HTMLElement
    const draggedRect = draggedElement.getBoundingClientRect()

    draggedTagIndex = index
    event.dataTransfer?.setData('text/plain', tags[index])
    event.dataTransfer?.setDragImage(
      draggedElement,
      event.clientX - draggedRect.left,
      event.clientY - draggedRect.top,
    )
  }

  function handleTagDragOver(event: DragEvent) {
    event.preventDefault()
    if (draggedTagIndex === null) return

    const dropIndex = getTagDropIndex(event)
    const nextIndex = dropIndex > draggedTagIndex ? dropIndex - 1 : dropIndex

    if (nextIndex < 0 || nextIndex >= tags.length || nextIndex === draggedTagIndex) return

    moveTag(draggedTagIndex, nextIndex)
    draggedTagIndex = nextIndex
  }

  function generateContent() {
    const nextTags = [parsedSecret.issuer, parsedSecret.account, parsedSecret.secret].filter(Boolean)
    const filenameParts = ['otp-tags', parsedSecret.issuer, parsedSecret.account].filter(Boolean)

    tags = nextTags
    filename = `${sanitizeFilename(filenameParts.join('-')) || 'otp-tags'}.txt`
  }

  function downloadPreview() {
    if (!filePreview) return

    const safeName = sanitizeFilename(filename) || 'otp-tags.txt'
    const blob = new Blob([filePreview], { type: 'text/plain;charset=utf-8' })
    const url = URL.createObjectURL(blob)
    const link = document.createElement('a')

    link.href = url
    link.download = safeName
    link.click()
    URL.revokeObjectURL(url)
  }

  function sanitizeFilename(value: string) {
    return value.trim().replace(/[^a-zA-Z0-9._-]+/g, '-').replace(/^-+|^\.+|-+$/g, '')
  }

  function updateFilename(event: Event) {
    filename = sanitizeFilename((event.currentTarget as HTMLInputElement).value)
  }
</script>

<main class="shell">
  <aside class="side-column">
    <section class="hero-card" aria-labelledby="title">
      <div class="hero-topline">
        <p class="eyebrow">{t.eyebrow}</p>
        <div class="language-switcher" aria-label={t.language}>
          <button type="button" class:active={language === 'en'} aria-pressed={language === 'en'} onclick={() => (language = 'en')}>
            EN
          </button>
          <button type="button" class:active={language === 'vi'} aria-pressed={language === 'vi'} onclick={() => (language = 'vi')}>
            VI
          </button>
        </div>
      </div>
      <h1 id="title">{t.title}</h1>
      <p class="intro">
        {t.intro}
      </p>
    </section>

    <section class="qr-card" aria-label={t.qrSectionLabel}>
      <label for="qr-image">{t.qrImage}</label>
      <div class="qr-actions">
        <div class="upload-box">
          <input id="qr-image" type="file" accept="image/*" onchange={decodeQrImage} />
          <div>
            <strong>{t.chooseQr}</strong>
            <span>{t.qrFormats}</span>
          </div>
        </div>
        <button type="button" class="camera-button" onclick={scanning ? stopCameraScan : startCameraScan}>
          {scanning ? t.stop : t.camera}
        </button>
      </div>

      {#if scanning}
        <div class="camera-preview">
          <video bind:this={videoElement} playsinline muted aria-label={t.cameraVideoLabel}></video>
          <span>{t.cameraHint}</span>
        </div>
      {:else if qrPreview}
        <img class="qr-preview" src={qrPreview} alt={t.qrPreviewAlt} />
      {:else}
        <div class="qr-placeholder" aria-hidden="true">QR</div>
      {/if}

      {#if qrError}
        <p class="error">{qrError}</p>
      {/if}
    </section>
  </aside>

  <section class="panel" aria-label={t.otpPanelLabel}>
    <div class="secret-heading">
      <label for="secret">{t.content}</label>
      <label class="open-file-button" for="secret-file">{t.loadContent}</label>
      <input id="secret-file" class="visually-hidden" type="file" accept=".txt,text/plain" onchange={openSecretFile} />
    </div>
    <textarea
      id="secret"
      bind:value={secretInput}
      autocomplete="off"
      spellcheck="false"
      placeholder={t.contentPlaceholder}
    ></textarea>

    {#if error}
      <p class="error">{error}</p>
    {/if}

    <fieldset class="mode-card">
      <legend>{t.otpType}</legend>
      <label class:active={otpMode === 'totp'}>
        <input type="radio" name="otp-mode" value="totp" bind:group={otpMode} />
        <span>{t.timeBased}</span>
      </label>
      <label class:active={otpMode === 'hotp'}>
        <input type="radio" name="otp-mode" value="hotp" bind:group={otpMode} />
        <span>{t.counterBased}</span>
      </label>
    </fieldset>

    {#if otpMode === 'hotp'}
      <div class="counter-row">
        <label for="hotp-counter">{t.counter}</label>
        <div>
          <input
            id="hotp-counter"
            type="number"
            min="0"
            step="1"
            value={hotpCounter}
            oninput={updateHotpCounter}
          />
          <button type="button" onclick={() => (hotpCounter += 1)}>+1</button>
        </div>
      </div>
    {/if}

    <div class="key-card">
      <div>
        <span class="muted">{t.extractedKey}</span>
        <p>{parsedSecret.secret || t.noKey}</p>
      </div>
      <button type="button" onclick={copyKey} disabled={!parsedSecret.secret}>
        {copiedKey ? t.copied : t.copyKey}
      </button>
    </div>

    <div class="otp-card">
      <div>
        <span class="muted">{t.currentCode}</span>
        <p class="otp-code" aria-live="polite">{formattedCode}</p>
      </div>
      <button type="button" onclick={copyCode} disabled={code === '------'}>
        {copied ? t.copied : t.copy}
      </button>
    </div>

    {#if otpMode === 'totp'}
      <div class="time-row">
        <span>{t.remaining} {secondsRemaining}s</span>
        <span>{STEP_SECONDS}s / {t.period}</span>
      </div>
      <div class="progress" aria-hidden="true">
        <span style={`width: ${progress}%`}></span>
      </div>
    {:else}
      <div class="time-row">
        <span>{t.currentCounter}</span>
        <span>{hotpCounter}</span>
      </div>
    {/if}

    {#if parsedSecret.issuer || parsedSecret.account}
      <dl class="meta">
        {#if parsedSecret.issuer}
          <div>
            <dt>{t.issuer}</dt>
            <dd>
              <span>{parsedSecret.issuer}</span>
              <button type="button" class="meta-copy" onclick={copyIssuer}>
                {copiedIssuer ? t.copied : t.copy}
              </button>
            </dd>
          </div>
        {/if}
        {#if parsedSecret.account}
          <div>
            <dt>{t.account}</dt>
            <dd>
              <span>{parsedSecret.account}</span>
              <button type="button" class="meta-copy" onclick={copyAccount}>
                {copiedAccount ? t.copied : t.copy}
              </button>
            </dd>
          </div>
        {/if}
      </dl>
    {/if}

    <section class="file-card" aria-labelledby="file-title">
      <div class="section-heading">
        <div>
          <span class="muted">{t.fileBuilder}</span>
          <h2 id="file-title">{t.fileTitle}</h2>
        </div>
        <button type="button" class="generate-button" onclick={generateContent} disabled={!parsedSecret.secret}>
          {t.generateContent}
        </button>
      </div>

      <div class="tag-input-row">
        <label for="tag-input">{t.addTag}</label>
        <div class="tag-controls">
          <input
            id="tag-input"
            type="text"
            bind:value={tagInput}
            onkeydown={handleTagInputKeydown}
            placeholder={t.tagPlaceholder}
          />
          <button type="button" onclick={addTag}>{t.add}</button>
        </div>
      </div>

      <div
        class="tag-box"
        role="list"
        aria-label={t.tagListLabel}
        ondragover={handleTagDragOver}
        ondrop={() => (draggedTagIndex = null)}
      >
        {#if tags.length > 0}
          {#each tags as tag, index}
            <div
              class:dragging={draggedTagIndex === index}
              class="tag-chip"
              role="listitem"
              draggable="true"
              ondragstart={(event) => handleTagDragStart(event, index)}
              ondragend={() => (draggedTagIndex = null)}
              aria-label={`${t.dragTagLabel} ${tag}`}
            >
              <span>{tag}</span>
              <button
                type="button"
                class="tag-remove"
                aria-label={`${t.removeTagLabel} ${tag}`}
                onclick={(event) => {
                  event.stopPropagation()
                  removeTag(index)
                }}
              >
                x
              </button>
            </div>
          {/each}
        {:else}
          <p class="tag-empty">{t.emptyTags}</p>
        {/if}
      </div>

      <label for="file-preview">{t.previewContent}</label>
      <textarea id="file-preview" class="preview-box" readonly value={filePreview}></textarea>

      <div class="download-row">
        <div>
          <label for="filename">{t.filename}</label>
          <input id="filename" type="text" value={filename} oninput={updateFilename} placeholder="example.txt" />
        </div>
        <button type="button" onclick={downloadPreview} disabled={!filePreview}>{t.download}</button>
      </div>
    </section>
  </section>
</main>
