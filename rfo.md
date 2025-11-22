Super — gəlin hər üç bölməni **daha dərindən**, proqramçı səviyyəsində və “sıfırdan anlayan” üçün addım-addım açıqlayaq. Hər bölmədə: **rəsmi təsdiq/idarəetmə**, **alternativlər**, **kənar hallar**, **kodun hər sətirinin nə etdiyi**, **komplekslik** və **test nümunələri** olacaq. Başlayaq!

# 📘 BÖLÜM 1 — Dinamik MEX (A, i, x sorğuları ilə)

## 🔷 Problem xülasəsi (qısa)

N elementli massiv var. Q sorğu: `i x` — A[i] = x edirik və mex(A)-ı çap edirik. N,Q ≤ 2e5, elementlər 0…1e9 aralığında.

## ✳️ Niyə MEX üçün yalnız 0…N lazımdır — dəqiq riyazi isbat

Massivin uzunluğu N-dir. 0..N aralığında (hər ikisi daxil) N+1 fərqli dəyər var. Amma massivdə cəmi N ədəd olduğundan ən azı bir dəyər bu aralıqda olmamalıdır — yəni mex(A) həmişə 0..N aralığında olur. Məsələn, əgər massiv 0..N-1 bütün elementləri ehtiva edirsə, mex = N. Buna görə biz elementlərin 0..N diapazonundakı tezliyinə baxsaq kifayət edir.

## ❌ Niyə hər sorğuda tam scan (O(N)) olmaz?

Q və N hər ikisi 2e5 ola bilər. Hər sorğuda O(N) baş verərsə, O(N·Q) ≈ 4·10^10 əməliyyat — TLE.

## ✅ Optimal ideya — `freq` + `missing set`

* `freq[v]` = 0..N aralığındakı dəyərin cari tezliyi. (Ən çox N+1 element saxlanır.)
* `missing` = set (ordered) içində 0..N aralığında **olan** və **hal-hazırda massivdə yoxdur** dəyərlər. MEX = ən kiçik `missing` elementi → `*missing.begin()`.

### Nə üçün bu O(log N) per update?

`set`-də `insert`/`erase`/`begin` əməliyyatları O(log N). Hər sorğuda maksimum iki `insert/erase` və bir `begin()` olur → O(log N).

## Kod (əsas) — tam və izahlı

(Mən kodu bir də göstərirəm amma indi sətir-sətir daha dərindən izah edəcəm.)

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int N, Q;
    cin >> N >> Q;

    vector<long long> A(N+1);
    vector<int> freq(N+1, 0);
    set<int> missing;

    for (int i = 0; i <= N; i++) missing.insert(i);

    for (int i = 1; i <= N; i++) {
        cin >> A[i];
        if (A[i] <= N) {
            freq[A[i]]++;
            if (freq[A[i]] == 1)
                missing.erase(A[i]);
        }
    }

    while (Q--) {
        int i;
        long long x;
        cin >> i >> x;

        long long old = A[i];

        if (old <= N) {
            freq[old]--;
            if (freq[old] == 0)
                missing.insert(old);
        }

        A[i] = x;
        if (x <= N) {
            freq[x]++;
            if (freq[x] == 1)
                missing.erase(x);
        }

        cout << *missing.begin() << '\n';
    }
}
```

### Sətir-sətir daha dərindən izah

* `#include <bits/stdc++.h>` — bütün standard C++ header-lərini yükləyir (contest üçün praktik).
* `ios::sync_with_stdio(false); cin.tie(nullptr);` — I/O sürətləndirilməsi. `sync_with_stdio(false)` C və C++ I/O sinxronizasiyasını söndürür; `cin.tie(nullptr)` `cout`-u `cin`-dən ayrır ki, flush-erlərlə vaxt itirməyək.
* `vector<long long> A(N+1);` — massiv 1-based indekslənir (tapşırıqda çox vaxt 1-index olur). `long long` istifadə etdik ki, A[i] 1e9 ola bilər.
* `vector<int> freq(N+1, 0);` — yalnız 0..N üçün tezlik saxlayırıq, ona görə N+1 ölçü.
* `set<int> missing;` — 0..N arasındakı hələ massivdə olmayan ədədlər; `set` ordered olduğuna görə `.begin()` ən kiçiyi verir.
* `for (i=0..N) missing.insert(i);` — əvvəl hamısını missing sayırıq; sonra giriş oxunanda mövcud olanları çıxarırıq.
* `if (A[i] <= N) { freq[A[i]]++; if (freq[A[i]]==1) missing.erase(A[i]); }` — yalnız 0..N diapazonundan olan dəyərlər freq-ə düşür. Əgər tezlik 1 oldu — yəni əvvəllər missing idi — onu missing-dən çıxarırıq.
* Sorğu zamanı:

  * `old = A[i]` — köhnə dəyəri götürürük.
  * əgər `old <= N`: `freq[old]--` və əgər indi sıfırdırsa `missing.insert(old)` (indi onu artıq massivdə yoxdur).
  * `A[i] = x` və dəyişənin yeni dəyərini yeniləyirik: əgər `x <= N`, `freq[x]++`, və `freq[x]==1` olarsa `missing.erase(x)`.
  * `cout << *missing.begin()` — ən kiçik mövcud olmayan ədəd → mex.

