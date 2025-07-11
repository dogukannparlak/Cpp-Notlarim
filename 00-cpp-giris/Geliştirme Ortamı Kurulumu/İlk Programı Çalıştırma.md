# İlk C++ Programını Çalıştırma

Geliştirme ortamımızı kurduğumuza ve kod editörümüzü/IDE'mizi seçtiğimize göre, artık ilk C++ programımızı yazıp çalıştırmaya hazırız. Programlama öğrenirken geleneksel ilk adım "Merhaba, Dünya!" (Hello, World!) programını yazmaktır. Bu program, ekrana basit bir mesaj yazdırır ve kurulumunuzun doğru çalıştığını test etmenin en iyi yoludur.

## 1. Kodun Yazılması

Öncelikle, seçtiğiniz kod editörünü veya IDE'yi açın ve `merhaba.cpp` adında yeni bir dosya oluşturun. `.cpp` uzantısı, bu dosyanın bir C++ kaynak kodu dosyası olduğunu belirtir.

Dosyanın içine aşağıdaki kodu yazın veya kopyalayıp yapıştırın:

```cpp
// İlk C++ Programımız
#include <iostream>

// Programın başlangıç noktası
int main() {
    // Ekrana "Merhaba, Dünya!" yazdırır
    std::cout << "Merhaba, Dünya!" << std::endl;

    // Programın başarıyla bittiğini belirtir
    return 0;
}
```

### Kodun Açıklaması

-   `#include <iostream>`: Bu satır, "input/output stream" (giriş/çıkış akışı) kütüphanesini programımıza dahil eder. Bu kütüphane, `std::cout` gibi ekrana yazı yazdırmamızı sağlayan araçları içerir.
-   `int main()`: Bu, her C++ programının giriş noktasıdır. Program çalıştığında, işletim sistemi `main` fonksiyonunu çağırır ve içindeki kodlar sırayla çalıştırılır.
-   `std::cout`: "character output stream" (karakter çıkış akışı) anlamına gelir ve genellikle standart çıkış aygıtı olan ekrana veri yazdırmak için kullanılır.
-   `<<`: Bu operatör, "insertion operator" (ekleme operatörü) olarak bilinir. Sağındaki veriyi solundaki akışa (stream) gönderir. Yani `"Merhaba, Dünya!"` metnini `std::cout`'a göndererek ekrana yazdırır.
-   `std::endl`: "end line" (satır sonu) anlamına gelir. Ekrana bir yeni satır karakteri basar ve imleci bir sonraki satırın başına taşır. Aynı zamanda çıkış tamponunu (output buffer) temizler.
-   `return 0;`: `main` fonksiyonunun sonunda bu ifade, programın herhangi bir hata olmadan başarıyla tamamlandığını işletim sistemine bildirir. `0` genellikle başarı anlamına gelir.

## 2. Derleme ve Çalıştırma (Terminal Üzerinden)

Kodu yazdıktan sonra onu makine diline çevirmemiz (derleme) ve ardından çalıştırmamız gerekir. Bu işlemi terminal veya komut istemi üzerinden yapacağız.

**Adım 1: Terminali Açın ve Dosyanın Olduğu Dizine Gidin**
-   Terminalinizi (Windows'ta CMD/PowerShell, macOS/Linux'ta Terminal) açın.
-   `cd` (change directory) komutunu kullanarak `merhaba.cpp` dosyasını kaydettiğiniz klasöre gidin. Örneğin: `cd Belgeler/CppProjeleri`

**Adım 2: Kodu Derleyin**
-   Aşağıdaki komutu kullanarak kodu derleyin. Bu komut, `g++` (veya `clang++`) derleyicisine `merhaba.cpp` dosyasını okumasını ve `merhaba` adında çalıştırılabilir bir dosya oluşturmasını söyler.
    ```bash
    g++ merhaba.cpp -o merhaba
    ```
-   **Komutun Açıklaması:**
    -   `g++`: Kullandığımız C++ derleyicisinin adı (GCC). Eğer macOS'ta Xcode Command Line Tools kurduysanız `clang++` da kullanabilirsiniz.
    -   `merhaba.cpp`: Derlemek istediğimiz kaynak dosyanın adı.
    -   `-o merhaba`: `-o` (output) bayrağı, derleyiciye oluşturulacak çalıştırılabilir dosyanın adını belirtir. Bu bayrağı kullanmazsanız, dosya varsayılan olarak Linux/macOS'ta `a.out`, Windows'ta `a.exe` olarak adlandırılır.

**Adım 3: Programı Çalıştırın**
-   Derleme işlemi başarılı olduysa (terminalde bir hata mesajı görmediyseniz), bulunduğunuz dizinde çalıştırılabilir dosyanız oluşmuş demektir.
-   Programı çalıştırmak için aşağıdaki komutu kullanın:

    -   **Windows'ta:**
        ```bash
        .\merhaba.exe 
        ```
        veya sadece
        ```bash
        merhaba
        ```

    -   **macOS ve Linux'ta:**
        ```bash
        ./merhaba
        ```
-   `./` ifadesi, komutu mevcut dizinde aramasını söyler.

**Beklenen Çıktı:**
Terminal ekranında aşağıdaki çıktıyı görmelisiniz:
```
Merhaba, Dünya!
```

Tebrikler! İlk C++ programınızı başarıyla derleyip çalıştırdınız. Bu, geliştirme ortamınızın doğru bir şekilde kurulduğunu ve C++ dünyasına ilk adımınızı attığınızı gösterir.

### IDE Kullanıyorsanız

Visual Studio, CLion, Code::Blocks gibi bir IDE kullanıyorsanız, bu derleme ve çalıştırma adımları genellikle otomatikleştirilmiştir. Genellikle arayüzde bulunan "Run" (Çalıştır) veya "Build and Run" (Derle ve Çalıştır) gibi bir düğmeye (genellikle yeşil bir oynat butonu ikonu vardır) tıklamanız yeterlidir. IDE, arka planda derleyiciyi çağıracak ve programı sizin için çalıştıracaktır.
