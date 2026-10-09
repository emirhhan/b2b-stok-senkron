# B2B → WooCommerce Stok Senkronu

**Sorun:** E-ticaret mağazası stok tutmadan, tedarikçilerin B2B portallarındaki stokla satış yapıyordu. Tedarikçi stoğu hızla tükendiği için siparişlerin önemli bir kısmı "ürün kalmadı" nedeniyle iptal oluyordu. Bu hem ciroyu hem reklam verimini yakıyordu.

**Çözüm:** İki farklı tedarikçinin B2B portalından beden bazında stok okuyup WooCommerce'e yazan, insan onaylı bir senkron sistemi.

## Nasıl çalışır

```
Tedarikçi A portalı ─┐                         ┌─ WooCommerce (REST, imzalı)
 (AJAX + HTML)       ├─► Senkron paneli ─► eşleştirme ─► kurallar ─► dry-run raporu ─► onay ─► canlı yazma
Tedarikçi B portalı ─┘   (yerel, Node)      (isim/ID)    (beden serisi,                         │
 (JSON API)                                               mükerrer, uyarı)                       └─ ürün sayfasında "son bedenler" uyarısı
```

- **İki farklı kaynak, tek model:** Biri sunucu-render HTML + AJAX, diğeri token yenilemeli JSON API. Her ikisi ortak "ürün → beden → adet" modeline indirgenir.
- **Güvenli eşleştirme:** Önce tedarikçi kimliği, sonra normalize isim. Bulanık eşleştirme bilinçli olarak *otomatik yapılmaz* (aynı modelin renk kodları farklı ürünlerdir) — öneri olarak panele düşer, insan bağlar.
- **İş kuralları:** Mükerrer listelemelerde beden başına MAX (asla toplama), farklı beden sistemlerini (bel ölçüsü, birleşik bedenler) atlama, son 1–2 beden kaldığında ürün sayfasında uyarı.
- **Güvenli yayın:** Varsayılan mod `dry` — hiçbir şey yazmaz, değişiklik raporu üretir. Canlıya geçiş elle. REST uçları anahtar ister; sunucuda yalnızca SHA-256 özeti tutulur.
- **Dayanıklılık:** Hız sınırına (429) karşı geri çekilme, oturum düşünce turu durdurup bildirim, spot-check ile canlı kaynağa karşı doğrulama.

## Teknoloji
PHP (WordPress/WooCommerce REST), Node.js 22 (bağımlılıksız), Chrome DevTools Protocol.

## Pazarlama açısından neden önemli
Stok doğruluğu, reklam bütçesinin olmayan ürüne harcanmasını önler; iptal oranı düşer, ölçülen dönüşüm gerçek ciroyu yansıtır.

---

## 🔒 Kaynak kod

Kaynak kod bu depoda **AES-256 ile şifrelenmiş** olarak durur (`kaynak.tar.gz.enc`). API anahtarları, mağaza kimlikleri ve iş ortaklarının bilgileri şifrelemeden önce koddan temizlendi.

Kodu incelemek isteyen işverenler ve ekipler şifreyi doğrudan benden alabilir:
**[LinkedIn](https://www.linkedin.com/in/emirhanterzi/)** · **[Profil](https://www.motogate.com.tr/emirhan-terzi/)**

```bash
openssl enc -d -aes-256-cbc -pbkdf2 -iter 300000 -in kaynak.tar.gz.enc | tar xz
```

© Emirhan Terzi. Tüm hakları saklıdır. Kod lisanslı değildir; izinsiz kopyalanamaz, kullanılamaz.