### Komplekslik

* Başlanğıc: set insert N+1 dəfə → O(N log N) (amma N ≤ 2e5).
* Hər sorğu: daimi sayda `freq` dəyişməsi + `set insert/erase` (hər biri O(log N)). Beləliklə ümumi O((N + Q) log N).
* Yaddaş: O(N) (A, freq, set).

### Kənar hallar, nümunələr

* Elementlər > N → saxlanmırlar (`freq`-də yox). Bu səbəbdən massivdə 1e9 kimi böyük ədədlər olsa da, mex dəyişməyə bilər.
* İndeks 1-baseddirsə istifadəçi input da 1-based verir — real contestdə bu barədə diqqətli ol.
* N = 0 halı: çox nadir, amma `missing` → {0} olur; mex 0-dır.

### Alternativ / optimallaşdırma

* `set` O(log N) yetərlidir. Amma əgər istəyirsənsə:

  * **Fenwick/Segment Tree**: hər pozisiyada 1 varsa 1, əks halda 0 — sonra `kth`-smallest 0-nu tapmaq üçün segment tree ilə axtarış. Bu da O(log N) per query, amma daha ağır kod.
  * **Union-Find trick**: statik mex üçün “next pointer” DSU ilə tez mex tapmaq olur (amortized α(N)). Dinamik (point updates) üçün daha çətindir.
  * `ordered_set` lazım deyil — normal `set` kifayətdir.

### Test nümunələri

1. N=4, A=[0,1,2,4], Q=1, change (4,3) → əvvəl mex=3, dəyişəndən sonra A=[0,1,2,3] mex=4.
2. Dəyişən `x` böyükdür (>>N) → mex-ə təsir etmir.

---

# 📘 BÖLÜM 2 — “OD of Legend” (Blue vs Red) — daha dərin izah

## 🔷 Problem xülasəsi

İki komanda, hərəsinin 5 HP-ləri var: B[0..4] və R[0..4]. Hər gedişdə 1 HP zərər vurulur. Blue hər tur əvvəl başlayır. Kim əvvəl rəqibin cəmi HP-sini 0 etdirər, o qalibdir.

## ✔️ Formal məntiq və sübut

Hər gediş 1 HP endirir. Əgər Blue ilk gedişi atırsa, gedişlər növbə ilə gedir: Blue, Red, Blue, Red, ...

* Blue-nun rəqibi (Red) öldürməsi üçün lazım olan zərbələrin sayı `S_R = sum(R_i)`.
* Red-in Blue-nu öldürməsi üçün lazım olan zərbələrin sayı `S_B = sum(B_i)`.

İndi bir sıra zərbələrin kim tərəfindən vurulduğunu düşünək. Blue ilk olduğuna görə əgər `S_R <= S_B`, Blue həmin `S_R`-ci zərbəni vuraraq Red-in cəmini 0 edir və qalib olur.
Əgər `S_R > S_B`, Red `S_B`-ci zərbəni vurduqdan sonra Blue-un HP 0 olar → Red qalib olur.

Bu sübut düz və tamdır — zərbələr vahid vahiddir və hər bir zərbə yalnız rəqibə təsir edir.

## ✅ Kod (sadə)

```cpp
#include <bits/stdc++.h>
using namespace std;

int main(){
    long long B[5], R[5];
    long long SB = 0, SR = 0;

    for(int i=0;i<5;i++) cin >> B[i], SB += B[i];
    for(int i=0;i<5;i++) cin >> R[i], SR += R[i];

    if(SR <= SB) cout << "Blue";
    else cout << "Red";
}
```

