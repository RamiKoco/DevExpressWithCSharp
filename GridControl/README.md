# DevExpress GridControl

## Giriş

**GridControl**, DevExpress WinForms'un en çok kullanılan bileşenidir. Kurumsal (LOB — line of business) uygulamalarda neredeyse her liste ekranının temelinde o vardır: ürünler, siparişler, müşteriler... hepsi bir grid'de listelenir.

Bu yazı, GridControl'ü pratik açıdan ele alır: temel kavramlar, veri bağlama, kolonlar, hazır gelen özellikler ve sık kullanılan ipuçları. Örnekler generic bir `Urun` modeli üzerinden ilerler:

```csharp
public class Urun
{
    public int Id { get; set; }
    public string Ad { get; set; }
    public string Kategori { get; set; }
    public decimal Fiyat { get; set; }
    public int Stok { get; set; }
}
```

---

## En Kritik Ayrım: GridControl ve GridView

Yeni başlayanların en çok karıştırdığı nokta budur ve her şeyin temelidir:

* **GridControl** — forma bıraktığın asıl bileşendir (kapsayıcı). Veriyi (`DataSource`) o taşır.
* **GridView** — GridControl'ün içinde veriyi **gösteren** görünümdür. Kolonlar, ayarlar ve olaylar onun üzerindedir.

Yani veriyi **GridControl'e** bağlarsın, ama kolonları ve davranışı **GridView'de** yapılandırırsın:

```csharp
gridControl1.DataSource = urunler;   // veri GridControl'e
// ama kolonlar, ayarlar, olaylar → gridView1 üzerinde
```

Bir GridControl birden çok görünüm tipini barındırabilir (GridView, BandedGridView, CardView...), ama varsayılan ve en yaygın olanı GridView'dir. Bu ayrımı kavradığında, dokümantasyondaki her şey yerine oturur.

---

## Veri Bağlama (Data Binding)

GridControl'e bir liste, `BindingList`, `DataTable` ya da herhangi bir `IList` bağlayabilirsin:

```csharp
List<Urun> urunler = UrunleriGetir();
gridControl1.DataSource = urunler;
```

GridView, kaynaktaki property'lerden kolonları **otomatik üretir**. Kolonları elle kontrol etmek istersen, otomatik üretimi kapatıp kendin tanımlarsın.

---

## Kolonlar (Columns)

Her kolon bir `GridColumn`'dur. İki anahtar özelliği vardır: `FieldName` (hangi property'ye bağlı) ve `Caption` (başlıkta görünen metin):

```csharp
gridView1.Columns.Clear();

var colAd = gridView1.Columns.AddVisible("Ad", "Ürün Adı");
var colFiyat = gridView1.Columns.AddVisible("Fiyat", "Birim Fiyat");
colFiyat.DisplayFormat.FormatType = FormatType.Numeric;
colFiyat.DisplayFormat.FormatString = "c2";   // para formatı
```

`VisibleIndex` ile sıralarını, `Visible` ile görünürlüklerini yönetirsin.

---

## Hazır Gelen Özellikler

GridControl'ün asıl gücü, **sıfır kodla** gelen özelliklerdir. Standart bir grid'de elle yazman gereken çoğu şey burada hazırdır:

* **Sıralama** — kolon başlığına tıkla
* **Gruplama** — kolonu grup paneline sürükle
* **Filtreleme** — kolon başlığındaki filtre simgesi
* **Özetler (summary)** — toplam, ortalama, sayım alt bilgide
* **Kolon yönetimi** — kullanıcı kolonları gizleyip gösterebilir, sırayı değiştirebilir

Bu özellikler için ekstra kod yazmazsın; GridControl bunları kullanıcıya doğrudan sunar. Kurumsal uygulamalarda tercih edilmesinin başlıca sebebi budur.

---

## In-place Editors (RepositoryItem)

Hücreler sadece metin göstermez; içlerinde **zengin editor'ler** barındırabilir. Bunun için `RepositoryItem` kullanılır. Örneğin Kategori kolonunu bir açılır listeye dönüştürmek:

