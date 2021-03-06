---
id: concurrent-mode-intro
title: Konkurrent Rejiminə Giriş (Eksperimental)
permalink: docs/concurrent-mode-intro.html
next: concurrent-mode-suspense.html
---

<style>
.scary > blockquote {
  background-color: rgba(237, 51, 21, 0.2);
  border-left-color: #ed3315;
}
</style>

<div class="scary">

>Xəbərdarlıq:
>
>Bu səhifədə **stabil buraxılışlarda [mövcud olmayan](/docs/concurrent-mode-adoption.html) eksperimental xüsusiyyətlərdən danışılır**. Produksiya applikasiyalarında eksperimental qurulmalardan istifadə etməyin. Buradakı xüsusiyyətlər React-in bir hissəsi olana kimi xəbərdarlıq verilmədən əhəmiyyətli dərəcədə dəyişilə bilər.
>
>Bu sənədlər erkən yoxlamaq istəyən və maraqlanan proqramçılar üçün yönəldilib. **Əgər React-ə yeni başlayırsınızsa, burada danışılan xüsusiyyətlərdən narahat olmayın** -- bu xüsusiyyətləri indi öyrənmək lazım deyil.

</div>

Bu səhifə Konkurrent Rejiminin nəzəri icmalını təqdim edir. **Daha praktik giriş üçün növbəti bölümləri nəzərdən keçirin:**

* [Məlumat Yüklənməsi üçün Suspense](/docs/concurrent-mode-suspense.html), React komponentlərində məlumat yükləmək üçün yeni bir mexanizmi təsvir edir.
* [Konkurrent UI Həlləri](/docs/concurrent-mode-patterns.html), Konkurrent Rejimi və Suspense ilə mümkün olan bəzi UI həllərini göstərir.
* [Konkurrent Rejiminə Uyğunlaşma](/docs/concurrent-mode-adoption.html), layihənizdə Konkurrent Rejimini necə sınaya biləcəyinizi izah edir.
* [Konkurrent Rejiminin API Arayışı](/docs/concurrent-mode-reference.html), eksperimental qurulmalarda mövcud olan yeni API sənədləridir.

## Konkurrent Rejimi Nədir? {#what-is-concurrent-mode}

Konkurrent Rejimi, React applikasiyalarına responsive qalmağa və istifadəçinin cihaz imkanlarına və şəbəkə sürətinə düzgün cavab verməyə kömək edən yeni xüsusiyyətlər toplusudur.

Bu xüsusiyyətlər hələ də eksperimentaldır və dəyişə bilərlər. Bunlar sabit React buraxılışının bir hissəsi deyillər və ancaq eksperimental qurulma ilə bunu sınamaq olar.

## Bloklama ilə Kəsilə Bilən Render Etmənin Müqayisəsi {#blocking-vs-interruptible-rendering}

**Konkurrent Rejimi izah etmək üçün versiya nəzarəti sistemini metafora kimi istifadə edəcəyik.** Əgər komandada işləyirsinizsə, böyük ehtimal Git kimi versiya nəzarət sistemindən və budaqlardan istifadə edirsiniz və budaqlardan istifadə edirsiniz. İşlədiyiniz budaq hazır olduqda, işinizi əsas budağa (master) birləşdirə bilərsiniz və digər proqramçılar bunu pull edə bilərlər.

Versiya nəzarəti sisteminin mövcudluğundan əvvəl, işləmə axını çox fərqli idi. Budaq anlayışı yox idi. Bəzi sənədləri redaktə etmək istədiyinizdə, hər kəsə işinizi qurtarana qədər həmin sənədlərə toxunmamasını deməliydiniz. Bir faylda digər proqramçı ilə eyni vaxtda işləmək mümkün deyildi və sözün əsl mənasında siz digər proqramçıların işini **bloklayırdınız**.

Bu, React daxil olmaqla, UI kitabxanalarının bu gün necə işlədiyini göstərir. Bu kitabxanalarda, yeni DOM nodunu yaratma və komponentlərin içərisindəki kodu işlətmə daxil olmaqla, yeniləmə render edilməyə başladıqda bu işi dayandırmaq mümkün deyil. Biz bu yanaşmanı "bloklayan render etmə" adlandıracağıq.

Konkurrent Rejimində render etmə bloklanmır və kəsilə bilir. Bu, istifadəçi təcrübəsini yaxşılaşdırır və bundan əvvəl mümkün olmayan yeni xüsusiyyətlərə yol açır. [Növbəti](/docs/concurrent-mode-suspense.html) [bölümlərdəki](/docs/concurrent-mode-patterns.html) konkret nümunələrə baxmadan əvvəl yeni xüsusiyyətləri ümumi olaraq gözdən keçirəcəyik.

### Kəsilə Bilən Render Etmə {#interruptible-rendering}

