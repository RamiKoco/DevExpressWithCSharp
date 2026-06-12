# DevExpress SchedulerControl

## Giriş

**SchedulerControl**, DevExpress'in takvim ve planlama bileşenidir. Randevular, rezervasyonlar, üretim planlama, araç/personel sevkiyatı (dispatch) gibi zaman ve kaynak yönetimi gerektiren her senaryoda kullanılır.

Bu yazı SchedulerControl'ü pratik açıdan ele alır: temel mimari, randevular, kaynaklar, görünümler ve özellikle performans için kritik olan toplu güncelleme tekniği. Örnekler generic bir `Randevu`/`Kaynak` modeli üzerinden ilerler.

---

## En Kritik Ayrım: SchedulerControl ve SchedulerStorage

GridControl'deki GridControl/GridView ayrımına benzer bir yapı burada da var:

* **SchedulerControl** — ekranda gördüğün görsel takvim (görünümler, sürükle-bırak, çizim).
* **SchedulerStorage** — veriyi tutan ve modeline eşleyen kısım (randevular ve kaynaklar).

Yani veriyi **storage**'a bağlar, görünümü **control**'de yapılandırırsın. Bu ayrımı kavramak, scheduler'ı anlamanın anahtarıdır.

---

## Randevular (Appointments) ve Mapping

Kendi veri modelini, scheduler'ın randevu alanlarına **mapping** ile eşlersin. Storage'a veri kaynağını verir, hangi property'nin neye karşılık geldiğini söylersin:

```csharp
schedulerStorage1.Appointments.DataSource = randevular;   // List<Randevu>

var map = schedulerStorage1.Appointments.Mappings;
map.Start   = "Baslangic";    // DateTime
map.End     = "Bitis";        // DateTime
map.Subject = "Konu";
map.Description = "Aciklama";
```

`Start`, `End` ve `Subject` temel zorunlu eşlemelerdir. Mapping kurulduğunda, listendeki her kayıt takvimde bir randevu olarak görünür.

---

## Kaynaklar (Resources)

Scheduler'ın güçlü yanı, randevuları **kaynaklara** göre düzenleyebilmesidir: odalar, personel, makineler, araçlar... Her kaynak kendi sütununda gösterilir:

```csharp
schedulerStorage1.Resources.DataSource = kaynaklar;   // odalar / personel / makineler
schedulerStorage1.Resources.Mappings.Id = "Id";
schedulerStorage1.Resources.Mappings.Caption = "Ad";
```

Randevunun hangi kaynağa ait olduğunu da randevu mapping'inde belirtirsin (`map.ResourceId = "KaynakId"`). Böylece takvim, kaynak bazında gruplanır — örneğin her makinenin kendi zaman çizelgesi.

---

## Görünümler (Views)

Scheduler birden çok görünüm sunar ve aralarında geçiş yapılabilir:

```csharp
schedulerControl1.ActiveViewType = SchedulerViewType.Week;
```

Başlıca görünümler: **Day** (gün), **Week** (hafta), **WorkWeek** (iş haftası), **Month** (ay), **Timeline** (zaman çizelgesi) ve **Gantt**. Timeline ve Gantt, kaynak bazlı planlamada (üretim, sevkiyat) özellikle kullanışlıdır.

---

## Performans: BeginUpdate / EndUpdate

En kritik pratik ipucu budur. Çok sayıda randevu yüklerken veya toplu değişiklik yaparken, scheduler her değişiklikte yeniden çizim yaparsa ciddi şekilde yavaşlar (donma yaşanabilir). Çözüm, işlemi `BeginUpdate`/`EndUpdate` arasına almaktır:

```csharp
schedulerControl1.BeginUpdate();
try
{
    // toplu yükleme veya çok sayıda randevu değişikliği
}
finally
{
    schedulerControl1.EndUpdate();   // tek seferde yeniden çiz
}
```

Bu sayede scheduler, her adımda değil yalnızca işlem bittiğinde bir kez çizim yapar. Yoğun veride bu, saniyeler süren bir donmayı neredeyse anlık hâle getirebilir — yüksek hacimli planlama ekranlarında olmazsa olmazdır.

---

## Çakışma Yönetimi

Aynı kaynakta üst üste binen randevulara izin verilip verilmeyeceğini kontrol edebilirsin:

```csharp
schedulerControl1.OptionsCustomization.AllowAppointmentConflicts =
    DevExpress.XtraScheduler.AppointmentConflictsMode.Never;   // çakışmaya izin verme
```

Bu, randevu/rezervasyon sistemlerinde çift kayıt (double-booking) önlemek için işe yarar.

---

## Kendi Takvimini Yazmak ve SchedulerControl Farkı

| Elle (custom) takvim UI            | DevExpress SchedulerControl            |
| ---------------------------------- | -------------------------------------- |
| Görünümleri elle çizersin          | Day/Week/Month/Timeline/Gantt hazır    |
| Sürükle-bırak elle kurulur         | Yerleşik sürükle-bırak                  |
| Kaynak gruplama yok                | Kaynak bazlı gruplama hazır            |
| Çakışma kontrolü elle yazılır      | AllowAppointmentConflicts ile hazır    |
| Veri bağlama elle                  | SchedulerStorage mapping ile           |

---

## Pratik İpuçları

* **BeginUpdate/EndUpdate** — toplu yükleme/değişiklikte performansın anahtarı; unutma.
* **Mapping'i doğru kur** — `Start`, `End`, `Subject` olmadan randevular görünmez.
* **ResourceId mapping** — kaynak bazlı gruplama istiyorsan randevuda `ResourceId`'yi eşle.
* **Control/storage ayrımını koru** — veri storage'a, görünüm control'e; ikisini karıştırma.
* **Kullanım alanı** — scheduler sadece "takvim" değildir; üretim planlama, sevkiyat ve rezervasyon gibi kaynak+zaman senaryolarına da oturur.

---

## Sonuç

SchedulerControl, DevExpress'in takvim ve planlama bileşenidir. En kritik kavram, **SchedulerControl (görsel takvim) ile SchedulerStorage (veri ve mapping)** ayrımıdır — tıpkı GridControl/GridView gibi.

Randevuları kendi modeline mapping ile bağlar, kaynaklar üzerinden gruplar ve farklı görünümlerle (Day/Week/Month/Timeline/Gantt) sunarsın. Performansın anahtarı ise toplu işlemleri `BeginUpdate`/`EndUpdate` arasına almaktır; bu, yoğun veride donmayı önleyen en önemli tekniktir.

Scheduler'ın kaynak + zaman modeli, basit takvimlerin ötesinde üretim planlama ve sevkiyat gibi senaryolara da uyduğu için, kurumsal uygulamalarda GridControl kadar güçlü bir araçtır.