### Daha dərindən izah

* `long long` istifadə etmək lazımdır — HP-lər böyük ola bilər (problem limitlərindən asılı).
* `SR <= SB` bərabər halda Blue qalibdir — çünki Blue ilk gediş edir və eyni say zərbə olsa Blue sonuncu vuracaq (məsələn, əgər hərəkətlər eyni sayda olarsa, Blue-in son zərbəsi Red-in HP-sini 0 edir).
* Bu funksiya **O(1)** vaxtdır (sadəcə beş ədədi toplamaq).

### Düşüncə şablonu (oyun tipləri)

* Bu tip suallarda tez-tez “hansı tərəf ilk başlayır?”, “hansı hərəkət vahid / bərabər dəyərdir?” suallarını soruş.
* Əgər zərbələr fərqli dəyərdə olsa, ya hərəkət planlaması (greedy/dp) gərəkə bilər; ancaq burada hər gediş sabit — sadəcə cəmlər müqayisə olunur.

### Məsələn

* B = [1,1,1,1,1] → SB=5, R=[2,2,2,2,2] → SR=10 → SR>SB → Red qalibdir.
* B=[0,0,0,0,1] SB=1, R=[1,0,0,0,0] SR=1 → SR<=SB → Blue qalibdir.

---

# 📘 BÖLÜM 3 — Sadə ədəd məsələsi: p = a + b (a və b sadə)

## 🔷 Problem xülasəsi

Verilmiş p (sadə) üçün iki sadə ədəd `a` və `b` tap: `p = a + b`. Tapılmazsa -1 çap et.

## 🔍 Riyazi müşahidə və tam isbat

* Bütün sadə ədədlərdən yalnız **2** cütdür; bütün digərləri təkdir.
* `p`-nin cüt olması yalnız `p = 2` ola bilər. `2 = 1 + 1` — 1 sadə deyil. Yəni p=2 üçün cavab yoxdur → -1.
* İndi p tək (p odd). Əgər p = a + b və a,b həm sadədirsə:

  * Əgər həm a,b təkdirsə → tək + tək = cüt, amma p tək — ziddiyyət.
  * Buna görə birinin cüt, digərinin tək olması lazımdır. Yeganə cüt sadə 2-dir.
  * Deməli `a = 2`, `b = p - 2` (və ya əksinə).
  * Cavab yalnız p-2 primdirsə var.

## Kod (klassiski isPrime ilə)

```cpp
#include <bits/stdc++.h>
using namespace std;

bool isPrime(long long x) {
    if (x < 2) return false;
    if (x % 2 == 0) return x == 2;
    for (long long i = 3; i * i <= x; i += 2)
        if (x % i == 0) return false;
    return true;
}

int main() {
    long long p;
    cin >> p;

    if (p == 2) {
        cout << -1;
        return 0;
    }

    if (isPrime(p - 2))
        cout << 2 << " " << p - 2;
    else
        cout << -1;
}
```

### Sətir-sətir daha dərindən izah

* `isPrime()`:

  * `x < 2` → false (0,1 prim deyil).
  * `x % 2 == 0` → cütləri burda yoxlayırıq, yalnız 2 primdir.
  * `for (i=3; i*i<=x; i+=2)` — 3-dən başlayıb √x-ə qədər yalnız tək ədədlərlə bölünməni yoxlayırıq (çünki cütlər artıq yoxlanılıb). Bu √x yoxlaması klassik və düzgün üsuldur.
* `p == 2` üçün birbaşa -1 qaytarırıq.
* Əks halda `isPrime(p-2)` yoxlanılır — doğrudurursa `2` və `p-2` çap olunur, yoxsa -1.

### Komlekslik

* `isPrime(p-2)`-nin vaxtı O(√p). Əgər p ≤ 1e9, √p ≤ 31623 → bu praktiki contest üçün rahatdır.
* Əgər p daha böyükdürsə (64-bit), istifadə etmək üçün Miller-Rabin deteriministik test tövsiyə olunur.

### Optimallaşdırılmış isPrime (bir az sürətli)

Trial division metodunu 6k ± 1 optimizasiyası ilə yaza bilərsən — hər i-nin 6k±1 formalarını yoxlayıb iterasiya sayını yarıdan da azaldırsan.

