# Notlar

```php
class Chirp extends Model
{
    public function store(Request $request): RedirectResponse
    {
        $validated = $request->validate([
            'message' => 'required|string|max:255',
        ]);
        $request->user()->chirps()->create($validated);
        return redirect(route('chirps.index'));
    }
}


class Chirp extends Model
{
    public function store(Request $request): RedirectResponse
    {
        $validated = $request->validate([
            'message' => 'required|string|max:255',
        ]);
        $request->user()->chirps()->insert($validated);
        return redirect(route('chirps.index'));
    }
}
```

## insert() vs create() Farkı

**`$request->user()->chirps()->create($validated);` ile `$request->user()->chirps()->insert($validated);` Arasındaki Fark**

İki kod örneği arasındaki fark, veri tabanına veri ekleme yönteminde yatmaktadır. Laravel'de veri eklemek için farklı yöntemler kullanabilirsiniz: `create()` ve `insert()`. Bu iki yöntem arasındaki farklar şunlardır:

### 1. **`create()` Kullanımı**

```php
$request->user()->chirps()->create($validated);
```

- **Amaç**: `create()` metodu, Eloquent modelinin bir örneğini oluşturur ve veri tabanına kaydeder. Bu yöntem, modelin `fillable` veya `guarded` özelliğine göre **veri doğrulama ve kütüphane özellikleri** (örneğin, otomatik zaman damgaları gibi) ile uyumlu bir şekilde çalışır.
- **Çalışma şekli**: `create()` metodu, önceden doğrulanan veriyi alır ve modelin **yeni bir örneğini** oluşturur. Bu işlem sırasında modeldeki `created_at` ve `updated_at` gibi otomatik alanlar da doldurulur (bu Laravel'in Eloquent özelliği ile sağlanır).
- **Veri doğrulama**: `create()` metodu sadece veriyi eklemekle kalmaz, aynı zamanda modelin **başka özelliklerini de yönetebilir** (örneğin, otomatik oluşturulacak alanlar gibi).
- **Eloquent ile uyumlu**: Bu yöntem, Laravel'in Eloquent ORM (Object-Relational Mapping) yapısını kullanarak veri ekler ve model özelliklerine tamamen entegre olur.

### 2. **`insert()` Kullanımı**

```php
$request->user()->chirps()->insert($validated);
```

- **Amaç**: `insert()` metodu, doğrudan SQL sorgusu kullanarak veri tabanına veri ekler. Bu yöntem, Eloquent modelini atlar ve doğrudan veri ekler.
- **Çalışma şekli**: `insert()` metodu, doğrulanan veriyi doğrudan veri tabanına ekler. Ancak, bu yöntem **modelin özelliklerini** (örneğin, `fillable`, `guarded`, `created_at`, `updated_at` gibi) dikkate almaz. Bu nedenle, eğer `created_at` veya `updated_at` gibi alanların otomatik olarak doldurulmasını istiyorsanız, bu değerlerin manuel olarak belirtilmesi gerekir.
- **Performans**: `insert()` metodu genellikle **daha hızlıdır** çünkü Eloquent'in model örneklerini oluşturmaz ve daha düşük seviyede çalışır.
- **Veri doğrulama**: `insert()` sadece veriyi ekler ve herhangi bir model özelliklerini veya ilişkileri yönetmez. `fillable` veya `guarded` gibi özellikler de bu işlemde geçerli değildir.

### **Özet Farklar**

1. **`create()`**:

   - Eloquent ORM ile uyumlu.
   - `fillable` ve `guarded` özelliklerine dikkat eder.
   - Otomatik olarak `created_at` ve `updated_at` gibi alanları doldurur.
   - Daha fazla özellik sunar, daha ağırdır.

2. **`insert()`**:
   - Doğrudan SQL sorgusu kullanır, Eloquent ORM'den bağımsızdır.
   - `fillable` ve `guarded` gibi model özelliklerini göz önünde bulundurmaz.
   - `created_at` gibi otomatik alanları manuel olarak ayarlamanız gerekir.
   - Daha hızlıdır, daha az kaynak kullanır.

### **Hangisini Kullanmalı?**

- Eğer modelin **otomatik alanlarını**, ilişkilerini ve diğer Eloquent özelliklerini kullanmak istiyorsanız `create()` metodunu tercih etmelisiniz.
- Eğer **performans odaklı** çalışıyorsanız ve sadece veriyi eklemek istiyorsanız, `insert()` metodunu kullanabilirsiniz. Ancak, bu durumda `created_at` gibi otomatik alanları manuel olarak ayarlamanız gerekebilir.

### **Sonuç**

- `create()` daha kapsamlı ve Eloquent özellikleriyle uyumludur.
- `insert()` ise daha hızlı ve doğrudan SQL işlemi yapar, ancak modelin sağladığı avantajları kaybetmiş olursunuz.

## Eloquent Model 1

Eloquent, Laravel'in ORM (Object-Relational Mapping) katmanıdır ve birçok kullanışlı metod sağlar. Model üzerinde çalışan bu metodlar, genellikle `Model` sınıfına ve türetilmiş sınıflara (örneğin, `User`, `Post` gibi) aittir. Aşağıda, Laravel Eloquent modelinde yer alan bazı önemli metodları ve bunların hangi sınıflara ait olduklarını içeren bir tabloyu bulabilirsiniz.

### Eloquent Model Metodları

| **Metod**       | **Ait Olduğu Sınıf**                 | **Açıklama**                                                                                                                      |
| --------------- | ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| `create()`      | `Illuminate\Database\Eloquent\Model` | Yeni bir model örneği oluşturur ve veri tabanına kaydeder. Modelin `fillable` ya da `guarded` özelliklerine göre veri kabul eder. |
| `update()`      | `Illuminate\Database\Eloquent\Model` | Mevcut bir model örneğini günceller. `save()` metoduyla benzer işlev görür, ancak belirli veriyle güncelleme yapar.               |
| `delete()`      | `Illuminate\Database\Eloquent\Model` | Mevcut bir model örneğini siler.                                                                                                  |
| `save()`        | `Illuminate\Database\Eloquent\Model` | Mevcut model örneğini kaydeder. Ekleme veya güncelleme işlemi yapar.                                                              |
| `all()`         | `Illuminate\Database\Eloquent\Model` | Tüm model örneklerini veri tabanından çeker ve bir koleksiyon (Collection) olarak döner.                                          |
| `find()`        | `Illuminate\Database\Eloquent\Model` | Verilen ID ile modelin bir örneğini bulur.                                                                                        |
| `first()`       | `Illuminate\Database\Eloquent\Model` | İlk kayıtlı model örneğini döner.                                                                                                 |
| `where()`       | `Illuminate\Database\Eloquent\Model` | Belirli bir koşula göre veri filtrelemesi yapar.                                                                                  |
| `orWhere()`     | `Illuminate\Database\Eloquent\Model` | `where` gibi çalışır ancak, ek bir `OR` koşulu ekler.                                                                             |
| `pluck()`       | `Illuminate\Database\Eloquent\Model` | Belirli bir sütunun değerlerini döner.                                                                                            |
| `count()`       | `Illuminate\Database\Eloquent\Model` | Modelin kaç kaydı olduğunu döner.                                                                                                 |
| `exists()`      | `Illuminate\Database\Eloquent\Model` | Verilen koşulda herhangi bir kayıt olup olmadığını kontrol eder.                                                                  |
| `increment()`   | `Illuminate\Database\Eloquent\Model` | Belirtilen bir sütunun değerini arttırır.                                                                                         |
| `decrement()`   | `Illuminate\Database\Eloquent\Model` | Belirtilen bir sütunun değerini düşürür.                                                                                          |
| `has()`         | `Illuminate\Database\Eloquent\Model` | Bir ilişkili modelin varlığını kontrol eder (örneğin, bir yazı ile etiketlerin ilişkisini kontrol etme).                          |
| `with()`        | `Illuminate\Database\Eloquent\Model` | Model ile birlikte ilişkili modelleri önceden yükler (Eager Loading).                                                             |
| `load()`        | `Illuminate\Database\Eloquent\Model` | Modelin ilişkili modellerini yükler (genellikle `with()` metodunun ardından).                                                     |
| `touch()`       | `Illuminate\Database\Eloquent\Model` | Modelin zaman damgalarını (created_at ve updated_at) günceller.                                                                   |
| `findOrFail()`  | `Illuminate\Database\Eloquent\Model` | ID ile model örneğini bulur. Eğer model bulunmazsa, bir `ModelNotFoundException` hatası fırlatır.                                 |
| `firstOrFail()` | `Illuminate\Database\Eloquent\Model` | İlk modeli bulur, bulunmazsa `ModelNotFoundException` hatası fırlatır.                                                            |
| `forceDelete()` | `Illuminate\Database\Eloquent\Model` | Silinmiş (soft deleted) olan bir kaydı veri tabanından tamamen siler.                                                             |
| `restore()`     | `Illuminate\Database\Eloquent\Model` | Soft deleted (yumuşak silinmiş) bir kaydı geri yükler.                                                                            |
| `toArray()`     | `Illuminate\Database\Eloquent\Model` | Modeli bir diziye dönüştürür.                                                                                                     |
| `toJson()`      | `Illuminate\Database\Eloquent\Model` | Modeli JSON formatında döner.                                                                                                     |

### **Eloquent İlişkilerle İlgili Metodlar (Model İlişkileri)**

Eloquent, modeller arasındaki ilişkileri yönetmek için çeşitli metodlar sağlar. İşte bazıları:

| **Metod**         | **Ait Olduğu Sınıf**                 | **Açıklama**                                                                                                |
| ----------------- | ------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| `hasMany()`       | `Illuminate\Database\Eloquent\Model` | Bir modelin başka bir modelle `hasMany` (birden çoğa) ilişkisini tanımlar.                                  |
| `belongsTo()`     | `Illuminate\Database\Eloquent\Model` | Bir modelin başka bir modelle `belongsTo` (birden bire) ilişkisini tanımlar.                                |
| `belongsToMany()` | `Illuminate\Database\Eloquent\Model` | Bir modelin başka bir modelle `belongsToMany` (çoktan çoka) ilişkisini tanımlar.                            |
| `hasOne()`        | `Illuminate\Database\Eloquent\Model` | Bir modelin başka bir modelle `hasOne` (birden bire) ilişkisini tanımlar.                                   |
| `morphTo()`       | `Illuminate\Database\Eloquent\Model` | Polimorfik (çok biçimli) ilişkileri tanımlar (örneğin, yorumlar bir yazıya ya da bir videoya ait olabilir). |
| `morphMany()`     | `Illuminate\Database\Eloquent\Model` | Polimorfik çoktan çoğa ilişkiyi tanımlar.                                                                   |
| `morphToMany()`   | `Illuminate\Database\Eloquent\Model` | Polimorfik çoktan çoka ilişkisini tanımlar.                                                                 |

### **Eloquent Global Scope ve Local Scope Metodları**

Ayrıca, Eloquent modellerine özel scope'lar eklemek için de metodlar kullanabilirsiniz.

| **Metod**               | **Ait Olduğu Sınıf**                 | **Açıklama**                                                                                                                 |
| ----------------------- | ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| `scopeXyz()`            | `Illuminate\Database\Eloquent\Model` | `scope` ile başlayan metotlar, Eloquent'e özgü `Local Scopes` oluşturur. Örneğin, `scopeActive()` aktif kullanıcıları döner. |
| `withoutGlobalScopes()` | `Illuminate\Database\Eloquent\Model` | Global scope'ları devre dışı bırakır.                                                                                        |

### **Eloquent ve Pivot Tablosu Metodları**

Pivot tablosu üzerinde işlem yapmak için bazı metodlar da bulunur.

| **Metod**  | **Ait Olduğu Sınıf**                                   | **Açıklama**                                                                                  |
| ---------- | ------------------------------------------------------ | --------------------------------------------------------------------------------------------- |
| `attach()` | `Illuminate\Database\Eloquent\Relations\BelongsToMany` | Pivot tablosuna yeni bir ilişki ekler.                                                        |
| `detach()` | `Illuminate\Database\Eloquent\Relations\BelongsToMany` | Pivot tablosundan bir ilişkiyi kaldırır.                                                      |
| `sync()`   | `Illuminate\Database\Eloquent\Relations\BelongsToMany` | Pivot tablosunda ilişkileri senkronize eder. (Yani, ilişkiyi ekler ve var olanları kaldırır.) |

### **Sonuç**

Laravel Eloquent, güçlü bir ORM katmanı sunarak veri tabanı işlemlerini kolaylaştırır. `Model` sınıfı üzerinde çalışan metodlar, CRUD işlemleri (Create, Read, Update, Delete), ilişkiler, doğrulama ve zaman damgası gibi birçok işlemi yönetebilir. Ayrıca, ilişkili modelleri yönetmek için birçok metod da bulunur ve her biri veritabanı etkileşimlerini kolaylaştırır.

## Eloquent Model 2

Eloquent modeli, Laravel'de veritabanı işlemlerini kolaylaştırmak için pek çok metod sunar. Bunlar, verileri ekleme (create), güncelleme (update), silme (delete) gibi temel CRUD (Create, Read, Update, Delete) işlemleriyle birlikte, ilişkili verilerle çalışma, sorgu oluşturma, ve daha fazlasını içerir. Aşağıda, **`create()`** ve benzeri metodlar için ayrıntılı bir açıklama ve ilgili sınıfların ve kullanımlarının bir tablosunu bulabilirsiniz.

### **Eloquent Model Metodları ve İşlevleri**

| **Metod**          | **Ait Olduğu Sınıf**                 | **Açıklama**                                                                                                                                                   |
| ------------------ | ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `create()`         | `Illuminate\Database\Eloquent\Model` | Yeni bir model örneği oluşturur ve veritabanına kaydeder. Bu metod, genellikle `fillable` veya `guarded` özelliğine göre veriyi kabul eder.                    |
| `update()`         | `Illuminate\Database\Eloquent\Model` | Mevcut bir model örneğini günceller. Modeldeki veriyi güncellemek için kullanılır.                                                                             |
| `save()`           | `Illuminate\Database\Eloquent\Model` | Yeni bir model ekler veya mevcut modeli günceller. Modeldeki tüm özellikler kaydedilir.                                                                        |
| `firstOrCreate()`  | `Illuminate\Database\Eloquent\Model` | Verilen koşulla ilk kaydı bulur, yoksa yeni bir model oluşturur ve kaydeder.                                                                                   |
| `updateOrCreate()` | `Illuminate\Database\Eloquent\Model` | Verilen koşulda bir kayıt bulur, yoksa yeni bir model oluşturur ve kaydeder.                                                                                   |
| `insert()`         | `Illuminate\Database\Eloquent\Model` | Veritabanına bir dizi veriyi doğrudan ekler (toplu ekleme). Bu metod `save()` gibi otomatik olarak modelin `created_at` ve `updated_at` değerlerini ayarlamaz. |
| `createMany()`     | `Illuminate\Database\Eloquent\Model` | Birden fazla kayıt eklemek için kullanılır. Çoğunlukla toplu veri eklemek için tercih edilir.                                                                  |
| `delete()`         | `Illuminate\Database\Eloquent\Model` | Veritabanındaki bir modeli siler. Silme işlemi `forceDelete()` ile tam anlamıyla yapılabilir (soft delete dışında).                                            |
| `destroy()`        | `Illuminate\Database\Eloquent\Model` | Veritabanındaki bir veya daha fazla modeli ID'leriyle siler.                                                                                                   |
| `first()`          | `Illuminate\Database\Eloquent\Model` | Veritabanından ilk modeli çeker.                                                                                                                               |
| `find()`           | `Illuminate\Database\Eloquent\Model` | ID'ye göre bir model örneği bulur. Bulamazsa `null` döner.                                                                                                     |
| `findOrFail()`     | `Illuminate\Database\Eloquent\Model` | ID'ye göre bir model bulur, bulamazsa `ModelNotFoundException` hatası fırlatır.                                                                                |
| `pluck()`          | `Illuminate\Database\Eloquent\Model` | Belirli bir sütunun tüm değerlerini döner.                                                                                                                     |
| `count()`          | `Illuminate\Database\Eloquent\Model` | Bir modelin veritabanındaki toplam sayısını döner.                                                                                                             |
| `exists()`         | `Illuminate\Database\Eloquent\Model` | Belirtilen koşula göre modelin var olup olmadığını kontrol eder.                                                                                               |
| `increment()`      | `Illuminate\Database\Eloquent\Model` | Bir sütunun değerini arttırır.                                                                                                                                 |
| `decrement()`      | `Illuminate\Database\Eloquent\Model` | Bir sütunun değerini düşürür.                                                                                                                                  |
| `touch()`          | `Illuminate\Database\Eloquent\Model` | `updated_at` ve `created_at` zaman damgalarını günceller.                                                                                                      |
| `forceDelete()`    | `Illuminate\Database\Eloquent\Model` | Soft deleted (yumuşak silinmiş) bir kaydı veri tabanından tamamen siler.                                                                                       |
| `restore()`        | `Illuminate\Database\Eloquent\Model` | Soft deleted (yumuşak silinmiş) bir kaydı geri yükler.                                                                                                         |
| `toArray()`        | `Illuminate\Database\Eloquent\Model` | Modeli dizi formatında döndürür.                                                                                                                               |
| `toJson()`         | `Illuminate\Database\Eloquent\Model` | Modeli JSON formatında döndürür.                                                                                                                               |

### **Metodların Açıklamaları ve Kullanım Senaryoları**

1. **`create()`**

   - **Açıklama**: Bu metod, yeni bir model örneği oluşturur ve veritabanına kaydeder. Genellikle verilerin `fillable` veya `guarded` özelliğine uygun olup olmadığını kontrol eder.
   - **Kullanım**:

     ```php
     $user = User::create([
         'name' => 'John Doe',
         'email' => 'john.doe@example.com',
     ]);
     ```

2. **`update()`**

   - **Açıklama**: Var olan bir modelin özelliklerini günceller. Bu, modelin kaydını veritabanında günceller.
   - **Kullanım**:

     ```php
     $user = User::find(1);
     $user->update([
         'name' => 'Updated Name',
     ]);
     ```

3. **`save()`**

   - **Açıklama**: Modelin güncel haliyle kaydedilmesini sağlar. Eğer model yeni bir modelse, `create()` gibi bir işlem yapılır, eğer var olan bir modelse, güncellenir.
   - **Kullanım**:

     ```php
     $user = new User();
     $user->name = 'Jane Doe';
     $user->email = 'jane.doe@example.com';
     $user->save();
     ```

4. **`insert()`**

   - **Açıklama**: Çok sayıda veriyi topluca ekler. Ancak, `insert` metoduyla eklenen kayıtlarda, `created_at` ve `updated_at` gibi zaman damgaları otomatik olarak ayarlanmaz.
   - **Kullanım**:

     ```php
     User::insert([
         ['name' => 'John Doe', 'email' => 'john@example.com'],
         ['name' => 'Jane Doe', 'email' => 'jane@example.com'],
     ]);
     ```

5. **`firstOrCreate()`**

   - **Açıklama**: Verilen koşula göre bir model bulur. Eğer model mevcut değilse, yeni bir model oluşturur ve kaydeder.
   - **Kullanım**:

     ```php
     $user = User::firstOrCreate(
         ['email' => 'john.doe@example.com'],
         ['name' => 'John Doe']
     );
     ```

6. **`updateOrCreate()`**

   - **Açıklama**: Verilen koşulda bir model bulur ve günceller. Eğer model mevcut değilse, yeni bir model oluşturur.
   - **Kullanım**:

     ```php
     $user = User::updateOrCreate(
         ['email' => 'john.doe@example.com'],
         ['name' => 'John Doe Updated']
     );
     ```

7. **`delete()`**

   - **Açıklama**: Mevcut bir kaydı veritabanından siler. Soft delete kullanılıyorsa, sadece `deleted_at` alanı güncellenir.
   - **Kullanım**:

     ```php
     $user = User::find(1);
     $user->delete();
     ```

8. **`destroy()`**

   - **Açıklama**: Veritabanındaki bir veya daha fazla modeli ID'leriyle siler.
   - **Kullanım**:

     ```php
     User::destroy([1, 2, 3]);
     ```

9. **`forceDelete()`**

   - **Açıklama**: Soft delete yapılmış bir kaydı tamamen siler.
   - **Kullanım**:

     ```php
     $user = User::find(1);
     $user->forceDelete();
     ```

10. **`restore()`**

    - **Açıklama**: Soft delete yapılmış bir kaydı geri yükler.
    - **Kullanım**:

      ```php
      $user = User::withTrashed()->find(1);
      $user->restore();
      ```

### **Sonuç**

Laravel Eloquent, veritabanı işlemlerini basit ve etkili bir şekilde yönetmek için çeşitli metodlar sunar. **`create()`**, **`update()`**, **`insert()`**, **`delete()`** ve diğer CRUD metodları, veritabanı işlemlerini kolaylaştırırken, ilişkili verilerle çalışma, toplu ekleme ve silme işlemleri de oldukça kullanışlıdır. Bu metodlar sayesinde, Eloquent ile etkileşimli ve esnek bir veritabanı yönetimi gerçekleştirebilirsiniz.

## Elequent CRUD Metodları

Eloquent modelinde **`insert()`** ve diğer CRUD metodları, veritabanı işlemlerini gerçekleştiren temel fonksiyonlardır.

### **CRUD Metodları**

| **Metod**          | **Ait Olduğu Sınıf**                 | **Açıklama**                                                                                                                                       |
| ------------------ | ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `insert()`         | `Illuminate\Database\Eloquent\Model` | Veritabanına bir veya birden fazla kaydı doğrudan ekler. Bu metod, **`created_at`** ve **`updated_at`** zaman damgalarını otomatik olarak eklemez. |
| `create()`         | `Illuminate\Database\Eloquent\Model` | Yeni bir model örneği oluşturur ve veritabanına kaydeder. Genellikle `fillable` veya `guarded` özelliğine göre veriyi kabul eder.                  |
| `save()`           | `Illuminate\Database\Eloquent\Model` | Yeni bir model ekler veya mevcut modeli günceller. Modeldeki tüm özellikler kaydedilir.                                                            |
| `update()`         | `Illuminate\Database\Eloquent\Model` | Mevcut bir modelin özelliklerini günceller.                                                                                                        |
| `updateOrCreate()` | `Illuminate\Database\Eloquent\Model` | Belirtilen koşula göre bir kayıt bulur, yoksa yeni bir kayıt oluşturur ve kaydeder.                                                                |
| `delete()`         | `Illuminate\Database\Eloquent\Model` | Veritabanındaki bir modeli siler. Soft delete (yumuşak silme) kullanılıyorsa, sadece `deleted_at` alanı güncellenir.                               |
| `destroy()`        | `Illuminate\Database\Eloquent\Model` | Veritabanındaki bir veya birden fazla modeli ID'leriyle siler.                                                                                     |
| `first()`          | `Illuminate\Database\Eloquent\Model` | Veritabanından ilk modeli çeker.                                                                                                                   |
| `find()`           | `Illuminate\Database\Eloquent\Model` | ID'ye göre bir model örneği bulur. Bulamazsa `null` döner.                                                                                         |
| `findOrFail()`     | `Illuminate\Database\Eloquent\Model` | ID'ye göre bir model bulur, bulamazsa `ModelNotFoundException` hatası fırlatır.                                                                    |
| `pluck()`          | `Illuminate\Database\Eloquent\Model` | Belirli bir sütunun tüm değerlerini döner.                                                                                                         |
| `count()`          | `Illuminate\Database\Eloquent\Model` | Bir modelin veritabanındaki toplam sayısını döner.                                                                                                 |
| `exists()`         | `Illuminate\Database\Eloquent\Model` | Belirtilen koşula göre modelin var olup olmadığını kontrol eder.                                                                                   |
| `increment()`      | `Illuminate\Database\Eloquent\Model` | Bir sütunun değerini arttırır.                                                                                                                     |
| `decrement()`      | `Illuminate\Database\Eloquent\Model` | Bir sütunun değerini düşürür.                                                                                                                      |
| `touch()`          | `Illuminate\Database\Eloquent\Model` | `updated_at` ve `created_at` zaman damgalarını günceller.                                                                                          |
| `forceDelete()`    | `Illuminate\Database\Eloquent\Model` | Soft delete (yumuşak silme) yapılmış bir kaydı veritabanından tamamen siler.                                                                       |
| `restore()`        | `Illuminate\Database\Eloquent\Model` | Soft delete (yumuşak silme) yapılmış bir kaydı geri yükler.                                                                                        |
| `toArray()`        | `Illuminate\Database\Eloquent\Model` | Modeli dizi formatında döndürür.                                                                                                                   |
| `toJson()`         | `Illuminate\Database\Eloquent\Model` | Modeli JSON formatında döndürür.                                                                                                                   |

### **`insert()` Metodu**

- **Açıklama**: `insert()` metodu, veritabanına bir veya birden fazla kaydı doğrudan ekler. Ancak, bu metod otomatik olarak `created_at` ve `updated_at` alanlarını güncellemez. Bu nedenle, bu metodu kullandığınızda manuel olarak tarih eklemek isteyebilirsiniz.
- **Kullanım**:

  ```php
  // Tek bir kayıt ekleme
  User::insert([
      'name' => 'John Doe',
      'email' => 'john.doe@example.com',
      'created_at' => now(),
      'updated_at' => now(),
  ]);

  // Birden fazla kayıt ekleme
  User::insert([
      ['name' => 'Jane Doe', 'email' => 'jane.doe@example.com', 'created_at' => now(), 'updated_at' => now()],
      ['name' => 'Robert Smith', 'email' => 'robert.smith@example.com', 'created_at' => now(), 'updated_at' => now()],
  ]);
  ```

### **`create()` Metodu**

- **Açıklama**: `create()` metodu, yeni bir model örneği oluşturur ve veritabanına kaydeder. Bu metod, **`fillable`** veya **`guarded`** özelliklerine göre veri alır ve otomatik olarak `created_at` ve `updated_at` zaman damgalarını ekler.
- **Kullanım**:

  ```php
  $user = User::create([
      'name' => 'John Doe',
      'email' => 'john.doe@example.com',
  ]);
  ```

### **`save()` Metodu**

- **Açıklama**: `save()` metodu, modelin verilerini kaydeder. Eğer model daha önce var ise, veritabanında güncelleme yapılır, eğer model yeni bir modelse, yeni bir kayıt eklenir.
- **Kullanım**:

  ```php
  $user = new User();
  $user->name = 'John Doe';
  $user->email = 'john.doe@example.com';
  $user->save();
  ```

### **`update()` Metodu**

- **Açıklama**: `update()` metodu, mevcut bir modelin özelliklerini günceller. Bu metod sadece var olan bir modeli günceller. Yeni bir model eklemez.
- **Kullanım**:

  ```php
  $user = User::find(1);
  $user->update([
      'name' => 'Updated Name',
  ]);
  ```

### **`updateOrCreate()` Metodu**

- **Açıklama**: `updateOrCreate()` metodu, belirli bir koşulda bir kayıt arar. Eğer kayıt bulunursa, güncellenir, yoksa yeni bir kayıt oluşturulur.
- **Kullanım**:

  ```php
  $user = User::updateOrCreate(
      ['email' => 'john.doe@example.com'],
      ['name' => 'Updated Name']
  );
  ```

### **`delete()` Metodu**

- **Açıklama**: `delete()` metodu, veritabanındaki bir kaydı siler. Soft delete (yumuşak silme) kullanılıyorsa, sadece `deleted_at` alanı güncellenir.
- **Kullanım**:

  ```php
  $user = User::find(1);
  $user->delete();
  ```

### **`destroy()` Metodu**

- **Açıklama**: `destroy()` metodu, veritabanındaki bir veya birden fazla kaydı ID'leriyle siler.
- **Kullanım**:

  ```php
  User::destroy([1, 2, 3]);
  ```

### **`forceDelete()` Metodu**

- **Açıklama**: Soft delete yapılmış bir kaydı tamamen siler.
- **Kullanım**:

  ```php
  $user = User::find(1);
  $user->forceDelete();
  ```

### **`restore()` Metodu**

- **Açıklama**: Soft delete yapılmış bir kaydı geri yükler.
- **Kullanım**:

  ```php
  $user = User::withTrashed()->find(1);
  $user->restore();
  ```

### **`first()` Metodu**

- **Açıklama**: Veritabanından ilk modeli döner. Eğer kayıt yoksa `null` döner.
- **Kullanım**:

  ```php
  $user = User::first();
  ```

### **`find()` Metodu**

- **Açıklama**: ID'ye göre bir model bulur. Eğer bulunamazsa `null` döner.
- **Kullanım**:

  ```php
  $user = User::find(1);
  ```

### **`findOrFail()` Metodu**

- **Açıklama**: ID'ye göre bir model bulur, bulamazsa `ModelNotFoundException` hatası fırlatır.
- **Kullanım**:

  ```php
  $user = User::findOrFail(1);
  ```

### **Sonuç**

- **`insert()`**, **`create()`**, **`save()`**, **`update()`**, **`delete()`** ve diğer CRUD metodları, Laravel'in veritabanı işlemlerini basit ve etkili bir şekilde yönetmek için sağladığı fonksiyonlardır. **`insert()`** metodu, doğrudan toplu veri ekleme

işlemleri için kullanılırken, **`save()`** ve **`update()`** gibi metodlar mevcut veriyi güncelleyebilir.

## ne zaman insert() ne zaman create()?

İşte **`insert()`** ve **`create()`** metodları arasındaki farkları ve hangi durumlarda kullanılmaları gerektiğini tablo halinde açıklayayım:

| **Metod**      | **Kullanım Durumu**                                                                                                          | **Açıklama**                                                                                                                                                                                                                                 |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`insert()`** | - Birden fazla kaydı topluca eklerken.                                                                                       | - **Toplu veri eklemek** için kullanılır.<br>- **`created_at`** ve **`updated_at`** zaman damgalarını **otomatik olarak eklemez**.<br>- Veritabanına doğrudan veri ekler.                                                                    |
|                | - Zaman damgası eklemek istemiyorsanız veya kendi tarihlerinizi manuel olarak eklemeniz gerekiyorsa.                         |                                                                                                                                                                                                                                              |
|                | - **Veritabanına doğrudan veri ekleme** işlemi yaparken (Eloquent model özellikleri kullanmaz).                              |                                                                                                                                                                                                                                              |
| **`create()`** | - **Yeni bir kayıt eklerken**, genellikle **`fillable`** veya **`guarded`** özelliği ile tanımlanmış alanlara veri eklerken. | - **Tek bir kayıt eklemek** için kullanılır.<br>- **`fillable`** veya **`guarded`** özelliği ile belirlenmiş alanlara otomatik olarak veri ekler.<br>- **`created_at`** ve **`updated_at`** zaman damgalarını **otomatik olarak ekler**.<br> |

### **Özetle:**

- **`insert()`**: Birden fazla kayıt eklemek, veritabanına doğrudan veri eklemek için, **`created_at`** ve **`updated_at`** alanlarına müdahale etmeyi istemiyorsanız kullanılır.
- **`create()`**: Eloquent modelinin özelliklerine (fillable/guarded) uyarak **tekil bir kayıt eklemek**, zaman damgalarının otomatik eklenmesini istemek ve modelin Eloquent özelliklerini kullanmak için tercih edilir.

### **Örnek Kullanım:**

#### **`insert()` Kullanımı (Toplu Veri Ekleme)**

```php
// Toplu veri eklemek
User::insert([
    ['name' => 'John Doe', 'email' => 'john.doe@example.com', 'created_at' => now(), 'updated_at' => now()],
    ['name' => 'Jane Doe', 'email' => 'jane.doe@example.com', 'created_at' => now(), 'updated_at' => now()],
]);
```

#### **`create()` Kullanımı (Tekil Kayıt Eklemek)**

```php
// Tek bir kayıt eklemek
$user = User::create([
    'name' => 'John Doe',
    'email' => 'john.doe@example.com',
]);
```

## create() ile birlikte, aynı kuralları destekleyerek kullanılabilecek komutlar?

**`create()`** metodu ile birlikte kullanılabilecek bazı yaygın komutlar, genellikle Eloquent modelinin işlevselliğini desteklemek ve doğrulama, ilişkiler gibi işlemleri daha etkili şekilde yapabilmek için kullanılır. İşte **`create()`** ile birlikte sıklıkla kullanılan bazı komutlar ve açıklamaları:

| **Komut**              | **Açıklama**                                                                                                                                         | **Örnek**                                                                                                                                     |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ------ | -------------- |
| **`validate()`**       | - Kullanıcıdan alınan verilerin doğruluğunu sağlamak için kullanılır.<br>- Genellikle bir formda alınan verilerin doğruluğu kontrol edilir.          | `$request->validate([ 'name' => 'required                                                                                                     | string | max:255', ]);` |
| **`fill()`**           | - Modeldeki alanları tek seferde doldurmak için kullanılır.<br>- **`create()`** yerine kullanılırsa, model oluşturulmaz, sadece veri doldurulur.     | `$user->fill(['name' => 'John Doe', 'email' => 'john.doe@example.com']);`                                                                     |
| **`associate()`**      | - İlişkili bir modelin bağlantısını oluşturur. Örneğin, bir **`Post`** modeline ait **`User`** modelini ilişkilendirmek için kullanılabilir.         | `$post->user()->associate($user);`                                                                                                            |
| **`save()`**           | - Veriyi kaydederken kullanılır.<br>- **`create()`** ile aynı işlevi görür, ancak önceden model oluşturulup verisi doldurulmuş olmalıdır.            | `$user = new User(); $user->name = 'John Doe'; $user->save();`                                                                                |
| **`forceCreate()`**    | - Veritabanı kaydını oluşturur, ancak modelin **`$timestamps`** özelliği varsayılan olarak **`true`** olduğunda zaman damgalarını atlamanızı sağlar. | `$user = User::forceCreate([ 'name' => 'John Doe', 'email' => 'john.doe@example.com']);`                                                      |
| **`createMany()`**     | - Birden fazla kayıt oluşturur.<br>- Bu, **`insert()`** gibi toplu işlem yaparken, Eloquent ilişkilerini de korur.                                   | `User::createMany([ ['name' => 'John Doe', 'email' => 'john.doe@example.com'], ['name' => 'Jane Doe', 'email' => 'jane.doe@example.com'] ]);` |
| **`firstOrCreate()`**  | - Eğer belirtilen kriterle eşleşen bir kayıt bulunursa, o kaydı döndürür; yoksa, yeni bir kayıt oluşturur.                                           | `$user = User::firstOrCreate([ 'email' => 'john.doe@example.com' ], [ 'name' => 'John Doe' ]);`                                               |
| **`updateOrCreate()`** | - Belirtilen kriterlere göre var olan kaydı günceller ya da kaydı oluşturur.                                                                         | `$user = User::updateOrCreate([ 'email' => 'john.doe@example.com' ], [ 'name' => 'John Doe' ]);`                                              |
| **`createFrom()`**     | - Başka bir modelin verilerini alarak yeni bir model oluşturur.                                                                                      | `$newUser = User::createFrom($otherUser);`                                                                                                    |

### **Örnekler:**

#### **`validate()` ile `create()` Kullanımı**

```php
public function store(Request $request)
{
    // Veriyi doğrulama
    $validated = $request->validate([
        'name' => 'required|string|max:255',
        'email' => 'required|email|unique:users,email',
    ]);

    // Veriyi kaydetme
    $user = User::create($validated);

    return redirect(route('users.index'));
}
```

#### **`fill()` ile `create()` Kullanımı**

```php
$user = new User();
$user->fill([
    'name' => 'John Doe',
    'email' => 'john.doe@example.com'
]);
$user->save();
```

#### **`firstOrCreate()` Kullanımı**

```php
$user = User::firstOrCreate(
    ['email' => 'john.doe@example.com'],
    ['name' => 'John Doe', 'password' => bcrypt('secret')]
);
```

#### **`updateOrCreate()` Kullanımı**

```php
$user = User::updateOrCreate(
    ['email' => 'john.doe@example.com'],
    ['name' => 'John Doe', 'password' => bcrypt('newpassword')]
);
```

### **Özetle:**

- **`validate()`**: **`create()`** ile birlikte kullanılarak, verilerin doğruluğunu sağlamada yardımcı olur.
- **`fill()`**: **`create()`** yerine kullanılabilir. Ancak **`create()`** otomatik olarak modelin **`fillable`** özelliğine uyar.
- **`firstOrCreate()`**: Eğer kayıt varsa o kaydı döndürür, yoksa yeni bir kayıt oluşturur.
- **`updateOrCreate()`**: Belirtilen kriterle bir kayıt bulur ve günceller, yoksa yeni bir kayıt oluşturur.

Bu metodlar, **`create()`** ile birlikte kullanılacak en yaygın Eloquent işlevleridir.

## DataBase Yapısını Görüntüleme

```bash
php artisan db:show
php artisan db:table users
```

# php artisan thinker

Laravel için otomatik olarak tinker konsolunu oluşturmaktadır. Thinker ile şunlar yapılabilir:

- Verileri almak
- Modeli oluşturmak
- Modeli güncellemek
- Modeli silmek
- Modeli kaydetmek

```bash
$ php artisan tinker
Psy Shell v0.12.7 (PHP 8.3.15 — cli) by Justin Hileman
> \App\Models\Chirp::all();
= Illuminate\Database\Eloquent\Collection {#6190
    all: [
      App\Models\Chirp {#6189
        id: 1,
        user_id: 1,
        message: "Nasılsınız?",
        created_at: "2025-01-07 11:18:36",
        updated_at: "2025-01-07 11:18:36",
      },
    ],
  }

> exit
```

# Laravel Eloquent ORM

Laravel Eloquent ORM, veritabanı tabloları arasındaki ilişkileri yönetmek için bir dizi ilişki türü sunar. Bu ilişkiler, veritabanındaki verileri daha verimli bir şekilde sorgulamanıza ve birbirine bağlamanızı sağlar. Laravel'de kullanabileceğiniz temel ilişki türleri şunlardır:

### 1. **`belongsTo`**

- **Açıklama:** Bu ilişki, "ters ilişki" olarak da bilinir. İki model arasında **"birçok-to-bir"** ilişkisini tanımlar.
- **Örnek Kullanım:** Bir `Post` modelinin bir `User` modeline ait olması.

```php
public function user(): BelongsTo
{
    return $this->belongsTo(User::class);
}
```

### 2. **`hasOne`**

- **Açıklama:** Bu ilişki, "bir-to-bir" ilişkiyi tanımlar. Yani bir model, diğer bir modele tek bir örnekle bağlanır.
- **Örnek Kullanım:** Bir `User` modelinin bir `Profile` modeline sahip olması.

```php
public function profile(): HasOne
{
    return $this->hasOne(Profile::class);
}
```

### 3. **`hasMany`**

- **Açıklama:** Bu ilişki, "bir-to-çok" ilişkiyi tanımlar. Bir modelin birden fazla ilgili modele sahip olduğunu belirtir.
- **Örnek Kullanım:** Bir `User` modelinin birden fazla `Post` modeline sahip olması.

```php
public function posts(): HasMany
{
    return $this->hasMany(Post::class);
}
```

### 4. **`belongsToMany`**

- **Açıklama:** Bu ilişki, "çok-to-çok" ilişkiyi tanımlar. İki modelin, genellikle ara bir tablo (pivot tablo) aracılığıyla birbirine bağlanmasını sağlar.
- **Örnek Kullanım:** Bir `User` modelinin birden fazla `Role` modeline sahip olması ve her rolün birden fazla kullanıcıya ait olması.

```php
public function roles(): BelongsToMany
{
    return $this->belongsToMany(Role::class);
}
```

### 5. **`hasManyThrough`**

- **Açıklama:** Bu ilişki, "iki ara model üzerinden çok-to-bir" ilişkisini tanımlar. Bir model, başka bir model ile ilişkili olabilir, ancak bu ilişkiyi geçici olarak bir üçüncü model üzerinden kurar.
- **Örnek Kullanım:** Bir `Country` modelinin, `Post` modelini `User` modeli üzerinden dolaylı olarak ilişkili olarak alması.

```php
public function posts(): HasManyThrough
{
    return $this->hasManyThrough(Post::class, User::class);
}
```

### 6. **`morphOne`**

- **Açıklama:** Polimorfik ilişkilerde, "bir-to-bir" ilişkisi için kullanılır. Bu ilişki, bir modelin başka bir modelin örneğini tek bir alanda tutmasını sağlar.
- **Örnek Kullanım:** Bir `Post` modelinin bir `Image` modeline ait olması, ancak `Image` modelinin farklı türlerdeki modellerle ilişkili olabilmesi.

```php
public function image(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable');
}
```

### 7. **`morphMany`**

- **Açıklama:** Polimorfik ilişkilerde, "bir-to-çok" ilişkisi için kullanılır. Bir model, başka bir modelin birden fazla örneğiyle ilişkili olabilir.
- **Örnek Kullanım:** Bir `Post` modelinin birden fazla `Comment` modeline ait olması, ancak `Comment` modeli farklı türlerdeki modellerle ilişkili olabilmesi.

```php
public function comments(): MorphMany
{
    return $this->morphMany(Comment::class, 'commentable');
}
```

### 8. **`morphTo`**

- **Açıklama:** Polimorfik ilişkilerde, bir modelin farklı model türleriyle ilişkili olabileceğini belirtir. Polimorfik ilişkilerin tersidir.
- **Örnek Kullanım:** Bir `Comment` modelinin, farklı model türlerine ait olabilmesi (örneğin, `Post`, `Video` vs.).

```php
public function commentable(): MorphTo
{
    return $this->morphTo();
}
```

### 9. **`morphedByMany`**

- **Açıklama:** Polimorfik çok-to-çok ilişkilerde kullanılır. Bir model birden fazla modelle çok-to-çok ilişki kurar.
- **Örnek Kullanım:** Bir `Tag` modelinin birçok farklı modelle (örneğin, `Post`, `Video` vs.) çoklu ilişkisi olması.

```php
public function posts(): MorphedByMany
{
    return $this->morphedByMany(Post::class, 'taggable');
}
```

### 10. **`morphToMany`**

- **Açıklama:** Polimorfik çok-to-çok ilişkilerde, bir modelin birden fazla türdeki modele ait olmasını ifade eder.
- **Örnek Kullanım:** Bir `Post` modelinin birden fazla `Tag` modeline ait olması.

```php
public function tags(): MorphToMany
{
    return $this->morphToMany(Tag::class, 'taggable');
}
```

---

### Özet Tablosu

| İlişki Tipi      | Açıklama                                                                 | Kullanım Örneği                  |
| ---------------- | ------------------------------------------------------------------------ | -------------------------------- |
| `belongsTo`      | Bir modelin, diğer modelin "tek" bir örneğine ait olduğu ilişki.         | `Post` -> `User`                 |
| `hasOne`         | Bir modelin, diğer modelin "tek" bir örneğiyle ilişkisi.                 | `User` -> `Profile`              |
| `hasMany`        | Bir modelin, diğer modelin birden fazla örneğiyle ilişkisi.              | `User` -> `Post`                 |
| `belongsToMany`  | Bir modelin, diğer modelin birden fazla örneğiyle çoklu ilişkisi.        | `User` -> `Role`                 |
| `hasManyThrough` | Bir modelin, başka bir model üzerinden ilişkili olduğu modelle ilişkisi. | `Country` -> `Post` -> `User`    |
| `morphOne`       | Polimorfik "bir-to-bir" ilişki.                                          | `Post` -> `Image`                |
| `morphMany`      | Polimorfik "bir-to-çok" ilişki.                                          | `Post` -> `Comment`              |
| `morphTo`        | Polimorfik ilişkiyi tersine çevirir.                                     | `Comment` -> `Post` veya `Video` |
| `morphedByMany`  | Polimorfik çok-to-çok ilişki (geri).                                     | `Tag` -> `Post`, `Video`         |
| `morphToMany`    | Polimorfik çok-to-çok ilişki.                                            | `Post` -> `Tag`                  |

### Hangi İlişkiyi Ne Zaman Kullanmalısınız?

- **`belongsTo`** ve **`hasMany`**: Genellikle veritabanındaki "tek" ve "çok" ilişkilerini tanımlamak için kullanılır.
- **`belongsToMany`**: İki model arasında çoktan çoğa ilişki kurmak için.
- **`hasManyThrough`**: Bir modelin, başka bir model üzerinden ilişkili olduğu bir veri yapısı kurmak için.
- **Polimorfik İlişkiler (`morphOne`, `morphMany`, `morphedByMany`, `morphToMany`)**: Çeşitli model türlerinin aynı ilişkileri paylaşması gereken durumlarda kullanılır.

## belongsTo vs hasOne?

`belongsTo` ve `hasOne`, Laravel Eloquent ORM'deki ilişki türleridir ve her ikisi de **"bir-to-bir"** ilişkileri tanımlar. Ancak, bunlar arasındaki temel farklar, ilişkilerin nasıl tanımlandığı ve her iki modeldeki yönelimler ile ilgilidir.

### 1. **`belongsTo`**

- **Yön**: Bu ilişki, bir modelin başka bir modelin **"bir örneğine"** ait olduğunu belirtir. Yani, bir modelin **"bağlı olduğu"** modelin bir örneğini belirtir.
- **Kullanım**: `belongsTo`, genellikle **"bağlı"** modelde bir yabancı anahtar (foreign key) olduğunu ifade eder.
- **Örnek**: Bir `Post` modelinin bir `User` modeline ait olduğunu düşünün. Bu durumda `Post`, `User` modeline **aittir**.

  ```php
  // Post modelinde
  public function user(): BelongsTo
  {
      return $this->belongsTo(User::class);
  }
  ```

- **Veritabanı İlişkisi**: Bu ilişkiyi kullanırken, **"Post"** tablosunda genellikle `user_id` gibi bir yabancı anahtar bulunur ve bu anahtar, **"User"** tablosundaki bir kaydı işaret eder.

### 2. **`hasOne`**

- **Yön**: Bu ilişki, bir modelin başka bir modelin **"tek bir örneğiyle"** ilişkili olduğunu belirtir. Yani, bir model **"sahip"tir**.
- **Kullanım**: `hasOne` genellikle bir modelin **"sahip olduğu"** bir diğer modelin örneğini tanımlar.
- **Örnek**: Bir `User` modelinin bir `Profile` modeline sahip olduğunu düşünün. Bu durumda `User`, `Profile` modeline **sahiptir**.

  ```php
  // User modelinde
  public function profile(): HasOne
  {
      return $this->hasOne(Profile::class);
  }
  ```

- **Veritabanı İlişkisi**: Bu ilişkiyi kullanırken, **"Profile"** tablosunda genellikle `user_id` gibi bir yabancı anahtar bulunur ve bu anahtar, **"User"** tablosundaki bir kaydı işaret eder.

### Temel Farklar

| Özellik               | `belongsTo`                                                                          | `hasOne`                                                                                |
| --------------------- | ------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------- |
| **Yön**               | "Bağlı" modeldeki yabancı anahtara işaret eder (örneğin, `Post` -> `User`)           | "Sahip" modelin bir örneğine işaret eder (örneğin, `User` -> `Profile`)                 |
| **Yabancı Anahtar**   | Yabancı anahtar, bağlı modelde olur (örneğin, `Post` tablosunda `user_id` bulunur)   | Yabancı anahtar, sahip modelde olur (örneğin, `Profile` tablosunda `user_id` bulunur)   |
| **Kullanım Alanı**    | "Birçok-to-bir" ilişkilerde, bağlı modelde (örneğin, bir `Post` bir `User`'a aittir) | "Bir-to-bir" ilişkilerde, sahip modelde (örneğin, bir `User`'ın bir `Profile`'ı vardır) |
| **Veritabanı Yapısı** | Yabancı anahtar (`user_id`) ilişkili (bağlı) modelde yer alır.                       | Yabancı anahtar (`user_id`) sahip (ilişkili) modelde yer alır.                          |

### Uygulama Örnekleri

#### Örnek 1: `Post` ve `User` İlişkisi (belongsTo)

- **Yön**: Bir `Post` bir `User`'a ait olduğu için, `Post` modelinde `belongsTo` kullanılır.
- **Veritabanı**: `Post` tablosunda `user_id` adlı bir yabancı anahtar olur.

```php
class Post extends Model
{
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

#### Örnek 2: `User` ve `Profile` İlişkisi (hasOne)

- **Yön**: Bir `User` modelinin bir `Profile` modeline sahip olduğu için, `User` modelinde `hasOne` kullanılır.
- **Veritabanı**: `Profile` tablosunda `user_id` adlı bir yabancı anahtar olur.

```php
class User extends Model
{
    public function profile(): HasOne
    {
        return $this->hasOne(Profile::class);
    }
}
```

### Hangi Durumda `belongsTo`, Hangi Durumda `hasOne` Kullanılır?

- **`belongsTo` kullanın**:
  - Bir model, başka bir modelin **"bağımlısı"** olduğunda, yani **"yabancı anahtar"** modelde bulunduğunda.
  - Örneğin: `Post` modeli bir `User`'a ait olduğunda (`Post`'un `user_id`'si var).
- **`hasOne` kullanın**:
  - Bir model, başka bir modelin **"sahibi"** olduğunda, yani **"yabancı anahtar"** diğer modelde bulunduğunda.
  - Örneğin: `User` modelinin bir `Profile`'ı olduğunda (`Profile`'ın `user_id`'si var).

### Sonuç

- **`belongsTo`**: Bir model başka bir modele ait olduğunda (yabancı anahtar "ilişkili" modelde).
- **`hasOne`**: Bir model başka bir modeli sahipleniyor (yabancı anahtar "sahip" modelde).

Her iki ilişki de "bir-to-bir" ilişkiler olup, ilişkilerin yönünü belirlemek önemlidir.
