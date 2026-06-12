# DevExpress Editors

## Giriş

**Editors**, DevExpress'in giriş (input) kontrolleridir — standart WinForms `TextBox`, `ComboBox`, `DateTimePicker`'ın zengin karşılıkları. Her formda kullanılırlar ve GridControl yazısında gördüğümüz **in-place editor** kavramının da temelidir.

Bu yazıda DevExpress editor'lerini, özellikle de kurumsal uygulamaların bel kemiği olan `LookUpEdit`'i ele alıyoruz. Örnekler yine generic `Urun`/`Kategori` modeli üzerinden ilerler.

---

## Editor Nedir?

DevExpress editor'leri, ortak bir API paylaşan gelişmiş giriş kontrolleridir. Hepsinin temelinde `BaseEdit` vardır. Sık kullanılanlar:

* **TextEdit / MemoEdit** — metin girişi
* **SpinEdit** — artır/azalt oklu sayı girişi
* **DateEdit** — takvim açılır penceresiyle tarih
* **CheckEdit** — onay kutusu
* **ComboBoxEdit** — sabit listeden seçim
* **LookUpEdit / GridLookUpEdit** — veri kaynağına bağlı açılır liste

Hepsi tutarlı bir mantıkla çalışır: değeri `EditValue`, yapılandırması `Properties` üzerinden yönetilir.

---

## Ortak Kavram: EditValue ve Properties

Tüm editor'ler iki şeyi paylaşır:

* **`EditValue`** — editor'ün gerçek değeri
* **`Properties`** — editor'ün davranışını yapılandırdığın yer

```csharp
textEdit1.EditValue = "Merhaba";
dateEdit1.EditValue = DateTime.Today;
spinEdit1.EditValue = 42;
```

Bu tutarlılık sayesinde, bir editor'ü öğrendiğinde diğerlerini de büyük ölçüde bilmiş olursun.

---

## ComboBoxEdit — Sabit Liste

Seçenekler sabitse (veritabanından gelmiyorsa), `ComboBoxEdit` yeterlidir:

```csharp
comboBoxEdit1.Properties.Items.AddRange(new[] { "Elektronik", "Gıda", "Giyim" });
```

---

## LookUpEdit — Veriye Bağlı Açılır Liste

Kurumsal uygulamaların en çok kullandığı editor budur. Bir kaydı (örneğin kategoriyi) seçtirip, kullanıcıya **adını** gösterirken arka planda **Id**'sini saklar — yani yabancı anahtar (foreign key) seçimi için biçilmiş kaftandır:

```csharp
lookUpEdit1.Properties.DataSource = kategoriler;   // List<Kategori>
lookUpEdit1.Properties.DisplayMember = "Ad";       // kullanıcının gördüğü
lookUpEdit1.Properties.ValueMember = "Id";         // saklanan değer
```

Kullanıcı "Elektronik" görür ve seçer; `EditValue` ise o kategorinin `Id`'si olur. Çok kolonlu bir açılır liste gerekiyorsa (örneğin kod + ad + fiyat), `GridLookUpEdit` kullanılır — içinde tam bir grid barındırır.

---

## DateEdit ve Maskeleme

`DateEdit`, takvim açılır penceresiyle tarih seçtirir. Giriş formatını maske ile kontrol edebilirsin:

```csharp
dateEdit1.Properties.Mask.EditMask = "dd.MM.yyyy";
dateEdit1.Properties.Mask.UseMaskAsDisplayFormat = true;
```

Maskeleme yalnızca tarihe özgü değildir; telefon, sayı, para gibi biçimli girişler için de `Properties.Mask` kullanılır. Bu, standart WinForms'ta elle yazman gereken doğrulamayı hazır getirir.

---

## In-place Kullanım (RepositoryItem)

GridControl yazısından hatırla: bir editor, grid'in içinde kullanılacaksa **RepositoryItem** sürümüyle kullanılır. Yani aynı editor, iki bağlamda yaşar — forma doğrudan bırakılan standalone hâli ve grid'e gömülen RepositoryItem hâli:

```csharp
var repoLookUp = new RepositoryItemLookUpEdit();
repoLookUp.DataSource = kategoriler;
repoLookUp.DisplayMember = "Ad";
repoLookUp.ValueMember = "Id";

gridView1.Columns["KategoriId"].ColumnEdit = repoLookUp;   // grid hücresinde LookUpEdit
```

Bu ikili yapı (standalone + RepositoryItem) DevExpress editor'lerinin temel bir özelliğidir.

---

## Doğrulama (Validation)

Editor'ler, `DXValidationProvider` ile birlikte yerleşik doğrulama sunar: zorunlu alanları işaretleyebilir, geçersiz girişte hata ikonu gösterebilirsin. Böylece "bu alan boş olamaz", "geçerli bir e-posta gir" gibi kurallar, standart WinForms'taki kadar uğraştırmadan kurulur.

---

## Standart WinForms Kontrolleri ve DevExpress Editors Farkı

| Standart WinForms                  | DevExpress Editors                     |
| ---------------------------------- | -------------------------------------- |
| TextBox, ComboBox, DateTimePicker  | TextEdit, ComboBoxEdit, DateEdit...    |
| Sınırlı görünüm, tema yok          | Skin uyumlu, tutarlı görünüm           |
| Maskeleme/doğrulama elle yazılır   | Yerleşik mask ve validation            |
| Grid içinde doğrudan kullanılamaz  | RepositoryItem ile grid'e gömülür      |
| Veriye bağlı dropdown zahmetli     | LookUpEdit ile kolay (FK seçimi)       |

---

## Pratik İpuçları

* **EditValue ile Text'i karıştırma** — `EditValue` gerçek değerdir (LookUpEdit'te `Id` gibi); `Text` ise ekranda görünen metindir.
* **Yapılandırma Properties'tedir** — editor'ün davranışını (`DataSource`, `Mask`, `NullText`...) hep `Properties` üzerinden ayarlarsın.
* **NullText** — LookUpEdit boşken görünecek metni (`Properties.NullText`) ayarla; kullanıcı ne seçmesi gerektiğini anlasın.
* **Çok kolonlu dropdown** — basit liste için `LookUpEdit`, grid gibi çok kolonlu seçim için `GridLookUpEdit`.

---

## Sonuç

DevExpress editor'leri, standart WinForms giriş kontrollerinin zengin karşılıklarıdır. Hepsi `EditValue` ve `Properties` üzerinden tutarlı biçimde çalışır; bu yüzden birini öğrenmek diğerlerini de büyük ölçüde öğrenmek demektir.

En değerlisi `LookUpEdit`'tir: kullanıcıya adı gösterip arka planda Id'yi sakladığı için yabancı anahtar seçiminin standart yoludur. Editor'ler ayrıca maskeleme ve doğrulamayı hazır sunar ve RepositoryItem sürümleriyle grid'in içine de gömülür.

Bu ikili yapı — standalone form kontrolü ve grid içi in-place editor — DevExpress'in form ile grid'i aynı zengin bileşenler üzerine kurmasını sağlar; GridControl'den sonra öğrenilecek en doğal konu da budur.
