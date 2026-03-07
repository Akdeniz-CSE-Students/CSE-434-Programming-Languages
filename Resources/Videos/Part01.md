# Lex ve Flex Eğitimi: Kapsamlı Dokümantasyon

Bu doküman, Jonathan R. Engelsma'nın hazırladığı "Lex/Flex Eğitimi" adlı videonun detaylı, hiçbir adımın atlanmadığı, öğretici yazılı bir versiyonudur. Öğrenme sürecini kolaylaştırmak için önce kavramların hızlı bir özeti, ardından konunun tüm teknik detayları ve uygulamalı kod örnekleri sunulmuştur.

---

## 1. KAVRAMLAR İÇİN HIZLI ÖZET

Aşağıda videoda geçen temel terimlerin ve kavramların kısa tanımları yer almaktadır:

*   **Compiler (Derleyici):** Yüksek seviyeli bir programlama dilinde yazılmış kaynak kodu, makine koduna veya hedef bir dile çeviren yazılımdır. Çeşitli aşamalardan oluşur.
*   **Lexical Analysis (Sözcük Analizi / Scanner):** Derleme sürecinin *ilk aşamasıdır*. Bir kaynak dosyasındaki karakter akışını (character stream) okur ve bunları anlamlı gruplara, yani "Token"lara dönüştürür.
*   **Token:** Kaynak kodda dilin kurallarına göre anlam ifade eden en küçük yapı taşlarıdır (örneğin; anahtar kelimeler, değişken isimleri, noktalama işaretleri).
*   **Syntax Analysis (Sözdizimi Analizi / Parser):** Scanner'dan gelen token akışını alır ve dilin dilbilgisi kurallarına göre kontrol ederek bir "Parse Tree" (Sözdizimi Ağacı) oluşturur.
*   **Lex:** Unix ortamları için geliştirilmiş orijinal, geleneksel "tarayıcı üretici" (scanner generator) aracıdır.
*   **Flex:** Lex'in açık kaynaklı (open source) versiyonudur. Günümüzde lex ve flex kelimeleri genellikle aynı aracı ifade etmek için kullanılır.
*   **Regular Expressions (Düzenli İfadeler / Regex):** Metinler içinde belirli desenleri (pattern) aramak ve eşleştirmek için kullanılan karakter dizileridir. Lex, tokenları tanımak için Regex kullanır.
*   **lex.yy.c:** Lex aracı tarafından, `.l` uzantılı girdi dosyanıza bakılarak otomatik olarak üretilen C kaynak kodudur. (Tarayıcının ta kendisidir).
*   **yylex():** Lex tarafından üretilen tarayıcının ana fonksiyonudur. Her çağrıldığında girdi metninden bir sonraki token'ı bulur ve döner.
*   **yytext:** Lex çalışırken geçerli eşleşen metnin (token'ın kendisinin) değerini tutan global karakter işaretçisidir (char pointer).
*   **yylineno:** Tarayıcının şu anda girdinin hangi satırında olduğunu tutan global değişkendir (hata yakalamada faydalıdır).
*   **yywrap():** Girdi dosyasının sonuna gelindiğinde (EOF) Lex tarafından çağrılan fonksiyondur. 1 dönerse işlemin bittiğini belirtir.

---

## 2. DETAYLI İÇERİK

### 2.1. Dil İşleme Yığını (Language Processing Stack)
Lex aracının nerede durduğunu anlamak için standart bir derleyicinin (compiler) aşamalarını bilmek gerekir. Bir derleyici şu aşamalardan oluşur:

1.  **Scanner (Lexical Analysis):** Karakter akışını alır, *Token Akışına (Token Stream)* dönüştürür. **Lex'in bulunduğu aşama burasıdır.**
2.  **Parser (Syntax Analysis):** Token akışını alır, dil kurallarına göre *Parse Tree (Sözdizimi Ağacı)* oluşturur.
3.  **Semantic Analysis and Code Generation:** Anlamsal analiz yapar ve bir *Abstract Syntax Tree (Soyut Sözdizimi Ağacı)* veya ara form (intermediate form) üretir.
4.  **Machine-independent code improvement (Opsiyonel):** Kodu donanımdan bağımsız olarak optimize eder.
5.  **Target code generation:** Hedef dile (örneğin Assembly diline) çeviri yapar.
6.  **Machine-specific code improvement (Opsiyonel):** Makineye özgü donanımsal optimizasyonlar yapar.

### 2.2. Lexical Analysis Nasıl Çalışır? (Örnek)
Elinizde C dilinde yazılmış basit bir fonksiyon olduğunu düşünün:
`void swap(int *v1, int *v2) { ... }`

Bu kaynak kodu aslında sadece yanyana duran bir harf kalabalığıdır. **Scanner (Tarayıcı)** bu metni alır ve parçalara böler:
`[void] [swap] [(] [int] [*] [v1] [,] [int] [*] [v2] [)][{]` ...
Her bir köşeli parantez içindeki ifade bir **Token**'dır. Scanner boşlukları atlar ve sadece parser'ın (ayrıştırıcının) ihtiyaç duyacağı anlamlı tokenları üretir.

### 2.3. Lex / Flex Nedir?
Lex, sıfırdan bir Lexical Analyzer (Scanner) yazmak yerine, size bir "Scanner Üreticisi" sunar.
*   Girdi olarak, hangi kelimelerin/desenlerin hangi tokenlara denk geldiğini anlatan **Düzenli İfadeler (Regular Expressions)** ve bu desenler bulunduğunda çalışacak **C kodlarını (Actions)** alır.
*   Çıktı olarak, tablo güdümlü (table-driven) hızlı çalışan bir C kodu dosyası üretir (genellikle `lex.yy.c` adında).
*   Manuel olarak bir if/else veya switch bloğuyla kelime aramak yerine, Lex bu süreci otomatize eder ve çok daha karmaşık diller için işi inanılmaz kolaylaştırır.

### 2.4. Lex Girdi Dosyasının Yapısı (.l uzantılı)
Lex dosyaları (genellikle `.l` uzantılıdır), üç temel bölümden oluşur ve bu bölümler `%%` sembolü ile birbirinden ayrılır.

```lex
/* 1. BÖLÜM (İsteğe Bağlı): Tanımlamalar ve Global C Kodu */
%{
  // Buraya yazılan C kodları (örn: #include işlemleri, global değişkenler)
  // üretilecek lex.yy.c dosyasının en üstüne olduğu gibi kopyalanır.
%}
%%

/* 2. BÖLÜM (Zorunlu): Kurallar (Patterns ve Actions) */
pattern     action
pattern     { blok_action; }
%%

/* 3. BÖLÜM (İsteğe Bağlı): Kullanıcı C Kodları */
// Buraya yazılan C kodları (fonksiyonlar vb.) üretilen C dosyasının en altına eklenir.
```

### 2.5. Temel Lex Kullanımı (hello world örneği)
Videodaki ilk örnek, `ex1.l` adında bir dosyadır. 1. ve 3. bölümleri boştur.

```lex
%%
"hello world"   printf("GOODBYE\n");
.               ;
%%
```
**Bu kod ne yapar?**
*   Girdide `"hello world"` metnini görürse ekrana `GOODBYE` yazar.
*   İkinci kural olan nokta (`.`) regex dilinde "herhangi bir karakter" demektir. Action kısmı sadece bir noktalı virgül `;` içerir (Yani boş C ifadesi, hiçbir şey yapma). Dolayısıyla hello world haricindeki her karakteri görmezden gelir.

**Nasıl Derlenir ve Çalıştırılır?**
1.  Terminalde: `lex ex1.l` komutu çalıştırılır. Bu, `lex.yy.c` dosyasını oluşturur.
2.  Daha sonra C derleyicisi ile derlenir: `cc lex.yy.c -ll`
    *   *Not:* `-ll` (veya sisteminize göre `-lfl`) bayrağı lex kütüphanesini bağlar. Bu kütüphane, kendi içinde programın çalışmasını sağlayan varsayılan bir `main()` fonksiyonu barındırır.
3.  Çalıştırma: `./a.out`
    *   Ekrana `hello world` yazdığınızda size `GOODBYE` cevabını verir. Başka bir şey yazarsanız tepki vermez.

### 2.6. Lex'te Düzenli İfade (Regex) Örnekleri
Lex'te tokenları eşleştirmek için kullanılan desenler:

*   `abc` : Sadece "abc" stringini eşleştirir.
*   `[a-zA-Z]` : Herhangi bir tek büyük veya küçük harfi eşleştirir.
*   `dog.*cat` : "dog" ile başlayan, arada sıfır veya daha fazla rastgele karakter (`.*`) olan ve "cat" ile biten stringleri eşleştirir.
*   `(ab)+` : "ab" deseninin bir veya daha fazla kez yanyana tekrarını eşleştirir (ab, abab, ababab vb.).
*   `[^a-z]+` : Küçük a-z harfleri **İÇERMEYEN** (şapka `^` işareti parantez içinde "değil/hariç" anlamındadır) bir veya daha fazla uzunluktaki karakter dizilerini eşleştirir.
*   `[+-]?[0-9]+` : İsteğe bağlı (`?`) bir artı veya eksi işareti olan ve ardından bir veya daha fazla rakam gelen tam sayıları eşleştirir.

---

### 2.7. UYGULAMALI ÖRNEK: Konfigürasyon Dosyası Tarayıcısı (Hands-on Demo)

Videonun en önemli kısmı, sıfırdan bir konfigürasyon dosyasını okuyup analiz eden gerçekçi bir C projesi yazmaktır.

**Hedef Girdi Dosyası (`config.in`):**
```text
db_type : mysql
db_name : testdata
db_table_prefix : test_
db_port : 1099
```

Program, bu dosyayı okuyacak, anahtar kelimeleri tanıyacak ve sözdizimi hatalarını (örneğin port numarası yerine harf girilmesi) yakalayacaktır. Bu işlem 3 adımda yapılır:

#### Adım 1: C Başlık Dosyasını Oluşturmak (`myscanner.h`)
Lex'in bulacağı tokenlara birer sayısal ID (Kimlik) vermeliyiz. Hem Lex dosyamız hem de C programımız aynı kimlikleri bilmelidir.

```c
// myscanner.h
#define TYPE 1
#define NAME 2
#define TABLE_PREFIX 3
#define PORT 4
#define COLON 5
#define IDENTIFIER 6
#define INTEGER 7
```

#### Adım 2: Lex Dosyasını Oluşturmak (`myscanner.l`)
Girdiyi okuyacak ve token'ları tespit ettiğinde `myscanner.h` içindeki numaraları C tarafına döndürecek kuralları tanımlarız.

```lex
// myscanner.l
%{
#include "myscanner.h"
%}

%%
:                       return COLON;
"db_type"               return TYPE;
"db_name"               return NAME;
"db_table_prefix"       return TABLE_PREFIX;
"db_port"               return PORT;
[a-zA-Z][_a-zA-Z0-9]*   return IDENTIFIER;
[1-9][0-9]*             return INTEGER;[ \t\n]                 ; /* Boşlukları, tabları ve satır atlamaları yoksay */
.                       printf("unexpected character\n");
%%

int yywrap(void)
{
    return 1;
}
```
*   **Önemli Kural (Sıralama):** Lex kuralları **yukarıdan aşağıya** doğru değerlendirilir. `"db_type"` gibi spesifik anahtar kelimeler, her türlü kelimeyi yakalayan `IDENTIFIER` kuralından **önce** yazılmalıdır. Aksi halde Lex "db_type" kelimesini de sıradan bir IDENTIFIER zannederek yanlış sonuç üretir.
*   **IDENTIFIER Kuralı (`[a-zA-Z][_a-zA-Z0-9]*`):** Bir harfle başlamalı, ardından sıfır veya daha fazla harf, rakam veya alt çizgi gelebilir.
*   **INTEGER Kuralı (`[1-9][0-9]*`):** Sıfırdan farklı bir rakamla başlamalı, ardından sıfır veya daha fazla rakam gelmelidir.
*   **yywrap:** Lex'in EOF (dosya sonu) durumunda ne yapacağını bilmesi için gereken standart bir fonksiyondur. 1 döndürmek analizin bittiği anlamına gelir.

#### Adım 3: Ana C Programını Yazmak (`myscanner.c`)
Lex'in ürettiği tarayıcıyı (`yylex`) yönetecek ana mantığı C dilinde yazarız.

```c
// myscanner.c
#include <stdio.h>
#include "myscanner.h"

// Lex tarafından oluşturulan değişkenleri ve fonksiyonları external olarak tanımlıyoruz
extern int yylex();
extern int yylineno; // Bulunulan satır numarası (Hata mesajları için)
extern char* yytext; // Eşleşen kelimenin string değeri

// Sadece ekrana daha güzel yazdırmak için oluşturduğumuz basit bir dizi
// Dizi index'leri myscanner.h dosyasındaki define numaraları ile aynı hizada
char *names[] = {NULL, "db_type", "db_name", "db_table_prefix", "db_port"};

int main(void) {
    int ntoken, vtoken;

    ntoken = yylex(); // İlk tokeni al (Örn: TYPE)

    while (ntoken) { // ntoken 0 (EOF) olmadığı sürece dön
        printf("%d\n", ntoken); // Sadece debug amacıyla token ID'sini yazdır

        // Kurala göre her anahtar kelimeden sonra bir iki nokta üst üste (COLON) gelmeli
        if (yylex() != COLON) {
            printf("Syntax error in line %d, Expected a ':' but found %s\n", yylineno, yytext);
            return 1;
        }

        // İki noktadan sonra gelen asıl değeri (value) al
        vtoken = yylex();

        // Beklenen anahtar kelime tipine göre gelen değerin formatını kontrol et
        switch (ntoken) {
            case TYPE:
            case NAME:
            case TABLE_PREFIX:
                // Bu anahtar kelimelerin karşılığı bir kelime (IDENTIFIER) olmalı
                if (vtoken != IDENTIFIER) {
                    printf("Syntax error in line %d, Expected an identifier but found %s\n", yylineno, yytext);
                    return 1;
                }
                printf("%s is set to %s\n", names[ntoken], yytext);
                break;
            
            case PORT:
                // Port değerinin karşılığı bir sayı (INTEGER) olmalı
                if (vtoken != INTEGER) {
                    printf("Syntax error in line %d, Expected an integer but found %s\n", yylineno, yytext);
                    return 1;
                }
                printf("%s is set to %s\n", names[ntoken], yytext);
                break;
            
            default:
                // Bilinmeyen bir anahtar kelime gelirse
                printf("Syntax error in line %d\n", yylineno);
                return 1;
        }

        // Bir sonraki konfigürasyon satırına geçmek için döngüyü besle
        ntoken = yylex();
    }
    return 0;
}
```
*Not: Videoda eğitmen kod yazarken bazı ufak syntax hataları (noktalı virgül unutma, `int main` fonksiyonunda return unutma vb.) yapmış ve bunları derleme aşamasında düzelterek ilerlemiştir. Yukarıdaki kod düzeltilmiş, çalışan nihai halidir.*

#### Derleme ve Çalıştırma İşlemleri
Bu 3 dosyayı birbirine bağlamak için terminalde sırasıyla şu adımlar izlenir:

1.  **Lex dosyasını C'ye çevir:**
    `lex myscanner.l`
    *(Bu komut arka planda `lex.yy.c` dosyasını oluşturur).*
2.  **C dosyalarını derle:**
    `gcc myscanner.c lex.yy.c -o myscanner`
    *(Hem kendi yazdığımız ana programı hem de lex'in ürettiği tarayıcıyı birlikte derleyip "myscanner" adında çalıştırılabilir bir dosya oluşturur).*
3.  **Programı çalıştır ve konfigürasyon dosyasını içeri aktar (redirect):**
    `./myscanner < config.in`

**Başarılı Çıktı:**
```text
1
db_type is set to mysql
2
db_name is set to testdata
3
db_table_prefix is set to test_
4
db_port is set to 1099
```

Eğer `config.in` dosyasına girip port numarasını `a1099` yaparsanız (yani bir integer kuralını bozarsanız), program size şu hatayı verecektir:
`Syntax error in line 4, Expected an integer but found a1099`

**Özetle:** Videoda, Lex/Flex'in derleyici mimarisindeki yeri açıklanmış, temel sözdizimi öğretilmiş ve son olarak Lex'in ürettiği otomatik tarayıcının kendi yazdığımız bir C programına nasıl entegre edileceği adım adım bir konfigürasyon parser'ı yazılarak uygulamalı olarak gösterilmiştir.