```cpp
bool isPrime(long long n) {
    if (n < 2) return false;
    if (n % 2 == 0) return n == 2;
    if (n % 3 == 0) return n == 3;
    for (long long i = 5; i * i <= n; i += 6) {
        if (n % i == 0 || n % (i + 2) == 0) return false;
    }
    return true;
}
```

Bu daha sürətlidir, çünki yalnız `i = 6k-1, 6k+1` nöqtələrinə baxır.

### Kənar hallar və nümunələr

* p = 3 → p-2 = 1 (sadə deyil) → -1 (doğru: 3 = 2+1, amma 1 deyil sadə)
* p = 5 → p-2 = 3 (sadə) → cavab: `2 3`.
* p = 4 — **amma** p sadə zəmanət verilib. Əgər p sadə deyilsə, problem tərifi pozular. Həmişə əvvəlcə p-nin sadə olub-olmamasını yoxlamaq lazım deyil əgər problem şərtin içində "p sadədir" varsa.

### Proqramçı düşüncə tərzi (primes ilə bağlı məsələlər)

* Paritet (cüt/tək) ilə başla — çox vaxt qısa nəticə verir.
* `2` xüsusi haldır — cüt və yeganə cüt prime.
* Kiçik limitlərdə trial division yetərli, böyük limitlərdə Miller-Rabin və ya precomputed sieve (bir neçə dəfə test olunacaqsa) daha məqsədəuyğundur.
* Goldbach tipli məsələlərdə `2 + (n-2)` və mod 2 ilə bağlı məntiq ilk yoxlanılmalı olanlardır.

---

# 🧭 Proqramçı kimi düşünmə bacarıqlarını inkişaf etdirmək üçün ümumi qeydlər

1. **İnvariant tap** — məsələn MEX üçün invariant: “mex ∈ [0,N]”. Hər əməliyyatda bu invariant yenilənir.
2. **Limitləri ilk oxu** — N,Q ≤ 2e5 → O(N log N) və ya O(N + Q log N) düşün.
3. **Pariteyi yoxla** — primes və oyun problemlərində paritet və rolların (kimin başladığı) əhəmiyyəti həmişə var.
4. **Kənar hallar yaz** — 0, 1, maksimum, minimal, böyük x (>N) kimi.
5. **Sadə nümunələrlə dry-run et** — klub testləri yaz: kiçik N ilə hamısını əllə simulasiya et və avtomatlaşdırılmış stress test apar.
6. **Test & Debug strategiyası**:

   * Random generator ilə brute force (kiçik N) + optimized ilə müqayisə et.
   * Assertlər əlavə et (məs. `assert(0 <= mex && mex <= N)`).
7. **Komplekslik və yaddaş ölçü** — həmişə O-ları yaz və limitlərlə də uyğunluğunu təsdiqlə.

---

# ✅ Sınaq planı (hansı testləri yığmalısan)

### MEX üçün

* Kiçik N=1..5 üçün bütün mümkün A-larla brute-force ilə yoxla.
* Q müxtəlif: 0 sorğu, max Q, eyni indeksə bir neçə dəfə set etmək.
* Dəyərlər böyük: A[i] = 1e9, x = 1e9 → mex dəyişməməlidir.

### Blue vs Red

* B və R hamısı 0.
* Bir komanda heç nə (hamısı 0), bərabərlikləri yoxla.

### Prime split

* p = 2,3,5,large prime (1e9+7 — əgər problem limitlərinə uyğundursa).
* p-2 composite və prime halları.

---

# Nəticə və növbəti addımlar

Sənə hər üç məsələ üçün:

* daha geniş məntiq izahı,
* kodun sətir-sətir izahı,
* komplekslik və optimizasiya,
* test və debugging strategiyası verdim.

İstəyirsən, növbəti addımlar:

* Hər məsələ üçün **sadə test generatoru** və brute-force yoxlama kodu yazım (C++/Python).
* MEX üçün **segment tree** və ya **Fenwick** (kəsilən 0/1 array) ilə alternativ tam kod göstərim.
* Sadə ədəd üçün **Miller-Rabin** implementasiyası əlavə edib 64-bit üçün təsdiqləyim.

Sən hansıını istəyirsən? De: **“MEX üçün segment tree versiyasını göstər”** və ya **“Brute-force test generator yaz”** və dərhal hazır kod + nümunə ilə verim.
