# C++ Öğrenme Roadmap'i - Dal Yapısı

## 🌱 Giriş (Introduction to Language)
```
C++ Giriş
├── C++ Nedir?
├── C++ Neden Kullanılır?
├── C vs C++
└── Geliştirme Ortamı Kurulumu
    ├── C++ Kurulumu
    ├── Code Editörleri / IDE'ler
    └── İlk Programı Çalıştırma
```

## 🌿 Başlangıç Konuları (Beginner Topics)
```
Başlangıç Konuları
├── Temel İşlemler
│   ├── Aritmetik Operatörler
│   ├── Mantıksal Operatörler
│   └── Bitwise Operatörler
├── Kontrol Akışı & İfadeler
│   ├── Döngüler (for/while/do-while)
│   └── Koşullar (if-else/switch/goto)
├── Fonksiyonlar
│   ├── Fonksiyonlar
│   ├── Statik Polimorfizm
│   ├── Fonksiyon Overloading
│   ├── Operator Overloading
│   └── Lambdalar
└── Veri Tipleri
    ├── Temel Veri Türleri
    ├── Statik Tipleme
    ├── Dinamik Tipleme
    └── RTTI (Runtime Type Information)
```

## 🌳 Orta Seviye Konular (Intermediate Topics)
```
Orta Seviye Konular
├── Pointer'lar ve Referanslar
│   ├── Pointerlar
│   ├── Referanslar
│   ├── Bellek Modeli
│   ├── Nesnelerin Yaşam Süresi
│   ├── Smart Pointer'lar
│   │   ├── weak_ptr
│   │   ├── shared_ptr
│   │   └── unique_ptr
│   ├── Raw Pointer'lar
│   ├── New/Delete Operatörleri
│   └── Bellek Sızıntısı
├── Kod Tabanı Yapılandırma
│   ├── Forward Declaration
│   ├── Kapsam (Scope)
│   ├── Namespace'ler
│   └── Header/CPP Dosyaları
├── Yapılar ve Sınıflar
│   ├── Sıfır, Beş, Üç Kuralı
│   └── Çoklu Kalıtım
├── Nesne Yönelimli Programlama
│   ├── Structures and Classes
│   ├── Sanal Metodlar
│   ├── Sanal Tablolar
│   └── Dinamik Polimorfizm
└── Hata Yönetimi
    ├── Çıkış Kodları
    ├── İstisnalar (Exceptions)
    └── Erişim İhlalleri
```

## 🌲 İleri Seviye Konular (Advanced Topics)
```
İleri Seviye Konular
├── Dil Kavramları
│   ├── auto (Otomatik Tip Çıkarımı)
│   ├── Tip Dönüşümü
│   │   ├── static_cast
│   │   ├── const_cast
│   │   ├── dynamic_cast
│   │   └── reinterpret_cast
│   ├── Tanımsız Davranış (UB)
│   ├── Argümana Bağlı Arama (ADL)
│   └── Name Mangling Makrolar
├── Standart Kütüphane + STL
│   ├── Iterator'lar
│   ├── iostream
│   ├── Algoritmalar
│   ├── Tarih/Zaman
│   ├── Çoklu İşlemciliği (Multithreading)
│   └── Konteynerler
├── Template'ler
│   ├── Variadic Template'ler
│   ├── Template Specialization
│   ├── Type Traits SFINAE
│   ├── Full Template Specialization
│   └── Partial Template Specialization
├── Idiomlar
│   ├── Non-Copyable / Non-Moveable
│   ├── Erase-Remove
│   ├── Copy and Swap
│   ├── Copy on Write
│   ├── RAII
│   ├── Pimpl
│   └── CRTP
└── Standartlar
    ├── C++11/14
    ├── C++17
    ├── C++20
    ├── En Yeni Standartlar
    └── C++0x
```

## 🔧 Araçlar ve Geliştirme Ortamı
```
Geliştirme Araçları
├── Debugger'lar
│   ├── Debugger Mesajlarını Anlama
│   ├── Debugging Sembolleri
│   ├── WinDBG
│   └── GDB
├── Derleyiciler
│   ├── Derleyici Aşamaları
│   ├── Derleyiciler ve Özellikler
│   ├── Clang++ / LLVM
│   ├── Intel C++
│   ├── MSVS C++
│   ├── GCC
│   └── MinGW
├── Build Sistemleri
│   ├── CMAKE
│   ├── Makefile
│   └── Ninja
└── Paket Yöneticileri
    ├── vcpkg
    ├── NuGet
    ├── Conan
    └── Spack
```

## 📚 Kütüphaneler ve Framework'ler
```
Kütüphaneler ve Framework'ler
├── Kütüphane Çalışması
│   ├── Kütüphane Dahil Etme
│   └── Lisanslama
├── Popüler Kütüphaneler
│   ├── Boost
│   ├── OpenCV
│   ├── POCO
│   ├── protobuf
│   ├── gRPC
│   ├── TensorFlow
│   ├── pybind11
│   ├── spdlog
│   ├── OpenCL
│   ├── fmt
│   └── ranges_v3
├── Test Kütüphaneleri
│   ├── gtest/gmock
│   └── Catch2
├── GUI Framework'leri
│   └── Qt
├── Profiling Araçları
│   └── Orbit Profiler
└── ML/AI Kütüphaneleri
    └── PyTorch C++
```

## 🎯 Öğrenme Sırası Önerileri

### 1. Temel Seviye (Başlangıç)
- C++ Giriş → Temel İşlemler → Kontrol Akışı → Fonksiyonlar → Veri Tipleri

### 2. Orta Seviye
- Pointer'lar → Kod Yapılandırma → Sınıflar → OOP → Hata Yönetimi

### 3. İleri Seviye
- Dil Kavramları → STL → Template'ler → Idiomlar → Standartlar

### 4. Pratik Uygulama
- Debugger'lar → Derleyiciler → Build Sistemleri → Kütüphaneler

## 📋 Notlar
- **Beginner Topics**: Burada başlayın
- **Intermediate Topics**: Sonraki adım
- **Advanced Topics**: Daha sonra öğrenin
- **Optional**: İsteğe bağlı konular

Bu roadmap, C++ öğreniminde sistematik bir yaklaşım sağlar ve her seviyede hangi konuların öğrenilmesi gerektiğini açıkça gösterir.