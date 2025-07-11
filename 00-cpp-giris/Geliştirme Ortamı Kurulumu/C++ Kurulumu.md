# C++ Geliştirme Ortamı Kurulumu

C++ kodlarını yazıp çalıştırabilmek için öncelikle bir **derleyiciye (compiler)** ihtiyacınız vardır. Derleyici, yazdığınız C++ kodunu makinenin anlayabileceği makine koduna çeviren bir araçtır. Popüler C++ derleyicileri arasında **GCC (GNU Compiler Collection)**, **Clang** ve **MSVC (Microsoft Visual C++ Compiler)** bulunur.

Bu rehber, farklı işletim sistemlerinde C++ geliştirme ortamını nasıl kuracağınızı adım adım anlatacaktır.

## Windows için Kurulum

Windows'ta C++ geliştirme ortamı kurmanın en popüler yollarından biri, **MinGW-w64** (Minimalist GNU for Windows) aracılığıyla GCC derleyicisini kurmaktır. Bunu yapmanın en kolay yolu ise **MSYS2** paket yöneticisini kullanmaktır.

### MSYS2 ile MinGW-w64 Kurulumu

1.  **MSYS2 Kurulumu:**
    -   [MSYS2 resmi web sitesine](https://www.msys2.org/) gidin ve en son sürüm yükleyiciyi indirin.
    -   İndirdiğiniz `.exe` dosyasını çalıştırarak kurulumu tamamlayın.

2.  **Paket Veritabanını Güncelleme:**
    -   Kurulum tamamlandıktan sonra MSYS2 terminalini başlatın.
    -   Aşağıdaki komutu çalıştırarak paket listesini ve temel paketleri güncelleyin. Terminali kapatıp tekrar açmanız istenebilir, istenirse yapın ve komutu tekrar çalıştırın.
        ```bash
        pacman -Syu
        ```

3.  **GCC Derleyici Kurulumu:**
    -   MSYS2 terminalinde, MinGW-w64 GCC toolchain'ini (derleyici, debugger vb. araçlar seti) kurmak için aşağıdaki komutu çalıştırın:
        ```bash
        pacman -S --needed base-devel mingw-w64-x86_64-toolchain
        ```
    -   Kurulum sırasında sizden paket seçimi yapmanız istenirse, varsayılan olarak tümünü seçmek için `Enter`'a basabilirsiniz.

4.  **PATH Ortam Değişkenini Ayarlama:**
    -   Derleyiciyi (`g++`) Windows'un herhangi bir komut satırından (CMD, PowerShell) tanıyabilmesi için MinGW-w64'ün `bin` klasörünü Windows'un `PATH` ortam değişkenine eklemeniz gerekir.
    -   Genellikle bu klasörün yolu `C:\msys64\mingw64\bin` şeklindedir.
    -   "Ortam Değişkenleri" ayarlarını açın, `Path` değişkenini bulun, "Düzenle" deyin ve yeni bir giriş olarak bu yolu ekleyin.

5.  **Kurulumu Doğrulama:**
    -   Yeni bir CMD veya PowerShell penceresi açın ve aşağıdaki komutu yazın:
        ```bash
        g++ --version
        ```
    -   Eğer `g++ (MinGW-w64)` gibi bir çıktı alıyorsanız, kurulum başarılıdır.

### Alternatif: Visual Studio

Microsoft'un kendi IDE'si olan **Visual Studio**'yu kurarak da C++ geliştirme ortamına sahip olabilirsiniz.
-   [Visual Studio Community](https://visualstudio.microsoft.com/tr/vs/community/) (ücretsiz) sürümünü indirin.
-   Yükleyiciyi çalıştırdığınızda "Workloads" (İş Yükleri) ekranında **"Desktop development with C++" (C++ ile Masaüstü Geliştirme)** seçeneğini işaretleyip kurun. Bu, MSVC derleyicisini ve gerekli tüm araçları otomatik olarak kuracaktır.

## macOS için Kurulum

macOS kullanıcıları için en kolay yol, Apple'ın sunduğu **Xcode Command Line Tools**'u kurmaktır. Bu paket, Clang derleyicisini içerir.

1.  **Terminal'i Açın:** Spotlight (Cmd + Space) ile "Terminal" aratıp açın.
2.  **Kurulum Komutunu Çalıştırın:**
    ```bash
    xcode-select --install
    ```
3.  Açılan pencerede kurulumu onaylayın.
4.  **Kurulumu Doğrulama:**
    -   Kurulum tamamlandıktan sonra terminalde aşağıdaki komutu çalıştırın:
        ```bash
        clang++ --version
        ```
    -   Eğer `Apple clang version...` gibi bir çıktı alıyorsanız, kurulum başarılıdır.

## Linux için Kurulum

Çoğu Linux dağıtımı, paket yöneticileri aracılığıyla C++ derleyicisini kurmayı çok kolay hale getirir.

### Debian/Ubuntu ve Türevleri

-   Terminali açın ve aşağıdaki komutları çalıştırın:
    ```bash
    sudo apt update
    sudo apt install build-essential
    ```
-   `build-essential` paketi, `g++` derleyicisini ve C++/C programları derlemek için gerekli diğer temel araçları içerir.

### Fedora/CentOS/RHEL ve Türevleri

-   Terminali açın ve aşağıdaki komutu çalıştırın:
    ```bash
    sudo dnf groupinstall "Development Tools"
    ```
-   Bu komut, `g++` dahil olmak üzere gerekli geliştirme araçları grubunu kurar.

### Kurulumu Doğrulama (Tüm Linux Dağıtımları)

-   Terminalde aşağıdaki komutu çalıştırın:
    ```bash
    g++ --version
    ```
-   `g++ (GCC)` ile başlayan bir çıktı görmelisiniz.

Bu adımlardan sonra C++ derleyiciniz kullanıma hazır olacaktır. Artık bir kod editörü veya IDE seçerek ilk programınızı yazmaya başlayabilirsiniz.
