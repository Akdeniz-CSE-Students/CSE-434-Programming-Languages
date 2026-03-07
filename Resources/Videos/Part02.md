# Yacc/Bison ve Lex Eğitimi Detaylı Dokümanı

Bu doküman, Lex ve Yacc (Bison) kullanarak dil işleme (language processing) ve derleyici/yorumlayıcı yapımı üzerine hazırlanmış eğitim videosunun eksiksiz ve detaylı bir özetidir. 

---

## BÖLÜM 1: KAVRAMLARIN HIZLI ÖZETİ

*   **Dil İşleme Yığını (Language Processing Stack):** Kaynak kodun makine koduna veya hedeflenen çıktıya dönüştürülme adımlarıdır.
*   **Lex (Sözcüksel Analiz - Lexical Analysis):** Kaynak kod (karakter akışı) dosyasını okur ve anlamlı yapı taşlarına (Token'lara) böler.
*   **Yacc (Sözdizimsel Analiz - Syntax Analysis / Parsing):** Lex'ten gelen Token akışını alır, önceden tanımlanmış dil bilgisi kurallarına (Grammar) göre ağaç yapıları (Parse Tree) oluşturur ve semantik işlemleri gerçekleştirir.
*   **Bison:** Yacc'ın GNU versiyonudur. Yacc ile geriye dönük (upwardly) tam uyumludur.
*   **Token:** Dilin en küçük anlamlı birimidir (örneğin; bir sayı, bir değişken adı, bir toplama işareti).
*   **Semantik Analiz (Semantic Analysis):** Yazılan kodun gramer olarak doğru olmasının yanı sıra, anlam olarak ne ifade ettiğini çözme işlemidir (örneğin toplama işlemi yapmak veya bir değişkene değer atamak).
*   **Yacc Dosya Yapısı:** `%%` sembolleri ile ayrılmış üç bölümden oluşur: C Tanımlamaları, Kurallar (Productions) ve C Kodları.
*   **Sembol Değerleri ($$, $1, $2 vb.):** Yacc kuralları içinde, sol taraftaki değer `$$` ile, sağ taraftaki elemanların değerleri sırasıyla `$1, $2, $3` vb. ile temsil edilir.
*   **%union:** Lex'ten dönen Token'ların farklı veri tiplerinde (int, char vb.) olabilmesine olanak tanıyan bir C birleşimi (union) tanımıdır.

---

## BÖLÜM 2: DETAYLI İÇERİK

### 1. Dil İşleme ve Lex/Yacc Etkileşimi
Bir dili işlemek için kod önce **Sözcüksel Analizden (Scanner/Lex)** geçer. Lex, `a = b + c * d` gibi bir ifadeyi alır ve bunu Token'lara çevirir: `[identifier] [=] [identifier] [+] [identifier] [*] [identifier]`.

Ardından bu Token akışı **Sözdizimsel Analize (Parser/Yacc)** girer. Yacc, bir "İçerikten Bağımsız Gramer (Context-Free Grammar)" kullanarak bu sembolleri bir sözdizimi ağacına (Parse Tree) dönüştürür. Ağaç oluşturulurken, kurallara bağladığımız aksiyonlar (actions) çalıştırılır. Bu eğitimde kod üretimi yerine, direkt yorumlayıcı (interpreter) mantığıyla anında işlem yapma (semantik süreç) işlenmiştir.

### 2. Dosyaların Derlenme Akışı (Workflow)
Bir proje geliştirirken dosyalar arası ilişki şu şekildedir:
1.  **mylang.y:** Yacc dil bilgisi dosyasıdır.
    *   `yacc -d mylang.y` komutu ile derlenir. (`-d` flag'i çok önemlidir, header dosyası üretir).
    *   Çıktı olarak `y.tab.c` (C kodlarını içeren parser) ve `y.tab.h` (Lex ile paylaşılacak token tanımlamaları) üretir.
2.  **mylang.l:** Lex dosyasıdır.
    *   İçerisinde `#include "y.tab.h"` bulundurur ki Yacc'ın beklediği token isimlerini bilebilsin.
    *   `lex mylang.l` komutu ile derlenir ve `lex.yy.c` dosyasını üretir.
3.  **GCC Derlemesi:**
    *   `gcc y.tab.c lex.yy.c -o mylang` komutu ile her iki C dosyası birleştirilerek çalıştırılabilir program (`mylang`) oluşturulur.

### 3. Yacc Dosya Yapısı (Yacc Input)
Bir Yacc dosyası (`.y` uzantılı) üç ana bölümden oluşur. Bölümler birbirlerinden `%%` ile ayrılır.

#### Birinci Bölüm: Tanımlamalar (Declarations)
*   **C Tanımlamaları:** `%{` ve `%}` arasına yazılır. Burada `#include` komutları, C değişkenleri ve C fonksiyon prototipleri yer alır.
*   **Yacc Tanımlamaları:**
    *   `%start [kural_adi]`: Gramerin kök (başlangıç) kuralını belirtir.
    *   `%token [isim]`: Lex'ten gelecek token'ları tanımlar.
    *   `%union { ... }`: Token'ların ve kuralların farklı veri tipleri taşıyabilmesi için kullanılır. Yacc'ın farklı veri tiplerini anlamasını sağlar.
    *   `%type <tip_adi> [kural_adi]`: Kendi tanımladığımız kuralların (non-terminal) hangi veri tipini döndüreceğini belirtir.

#### İkinci Bölüm: Kurallar (Productions / Grammar)
Bu bölüm dilin gramerini temsil eder.
*   Format: `sol_taraf : sag_taraf { aksiyonlar };`
*   Çoklu sağ taraf koşulları dik çizgi `|` ile ayrılır.
*   Özyinelemeli (Recursive) kurallar yazılabilir. Örn: `statements: statement | statement statements`
*   **Aksiyonlar (Actions):** Süslü parantezler `{}` arasına yazılan C kodlarıdır. Bir kural eşleştiğinde ne yapılacağını belirtir.
*   **Değerlere Erişim:** 
    *   Örnek Kural: `exp: exp '+' term`
    *   `$$` : Sol taraftaki `exp` öğesini temsil eder.
    *   `$1` : Sağ taraftaki ilk `exp` öğesini temsil eder.
    *   `$2` : Sağ taraftaki `'+'` karakterini temsil eder.
    *   `$3` : Sağ taraftaki `term` öğesini temsil eder.
    *   Örnek Aksiyon: `{ $$ = $1 + $3; }` (İlk ifadenin değeri ile üçüncü ifadenin değerini topla ve sonuca ata). Varsayılan aksiyon `{ $$ = $1; }` şeklindedir.

#### Üçüncü Bölüm: C Fonksiyonları (Third Part)
Gramer kurallarındaki aksiyonların kullandığı C fonksiyonlarının (örneğin sembol tablosu işlemleri) implementasyonu burada yapılır. Ayrıca zorunlu olan `yyerror(char *s)` fonksiyonu ve `main()` fonksiyonu genellikle buraya yazılır.

---

## BÖLÜM 3: ÖRNEK PROJE İNCELEMESİ (Hesap Makinesi - Calc)

Videoda basit bir hesap makinesi yorumlayıcısı (interpreter) inşa edilmiştir. Bu program:
*   A-Z ve a-z arası tek harfli 52 farklı değişkeni hafızasında tutabilir.
*   Sayıları alabilir, toplama ve çıkarma işlemi yapabilir.
*   Atama yapabilir (`a = 10 + 5;`).
*   Değişkenleri ve işlemleri ekrana yazdırabilir (`print a;`).
*   Programdan çıkış yapabilir (`exit;`).

### 1. calc.y (Yacc Dosyası) İncelemesi

**C Tanımlamaları Bölümü:**
*   Standart kütüphaneler eklenir (`stdio.h`, `stdlib.h`).
*   Değişkenleri tutmak için 52 elemanlı bir `symbols[52]` int dizisi tanımlanır (Sembol tablosu).
*   Yardımcı fonksiyon prototipleri yazılır (`computeSymbolIndex`, `symbolVal`, `updateSymbolVal`).

**Yacc Tanımlamaları Bölümü:**
```c
%union { int num; char id; }
%start line
%token print exit_command
%token <num> number
%token <id> identifier
%type <num> exp term
%type <id> assignment
```
*   `%union` ile Lex'in `yylval.num` ile tam sayı, `yylval.id` ile karakter döndürebileceği belirtiliyor.

**Kurallar Bölümü (Productions):**
```c
line: assignment ';'          {;}
    | exit_command ';'        { exit(EXIT_SUCCESS); }
    | print exp ';'           { printf("Printing %d\n", $2); }
    | line assignment ';'     {;}
    | line print exp ';'      { printf("Printing %d\n", $3); }
    | line exit_command ';'   { exit(EXIT_SUCCESS); }
    ;

assignment: identifier '=' exp { updateSymbolVal($1, $3); }
          ;

exp: term           { $$ = $1; }
   | exp '+' term   { $$ = $1 + $3; }
   | exp '-' term   { $$ = $1 - $3; }
   ;

term: number        { $$ = $1; }
    | identifier    { $$ = symbolVal($1); }
    ;
```
*Burada `updateSymbolVal` fonksiyonu $1 (identifier karakteri) içine $3 (hesaplanan exp değeri) değerini kaydeder.*

**C Fonksiyonları Bölümü:**
*   `computeSymbolIndex(char token)`: Gelen harfin (örn: 'A' veya 'a') 0-51 arası dizideki indeksini ASCII matematiği ile hesaplar.
*   `symbolVal(char symbol)`: Tablodan değeri okur.
*   `updateSymbolVal(char symbol, int val)`: Tabloya yeni değeri yazar.
*   `main()`: Diziyi sıfırlarla doldurur (başlangıç durumu) ve `yyparse()` fonksiyonunu çağırarak Lex ve Yacc'ı tetikler.
*   `yyerror()`: Beklenmeyen bir karakter veya syntax hatasında çağrılır.

### 2. calc.l (Lex Dosyası) İncelemesi
Yacc'ın beklediği tokenleri döndüren Lex kuralları:
```c
%{
#include "y.tab.h"
%}
%%
"print"                 { return print; }
"exit"                  { return exit_command; }
[a-zA-Z]                { yylval.id = yytext[0]; return identifier; }
[0-9]+                  { yylval.num = atoi(yytext); return number; }
[ \t\n]                 ; /* Boşlukları ve yeni satırları görmezden gel */
[-+=;]                  { return yytext[0]; } /* Operatörleri direkt karakter olarak döndür */
.                       { ECHO; yyerror("unexpected character"); }
%%
int yywrap(void) { return 1; }
```
*Önemli Not:* `yylval.id` ve `yylval.num` kullanımları Yacc dosyasında tanımlanan `%union` yapısından gelmektedir. Lex, yakaladığı değeri bu birleşim tipine atayarak Yacc'a gönderir.

### 3. Programın Derlenmesi ve Çalıştırılması
Videoda gösterilen adım adım derleme komutları:
1.  `yacc -d calc.y` *(y.tab.c ve y.tab.h oluşturur)*
2.  `lex calc.l` *(lex.yy.c oluşturur)*
3.  `gcc lex.yy.c y.tab.c -o calc` *(C kodlarını derler ve calc adında çalıştırılabilir dosya oluşturur)*
4.  `./calc` *(Programı çalıştırır)*

**Örnek Kullanım Akışı:**
```text
a = 1 + 100;
print a;
Printing 101

b = a - 10;
print b;
Printing 91

print a + b;
Printing 192

exit;
```
