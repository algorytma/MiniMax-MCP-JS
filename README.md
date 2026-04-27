# MiniMax-MCP-JS (Token Plan Max Düzeltilmiş)

[![MiniMax](https://img.shields.io/badge/MiniMax-MCP%20Server-00A9FF?style=flat-square)](https://github.com/MiniMax-AI/MiniMax-MCP-JS)
[![Token Plan Max](https://img.shields.io/badge/Token%20Plan%20Max-Compatible-10B981?style=flat-square)](#)

> MiniMax Token Plan Max hesapları için düzeltilmiş MCP server. Orijinal [MiniMax-AI/MiniMax-MCP-JS](https://github.com/MiniMax-AI/MiniMax-MCP-JS) reposundan fork edilmiştir.

## 📋 İçindekiler

- [Neden Bu Proje?](#neden-bu-proje)
- [Karşılaşılan Sorunlar](#karşılaşılan-sorunlar)
- [Çözüm](#çözüm)
- [Düzeltilen Hatalar](#düzeltilen-hatalar)
- [Kurulum](#kurulum)
- [Yapılandırma](#yapılandırma)
- [Kullanım](#kullanım)
- [Güncelleme](#güncelleme)
- [Katkı](#katkı)

---

## Neden Bu Proje?

MiniMax, AI araçları için Token Plan Max adında bir API servisi sunuyor. Bu API, [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) üzerinden erişilebilir.

Hermes Agent (NousResearch) üzerinde MiniMax MCP server'ını kullanmak istedik. Ancak orijinal `minimax-mcp-js` paketi **Mainland China** API hesapları için optimize edilmişti. Token Plan Max (Global) hesapları ile kullanıldığında çalışmıyordu.

---

## Karşılaşılan Sorunlar

### 1. API Host Uyumsuzluğu

| Hesap Tipi | Doğru Host | Paketteki Host |
|------------|------------|----------------|
| Token Plan Max (Global) | `api.minimax.io` | `api.minimax.chat` ❌ |
| Mainland China | `api.minimax.chat` | `api.minimax.chat` ✅ |

**Sorun:** Yanlış host kullanımı "Invalid API key" hatası veriyordu.

### 2. Güncellenmiş Model Adları

MiniMax API, Token Plan Max için model adlarını güncelledi:

| Özellik | Eski Model | Yeni Model |
|---------|-----------|------------|
| Text-to-Speech | `speech-02-hd` | `speech-2.8-hd` |
| Video Generation | `T2V-01` | `MiniMax-Hailuo-2.3` |
| Music Generation | `music-1.5` | `music-2.6` |

**Sorun:** Pakette eski model adları var, yeni modeller çalışmıyordu.

### 3. Video İşleme Süresi

Token Plan Max modelleri daha uzun işleme süresine sahip. Pakette sadece eski model için 60 retries vardı, yeni model için 30 retries (10 dakika) yetersiz kalıyordu.

### 4. Path Handling Crash

`expandHomeDir` fonksiyonu `undefined` filepath ile çağrıldığında crash oluyordu.

---

## Çözüm

Orijinal repoyu fork edip gerekli düzeltmeleri yaptık:

1. **API Host** → `api.minimax.io` (Global)
2. **Varsayılan Modeler** → Token Plan Max uyumlu
3. **Video Retries** → Yeni model için 60 retries (20 dakika)
4. **Path Handling** → Undefined kontrolü eklendi

---

## Düzeltilen Hatalar

### Düzeltme 1: API Host
```typescript
// src/const/index.ts
// ÖNCEKI (Hatalı):
export const DEFAULT_API_HOST = 'https://api.minimax.chat';

// SONRASI (Doğru):
export const DEFAULT_API_HOST = 'https://api.minimax.io';
```

### Düzeltme 2: TTS Modeli
```typescript
// src/const/index.ts
// ÖNCEKI (Hatalı):
export const DEFAULT_SPEECH_MODEL = 'speech-02-hd';

// SONRASI (Doğru):
export const DEFAULT_SPEECH_MODEL = 'speech-2.8-hd';
```

### Düzeltme 3: Video Modeli
```typescript
// src/const/index.ts
// ÖNCEKI (Hatalı):
export const DEFAULT_T2V_MODEL = 'T2V-01';

// SONRASI (Doğru):
export const DEFAULT_T2V_MODEL = 'MiniMax-Hailuo-2.3';
```

### Düzeltme 4: Music Modeli
```typescript
// src/const/index.ts
// ÖNCEKI (Hatalı):
export const DEFAULT_MUSIC_MODEL = 'music-1.5';

// SONRASI (Doğru):
export const DEFAULT_MUSIC_MODEL = 'music-2.6';
```

### Düzeltme 5: Video Model Listesi
```typescript
// src/const/index.ts
// ÖNCEKI:
export const VALID_VIDEO_MODELS = [
  'T2V-01', 'T2V-01-Director', 'I2V-01', 'I2V-01-Director',
  'I2V-01-live', 'S2V-01', 'MiniMax-Hailuo-02',
];

// SONRASI:
export const VALID_VIDEO_MODELS = [
  'T2V-01', 'T2V-01-Director', 'I2V-01', 'I2V-01-Director',
  'I2V-01-live', 'S2V-01', 'MiniMax-Hailuo-02', 'MiniMax-Hailuo-2.3',
];
```

### Düzeltme 6: Video Retries
```typescript
// src/api/video.ts
// ÖNCEKI:
const maxRetries = model === "MiniMax-Hailuo-02" ? 60 : 30;

// SONRASI:
const maxRetries = 60;
```

### Düzeltme 7: Path Handling
```typescript
// src/utils/file.ts
// ÖNCEKI:
function expandHomeDir(filepath: string): string {
  if (filepath.startsWith('~/')) {
    return path.join(os.homedir(), filepath.slice(2));
  }
  return filepath;
}

// SONRASI:
function expandHomeDir(filepath: string): string {
  if (!filepath) return filepath;
  if (filepath.startsWith('~/')) {
    return path.join(os.homedir(), filepath.slice(2));
  }
  return filepath;
}
```

---

## Kurulum

### Hermes Agent (Railway) ile Kullanım

#### 1. Fork'u GitHub'da Oluştur

Bu repoyu fork edin: `https://github.com/algorytma/MiniMax-MCP-JS`

#### 2. Hermes Agent Template'i Güncelleyin

`config.yaml` dosyasında MCP server ayarlarını güncelleyin:

```yaml
mcp_servers:
  minimax-media:
    command: "npx"
    args: ["-y", "algorytma/MiniMax-MCP-JS"]
    env:
      MINIMAX_API_KEY: "${MINIMAX_API_KEY}"
      MINIMAX_API_HOST: "https://api.minimax.io"
      MINIMAX_MCP_BASE_PATH: "/data/.hermes/mcp-output"
```

#### 3. Redeploy

Railway dashboard'tan projeyi yeniden deploy edin.

### Claude Desktop ile Kullanım

```json
{
  "mcpServers": {
    "minimax": {
      "command": "npx",
      "args": ["-y", "algorytma/MiniMax-MCP-JS"],
      "env": {
        "MINIMAX_API_KEY": "sk-cp-xxxxxx",
        "MINIMAX_API_HOST": "https://api.minimax.io",
        "MINIMAX_MCP_BASE_PATH": "/tmp/minimax-output"
      }
    }
  }
}
```

---

## Yapılandırma

### Ortam Değişkenleri

| Değişken | Açıklama | Varsayılan |
|----------|----------|------------|
| `MINIMAX_API_KEY` | Token Plan Max API anahtarınız (`sk-cp-...` formatında) | Gerekli |
| `MINIMAX_API_HOST` | API host adresi | `https://api.minimax.io` |
| `MINIMAX_MCP_BASE_PATH` | Çıktı dosyalarının kaydedileceği dizin | `~/Desktop` |
| `MINIMAX_RESOURCE_MODE` | `url` (URL döndür) veya `local` (dosya kaydet) | `url` |

### Token Plan Max — Doğru Değerler

| Özellik | Model | Endpoint |
|---------|-------|----------|
| Text-to-Speech | `speech-2.8-hd` | `/v1/t2a_v2` |
| Image Generation | `image-01` | `/v1/image_generation` |
| Video Generation | `MiniMax-Hailuo-2.3` | `/v1/video_generation` |
| Music Generation | `music-2.6` | `/v1/music_generation` |

---

## Kullanım

### MCP Araçları

| Araç | Açıklama | Parametreler |
|------|----------|--------------|
| `text_to_audio` | Metni sese dönüştür | `text`, `model`, `voiceId`, `speed` |
| `text_to_image` | Metinden görsel oluştur | `prompt`, `model`, `aspectRatio` |
| `generate_video` | Metin/görselden video oluştur | `prompt`, `model`, `firstFrameImage` |
| `music_generation` | Müzik oluştur | `prompt`, `lyrics` |
| `voice_clone` | Ses klonla | `audioFile`, `voiceId` |

---

## Güncelleme

### Orijinal Repodaki Değişiklikleri Almak

Bu repo fork olduğu için, orijinal repodaki güncellemeleri manuel olarak almak gerekebilir:

```bash
# Remote ekle
git remote add upstream https://github.com/MiniMax-AI/MiniMax-MCP-JS.git

# Orijinal repodan değişiklikleri çek
git fetch upstream

# Main branch'inde merge yap
git checkout main
git merge upstream/main

# Conflict varsa çöz
# Ardından push yap
git push origin main
```

### Önemli Not

Bu fork'ta yapılan düzeltmeler orijinal repoya **geri birleştirilmedi**. Güncellemeler alırken aşağıdaki noktalara dikkat edin:

- Orijinal repo `src/const/index.ts` dosyasındaki değerleri değiştirmiş olabilir
- Bu durumda çakışmaları manuel olarak çözmeniz gerekebilir
- Düzeltmelerimiz (`src/const/index.ts`, `src/api/video.ts`, `src/utils/file.ts`) korunmalı

---

## Test

Değişikliklerin çalıştığını doğrulamak için:

```bash
# Build et
npm install
npm run build

# Test et
npm test
```

---

## Katkı

1. Fork yapın
2. Branch oluşturun (`git checkout -b ozellik/yeni-ozellik`)
3. Değişikliklerinizi commit edin (`git commit -m 'Add: yeni özellik'`)
4. Branch'i push edin (`git push origin ozellik/yeni-ozellik`)
5. Pull Request açın

---

## Lisans

MIT License — Orijinal [MiniMax-AI/MiniMax-MCP-JS](https://github.com/MiniMax-AI/MiniMax-MCP-JS) ile aynısı.

---

## Referanslar

- [Orijinal MiniMax-MCP-JS](https://github.com/MiniMax-AI/MiniMax-MCP-JS)
- [MiniMax API Dokümantasyonu](https://platform.minimax.io/docs/token-plan/intro)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [Hermes Agent](https://github.com/NousResearch/hermes-agent)