```csharp
var comboBox = new RepositoryItemComboBox();
comboBox.Items.AddRange(new[] { "Elektronik", "Gıda", "Giyim" });

gridView1.Columns["Kategori"].ColumnEdit = comboBox;
```

Aynı şekilde `RepositoryItemLookUpEdit` (başka bir tablodan seçim), `RepositoryItemDateEdit` (tarih), `RepositoryItemCheckEdit` (onay kutusu) gibi editor'ler hücrelere yerleştirilir. Böylece grid, düzenlenebilir ve kullanıcı dostu hâle gelir.

---

## Satır Görünümünü Özelleştirme

Kurumsal uygulamalarda çok sık ihtiyaç duyulan bir şey: veriye göre satır/hücre renklendirmek. Örneğin stoğu biten ürünleri vurgulamak:

```csharp
private void gridView1_RowCellStyle(object sender, RowCellStyleEventArgs e)
{
    if (e.Column.FieldName == "Stok")
    {
        var stok = (int)gridView1.GetRowCellValue(e.RowHandle, "Stok");
        if (stok == 0)
            e.Appearance.BackColor = Color.MistyRose;   // stok bitenleri kırmızımsı yap
    }
}
```

Bu tür koşullu biçimlendirme, `RowCellStyle` veya `RowStyle` olaylarıyla yapılır ve grid'i çok daha okunur kılar.

---

## Master-Detail

GridControl, bir kaydın altında ona bağlı detayları **iç içe** gösterebilir. Örneğin her siparişin altında o siparişin kalemleri. Bu master-detail yapısı, kaynaktaki ilişkiler üzerinden kurulur ve kullanıcı ana satırı genişleterek detayları görür. Ayrı bir detay ekranı açmaya gerek kalmadan, ilişkili veriyi tek grid'de sunmanın güçlü bir yoludur.

---

## GridControl ve Standart DataGridView Farkı

| WinForms DataGridView (standart)   | DevExpress GridControl                 |
| ---------------------------------- | -------------------------------------- |
| Gruplama/filtreleme elle yazılır   | Hazır gelir, sıfır kod                 |
| Sınırlı hücre editor'leri          | Zengin in-place editor'ler (LookUp...) |
| Tema/skin yok                      | Skin ve görsel tema desteği            |
| Master-detail elle kurulur         | Built-in                               |
| Ücretsiz                           | Ticari lisans gerektirir               |

Kısacası: standart grid temel ihtiyaçları karşılar; GridControl, kurumsal uygulamaların beklediği zengin özellikleri hazır sunar — bedeli ise lisans maliyetidir.

---

## Pratik İpuçları

* **Kolonu GridControl'e değil, GridView'e ekle** — en sık yapılan hata budur; kolonlar görünümün üzerindedir.
* **Çok veride performans** — binlerce/milyonlarca satırda, tüm veriyi belleğe almak yerine *server mode* veya sanal veri kaynağı kullan.
* **Düzenlemeyi kapat** — salt-okunur liste için `gridView1.OptionsBehavior.Editable = false`.
* **Kolonları otomatik genişlet** — `gridView1.BestFitColumns()` ile içeriğe göre hizala.
* **Odaklı satırı al** — `gridView1.GetFocusedRow()` ile seçili kaydın nesnesine ulaş.

---

## Sonuç

GridControl, DevExpress WinForms'un kalbidir ve kurumsal liste ekranlarının temelini oluşturur. En kritik kavram, **GridControl (kapsayıcı, veriyi taşır) ile GridView (görünüm, kolonları ve davranışı taşır)** ayrımıdır; bunu kavramak gerisini kolaylaştırır.

Asıl gücü, sıralama, gruplama, filtreleme ve özetlerin sıfır kodla gelmesidir. In-place editor'ler ile düzenlenebilir, `RowCellStyle` ile koşullu biçimlendirilebilir, master-detail ile ilişkili veriyi tek ekranda gösterebilirsin.

Standart `DataGridView`'e göre çok daha zengindir; bedeli ticari lisanstır. Ama bir kez bu zenginliğe alıştığında, kurumsal uygulamaların neden DevExpress GridControl etrafında kurulduğunu net görürsün.
