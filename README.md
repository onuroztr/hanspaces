# Han Spaces Nidya Center

Esenyurt, İstanbul'daki Han Spaces Nidya Center için coworking, hazır ofis ve
toplantı odası hizmetlerini tanıtan, reklam trafiğinin indiği landing page.

## Teknoloji

Vite + React 18 + TypeScript, Tailwind CSS, shadcn/ui, react-router, Supabase
Edge Function (form gönderimi), react-helmet-async (SEO).

## Geliştirme

```bash
bun install
bun run dev       # sitemap üretip geliştirme sunucusunu başlatır
bun run build     # üretim derlemesi
bun run lint
bun run test
```

## Sayfa yapısı

Anasayfa (`src/pages/Index.tsx`) klasik bir dönüşüm hunisine göre sıralanmıştır:

| Bölüm | Bileşen | İşlevi |
| --- | --- | --- |
| Hero + form | `Hero` | Teklifi ve lead formunu ekranın üstünde sunar |
| Güven şeridi | `TrustBar` | İlk saniyede "burada ne var" cevabı |
| Hizmetler | `ServiceCards`, `FlexibleSolutions` | Üç hizmetin tanıtımı ve görselleri |
| Paketler | `PricingPlans` | Çalışma modelleri + teklif CTA'sı |
| Karşılaştırma | `ComparisonTable` | "Hangisi bana uygun" kararı |
| Hedef kitle | `WhoIsItFor` | Ziyaretçinin kendini tanıması |
| Kendi ofisin vs biz | `OfficeComparison` | Asıl rakip olan "kendi ofisimizi kuralım" fikrini hedefler |
| Tek fatura | `AllInclusive` | "Tüm giderler dahil" vaadini 10 kaleme döker |
| Olanaklar | `Amenities` | Dahil olan her şeyin tek tek sayılması |
| Avantajlar | `Features` | Neden biz |
| Süreç | `Process` | 4 adımda yerleşme |
| Galeri | `Gallery` | Mekan görselleri |
| Konum | `LocationDetail` | Harita, adres, ulaşım |
| Instagram | `InstagramFeed` | Sosyal kanıt |
| SSS | `Faq` | İtirazların kaldırılması + FAQPage yapısal verisi |
| Kapanış | `FinalCta` | Form / telefon / WhatsApp |

Üç hizmet sayfası (`/coworking`, `/hazir-ofis`, `/toplanti-odasi`) ortak
`ServicePageLayout` bileşenini kullanır; böylece reklam trafiği hangi sayfaya
inerse insin aynı dönüşüm yolunu görür.

## Dönüşüm takibi

`src/lib/analytics.ts` Google (GA4 + Google Ads) ve Meta Pixel'i tek yerden
yönetir. Kimlikler `.env` dosyasında tanımlı değilse **hiçbir ölçüm kodu
yüklenmez** ve site normal çalışır.

```
VITE_GA_MEASUREMENT_ID="G-XXXXXXXXXX"
VITE_GOOGLE_ADS_ID="AW-XXXXXXXXX"
VITE_GOOGLE_ADS_LEAD_LABEL="xxxxxxxxxxxxxxxxxx"
VITE_META_PIXEL_ID="XXXXXXXXXXXXXXX"
```

Gönderilen olaylar:

| Olay | Ne zaman | Meta karşılığı |
| --- | --- | --- |
| `page_view` | Her rota değişiminde | `PageView` |
| `form_start` | Forma ilk dokunuşta | — |
| `form_error` | Doğrulama hatasında | — |
| `generate_lead` | Form başarıyla gönderildiğinde | `Lead` |
| `contact_call` | Telefon bağlantısına tıklanınca | `Contact` |
| `contact_whatsapp` | WhatsApp bağlantısına tıklanınca | `Contact` |
| `cta_click` | Sayfa içi CTA'lara tıklanınca | — |
| `view_service` | Hizmet detayına gidilince | `ViewContent` |
| `scroll_depth` | %25 / 50 / 75 / 90 kaydırmada | — |
| `faq_open` | Bir SSS açıldığında | — |
| `gallery_open` | Galeri görseli büyütülünce | — |
| `page_not_found` | 404 sayfasına düşülünce | — |

`generate_lead` ayrıca `VITE_GOOGLE_ADS_LEAD_LABEL` tanımlıysa Google Ads
dönüşümü olarak da gönderilir.

## Reklam kaynağı (attribution)

`src/lib/attribution.ts`, ziyaretçi siteye ilk indiğinde URL'deki kampanya
parametrelerini (`utm_source`, `utm_medium`, `utm_campaign`, `utm_term`,
`utm_content`, `gclid`, `gbraid`, `wbraid`, `fbclid`, `msclkid`, `ttclid`)
yakalar ve oturum boyunca saklar (first-touch; kalıcı çerez yazılmaz).

Form gönderildiğinde bu özet `kaynak` alanıyla birlikte edge function'a
gider ve Google Sheets'te **F sütununa** yazılır. Örnek değer:

```
utm_source=google | utm_campaign=ofis | gclid=Cj0KCQ... | giris=/coworking
```

> **Dikkat:** Bu, `supabase/functions/sync-to-sheets` fonksiyonunda değişiklik
> gerektirdi (yazma aralığı `A2:E2` → `A2:F2`). Çalışması için fonksiyonu
> yeniden dağıtın ve tablodaki F sütununa bir başlık ekleyin (örn. "Kaynak").
> Fonksiyon dağıtılmazsa form eskisi gibi çalışmaya devam eder, yalnızca
> kaynak bilgisi kaydedilmez.

## İçerik notu

Sayfadaki tüm iddialar hizmetin gerçek özelliklerinden türetilmiştir. Müşteri
yorumu, puanlama ve fiyat rakamı **bilinçli olarak uydurulmamıştır**; bu veriler
elinizde olduğunda `src/i18n/translations.ts` üzerinden eklenebilir.
