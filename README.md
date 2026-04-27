# MiniMax-MCP-JS (Token Plan Max Düzeltilmiş)

[![MiniMax](https://img.shields.io/badge/MiniMax-MCP%20Server-00A9FF?style=flat-square)](https://github.com/MiniMax-AI/MiniMax-MCP-JS)
[![Token Plan Max](https://img.shields.io/badge/Token%20Plan%20Max-Compatible-10B981?style=flat-square)](#)

> MiniMax'ın anlamsız hardcoded sınırlarına ve sürekli değişen modellerine isyan olarak doğmuş, "Token Plan Max" hesapları ve Otonom Ajanlar için optimize edilmiş, kurşun geçirmez bir MCP server fork'u. Orijinal [MiniMax-AI/MiniMax-MCP-JS](https://github.com/MiniMax-AI/MiniMax-MCP-JS) reposundan fork edilmiştir. İhtiyacı olan tüm geliştiricilere feda olsun! 🍻

## 📋 İçindekiler

- [Neden Bu Proje? (Kısa Cevap: MiniMax'ın Mallığı Yüzünden)](#neden-bu-proje)
- [Karşılaşılan Sorunlar ve Saçmalıklar](#karşılaşılan-sorunlar)
- [Neleri Çözdük? (Bizim Çözümümüz)](#çözüm)
- [Düzeltilen Hatalar & Eklenen Özellikler](#düzeltilen-hatalar)
- [Kurulum](#kurulum)
- [Yapılandırma](#yapılandırma)
- [Kullanım](#kullanım)
- [Güncelleme](#güncelleme)
- [Katkı](#katkı)

---

## Neden Bu Proje? (Kısa Cevap: MiniMax'ın Mallığı Yüzünden) <a name="neden-bu-proje"></a>

MiniMax, muazzam kalitede ses, müzik ve video üreten harika yapay zeka modelleri sunuyor. Üstelik global kullanıcılar için "Token Plan Max" adında süper bir abonelik planları da var.

Fakat resmi [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) entegrasyonlarına baktığımızda, kodun **"Mainland China"** hesapları için beton gibi hardcoded değerlerle doldurulduğunu gördük! 

Hermes Agent (NousResearch) gibi otonom yapay zeka ajanlarımızla çalışırken karşılaştığımız tablo şuydu:
- Model isimleri koda **sabit bir enum listesi** olarak gömülmüştü. MiniMax yeni bir model çıkarıyor (`speech-2.8-hd` veya `music-2.6` gibi), ama biz MCP yüzünden bunu kullanamıyoruz çünkü kod enum'a takılıp "Geçersiz Model" hatası basıyordu!
- Sadece `api.minimax.chat` (Çin endpoint'i) kodlanmıştı. Token Plan Max kullanıcıları için gereken `api.minimax.io` endpoint'ini kullanamıyorduk.
- Otonom ajanlarımızın hata aldığında başka modele geçmesini (failover) beklerken, MCP sunucusu hatayı 500 dönüyor veya JSON formatına uymadan saçmalıyordu.

Kısacası MiniMax'ın kendi elleriyle yaptığı bu "katı ve statik" mimari yüzünden sistemlerimiz çöküyordu. Biz de "Yeter artık!" deyip kodu forklayarak, otonom ajanların ruhuna uygun, esnek, dinamik ve hataya toleranslı bu "Altın Standart" versiyonu oluşturduk. Bizim gibi çile çeken tüm geliştiricilere feda olsun!

---

## Karşılaşılan Sorunlar ve Saçmalıklar <a name="karşılaşılan-sorunlar"></a>

### 1. API Host Uyumsuzluğu
Resmi pakette host zorla `api.minimax.chat` olarak bırakılmıştı. Global hesap sahipleri sürekli "Invalid API key" hatasıyla karşılaşıyordu.

### 2. Beton Gibi Enum'lar (Geçmişte Kalan Modeller)
Yeni modeller çıkmasına rağmen paketteki Zod şemalarında eski modeller hardcoded enum olarak duruyordu. (Örn: `T2V-01`, `music-1.5`). Ajanımız `MiniMax-Hailuo-2.3` denemek istediğinde şema doğrulamasından geçemeyip patlıyordu.

### 3. Hata Yönetimi (Failover) Eksikliği
Ajanlar hata aldıklarında (örneğin yetersiz token veya geçersiz prompt) 500 hatası yerine, neyin yanlış gittiğini bilmek ister. Orijinal kod düz bir hata fırlatıyordu.

### 4. Uçucu Medya Linkleri
Üretilen videoların ve seslerin URL'leri kısa süre sonra zaman aşımına uğruyor (expiry). URL modunda çalışan ajanlarımız daha dosyayı indiremeden link patlıyordu.

---

## Neleri Çözdük? (Bizim Çözümümüz) <a name="çözüm"></a>

Orijinal repoyu esnek, otonom ajan dostu ve üretim ortamına (production-ready) uygun hale getirdik:

1. **Dinamik Model Desteği (Hardcoded Enumlar Çöpe):** Şemalardaki tüm kısıtlayıcı enum değerleri kaldırıldı. Ajanınız istediği string'i model olarak yollayabilir. MiniMax yarın `music-3.0` çıkarırsa, hiçbir kod güncellemesi yapmadan doğrudan kullanabilirsiniz!
2. **Global Endpoint Uyumluluğu:** Varsayılan host `api.minimax.io` olarak değiştirildi.
3. **Şeffaf Hata Yönetimi (`isError: true`):** Ajan hata aldığında artık bu bir sistem çökmesi (500) değil. Doğrudan `{ isError: true, content: "Hata detayı..." }` dönerek ajanın durumu algılayıp "failover" (başka modele geçiş) yapabilmesini sağladık.
4. **Çevre Değişkeni (Env Var) Odaklı Model Seçimi:** Varsayılan modeller `MINIMAX_DEFAULT_AUDIO_MODEL`, `MINIMAX_DEFAULT_VIDEO_MODEL` ve `MINIMAX_DEFAULT_MUSIC_MODEL` env variable'ları ile dinamik hale getirildi.
5. **Varsayılan Olarak Yerel Dosya (Local Resource Mode):** Üretilen medyalar uçucu URL'ler olarak değil, doğrudan yerel bir dizine (`MINIMAX_MCP_BASE_PATH`) indiriliyor. Böylece dosya kayıpları engellendi.

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
