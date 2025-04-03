# Solution to espeak-ng and phonemizer issues on Windows

## Overview

Hey everyone, I asked about this issue a couple of days ago and wanted to share the fix I discovered.

---

## 1. Install espeak-ng on Windows

1. Download and install the `.msi` file from the [espeak-ng releases page](https://github.com/espeak-ng/espeak-ng/releases).
2. After installing, set your PATH or use an environment variable:

   ```python
   os.environ["PHONEMIZER_ESPEAK_LIBRARY"] = "C:\\Program Files\\eSpeak NG\\espeak-ng.exe"
   ```

---

## 2. Why the standard phonemizer library fails on Windows

The default phonemizer library is mainly designed for Unix-like systems, and it struggles with Windows path handling and process execution. Even if you set the environment variable for `espeak-ng`, it might still fail.

---

## 3. Custom phonemizer for Windows

Instead of using:

```python
self.phonemizer = phonemizer.backend.EspeakBackend(
    language='en-us', preserve_punctuation=True, with_stress=True
)
```

Create a custom class that calls `espeak-ng` via `subprocess`:

```python
try:
    import subprocess
    import os

    class CustomPhonemizer:
        def __init__(self, espeak_path):
            self.espeak_path = espeak_path

        def phonemize(self, texts):
            results = []
            for text in texts:
                cmd = [self.espeak_path, "--ipa", "-q", text]
                result = subprocess.run(cmd, capture_output=True, text=True, encoding='utf-8')
                results.append(result.stdout.strip())
            return results

    self.phonemizer = CustomPhonemizer(os.environ["PHONEMIZER_ESPEAK_LIBRARY"])

except Exception:
    # Fallback to the default EspeakBackend if custom fails
    self.phonemizer = phonemizer.backend.EspeakBackend(
        language='en-us', preserve_punctuation=True, with_stress=True
    )
```

With this approach, you bypass the Windows compatibility issues in the phonemizer library by directly calling the `espeak-ng` executable.

---

## Conclusion

Hope this helps anyone else struggling with the same issue! If you have any questions or need more details, feel free to ask.