Filtrlənə bilən bir məhsul siyahısını təsvir edin. Siyahını filtr etmək üçün klaviatur düyməsinə basdığınızda kəkələmə effektini hiss etmisiniz? Çox güman ki, məhsul siyahısını yeniləmək üçün yeni DOM nodlarını yaratmaq və ya brauzer ilə elementləri yerləşdirmək kimi işlər qaçınılmazdır. Ancaq bu işi nə *vaxt* və *necə* icra etdiyimiz böyük rol oynayır.

Kəkələmənin ümumi bir həll yolu anket sahəsinə daxil etməni "debounce" etməkdir. Debounce etmə zamanı yalnız istifadəçi yazmağı dayandırdıqdan *sonra* siyahını yeniləyirik. Ancaq yazarkən UI yeniləməməsi bəzən məyusluğa səbəb ola bilər. Alternativ olaraq, anket sahəsinə daxil etməni "throttle" edə bilərik və siyahını müəyyən bir maksimum tezliklə yeniləyə bilərik. Lakin daha zəif cihazlarda yenə də kəkələmə baş verəcək. Həm debounce etmə, həm də throttle etmə standartdan aşağı istifadəçi təcrübəsi yaradır.

Kəkələmənin səbəbi sadədir: render etmə başladıqdan sonra onu kəsmək olmaz. Beləliklə, brauzer düyməni basdıqdan dərhal sonra mətn girişini yeniləyə bilmir. Əgər UI kitabxanası (məsələn, React kimi) testlərin tez işləməsinə baxmayaraq render etməni bloklayırsa, komponentlərinizdəki müəyyən bir iş həmişə kəkələnməyə səbəb olacaqdır. Və çox vaxt bunun asan bir həll yolu yoxdur.

**Konkurrent Rejimi bu təməl məhdudiyyəti kəsilə bilən render etmə ilə aradan qaldırır.** Bu o deməkdir ki, istifadəçi başqa bir düyməni basdıqda React, brauzerin mətn girişini yeniləməsinə mane olmayacaqdır. Bunun əvəzinə, React, brauzerin girişi yeniləməsinə icazə verə bilər və sonra yenilənmiş siyahını *yaddaşda* render etməyə davam edə bilər. Render etmə başa çatdıqda React, DOM-u yeniləyir və dəyişikliklər ekranda əks olunur.

