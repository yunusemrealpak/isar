---
title: Schema
---

# Schema

Uygulamanızın verilerini Isar ile sakladığınızda, koleksiyonlarla ilgileniyorsunuz. Bir koleksiyon, ilişkili Isar veritabanındaki bir veritabanı tablosu gibidir ve yalnızca tek türde Dart nesnesi içerebilir. Her koleksiyon nesnesi, karşılık gelen koleksiyondaki bir veri satırını temsil eder.

Bir koleksiyon tanımına "schema" denir. Isar Generator sizin için ağır işi yapar ve koleksiyonu kullanmak için ihtiyacınız olan çoğu kodu üretir.

## Bir koleksiyonun anatomisi

Her bir Isar koleksiyonunu, bir sınıfı `@collection` veya `@Collection()` ile etiketleyerek tanımlarsınız. Bir Isar koleksiyonu, veritabanındaki karşılık gelen tablodaki her sütun için alanlar içerir, bunların arasında birincil anahtarı oluşturan bir alan da bulunur.

Aşağıdaki kod, ID, ad ve soyad için sütunlar içeren bir `User` tablosunu tanımlayan basit bir koleksiyonun bir örneğidir:

```dart
@collection
class User {
  Id? id;

  String? firstName;

  String? lastName;
}
```

:::tip
Bir alanı kalıcı hale getirmek için, Isar'ın ona erişimi olması gerekir. Isar'ın bir alana erişimini, onu public yaparak veya getter ve setter metodları sağlayarak garanti altına alabilirsiniz.
:::

Koleksiyonu özelleştirmek için birkaç isteğe bağlı parametre bulunmaktadır:

| Config        | Açıklama                                                                                                      |
| ------------- | ---------------------------------------------------------------------------------------------------------------- |
| `inheritance` | Üst sınıfların ve mixin'lerin alanlarının Isar'da saklanıp saklanmayacağını kontrol eder. Varsayılan olarak etkindir.                  |
| `accessor`    | Varsayılan koleksiyon erişimcisini yeniden adlandırmanıza izin verir (örneğin, `Contact` koleksiyonu için `isar.contacts`). |
| `ignore`      | Belirli özelliklerin yok sayılmasına izin verir. Bu, üst sınıflar için de geçerlidir.                                  |

### Isar Id

Her koleksiyon sınıfı, bir nesneyi eşsiz bir şekilde tanımlayan `Id` tipinde bir id özelliği tanımlamalıdır. `Id`, sadece Isar Generator'ün id özelliğini tanımasına izin veren `int` için bir takma addır.

Isar, id alanlarını otomatik olarak indeksler, bu da nesneleri id'lerine göre verimli bir şekilde almanıza ve değiştirmenize olanak tanır.

Id'leri kendiniz belirleyebilir veya Isar'dan otomatik artan bir id ataması isteyebilirsiniz. Eğer `id` alanı `null` ve `final` değilse, Isar otomatik artan bir id atayacaktır. Eğer boş olmayan otomatik artan bir id istiyorsanız, `null` yerine `Isar.autoIncrement` kullanabilirsiniz.

:::tip
Otomatik artan id'ler, bir nesne silindiğinde yeniden kullanılmaz. Otomatik artan id'leri sıfırlamanın tek yolu veritabanını temizlemektir.
:::

### Koleksiyonları ve alanları yeniden adlandırma

Varsayılan olarak, Isar sınıf adını koleksiyon adı olarak kullanır. Benzer şekilde, Isar alan adlarını veritabanında sütun adları olarak kullanır. Eğer bir koleksiyonun veya alanın farklı bir adı olmasını istiyorsanız, `@Name` notasyonunu ekleyin. Aşağıdaki örnek, koleksiyon ve alanlar için özel adları göstermektedir:

```dart
@collection
@Name("User")
class MyUserClass1 {

  @Name("id")
  Id myObjectId;

  @Name("firstName")
  String theFirstName;

  @Name("lastName")
  String familyNameOrWhatever;
}
```

Özellikle, veritabanında zaten oluşturulmuş olan Dart alanlarını veya sınıflarını yeniden adlandırmak istiyorsanız, `@Name` notasyonunu kullanmayı düşünmelisiniz. Aksi takdirde, veritabanı alanı veya koleksiyonu silip yeniden oluşturacaktır.

### Alanları Yoksayma

Isar, bir koleksiyon sınıfının tüm public alanlarını kalıcı hale getirir. Bir özellik veya getter'ı `@ignore` ile etiketleyerek, onu kalıcılıktan çıkarabilirsiniz, aşağıdaki kod parçasında gösterildiği gibi:

```dart
@collection
class User {
  Id? id;

  String? firstName;

  String? lastName;

  @ignore
  String? password;
}
```

Bir koleksiyonun, bir üst koleksiyondan alanları miras aldığı durumlarda, genellikle `@Collection` notasyonunun `ignore` özelliğini kullanmak daha kolaydır:

```dart
@collection
class User {
  Image? profilePicture;
}

@Collection(ignore: {'profilePicture'})
class Member extends User {
  Id? id;

  String? firstName;

  String? lastName;
}
```

Bir koleksiyon, Isar tarafından desteklenmeyen bir tipe sahip bir alan içeriyorsa, bu alanı yoksaymanız gerekir.

:::warning
Isar nesnelerinde, kalıcı hale getirilmeyen bilgilerin saklanmasının iyi bir uygulama olmadığını unutmayın.
:::

## Desteklenen tipler

Isar aşağıdaki veri tiplerini destekler:

