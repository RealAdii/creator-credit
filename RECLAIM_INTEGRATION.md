# Reclaim Protocol JS SDK Integration Guide

Minimal guide to integrate Reclaim Protocol's zkTLS verification in the browser.

---

## Quick Start

```html
<script type="module">
  import { ReclaimProofRequest } from 'https://esm.sh/@reclaimprotocol/js-sdk@4.10.1';

  const APP_ID = 'your-app-id';
  const APP_SECRET = 'your-app-secret';
  const PROVIDER_ID = 'your-provider-id';

  async function startVerification() {
    // Initialize with portal URL config
    const proofRequest = await ReclaimProofRequest.init(APP_ID, APP_SECRET, PROVIDER_ID, {
      useAppClip: false,
      customSharePageUrl: 'https://portal.reclaimprotocol.org'
    });

    // Start session FIRST (before getting URL)
    await proofRequest.startSession({
      onSuccess: (proof) => {
        console.log('Proof received:', proof);
        // Extract data from proof.claimData.context
      },
      onError: (error) => {
        console.error('Verification failed:', error);
      }
    });

    // Get verification URL and open popup
    const requestUrl = await proofRequest.getRequestUrl();
    window.open(requestUrl, '_blank', 'width=500,height=700');
  }
</script>
```

---

## Common Errors & Fixes

### 1. `ReclaimProofRequest is not defined`

**Cause:** Using old v3 syntax or wrong import.

**Fix:**
```js
// WRONG - v3 syntax
const { Reclaim } = await import('...');
const request = new Reclaim.ProofRequest(APP_ID);

// CORRECT - v4 syntax
import { ReclaimProofRequest } from 'https://esm.sh/@reclaimprotocol/js-sdk@4.10.1';
const proofRequest = await ReclaimProofRequest.init(APP_ID, APP_SECRET, PROVIDER_ID, options);
```

### 2. Verification Opens App Store / Mobile App

**Cause:** `useAppClip` defaults to `true` on iOS.

**Fix:**
```js
await ReclaimProofRequest.init(APP_ID, APP_SECRET, PROVIDER_ID, {
  useAppClip: false,  // Force web flow
  customSharePageUrl: 'https://portal.reclaimprotocol.org'
});
```

### 3. Session Callbacks Never Fire

**Cause:** Calling `getRequestUrl()` before `startSession()`.

**Fix:**
```js
// WRONG order
const url = await proofRequest.getRequestUrl();
await proofRequest.startSession({ onSuccess, onError });

// CORRECT order - session first
await proofRequest.startSession({ onSuccess, onError });
const url = await proofRequest.getRequestUrl();
```

### 4. `share.reclaimprotocol.org` Redirects Don't Work

**Cause:** Default share URLs require mobile app handling.

**Fix:** Always set `customSharePageUrl` to use the web portal:
```js
{
  customSharePageUrl: 'https://portal.reclaimprotocol.org'
}
```

### 5. CORS / Module Loading Errors

**Cause:** Using wrong CDN or non-ESM bundle.

**Fix:** Use `esm.sh` with explicit version:
```js
// Recommended
import { ReclaimProofRequest } from 'https://esm.sh/@reclaimprotocol/js-sdk@4.10.1';

// Alternative - jsDelivr (may have issues)
import { ReclaimProofRequest } from 'https://cdn.jsdelivr.net/npm/@reclaimprotocol/js-sdk@4.10.1/+esm';
```

### 6. Popup Blocked

**Cause:** Opening popup outside user gesture.

**Fix:** Call `window.open` directly in click handler:
```js
button.addEventListener('click', async () => {
  // ... setup code ...
  window.open(requestUrl, '_blank', 'width=500,height=700'); // Inside click
});
```

---

## Full Working Example

```html
<!DOCTYPE html>
<html>
<head>
  <title>Reclaim Integration</title>
</head>
<body>
  <button id="verifyBtn">Verify Account</button>
  <div id="result"></div>

  <script type="module">
    import { ReclaimProofRequest } from 'https://esm.sh/@reclaimprotocol/js-sdk@4.10.1';

    // Get these from https://dev.reclaimprotocol.org
    const APP_ID = 'YOUR_APP_ID';
    const APP_SECRET = 'YOUR_APP_SECRET';
    const PROVIDER_ID = 'YOUR_PROVIDER_ID';

    document.getElementById('verifyBtn').addEventListener('click', async () => {
      const btn = document.getElementById('verifyBtn');
      btn.textContent = 'Connecting...';
      btn.disabled = true;

      try {
        const proofRequest = await ReclaimProofRequest.init(
          APP_ID,
          APP_SECRET,
          PROVIDER_ID,
          {
            useAppClip: false,
            customSharePageUrl: 'https://portal.reclaimprotocol.org'
          }
        );

        await proofRequest.startSession({
          onSuccess: (proof) => {
            document.getElementById('result').innerHTML = `
              <p>Verified!</p>
              <pre>${JSON.stringify(proof.claimData, null, 2)}</pre>
            `;
            btn.textContent = 'Verified';
          },
          onError: (error) => {
            document.getElementById('result').textContent = `Error: ${error.message}`;
            btn.textContent = 'Verify Account';
            btn.disabled = false;
          }
        });

        const requestUrl = await proofRequest.getRequestUrl();
        window.open(requestUrl, 'ReclaimVerify', 'width=500,height=700');

      } catch (err) {
        console.error(err);
        btn.textContent = 'Verify Account';
        btn.disabled = false;
      }
    });
  </script>
</body>
</html>
```

---

## SDK Versions

| Version | Status | Notes |
|---------|--------|-------|
| v4.10.1 | **Recommended** | Stable, portal support |
| v4.x | Works | Use latest 4.x |
| v3.x | Deprecated | Different API, don't use |

---

## Credentials

Get your credentials at: https://dev.reclaimprotocol.org

- **APP_ID**: Your application identifier
- **APP_SECRET**: Keep secret, used for signing
- **PROVIDER_ID**: Specific to data source (Twitter, YouTube, etc.)

---

## Support

- Docs: https://docs.reclaimprotocol.org
- Discord: https://discord.gg/reclaim
