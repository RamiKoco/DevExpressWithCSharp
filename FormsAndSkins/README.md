# DevExpress Forms & Skins

## Giriş

DevExpress yalnızca kontroller vermez; **formun kendisini** de değiştirir ve tüm uygulamaya tutarlı, temalanabilir bir görünüm kazandırır. Her DevExpress formu bir `XtraForm`'dur; **skin**'ler ise uygulamanın baştan sona görünümünü belirler.

Bu yazı, DevExpress'in görsel temelini ele alır: XtraForm, skin yönetimi ve en önemli pratik nokta olan skin uyumlu renklendirme.

---

## XtraForm — DevExpress Formu

Standart WinForms formları `Form`'dan türer. DevExpress formları ise `XtraForm`'dan türer. Sebep şudur: `XtraForm`, skin sistemine **dahildir** — başlık çubuğu, kenarlıkları ve çerçevesi aktif skin'i benimser. Klasik `Form` ise skin dışında kalır.

```csharp
public partial class UrunForm : DevExpress.XtraEditors.XtraForm
{
    public UrunForm() => InitializeComponent();
}
```

Benzer şekilde skinlenebilir bir UserControl için `XtraUserControl`, Office tarzı şerit menü için `RibbonForm` vardır. Genel kural: DevExpress kullanıyorsan formların `Form`'dan değil, `XtraForm`'dan türesin.

---

## Skin Nedir?

Skin, uygulamaya bütün olarak uygulanan bir görsel temadır: renkler, geçişler, kenarlıklar, kontrollerin görünümü. DevExpress çok sayıda hazır skin ile gelir (The Bezier, Office 2019 Colorful, Visual Studio Dark, WXI...). Skin'i değiştirmek, tüm uygulamayı anında yeniden temalar.

---

## Skin'i Ayarlamak

Varsayılan skin'i uygulama başlarken (`Program.cs` / `Main`) ayarlarsın:

```csharp
DevExpress.LookAndFeel.UserLookAndFeel.Default.SetSkinStyle("WXI");
```

Kullanıcının çalışma zamanında skin seçmesini istersen, `SkinDropDownButton` veya bir skin galerisi ile bunu tek satırlık bir menüye dönüştürebilirsin.

---

## Skin Uyumlu Renkler (En Önemli İpucu)

Sık yapılan bir hata: özel arayüzünde renkleri sabit kodlamak (`Color.Blue` gibi). Bu, koyu (dark) bir skin'e geçildiğinde göze batar veya okunamaz hâle gelir. Doğrusu, renkleri **aktif skin'in paletinden** almaktır; böylece özel UI'ın da skin değişince uyum sağlar:

```csharp
// Sabit renk yerine, aktif skin'in paletinden al:
var skin = DevExpress.Skins.CommonSkins.GetSkin(
    DevExpress.LookAndFeel.UserLookAndFeel.Default);

Color vurgu = skin.Colors["Highlight"];   // skin'in vurgu rengi
```

(Renk anahtarları skin'e göre değişebilir; fikir şudur: rengi sabitleme, aktif skin'den türet.) Bu yaklaşım, örneğin özel bir sekme/panel rengini skin'le uyumlu tutmak için kritiktir — kullanıcı açık bir skin'den koyu bir skin'e geçtiğinde arayüzün hâlâ tutarlı görünmesini sağlar.

---

## LookAndFeel — Görünüm Yönetimi

Görünümün merkezinde `UserLookAndFeel` vardır. `Default` tüm uygulamaya uygulanır; istersen tek bir kontrolün `LookAndFeel`'ini ayrı yönetebilirsin. Stil olarak `Skin`, `Flat`, `UltraFlat`, `Style3D` seçenekleri vardır — modern uygulamalar genelde `Skin` kullanır.

---

## Standart Form ve XtraForm Farkı

| Standart WinForms Form              | DevExpress XtraForm                    |
| ----------------------------------- | -------------------------------------- |
| `Form`'dan türer                    | `XtraForm`'dan türer                   |
| Skin dışında (klasik pencere çerçevesi) | Skin'li başlık ve çerçeve          |
| Uygulama geneli tema yok            | App-wide skin desteği                  |
| Özel renkleri elle yönetirsin       | Renkleri skin paletinden alabilirsin   |

---

## Pratik İpuçları

* **Tüm formları XtraForm'dan türet** — karışık (bazı `Form`, bazı `XtraForm`) bir uygulama tutarsız görünür.
* **Renkleri skin'den al, sabitleme** — dark skin'lerde sabit renkler okunamaz hâle gelir.
* **Skin'i tek yerden ayarla** — başlangıçta (`Main`) bir kez; dağıtık skin ayarı kafa karıştırır.
* **Ortak base form** — tüm formların türeyeceği ortak bir `XtraForm` temel sınıfı oluşturmak, görünümü ve davranışı tek yerden yönetmeyi sağlar (kurumsal uygulamalarda yaygın bir desendir).

---

## Sonuç

DevExpress, kontrollerin ötesinde formun kendisini ve uygulamanın genel görünümünü de yönetir. Her form `XtraForm`'dan türediğinde skin sistemine dahil olur; skin'ler ise tüm uygulamayı tek hamlede temalar.

En kritik pratik nokta, renkleri sabit kodlamak yerine **aktif skin'in paletinden almaktır** — böylece özel arayüzün de skin değişimlerine uyum sağlar. Ortak bir base form üzerinden tüm formları türetmek de görünümü ve davranışı tek merkezden yönetmenin yaygın yoludur.

GridControl ve Editors veriyi gösterip girdiyi alırken, Forms & Skins bunların üzerinde durduğu görsel zemini sağlar — bu yüzden DevExpress'in tutarlı görünümünün temelidir.