- `bool`
- `byte`
- `short`
- `int`
- `float`
- `double`
- `DateTime`
- `String`
- `List<bool>`
- `List<byte>`
- `List<short>`
- `List<int>`
- `List<float>`
- `List<double>`
- `List<DateTime>`
- `List<String>`

Ek olarak, gömülü nesneler ve enumlar da desteklenmektedir. Bunları aşağıda ele alacağız.

## byte, short, float

Birçok kullanım durumu için, 64-bit tamsayının veya double'ın tam aralığına ihtiyacınız olmayabilir. Isar, daha küçük sayıları saklarken yer ve bellek tasarrufu yapmanızı sağlayan ek türler destekler.

| Type       | Size in bytes | Range                                                   |
| ---------- | ------------- | ------------------------------------------------------- |
| **byte**   | 1             | 0 to 255                                                |
| **short**  | 4             | -2,147,483,647 to 2,147,483,647                         |
| **int**    | 8             | -9,223,372,036,854,775,807 to 9,223,372,036,854,775,807 |
| **float**  | 4             | -3.4e38 to 3.4e38                                       |
| **double** | 8             | -1.7e308 to 1.7e308                                     |

Ek sayı türleri, yerel Dart türleri için sadece takma adlardır, bu yüzden örneğin `short` kullanmak, `int` kullanmakla aynı şekilde çalışır.

İşte yukarıdaki türlerin tümünü içeren bir örnek koleksiyon:

```dart
@collection
class TestCollection {
  Id? id;

  late byte byteValue;

  short? shortValue;

  int? intValue;

  float? floatValue;

  double? doubleValue;
}
```

Tüm sayı türleri aynı zamanda listelerde de kullanılabilir. Baytları saklamak için, `List<byte>` kullanmalısınız.

## Null Olabilen Tipler

Isar'da nullability'nin nasıl çalıştığını anlamak esastır: Sayı türlerinin ayrı bir `null` temsili **YOKTUR**. Bunun yerine, belirli bir değer kullanılır:

| Type       | VM            |
| ---------- | ------------- |
| **short**  | `-2147483648` |
| **int**    |  `int.MIN`    |
| **float**  | `double.NaN`  |
| **double** |  `double.NaN` |

`bool`, `String` ve `List`'in ayrı bir null temsili vardır.

Bu davranış, performans iyileştirmelerini sağlar ve alanlarınızın nullability'sini, migration gerektirmeksizin veya `null` değerleriyle başa çıkmak için özel kod kullanmaksızın serbestçe değiştirmenize olanak tanır.

:::warning
`byte` türü null değerleri desteklemez.
:::

## DateTime

Isar, tarihlerinizin zaman dilimi bilgisini saklamaz. Bunun yerine, `DateTime`'ları saklamadan önce UTC'ye dönüştürür. Isar, tüm tarihleri yerel saat olarak döndürür.

`DateTime`'lar mikrosaniye hassasiyetiyle saklanır. Tarayıcılarda, JavaScript sınırlamaları nedeniyle yalnızca milisaniye hassasiyeti desteklenir.

## Enum

Isar, enum'ları diğer Isar türleri gibi saklamanıza ve kullanmanıza izin verir. Ancak, Isar'ın enum'u diske nasıl temsil etmesi gerektiğine karar vermelisiniz. Isar, dört farklı stratejiyi destekler:

| EnumType    | Açıklama                                                                                         |
| ----------- | --------------------------------------------------------------------------------------------------- |
| `ordinal`   | Enum'un indeksi `byte` olarak saklanır. Bu çok verimlidir ancak nullable enum'lara izin vermez |
| `ordinal32` | Enum'un indeksi `short` (4-byte tamsayı) olarak saklanır.                                     |
| `name`      | Enum adı `String` olarak saklanır.                                                            |
| `value`     | Bir özel özellik, enum değerini almak için kullanılır.                                               |

:::warning
`ordinal` ve `ordinal32`, enum değerlerinin sırasına bağlıdır. Sırayı değiştirirseniz, mevcut veritabanları yanlış değerler döndürecektir.
:::

Her strateji için bir örnek görelim.

```dart
@collection
class EnumCollection {
  Id? id;

  @enumerated // same as EnumType.ordinal
  late TestEnum byteIndex; // cannot be nullable

  @Enumerated(EnumType.ordinal)
  late TestEnum byteIndex2; // cannot be nullable

  @Enumerated(EnumType.ordinal32)
  TestEnum? shortIndex;

  @Enumerated(EnumType.name)
  TestEnum? name;

  @Enumerated(EnumType.value, 'myValue')
  TestEnum? myValue;
}

enum TestEnum {
  first(10),
  second(100),
  third(1000);

  const TestEnum(this.myValue);

  final short myValue;
}
```

Elbette, Enum'lar listelerde de kullanılabilir.

## Gömülü nesneler

Koleksiyon modelinizde iç içe nesnelerin olması sıklıkla faydalıdır. Nesneleri ne kadar derin iç içe yerleştirebileceğinize dair bir sınır yoktur. Ancak, derin iç içe bir nesneyi güncellerken, tüm nesne ağacını veritabanına yazmanız gerekeceğini unutmayın.

```dart
@collection
class Email {
  Id? id;

  String? title;

  Recepient? recipient;
}

@embedded
class Recepient {
  String? name;

  String? address;
}
```

Gömülü nesneler nullable olabilir ve diğer nesnelerden extend olabilir. Tek gereklilik, `@embedded` ile etiketlenmeleri ve gerekli parametreleri olmayan bir varsayılan constructor'a sahip olmalarıdır.
