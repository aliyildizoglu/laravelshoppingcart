# Laravel 5 & 6, 7 & 9 Alışveriş Sepeti

[![Build Durumu](https://travis-ci.org/darryldecode/laravelshoppingcart.svg?branch=master)](https://travis-ci.org/darryldecode/laravelshoppingcart)
[![Toplam İndirmeler](https://poser.pugx.org/darryldecode/cart/d/total.svg)](https://packagist.org/packages/darryldecode/cart)
[![Lisans](https://poser.pugx.org/darryldecode/cart/license.svg)](https://packagist.org/packages/darryldecode/cart)
Laravel Framework için Bir Alışveriş Sepeti Uygulaması

## HIZLI KISIMSEL DEMO

Demo: https://shoppingcart-demo.darrylfernandez.com/cart

Demo'nun Git deposu: https://github.com/darryldecode/laravelshoppingcart-demo

## KURULUM

Paketi [Composer](http://getcomposer.org/) aracılığıyla kurun.

Laravel 5.1~ için:
`composer require "darryldecode/cart:~2.0"`

Laravel 5.5, 5.6 veya 5.7~, 9 için:

`composer require "darryldecode/cart:~4.0"` veya
`composer require "darryldecode/cart"`

## YAPILANDIRMA

1. config/app.php dosyasını açın ve bu satırı Hizmet Sağlayıcılar Dizisine ekleyin.

```php
Darryldecode\Cart\CartServiceProvider::class

```

2. config/app.php dosyasını açın ve bu satırı Takma Adlara (Aliases) ekleyin

```php
  'Cart' => Darryldecode\Cart\Facades\CartFacade::class
```

3. İsteğe bağlı yapılandırma dosyası (tam kontrol sahibi olmayı planlıyorsanız)

```php
php artisan vendor:publish --provider="Darryldecode\Cart\CartServiceProvider" --tag="config"
```

## HOW TO USE

- [Quick Usage](#usage-usage-example)
- [Usage](#usage)
- [Conditions](#conditions)
- [Items](#items)
- [Associating Models](#associating-models)
- [Instances](#instances)
- [Exceptions](#exceptions)
- [Events](#events)
- [Format Response](#format)
- [Examples](#examples)
- [Using Different Storage](#storage)
- [License](#license)

## Quick Usage Example

```php
// Ürün Model İlişkilendirmesi ve Kullanıcı oturum bağlamasıyla Hızlı Kullanım

$Urun = Urun::find($urunId); // varsayılan olarak id, ad, açıklama ve fiyat özellikleri olan bir Urun modeliniz olduğunu varsayalım
$rowId = 456; // benzersiz bir() satır kimliği oluşturun
$kullaniciID = 2; // sepet içeriğini bağlamak için kullanıcı kimliği

// ürünü sepete ekle
\Cart::session($kullaniciID)->add(array(
    'id' => $rowId,
    'name' => $Urun->ad,
    'price' => $Urun->fiyat,
    'quantity' => 4,
    'attributes' => array(),
    'associatedModel' => $Urun
));

// sepet üzerindeki öğeyi güncelle
\Cart::session($kullaniciID)->update($rowId,[
	'quantity' => 2,
	'price' => 98.67
]);

// sepetten öğeyi sil
\Cart::session($kullaniciID)->remove($rowId);

// sepet öğelerini görüntüle
$urunler = \Cart::getContent();
foreach($urunler as $satir) {

	echo $satir->id; // satır kimliği
	echo $satir->name;
	echo $satir->qty;
	echo $satir->price;
	
	echo $satir->associatedModel->id; // modelinizin sahip olduğu özellikler
    echo $satir->associatedModel->ad; // modelinizin sahip olduğu özellikler
    echo $satir->associatedModel->açıklama; // modelinizin sahip olduğu özellikler
}

// TAM KULLANIM İÇİN AŞAĞIYA BAKIN..

```

## Usage

### IMPORTANT NOTE!

Varsayılan olarak, sepet, sepet verilerini tutan varsayılan bir oturum anahtarı olan "sessionKey"e sahiptir. Bu aynı zamanda bir sepeti belirli bir kullanıcıya bağlamak için kullanabileceğiniz benzersiz bir sepet tanımlayıcısı olarak hizmet eder. Bu varsayılan oturum anahtarını geçersiz kılmak için, HERHANGİ BİR BAŞKA YÖNTEM ÇAĞRISINDAN ÖNCE \Cart::session($sessionKey) yöntemini basitçe çağırmanız gerekir.

Örnek:

```php
$userId // mevcut giriş yapmış kullanıcının kimliği

// Bu, sepete yalnızca belirli bir kullanıcının sepet verilerine ihtiyacımız veya bunları manipüle etmemiz gerektiğini söyler.
// Bu, $userId olması gerekmez, kullanıcıyı veya müşteriyi temsil eden herhangi bir benzersiz anahtar kullanabilirsiniz.
// Temel olarak, bu sepeti belirli bir kullanıcıya bağlar.
\Cart::session($userId);

// ardından normal sepet kullanımı gelir
\Cart::add();
\Cart::update();
\Cart::remove();
\Cart::condition($condition1);
\Cart::getTotal();
\Cart::getSubTotal();
\Cart::getSubTotalWithoutConditions();
\Cart::addItemCondition($productID, $coupon101);
// ve benzeri..

```

Aşağıda Daha Fazla Örnek:

Sepete Ürün Ekleme: **Cart::add()**

Sepete ürün eklemenin birkaç yolu bulunmaktadır, aşağıda görebilirsiniz:

```php
/**
 * Sepete ürün ekler, dizi veya çok boyutlu dizi olabilir
 *
 * @param string|array $id
 * @param string $name
 * @param float $price
 * @param int $quantity
 * @param array $attributes
 * @param CartCondition|array $conditions
 * @return $this
 * @throws InvalidItemException
 */

 # HER ZAMAN HERHANGİ BİR SEPET FONKSİYONUNU ÇAĞIRMADAN ÖNCE SEPETİ BİR KULLANICIYA BAĞLAMAYI UNUTMAYIN
 # BÖYLECE SEPET, HANGİ KULLANICININ SEPET VERİSİNİ MANİPÜLE ETMEK İSTEDİĞİNİ BİLECEKTİR. YUKARIDAKİ ÖNEMLİ NOTU GÖRÜNTÜLEYİN.
 # ÖRNEK: \Cart::session($userId); ardından normal sepet kullanımına devam edebilirsiniz.

 # NOT:
 # Yeni bir öğe sepete eklerken 'id' alanı Model Kimliği (örneğin Ürün Kimliği) için değildir.
 # Bunun yerine, her benzersiz ürün veya kendi benzersiz fiyatına sahip ürün için benzersiz bir kimlik eklemelisiniz,
 # çünkü bu, sepeti güncelleme ve sepetteki her bir öğenin nasıl hesaplandığı ve miktarlarının nasıl ayrıldığı konusunda kullanılır.
 # Tam esneklik için model_id'yi bir öznitelik olarak ekleyebilirsiniz.
 # Örnek olarak, aynı ürünleri farklı özellikler ve fiyatlarla sepete eklemek istiyorsanız.
 # Ürün Kimliğini sepetin 'id' alanına koyarsanız, bunun yerine miktarın artması sonucu oluşur
 # benzersiz bir ürün olarak eklemek yerine.

// En basit haliyle öğe ekleme
Cart::add(455, 'Örnek Ürün', 100.99, 2, array());

// Dizi formatı
Cart::add(array(
    'id' => 456, // benzersiz satır kimliği
    'name' => 'Örnek Ürün',
    'price' => 67.99,
    'quantity' => 4,
    'attributes' => array()
));

// Aynı anda birden çok öğe eklemek
Cart::add(array(
  array(
      'id' => 456,
      'name' => 'Örnek Ürün 1',
      'price' => 67.99,
      'quantity' => 4,
      'attributes' => array()
  ),
  array(
      'id' => 568,
      'name' => 'Örnek Ürün 2',
      'price' => 69.25,
      'quantity' => 4,
      'attributes' => array(
        'size' => 'L',
        'color' => 'mavi'
      )
  ),
));

// Sepete öğeler eklemek için belirli bir kullanıcıya öğeler eklemek
$userId = auth()->user()->id; // veya kullanıcı kimliğini temsil eden herhangi bir dize
Cart::session($userId)->add(array(
    'id' => 456, // benzersiz satır kimliği
    'name' => 'Örnek Ürün',
    'price' => 67.99,
    'quantity' => 4,
    'attributes' => array(),
    'associatedModel' => $Ürün
));

// NOT:
// Lütfen unutmayın ki bir öğe sepete eklerken, "id" benzersiz olmalıdır çünkü satır kimliği olarak hizmet eder.
// Aynı ID'yi sağlarsanız, işlemin miktarını güncellemek olarak kabul eder,
// sepet öğesi yinelenmelerini önlemek için
```
Bir öğeyi sepet üzerinde güncelleme: Cart::update()

Sepet üzerindeki bir öğeyi güncellemek çok basittir:

```php
/**
 * Sepeti günceller
 *
 * @param $id (öğe Kimliği)
 * @param array $veri
 *
 * $veri, ilişkilendirilmiş bir dizi olacak, tüm verileri iletmek zorunda değilsiniz, sadece güncellemek istediğiniz öğenin
 * anahtar-değer çiftini iletebilirsiniz
 */

Cart::update(456, array(
  'name' => 'Yeni Ürün Adı', // yeni öğe adı
  'price' => 98.67, // yeni öğe fiyatı, fiyat ayrıca '98.67' gibi bir dize formatında da olabilir
));

// ayrıca bir ürünün miktarını güncellemek isteyebilirsiniz
Cart::update(456, array(
  'quantity' => 2, // böylece mevcut ürünün miktarı 4 ise, 2 daha eklenir ve bu 6'ya ulaşır
));

// ayrıca ürünün miktarını azaltarak güncellemek isteyebilirsiniz, bunu aşağıdaki gibi yaparsınız:
Cart::update(456, array(
  'quantity' => -1, // böylece mevcut ürünün miktarı 4 ise, 1 çıkarır ve sonuç olarak 3 olur
));

// NOT: Gördüğünüz gibi, varsayılan olarak miktar güncellemesi, mevcut değerine göre göreceli olarak yapılır
// mevcut miktar değerini artırmak veya azaltmak yerine miktarı tamamen değiştirmek istiyorsanız
// miktar değeri olarak bir dizi geçebilirsiniz, örneğin:
Cart::update(456, array(
  'quantity' => array(
      'relative' => false,
      'value' => 5
  ),
));
// yukarıdaki kodla, relative değeri false olarak işaretlendiğinde, öğenin miktarı önce 2 ise, şimdi 5 olur 2 + 5 yerine
// göreceli olarak güncellendiğinde 7 olur.

// belirli bir kullanıcı için sepeti güncelleme
$userId = auth()->user()->id; // veya kullanıcı kimliğini temsil eden herhangi bir dize
Cart::session($userId)->update(456, array(
  'name' => 'Yeni Ürün Adı', // yeni öğe adı
  'price' => 98.67, // yeni öğe fiyatı, fiyat ayrıca '98.67' gibi bir dize formatında da olabilir
));

```
Bir öğeyi sepetten kaldırma: Cart::remove()

Bir öğeyi sepetten kaldırmak çok kolaydır:

```php
/**
 * Sepetteki bir öğeyi öğe Kimliği ile kaldırır
 *
 * @param $id
 */

Cart::remove(456);

// Belirli bir kullanıcının sepetinden öğe kaldırma
$userId = auth()->user()->id; // veya kullanıcı kimliğini temsil eden herhangi bir dize
Cart::session($userId)->remove(456);

```

Bir sepet öğesini almak için: **Cart::get()**

```php
/**
 * Bir öğeyi öğe Kimliği ile sepetten alır
 * Eğer öğe Kimliği bulunamazsa, null döner
 *
 * @param $itemId
 * @return null|array
 */

$itemId = 456;

Cart::get($itemId);

// Aynı zamanda Öğenin miktarıyla çarpılmış toplam fiyatını da alabilirsiniz, aşağıda görüldüğü gibi:
$toplamFiyat = Cart::get($itemId)->getPriceSum();

// Belirli bir kullanıcının sepetinde öğeyi öğe Kimliği ile almak için:
$userId = auth()->user()->id; // veya kullanıcı kimliğini temsil eden herhangi bir dize
Cart::session($userId)->get($itemId);

```

Sepetin içeriğini ve öğe sayısını almak için: **Cart::getContent()**

```php

/**
 * Sepeti alır
 *
 * @return CartCollection
 */

$cartCollection = Cart::getContent();

// NOT: Çünkü sepet koleksiyonu Laravel'in Koleksiyonunu genişletir
// Zaten bildiğiniz Laravel Koleksiyonu yöntemlerini kullanabilirsiniz
// Aşağıda bazı yöntemlerini görebilirsiniz:

// sepet içeriğini sayma
$cartCollection->count();

// dönüşümler
$cartCollection->toArray();
$cartCollection->toJson();

// Belirli bir kullanıcının sepet içeriğini almak için
$userId = auth()->user()->id; // veya kullanıcı kimliğini temsil eden herhangi bir dize
Cart::session($userId)->getContent($itemId);

```

Sepetin boş olup olmadığını kontrol etmek için: **Cart::isEmpty()**

```php
/**
* Sepetin boş olup olmadığını kontrol eder
*
* @return bool
*/
Cart::isEmpty();

// Belirli bir kullanıcının sepet içeriğinin boş olup olmadığını kontrol etme
$userId = auth()->user()->id; // veya kullanıcı kimliğini temsil eden herhangi bir dize
Cart::session($userId)->isEmpty();

```

Sepetin toplam miktarını almak için: **Cart::getTotalQuantity()**

```php
/**
* Sepetteki toplam öğe miktarını alır
*
* @return int
*/
$cartTotalQuantity = Cart::getTotalQuantity();

// Belirli bir kullanıcı için
$cartTotalQuantity = Cart::session($userId)->getTotalQuantity();

```

Sepetin ara toplamını almak için: **Cart::getSubTotal()**

```php
/**
* Sepetin ara toplamını alır
*
* @return float
*/
$subTotal = Cart::getSubTotal();

// Belirli bir kullanıcı için
$subTotal = Cart::session($userId)->getSubTotal();

// Koşullar olmadan sepet ara toplamını almak için
$subTotalWithoutConditions = Cart::getSubTotalWithoutConditions();

```

 Koşullar olmaksızın sepetin ara toplamını alır **Cart::getSubTotalWithoutConditions()**

```php
/**
* Koşullar olmaksızın sepetin ara toplamını alır
*
* @param bool $formatted
* @return float
*/
$subTotalWithoutConditions = Cart::getSubTotalWithoutConditions();

// Belirli bir kullanıcı için
$subTotalWithoutConditions = Cart::session($userId)->getSubTotalWithoutConditions();

```

Sepet toplamını alır: **Cart::getTotal()**

```php
/**
 * Koşulların zaten uygulandığı yeni toplamı alır
 *
 * @return float
 */
$total = Cart::getTotal();

// Belirli bir kullanıcı için
$total = Cart::session($userId)->getTotal();

```
Sepeti Temizleme: **Cart::clear()**

```php
/**
* Sepeti temizler
*
* @return void
*/
Cart::clear();
Cart::session($userId)->clear();

```

## Conditions

* Laravel Alışveriş Sepeti koşulları destekler.
* Koşullar, (kuponlar, indirimler, indirimli satışlar, ürün bazlı indirimler vb.) açısından çok kullanışlıdır.
* Aşağıdaki örneği dikkatlice inceleyin, nasıl kullanılacağını görebilirsiniz.

Koşullar aşağıdaki şekillerde eklenir:

1.) Tüm Sepet Değeri Temeline Göre

2.) Her Öğe Temeline Göre

Öncelikle Sepet Temeline Bir Koşul Ekleyelim:

Sepet üzerine bir koşul eklemek için birkaç yöntem vardır:
NOT:

Sepet temelinde bir koşul eklerken, 'target' değeri 'subtotal' veya 'total' olmalıdır.
Eğer 'target' "subtotal" ise, bu koşul ara toplama uygulanacaktır.
Eğer 'target' "total" ise, bu koşul toplama uygulanacaktır.
Koşulları eklerken işlem sırası, eklediğiniz koşulların sırasına göre değişecektir.

Ayrıca, koşulları eklerken 'value' alanı hesaplamanın temelini oluşturacaktır. Bu sırayı değiştirebilirsiniz
CartCondition içinde 'order' parametresini ekleyerek.

```php

// Sepet temeline tek bir koşul eklemek
$condition = new \Darryldecode\Cart\CartCondition(array(
    'name' => 'KDV 12.5%',
    'type' => 'tax',
    'target' => 'subtotal', // bu koşul, getSubTotal() çağrıldığında sepetin ara toplamına uygulanır.
    'value' => '12.5%',
    'attributes' => array(
        'description' => 'Katma Değer Vergisi',
        'more_data' => 'burada daha fazla veri'
    )
));

Cart::condition($condition);
Cart::session($userId)->condition($condition); // belirli bir kullanıcının sepeti için

// veya farklı koşul örneklerinden birden fazla koşulu eklemek
$condition1 = new \Darryldecode\Cart\CartCondition(array(
    'name' => 'KDV 12.5%',
    'type' => 'tax',
    'target' => 'subtotal', // bu koşul, getSubTotal() çağrıldığında sepetin ara toplamına uygulanır.
    'value' => '12.5%',
    'order' => 2
));
$condition2 = new \Darryldecode\Cart\CartCondition(array(
    'name' => 'Express Kargo 15 TL',
    'type' => 'shipping',
    'target' => 'subtotal', // bu koşul, getSubTotal() çağrıldığında sepetin ara toplamına uygulanır.
    'value' => '+15',
    'order' => 1
));
Cart::condition($condition1);
Cart::condition($condition2);

// Dikkat edin ki, ara toplama uygulanan koşullar ekledikten sonra, getTotal() üzerindeki sonuç da etkilenecektir,
// çünkü getTotal(), ara toplama bağlıdır, ki bu da ara toplamıdır.

// yalnızca toplamlara uygulanacak koşul eklemek
$shippingCondition = new \Darryldecode\Cart\CartCondition(array(
    'name' => 'Express Kargo 15 TL',
    'type' => 'shipping',
    'target' => 'total', // bu koşul, getTotal() çağrıldığında sepetin toplamına uygulanır.
    'value' => '+15',
    'order' => 1 // hesaplama sırası kart tabanlı koşulların sırası. Sıra ne kadar büyükse, daha sonra uygulanır.
));
Cart::condition($shippingCondition);

// 'order' özelliği hesaplamada koşulların sırasını kontrol etmenizi sağlar. Aynı zamanda birden fazla koşulu eklemek
// için farklı sayfalar ile örneğin çok sayfalı bir alışveriş sürecinde sırayı ayarlamak için kullanılabilir.
// Sıra tanımlanmadığında varsayılan olarak 0 olarak kabul edilir.

// NOT!! Mevcut sürümde, 'order' parametresi yalnızca sepet temelindeki koşullar için geçerlidir. Her öğe bazlı koşullarda desteklenmez.

// veya birden fazla koşulu dizi olarak eklemek
Cart::condition([$condition1, $condition2]);

// Sepete uygulanan tüm koşulları almak için aşağıdakini kullanın:
$cartConditions = Cart::getConditions();
foreach ($cartConditions as $cartCondition) {
    $cartCondition->getTarget(); // koşulun uygulandığı hedef
    $cartCondition->getName(); // koşulun adı
    $cartCondition->getType(); // tür
    $cartCondition->getValue(); // koşulun değeri
    $cartCondition->getOrder(); // koşulun sırası
    $cartCondition->getAttributes(); // koşulun özellikleri, özellik eklenmediyse boş [] döner
}

// Ayrıca koşulun adını kullanarak sepette uygulanan bir koşulu alabilirsiniz, aşağıdaki gibi kullanın:
$conditionByName = Cart::getCondition('KDV 12.5%');
$conditionByName->getTarget(); // koşulun uygulandığı hedef
$conditionByName->getName(); // koşulun adı
$conditionByName->getType(); // tür
$conditionByName->getValue(); // koşulun değeri
$conditionByName->getAttributes(); // koşulun özellikleri, özellik eklenmediyse boş [] döner

// Koşulların hesaplanmış değerini ara toplamı sağlayarak alabilirsiniz, aşağıda görüldüğü gibi:
$subTotal = Cart::getSubTotal();
$calculatedValue = $conditionByName->getCalculatedValue($subTotal);

```

> NOT: Tüm sepet tabanlı koşullar, **Cart::getTotal()** çağrılmadan önce sepetin koşullarına eklenmelidir.
> ve ayrıca ara toplama uygulanması hedeflenen koşullar varsa, bunlar da sepetin koşullarına eklenmelidir.
>  **Cart::getSubTotal()**  çağrılmadan önce.

```php
$araToplamKosullu = Cart::getSubTotal(); // "ara toplama" hedefine uygulanan koşullarla
$toplamKosullu = Cart::getTotal(); // "toplam" hedefine uygulanan koşullarla
$araToplamKosulluKullanici = Cart::session($userId)->getSubTotal(); // belirli bir kullanıcının sepeti için
$toplamKosulluKullanici = Cart::session($userId)->getTotal(); // belirli bir kullanıcının sepeti için

```

Sıradaki konu Öğe Temeline Koşullardır.

Bu, özellikle bir kupona yalnızca bir öğe üzerinde değil tüm sepet değeri üzerinde uygulanmasını istediğiniz durumlar için çok kullanışlıdır.

> NOT: Öğe temelinde bir koşul eklerken, 'target' parametresine ihtiyaç yoktur veya atlanabilir.
> sepet temelinde koşullar eklerken olduğu gibi.

Şimdi bir öğe üzerine koşul ekleyelim:.

```php

// Önce koşul örneğimizi oluşturalım
$saleCondition = new \Darryldecode\Cart\CartCondition(array(
    'name' => 'İNDİRİM 5%',
    'type' => 'tax',
    'value' => '-5%',
));

// Şimdi sepete eklenmesi gereken ürün
$ürün = array(
    'id' => 456,
    'name' => 'Örnek Ürün 1',
    'price' => 100,
    'quantity' => 1,
    'attributes' => array(),
    'conditions' => $saleCondition
);

// Son olarak ürünü sepete ekleyin
Cart::add($ürün);

// Bir öğeye birden fazla koşul da ekleyebilirsiniz
$ürünKoşulu1 = new \Darryldecode\Cart\CartCondition(array(
    'name' => 'İNDİRİM 5%',
    'type' => 'sale',
    'value' => '-5%',
));
$ürünKoşulu2 = new \Darryldecode\Cart\CartCondition(array(
    'name' => 'Ürün Hediye Paketi 25.00',
    'type' => 'promo',
    'value' => '-25',
));
$ürünKoşulu3 = new \Darryldecode\Cart\CartCondition(array(
    'name' => 'MİSC',
    'type' => 'misc',
    'value' => '+10',
));

$ürün = array(
    'id' => 456,
    'name' => 'Örnek Ürün 1',
    'price' => 100,
    'quantity' => 1,
    'attributes' => array(),
    'conditions' => [$ürünKoşulu1, $ürünKoşulu2, $ürünKoşulu3]
);

Cart::add($ürün);

```

>NOT: Tüm sepet öğe başına koşullar, **Cart::getSubTotal()** çağrılmadan önce eklenmelidir.

Son olarak, her bir öğeye uygulanan koşullarla birlikte Sepet ara toplamını almak için **Cart::getSubTotal()** yöntemini çağırabilirsiniz.

```php
// Ara toplam, hedefi "ara toplam" olan eklenen koşullara ve öğe başına eklenen koşullara göre hesaplanır
$sepetAraToplamı = Cart::getSubTotal();
```

Mevcut bir öğeye koşul eklemek için: **Cart::addItemCondition($productId, $itemCondition)**

Sepetteki mevcut bir öğeye koşul eklemek de oldukça basittir.

Bu, özellikle ödeme işlemi sırasında kuponlar ve promosyon kodları gibi yeni koşulları bir öğeye eklemek için çok kullanışlıdır.
Nasıl yapılacağına dair örnek aşağıda verilmiştir:
```php
$productID = 456;
$coupon101 = new CartCondition(array(
            'name' => 'COUPON 101',
            'type' => 'coupon',
            'value' => '-5%',
        ));

Cart::addItemCondition($productID, $coupon101);
```

Sepet Koşullarını Temizleme: **Cart::clearCartConditions()**

```php
/**
* sepetin tüm koşullarını temizler,
* bunlar özellikle bir öğeye/ürüne eklenen koşulları silmez.
* Belirli bir öğeye ait bir koşulu kaldırmak isterseniz, removeItemCondition($itemId,$conditionName) yöntemini kullanabilirsiniz.
*
* @return void
*/
Cart::clearCartConditions()
```

Belirli Bir Sepet Koşulunu Kaldırma: **Cart::removeCartCondition($conditionName)**

```php
/**
* koşulu koşul adına göre bir sepetten kaldırır,
* bununla yalnızca sepet tabanlı eklenen koşulları kaldırabilirsiniz, bir öğe/ürün için eklenen koşulları değil.
* Bir öğe/ürün için eklenen bir koşulu kaldırmak isterseniz,
* bunun yerine removeItemCondition(itemId, conditionName) yöntemini kullanabilirsiniz.
*
* @param $conditionName
* @return void
*/
$koşulAdı = 'Yaz İndirimi 5%';

Cart::removeCartCondition($koşulAdı);
```

Belirli Bir Öğe Koşulunu Kaldırma: **Cart::removeItemCondition($itemId, $conditionName)**

```php
/**
* sepete zaten eklenmiş bir öğeye uygulanmış bir koşulu kaldırır
*
* @param $itemId
* @param $conditionName
* @return bool
*/
Cart::removeItemCondition($itemId, $koşulAdı);)
```

Tüm Öğe Koşullarını Temizleme: **Cart::clearItemConditions($itemId)**

```php
/**
* sepete zaten eklenmiş bir öğeye uygulanmış tüm koşulları kaldırır
*
* @param $itemId
* @return bool
*/
Cart::clearItemConditions($itemId);
```

Türüne Göre Koşulları Al: **Cart::getConditionsByType($tip)**

```php
/**
* Tüm koşulları Türüne Göre Filtrelenmiş Halde Al
* Lütfen unutmayın ki, bu yalnızca sepet temelinde eklenen koşulları döndürecektir, özellikle öğe bazında eklenen
* koşulları döndürmeyecektir
*
* @param $type
* @return CartConditionCollection
*/
public function getConditionsByType($tip)
```

Türüne Göre Koşulları Kaldır: **Cart::removeConditionsByType($tip)**

```php
/**
* Belirtilen $tip'e sahip tüm koşulları kaldırır
* Lütfen unutmayın ki, bu yalnızca sepet temelinde eklenen koşulları kaldıracaktır, özellikle öğe bazında eklenen
* koşulları kaldırmayacaktır
*
* @param $type
* @return $this
*/
public function removeConditionsByType($tip)
```

## Items

**Cart::getContent()** yöntemi öğelerin bir koleksiyonunu döndürür.

Bir öğenin kimliğini almak için **\$item->id** özelliğini kullanın.

Bir öğenin adını almak için **\$item->name** özelliğini kullanın.

Bir öğenin miktarını almak için **\$item->quantity** özelliğini kullanın.

Bir öğenin niteliklerini almak için **\$item->attributes** özelliğini kullanın.

Koşullar uygulanmadan tek bir öğenin fiyatını almak için **\$item->price** özelliğini kullanın.

Koşullar uygulanmadan bir öğenin ara toplamını almak için **\$item->getPriceSum()** yöntemini kullanın.

```php
/**
* fiyatın toplamını al
*
* @return mixed|null
*/
public function getPriceSum()

```

Tek bir öğenin koşullar uygulanmadan fiyatını almak için yöntemi kullanın

**\$item->getPriceWithConditions()**.

```php
/**
* Koşulların zaten uygulandığı tek fiyatı almak için
*
* @return mixed|null
*/
public function getPriceWithConditions()

```

Öğenin koşullar uygulanmış ara toplamını almak için yöntemi kullanın.

**\$item->getPriceSumWithConditions()**

```php
/**
* Koşulların zaten uygulandığı fiyat toplamını almak için
*
* @return mixed|null
*/
public function getPriceSumWithConditions()


```

**NOTE**:Koşullar uygulanmış fiyatı aldığınızda, yalnızca mevcut öğeye atanan koşullar hesaplanır.
Sepet koşulları fiyata uygulanmaz.

## Associating Models

Bir öğeyi bir modele bağlayabilirsiniz. Diyelim ki uygulamanızda bir `Product` modeliniz var. `associate()` yöntemi ile sepete, sepet içindeki bir öğenin `Product` modeli ile ilişkilendirildiğini belirtebilirsiniz.

Bu şekilde modele **\$item->model** özelliği kullanarak erişebilirsiniz.

İşte bir örnek:

```php

// öğeyi sepete ekleyin.
$sepetOgesi = Cart::add(455, 'Örnek Ürün', 100.99, 2, array())->associate('Product');

// dizi formatı
Cart::add(array(
    'id' => 456,
    'name' => 'Örnek Ürün',
    'price' => 67.99,
    'quantity' => 4,
    'attributes' => array(),
    'associatedModel' => 'Product'
));

// bir seferde birden fazla öğe ekleyin
Cart::add(array(
  array(
      'id' => 456,
      'name' => 'Örnek Ürün 1',
      'price' => 67.99,
      'quantity' => 4,
      'attributes' => array(),
      'associatedModel' => 'Product'
  ),
  array(
      'id' => 568,
      'name' => 'Örnek Ürün 2',
      'price' => 69.25,
      'quantity' => 4,
      'attributes' => array(
        'boyut' => 'L',
        'renk' => 'mavi'
      ),
      'associatedModel' => 'Product'
  ),
));

// Şimdi, sepet içeriğini dolaşırken modele erişebilirsiniz.
foreach(Cart::getContent() as $row) {
	echo 'Sepetinizde ' . $row->qty . ' adet ' . $row->model->name . ' adlı ürününüz bulunuyor. Açıklama: "' . $row->model->description . '"';
}
```

**NOTE**:Bu sadece bir öğeyi sepete eklerken çalışır.

## Instances

Aynı sayfada çakışmalar olmadan birden çok sepet örneği oluşturmanız gerekebilir.
Bunu yapmak için,

Yeni bir Hizmet Sağlayıcı oluşturun ve ardından register() yöntemi içine şunu ekleyebilirsiniz:

```php
$this->app['wishlist'] = $this->app->share(function($app)
{
$storage = $app['session']; // Laravel oturum depolama
$events = $app['events']; // Laravel olay işleyici
$instanceName = 'wishlist'; // sepet örneği adınız
$session_key = 'AsASDMCks0ks1'; // öğeleri tutmak için benzersiz oturum anahtarınız

		return new Cart(
    $storage,
    $events,
    $instanceName,
    $session_key
);

});

// 5.4 veya daha yeni sürümler için
use Darryldecode\Cart\Cart;
use Illuminate\Support\ServiceProvider;

class WishListProvider extends ServiceProvider
{
/**
* Uygulama hizmetleri başlatın.
*
* @return void
/
public function boot()
{
//
}
/*
* Uygulama hizmetlerini kaydedin.
*
* @return void
*/
public function register()
{
$this->app->singleton('wishlist', function($app)
{
$storage = $app['session'];
$events = $app['events'];
$instanceName = 'cart_2';
$session_key = '88uuiioo99888';
return new Cart(
$storage,
$events,
$instanceName,
$session_key,
config('shopping_cart')
);
});
}
}
```

Birden çok sepet örneği ile sorun yaşıyorsanız, lütfen bu demo deposundaki kodlara göz atın [DEMO](https://github.com/darryldecode/laravelshoppingcart-demo)

## Exceptions

Şu anda yalnızca iki istisna bulunmaktadır.

| İstisna                   | Açıklama                                                                  |
| --------------------------- | ------------------------------------------------------------------------- |
| _InvalidConditionException_ | Yeni bir Koşul oluşturulurken geçersiz bir alan değeri olduğunda           |
| _InvalidItemException_      | Yeni bir ürünün geçersiz alan değerleri var (id,ad,fiyat,miktar)           |
| _UnknownModelException_     | Bir sepet öğesine mevcut olmayan bir modeli ilişkilendirmeye çalıştığınızda |

## Events

Şu anda sepetin dinleyebileceğiniz ve bazı işlemler yapabileceğiniz 9 etkinliği bulunmaktadır.

| Etkinlik                        | Oluştuğunda                            |
| ---------------------------- | -------------------------------------- |
| cart.created(\$cart)         | Bir sepet örneği oluşturulduğunda            |
| cart.adding($items, $cart)   | Bir öğenin sepete eklenmeye çalışıldığında   |
| cart.added($items, $cart)    | Bir öğe sepete eklendiğinde               |
| cart.updating($items, $cart) | Bir öğe güncellenmeye çalışıldığında     |
| cart.updated($items, $cart)  | Bir öğe güncellendiğinde                  |
| cart.removing($id, $cart)    | Bir öğenin çıkarılmaya çalışıldığında    |
| cart.removed($id, $cart)     | Bir öğenin çıkarıldığında                 |
| cart.clearing(\$cart)        | Bir sepetin temizlenmeye çalışıldığında   |
| cart.cleared(\$cart)         | Bir sepetin temizlendiğinde               |

**NOTE**:Farklı sepet örnekleri için etkinliklerle ilgilenmek oldukça basittir. Örneğin, "wishlist" adında bir örnek oluşturdunuz. Etkinlikler aşağıdaki gibi olacaktır: {$instanceName}.created($cart)

Bu durumda wishlist sepet örneği için etkinlikler şu şekilde olacaktır:

- wishlist.created(\$cart)
- wishlist.adding($items, $cart)
- wishlist.added($items, $cart) ve benzeri...

## Format Response

Now you can format all the responses. You can publish the config file from the package or use env vars to set the configuration.
The options you have are:

- format_numbers or env('SHOPPING_FORMAT_VALUES', false) => Activate or deactivate this feature. Default to false,
- decimals or env('SHOPPING_DECIMALS', 0) => Number of decimals you want to show. Defaults to 0.
- dec_point or env('SHOPPING_DEC_POINT', '.') => Decimal point type. Defaults to a '.'.
- thousands_sep or env('SHOPPING_THOUSANDS_SEP', ',') => Thousands separator for value. Defaults to ','.

## Examples

```php

    // Ürünleri sepete ekleyin
Cart::add(array(
    array(
        'id' => 456,
        'name' => 'Örnek Ürün 1',
        'price' => 67.99,
        'quantity' => 4,
        'attributes' => array()
    ),
    array(
        'id' => 568,
        'name' => 'Örnek Ürün 2',
        'price' => 69.25,
        'quantity' => 4,
        'attributes' => array(
            'size' => 'L',
            'color' => 'blue'
        )
    ),
));

// Ardından aşağıdaki gibi yapabilirsiniz:
$items = Cart::getContent();

foreach($urunler as $item)
{
    $item->id; // Ürünün Kimliği
    $item->name; // Ürünün Adı
    $item->price; // Koşullar uygulanmadan tekil fiyat
    $item->getPriceSum(); // Koşullar uygulanmadan ara toplam
    $item->getPriceWithConditions(); // Koşullar uygulandığında tekil fiyat
    $item->getPriceSumWithConditions(); // Koşullar uygulandığında ara toplam
    $item->quantity; // Miktar
    $item->attributes; // Özellikler

    // Özellik, ItemAttributeCollection adlı yerel Laravel koleksiyonunu genişleten bir nesneyi döndürür
    // Bu nedenle aşağıdaki gibi işlemler yapabilirsiniz:

    if( $urun->attributes->has('size') )
    {
        // Ürün boyut özelliğine sahip
    }
    else
    {
        // Ürün boyut özelliğine sahip değil
    }
}

// veya
$items->each(function($item)
{
    $item->id; // Ürünün Kimliği
    $item->name; // Ürünün Adı
    $item->price; // Koşullar uygulanmadan tekil fiyat
    $item->getPriceSum(); // Koşullar uygulanmadan ara toplam
    $item->getPriceWithConditions(); // Koşullar uygulandığında tekil fiyat
    $item->getPriceSumWithConditions(); // Koşullar uygulandığında ara toplam
    $item->quantity; // Miktar
    $item->attributes; // Özellikler

    if( $item->attributes->has('size') )
    {
        // Ürün boyut özelliğine sahip
    }
    else
    {
        // Ürün boyut özelliğine sahip değil
    }
});

```

## Storage

Sepet öğeleri için farklı depolama kullanmak oldukça basittir. Sepet örneğine enjekte edilen depolama sınıfı yalnızca yöntemlere ihtiyaç duyacaktır.

Örnek olarak bir dilek listesi gerekebilir ve bunun anahtar-değer çiftlerini varsayılan oturum yerine veritabanında saklamak isteyebiliriz.

Bunu yapmak için öncelikle sepet verilerimizi tutacak bir veritabanı tablosuna ihtiyacımız olacaktır. Şu komutu kullanarak tabloyu oluşturalım: `php artisan make:migration create_cart_storage_table`

Örnek Kod:

```php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateCartStorageTable extends Migration
{
    /**
     * Veritabanı tablosunu oluşturun.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('cart_storage', function (Blueprint $table) {
            $table->string('id')->index();
            $table->longText('cart_data');
            $table->timestamps();

            $table->primary('id');
        });
    }

    /**
     * Göçleri geri alın.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('cart_storage');
    }
}

```

Sonraki adımda, verilerle kolayca işlem yapabilmek için bu tabloya bir Eloquent Modeli oluşturalım. Modeli nerede saklamak istediğiniz size bağlıdır. Bu örnek için, modeli App ad alanında saklayacağımızı varsayalım.

Kod:

```php
namespace App;

use Illuminate\Database\Eloquent\Model;

class DatabaseStorageModel extends Model
{
    protected $table = 'cart_storage';

    /**
     * Doldurulabilir öznitelikler.
     *
     * @var array
     */
    protected $fillable = [
        'id', 'cart_data',
    ];

    public function setCartDataAttribute($value)
    {
        $this->attributes['cart_data'] = serialize($value);
    }

    public function getCartDataAttribute($value)
    {
        return unserialize($value);
    }
}

```

Sonraki adım, sepet örneğimize enjekte edilecek depolama sınıfını oluşturmaktır:

Örnek:

```php
class DBStorage {

    public function has($key)
    {
        return DatabaseStorageModel::find($key);
    }

    public function get($key)
    {
        if($this->has($key))
        {
            return new CartCollection(DatabaseStorageModel::find($key)->cart_data);
        }
        else
        {
            return [];
        }
    }

    public function put($key, $value)
    {
        if($row = DatabaseStorageModel::find($key))
        {
            // update
            $row->cart_data = $value;
            $row->save();
        }
        else
        {
            DatabaseStorageModel::create([
                'id' => $key,
                'cart_data' => $value
            ]);
        }
    }
}
```

Aşağıdaki örneği inceleyerek Laravel'ın Önbellekleme sistemini (redis, memcached, file, dynamo vb.) nasıl kullanabileceğinizi görebilirsiniz. Örnek ayrıca çerez kalıcılığını da içerir, böylece sepetiniz 30 gün boyunca hala erişilebilir olur. Oturumlar varsayılan olarak yalnızca 20 dakika boyunca kalıcıdır.

```php
namespace App\Cart;

use Carbon\Carbon;
use Cookie;
use Darryldecode\Cart\CartCollection;

class CacheStorage
{
    private $data = [];
    private $cart_id;

    public function __construct()
    {
        $this->cart_id = \Cookie::get('cart');
        if ($this->cart_id) {
            $this->data = \Cache::get('cart_' . $this->cart_id, []);
        } else {
            $this->cart_id = uniqid();
        }
    }

    public function has($key)
    {
        return isset($this->data[$key]);
    }

    public function get($key)
    {
        return new CartCollection($this->data[$key] ?? []);
    }

    public function put($key, $value)
    {
        $this->data[$key] = $value;
        \Cache::put('cart_' . $this->cart_id, $this->data, Carbon::now()->addDays(30));

        if (!Cookie::hasQueued('cart')) {
            Cookie::queue(
                Cookie::make('cart', $this->cart_id, 60 * 24 * 30)
            );
        }
    }
}
```
Elbette, metni çevirebilirim. İşte çevirisi:

"Bu depoyu varsayılan depo olarak ayarlamak için, önce sepetin yapılandırma dosyasını güncellememiz gerekiyor.
İlk olarak, depoyu geçersiz kılmamıza olanak tanımak için sepet yapılandırma dosyasını yayınlayalım.
`php artisan vendor:publish --provider="Darryldecode\Cart\CartServiceProvider" --tag="config"`
Bu komutu çalıştırdıktan sonra, `config` klasörünüzde `shopping_cart.php` adında yeni bir dosya oluşmalı.

Bu dosyayı açalım ve depolama kullanımını güncelleyelim. `'storage' => null,` şeklinde olan anahtarı bulun,
ve yeni oluşturulan `DBStorage` sınıfınıza güncelleyin, örneğimizde olduğu gibi:
`'storage' => \App\DBStorage::class,`

VEYA Birden fazla sepet örneğiniz varsa (örneğin WishList), özel veritabanı depolamanızı sepet örneğinize enjekte edebilirsiniz.
Bunu yapmak için wishlist sepetinizin hizmet sağlayıcısına enjekte ederek depoyu özelleştirebilirsiniz.
Aşağıdaki gibi özel depolamanızı kullanacak şekilde depolamayı değiştirirsiniz:
"

```php
use Darryldecode\Cart\Cart;
use Illuminate\Support\ServiceProvider;

class WishListProvider extends ServiceProvider
{
    /**
     * Uygulama servislerini başlat.
     *
     * @return void
     */
    public function boot()
    {
        //
    }

    /**
     * Uygulama servislerini kaydet.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton('wishlist', function($app)
        {
            $storage = new DBStorage(); // <-- Yeni özel depolama sınıfınız
            $events = $app['events'];
            $instanceName = 'cart_2';
            $session_key = '88uuiioo99888';
            return new Cart(
                $storage,
                $events,
                $instanceName,
                $session_key,
                config('shopping_cart')
            );
        });
    }
}

```

Özel veritabanı depolamasını nasıl yapacağınıza dair hala kararsız mısınız? Ya da belki birden fazla sepet örneği oluşturmak konusunda muğlaklık yaşıyorsunuz? Aşağıdaki bağlantıları inceleyerek, kodları ve ihtiyacınıza göre nasıl genişletebileceğinizi veya rehber ve referans olarak nasıl kullanabileceğinizi görebilirsiniz:

(Maalesef, dış bağlantılara doğrudan erişimim olmadığı için verdiğiniz bağlantıları görüntüleyemem.)

Ancak, özel veritabanı depolamasını veya birden fazla sepet örneği yönetimini Laravel'de nasıl gerçekleştireceğiniz konusundaki belirli soruları veya kod örneklerini paylaşırsanız, size anlamada yardımcı olmaktan memnuniyet duyarım. Lütfen yardımcı olabileceğim özel endişelerinizi veya kod bölümlerini paylaşmaktan çekinmeyin.
[See Demo App Here](https://shoppingcart-demo.darrylfernandez.com/cart)

Veya

[See Demo App Repo Here](https://github.com/darryldecode/laravelshoppingcart-demo)

## License

"The Laravel Shopping Cart, açık kaynaklı bir yazılımdır ve lisanslanmıştır under the "
 [MIT license](http://opensource.org/licenses/MIT)

### Disclaimer

"BU YAZILIM "OLDUĞU GİBİ" SAĞLANMIŞ OLUP, AÇIKÇA BELİRTİLEN VEYA ZIMNEN İFADE EDİLEN GARANTİLER, ANCAK BUNLARLA SINIRLI OLMAMAK ÜZERE SATILABİLİRLİK VE BELİRLİ BİR AMACA UYGUNLUK GİBİ ZIMNİ GARANTİLER REDDEDİLİR. YAZAR VE KATKIDA BULUNANLARDAN HİÇBİRİ, DİREKT, DOLAYLI, TESADÜFİ, ÖZEL, ÖRNEK VEYA SONUÇ OLARAK ORTAYA ÇIKAN ZARARLARDAN (BUNLARLA SINIRLI OLMAMAK ÜZERE, YEDEK MAL VEYA HİZMET TEMİNİ; KULLANIM KAYBI, VERİ KAYBI YA DA KAZANÇ KAYBI; YA DA İŞ KESİNTİSİ) SORUMLU DEĞİLDİR, HANGİ SEBEP İLE OLURSA OLSUN VE HANGİ SORUMLULUK TEORİSİNE DAYANARAK OLURSA OLSUN, BU YAZILIMIN KULLANIMINDAN DOĞAN HERHANGİ BİR ZARAR İÇİN BİLE (İHMAL DE DAHİL OLMAK ÜZERE) SORUMLULUK TAŞIMAZ, HATTA BU TÜR BİR ZARAR İHTİMALİNDEN HABERDAR EDİLMİŞ OLSA BİLE."
