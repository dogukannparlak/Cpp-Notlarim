# C ve C++ Arasındaki Farklar

C ve C++, yazılım geliştirme dünyasında yaygın olarak kullanılan iki önemli programlama dilidir. C++, C'nin bir uzantısı olarak geliştirilmiş olsa da, iki dil arasında temel farklılıklar bulunmaktadır. Bu farkları anlamak, hangi dilin projeniz için daha uygun olduğuna karar vermenize yardımcı olur.

## 1. Programlama Paradigması

-   **C:** Prosedürel bir programlama dilidir. Programlar, fonksiyonlar (prosedürler) etrafında yapılandırılır. Veri ve fonksiyonlar birbirinden ayrıdır.
-   **C++:** Çoklu paradigma destekleyen bir dildir. Prosedürel programlamanın yanı sıra, temel olarak **Nesne Yönelimli Programlama (OOP)** paradigmasını destekler. Ayrıca fonksiyonel ve genel programlama özelliklerini de barındırır.

## 2. Nesne Yönelimli Programlama (OOP)

-   **C:** OOP özelliklerini (Sınıflar, Nesneler, Kalıtım, Polimorfizm, Kapsülleme) doğrudan desteklemez. Bu kavramlar `struct` ve fonksiyon işaretçileri kullanılarak taklit edilebilir ancak bu, dilin doğal bir özelliği değildir.
-   **C++:** OOP, C++'ın temel taşlarından biridir.
    -   **Sınıflar (Classes):** Veri ve bu veriyi işleyen fonksiyonları bir araya getirir.
    -   **Kalıtım (Inheritance):** Mevcut sınıflardan yeni sınıflar türetilmesini sağlar.
    -   **Polimorfizm (Polymorphism):** Farklı nesnelerin aynı mesaja farklı şekillerde yanıt vermesini sağlar (örneğin, sanal fonksiyonlar).
    -   **Kapsülleme (Encapsulation):** Veriyi dış erişimden korur ve sadece belirli fonksiyonlar aracılığıyla erişilmesini sağlar.

## 3. Standart Kütüphaneler

-   **C:** Standart kütüphanesi daha temeldir ve genellikle G/Ç işlemleri, string manipülasyonu ve bellek yönetimi gibi temel işlevleri içerir.
-   **C++:** Çok daha zengin bir standart kütüphaneye sahiptir. **Standart Şablon Kütüphanesi (STL - Standard Template Library)**, `vector`, `list`, `map` gibi konteynerler; `sort`, `find` gibi algoritmalar ve iterator'lar gibi güçlü araçlar sunar.

## 4. Bellek Yönetimi

-   **C:** Bellek yönetimi `malloc()`, `calloc()`, `realloc()` ve `free()` fonksiyonları ile manuel olarak yapılır.
-   **C++:** C'deki fonksiyonları desteklese de, `new` ve `delete` operatörlerini sunar. Daha da önemlisi, **RAII (Resource Acquisition Is Initialization)** prensibi ve Akıllı İşaretçiler (`unique_ptr`, `shared_ptr`, `weak_ptr`) sayesinde bellek sızıntılarını ve hatalarını büyük ölçüde önleyen modern ve daha güvenli bellek yönetimi teknikleri sunar.

## 5. Diğer Önemli Farklar

-   **Referanslar (References):** C++ referansları desteklerken, C yalnızca işaretçileri (pointers) kullanır. Referanslar, bir değişkene takma ad (alias) olarak davranır ve genellikle daha güvenli ve kullanımı kolaydır.
-   **İsim Alanları (Namespaces):** C++, büyük projelerde isim çakışmalarını önlemek için `namespace` özelliğini sunar. C'de bu özellik yoktur.
-   **Fonksiyon Aşırı Yükleme (Function Overloading):** C++'da aynı isme sahip ancak farklı parametrelere sahip birden fazla fonksiyon tanımlanabilir. C'de bu mümkün değildir.
-   **İstisna Yönetimi (Exception Handling):** C++, `try`, `catch`, `throw` anahtar kelimeleriyle modern istisna yönetimi mekanizması sunar. C'de hata yönetimi genellikle dönüş kodları veya global hata değişkenleri ile yapılır.
-   **Şablonlar (Templates):** C++, türden bağımsız kod yazmayı sağlayan şablonları destekler. Bu, genel programlama (generic programming) yapmayı ve STL gibi kütüphanelerin oluşturulmasını mümkün kılar.

## Özet Tablo

| Özellik | C | C++ |
| :--- | :--- | :--- |
| **Paradigma** | Prosedürel | Prosedürel, OOP, Fonksiyonel |
| **OOP Desteği** | Hayır | Evet (Sınıflar, Kalıtım, Polimorfizm) |
| **Kütüphane** | Temel Standart Kütüphane | Zengin Standart Kütüphane + STL |
| **Bellek Yönetimi** | `malloc`/`free` | `new`/`delete`, Akıllı İşaretçiler |
| **Referanslar** | Hayır | Evet |
| **Namespaces** | Hayır | Evet |
| **Fonksiyon Aşırı Yükleme**| Hayır | Evet |
| **İstisna Yönetimi**| Hayır (Dönüş kodları ile) | Evet (`try`/`catch`) |
| **Şablonlar** | Hayır | Evet |

Sonuç olarak C++, C'nin üzerine inşa edilmiş, daha modern, güvenli ve güçlü özellikler sunan bir dildir. C hala gömülü sistemler, işletim sistemleri ve performansın kritik olduğu bazı alanlarda popülerliğini korurken, C++ uygulama geliştirmede, oyun motorlarında ve karmaşık sistemlerde daha yaygın olarak tercih edilmektedir.