Fikir olaraq, React-in hər yeniləməni bir "budaqda" hazırladığını düşünə bilərsiniz. Budaqlarda işdən imtina və ya onların arasında keçid edə bildiyiniz kimi, Konkurrent Rejimində React daha vacib bir işi görmək üçün davam edən bir yeniləməni kəsə bilər və sonra əvvəl etdiyi işə qayıda bilər. Bu üsul, video oyunlarında [ikiqat buffer etməni](https://wiki.osdev.org/Double_Buffering) xatırlada bilər.

Konkurrent Rejimi üsulu UI-da debounce və throttle etmə ehtiyacını azaldır. Render etmə kəsilə bilən olduğu üçün, React kəkələmənin qarşısını almaq üçün işləri süni şəkildə *təxirə salmağa* ehtiyac duymur. Applikasiyanı responsiv saxlamaq üçün React render edilən işi dayandıra bilər.

### Qəsdən Yükləmə Ardıcıllığı {#intentional-loading-sequences}

Bundan əvvəl dedik ki, Konkurrent Rejimi, "budaq üzərində" işləməyə bənzəyir. Budaqlar yalnız qısamüddətli düzəlişlər üçün deyil, həm də uzun müddət icra olunan xüsusiyyətlər üçün də faydalıdır. Bəzən işlənilən xüsusiyyətin “kifayət qədər yaxşı vəziyyətdə” olub master budağına merge edilməsi bir neçə həftə çəkə bilər. Versiya nəzarət sistemi metaforamızın bu tərəfi də render etməyə aiddir.

Bir applikasiyada iki ekran arasında naviqasiya etdiyimizi düşünün. Bəzən yeni ekranda istifadəçiyə "kifayət qədər yaxşı" yükləmə vəziyyətini göstərmək üçün yetərincə yüklənmiş kod və məlumat olmaya bilər. Boş ekrana və ya böyük yükləmə göstəricisinə keçid çaşdırıcı ola bilər. Bununla yanaşı, çox vaxt lazımlı kodun və məlumatların yüklənməsi uzun çəkmir. **React köhnə ekranda bir az daha uzun qala bilsə və yeni ekranı nümayiş etdirmədən əvvəl "pis yükləmə vəziyyətini" "atsa", daha yaxşı olmazmı?**

Bunu etmək mümkün olsa da, təşkil etmək çətindir. Konkurrent Rejiminə bu xüsusiyyət daxildir. İlk öncə React yeni ekranı yaddaşda və ya metaforamızın dediyi kimi "fərqli bir budaqda" hazırlamağa başlayır. Beləliklə, React, DOM-u yeniləməmişdən öncə gözləyərək daha çox kontenti yükləyə bilər. Konkurrent Rejimində React-ə köhnə ekranı tam interaktiv şəkildə və yükləmə göstəricisi ilə göstərməsini bildirə bilərik. Yeni ekran hazır olduqda React bizi buna apara bilər.

### Konkurrentlik {#concurrency}

Yuxarıdakı iki nümunəni nəzərdən keçirərək Konkurrent Rejiminin bunları necə birləşdirdiyini görək. Budaqların fərqli komanda üzvlərinin müstəqil işləməsinə imkan verdiyi kimi **Konkurrent Rejimində React, *eyni vaxtda* bir neçə state yeniləməsi üzərində işləyə bilər**:

* CPU ilə əlaqəli yeniləmələr üçün (məsələn, DOM nodlarının yaradılması və komponent kodunun işlədilməsi kimi) konkurrentlik, daha vacib yeniləmənin digər başlamış render etməni "kəsə" biləcəyi deməkdir.
* IO ilə əlaqəli yeniləmələr üçün (məsələn, şəbəkədən kodu və ya məlumatı yükləmək kimi) konkurrentlik, bütün məlumatları yüklənmədən əvvəl React-in yaddaşda render etməyə başlaya biləcəyi və boş yükləmə vəziyyətlərinin atlatıla biləcəyi deməkdir.

Əhəmiyyətli olan React-i eyni formada *istifadə etməyimizdir*. Komponentlər, state və proplar kimi anlayışlar eyni şəkildə işləyirlər. Ekranı yeniləmək istədikdə state-i yeniləyirsiniz.

React yeniləmənin nə qədər "təcili" olduğuna qərar vermək üçün evristikadan istifadə edir və hər bir interaksiya üçün istədiyiniz istifadəçi təcrübəsini bir neçə kod sətri ilə düzəltməyə imkan verir.

## Tədqiqatı Produksiyada İstifadə Etmək {#putting-research-into-production}

Konkurrent Rejiminin xüsusiyyətləri ətrafında ümumi bir mövzu var. **Bu mövzunun vəzifəsi İnsan-Kompüter İnteraksiya nəticələrini real UI-lara inteqrasiya etməyə kömək etməkdir.**

Məsələn, tədqiqatlar göstərir ki, ekranlar arasında keçid zamanı çoxlu aralıq yükləmə vəziyyətinin göstərilməsi keçidin daha *yavaş* olduğunun hissiyatını verir. Buna görə Konkurrent Rejimi qəfil və çox tez-tez yenilənmələrin qarşısını almaq üçün yeni yükləmə vəziyyətlərini sabit "cədvəldə" göstərir.

Eynilə, tədqiqatlardan bilirik ki, hover və mətn daxil etmə kimi inteqrasiyaların çox qısa bir müddət ərzində idarə edilməsi lazımdır. Lakin klik və səhifə keçidləri gecikmə hissi vermədən bir az daha gözləyə bilərlər. Konkurrent Rejiminin daxilində istifadə edilən fərqli "prioritetlər", insanın qavrayış tədqiqatındakı inteqrasiya kateqoriyalarına uyğundur.

İstifadəçi təcrübəsinə güclü diqqət yetirən komandalar bəzən oxşar problemləri birdəfəlik həll yolu ilə həll edirlər. Lakin, bu həllərin saxlanılması çətin olduğundan nadirən uzun müddət qorunurlar. Konkurrent Rejimi ilə məqsədimiz UI tədqiqat nəticələrini abstraksiyaya çevirərək istifadə üçün idiomatik yollar təmin etməkdir. Bir UI kitabxanası olaraq React bunun üçün çox uyğun gəlir.

## Növbəti Addımlar {#next-steps}

İndi Konkurrent Rejiminin nə olduğunu bilirsiniz!

Növbəti səhifələrdə müəyyən mövzular haqqında daha çox məlumat əldə edəcəksiniz:

* [Məlumat Yüklənməsi üçün Suspense](/docs/concurrent-mode-suspense.html), React komponentlərində məlumat yükləmək üçün yeni bir mexanizmi təsvir edir.
* [Konkurrent UI Həlləri](/docs/concurrent-mode-patterns.html), Konkurrent Rejimi və Suspense ilə mümkün olan bəzi UI həllərini göstərir.
* [Konkurrent Rejiminə Uyğunlaşma](/docs/concurrent-mode-adoption.html), layihənizdə Konkurrent Rejimini necə sınaya biləcəyinizi izah edir.
* [Konkurrent Rejiminin API Arayışı](/docs/concurrent-mode-reference.html), eksperimental qurulmalarda mövcud olan yeni API sənədləridir.
