# vimics.github.io: vimics.vet sitesi, Claude ve ekip kuralları

Bu repo vimics.vet'in kendisidir. `main` dalına giren her commit birkaç dakika içinde canlıya çıkar (GitHub Pages, `main` / kök).
Ortak çalışanlar: Onur Demirak (`vimics` hesabı, repo sahibi) ve Umut Tutal (`umuttutal`). İkisi de `main`'e doğrudan push eder. Bu yüzden aşağıdaki kurallar conflict'i ve kaybı önlemek için zorunludur.

> Bu dosya `.claude/` altında. Nokta ile başlayan klasörleri Jekyll yayınlamaz, yani dosya vimics.vet'te görünmez. Repo public olduğu için GitHub'da herkes okuyabilir: buraya fiyat, maliyet, müşteri, iş birliği ya da iç not yazma.

---

## 1. Git akışı (conflict önleme)

**Oturum başında, her şeyden önce:**
```zsh
git fetch origin && git status -sb      # geride misin, yerelde commit'lenmemiş iş var mı?
git pull --rebase origin main           # güncelle
```
Tek seferlik ayar (her iki bilgisayarda): `git config pull.rebase true` ve `git config rebase.autoStash true`.

**Çalışırken:**
1. **Küçük ve sık commit.** Bir iş bitince hemen commit + push. Yerelde push edilmemiş commit'i gün sonuna bırakma.
2. **Push öncesi her zaman:** `git pull --rebase origin main` → `python3 -I _src/build.py` → gerekirse yeniden commit → `git push origin main`.
3. **Push reddedilirse** (`rejected`, `non-fast-forward`): asla `--force` kullanma. `git pull --rebase origin main`, derle, push.
4. **Paylaşılan dosyaya büyük değişiklik yapmadan önce haber ver:** `assets/site.css`, `_src/build.py` ve `_src/index.src.html` iki kişinin de sık dokunduğu dosyalar. Yarım saatten uzun sürecek bir değişiklik yapacaksan önce diğer kişiye yaz ("site.css'te header'ı değiştiriyorum").
5. **Birkaç günlük ya da yarım kalabilecek işler dalda yapılır:** `git switch -c onur/vcage-yeni-tablo` (önek: `onur/`, `umut/`). Dal düzenli olarak `git rebase origin/main` ile güncel tutulur, bitince `main`'e birleştirilir ve silinir. Yarım iş `main`'e girmez, çünkü `main` canlıdır.

**Yasaklar:**
- `git push --force` / `--force-with-lease` `main`'e: yasak.
- Push edilmiş commit'i değiştirmek (`commit --amend`, `rebase -i`, `reset --hard` + push): yasak.
- Başkasının dalını izinsiz rebase etmek ya da silmek: yasak.
- `main` dalını silmek, Pages ayarlarını ya da `CNAME`'i değiştirmek: yalnız Onur.

## 2. Üretilen dosyalarda conflict

Yalnız kaynak dosyalar elle düzenlenir: `_src/`, `assets/`. Şunlar **üretilir**, elle düzenlenmez:
`index.html`, `dicom/`, `vcage/`, `vicage/`, `blog/`, `sitemap.xml`, `robots.txt`, `CNAME`, `assets/img/favicon.svg`.

`build.py` deterministiktir (aynı kaynak → aynı çıktı). Rebase/merge sırasında conflict çıkarsa:
```zsh
# 1) Yalnız _src/ ve assets/ içindeki conflict'leri elle çöz (<<<<<<< işaretlerini kaldır)
# 2) Üretilen dosyalardaki conflict'leri ELLE ÇÖZME, herhangi bir tarafı al ve yeniden derle:
git checkout --theirs -- index.html dicom vcage vicage blog sitemap.xml robots.txt 2>/dev/null
python3 -I _src/build.py
git add -A && git rebase --continue      # (merge ise: git commit)
```
Sonra tarayıcıda yerel olarak kontrol et (`python3 -m http.server -d .` → http://localhost:8000).
Emin değilsen dur ve diğer kişiye sor. Conflict'i "benim tarafım kazansın" diye toptan çözme; karşı tarafın değişikliği kaybolur.

## 3. Claude oturumları için

- Oturum başında §1'deki `fetch` + `status` komutlarını çalıştır ve sonucu kullanıcıya bildir (geride/ileride, kirli çalışma alanı).
- **`git push` yalnız oturumu kullanan kişi açıkça "push et" dediğinde yapılır.** Commit serbest, push değil.
- Push'tan sonra Pages derlemesini bekle ve canlı adresi kontrol et:
  ```zsh
  gh api repos/vimics/vimics.github.io/pages/builds/latest --jq '.status+" "+.commit[0:7]'   # "built <commit>"
  curl -s -o /dev/null -w '%{http_code}\n' https://vimics.vet/
  ```
- Force push, history yeniden yazma, dal silme, repo/Pages ayarı değiştirme: kullanıcı açıkça istese bile önce ne kaybolacağını söyle ve teyit al.
- Kullanıcıyla Türkçe konuş; site metni İngilizce.
- Commit mesajı: İngilizce, emir kipi, tek satır özet ("Add VCage size table to order form").

## 4. Derleme ve yapı

```zsh
python3 -I _src/build.py          # tüm HTML, sitemap, robots, CNAME, favicon.svg üretilir
```
- `_src/*.src.html`: sayfa kaynakları. `_src/build.py`: derleme, `UPLOAD_URL`, `VICAGE_SIZES`, `BLOG_POSTS`.
- `_` ile başlayan klasörleri Jekyll yayınlamaz; `_src/` dışarıdan görünmez.
- Yeni blog yazısı: `_src/blog-<ad>.src.html` + `build.py` → `BLOG_POSTS` + `blog.src.html` listesine kart.

## 5. Değişmeyen içerik kuralları

- Tasarım: Vimics Design System, https://claude.ai/artifact/8A46AUCeVLFiiFQ87dr2FU (renk, IBM Plex, logo, ikonlar oradan).
- Yasak ifadeler: CE/FDA onaylı, "iyileşir", "en iyi / lider / tek / ilk / öncü", "%100 kendi üretimimiz", doğrulanmamış sayılar, ünlem, emoji, sonuç garantisi.
- Sitede telefon, fiyat ve teslim süresi yer almaz. Blog'a ticari bilgi (fiyat, maliyet, tedarikçi, rakip, iç geri bildirim, iş birliği detayı) girmez.
- `assets/img/email/` altındaki dosyalar **taşınmaz, yeniden adlandırılmaz**: gönderilmiş e-posta imzaları bu adreslerden yükleniyor.
- Hasta verisi için paylaşım linki (`UPLOAD_URL`) değiştirmeden önce Onur'a danış; link anonim ziyaretçiye klasörü göstermemelidir.
- GoDaddy DNS'te MX/TXT kayıtlarına dokunulmaz (info@vimics.vet e-postası).

## 6. Diğer repolar ve yol çakışması

- `vimics.vet/segmentation` → ayrı repo: **vimics/segmentation**. Bu repoda `segmentation/` adlı klasör **oluşturma**; yol çakışır.
- Yeni bir alt site gerekiyorsa aynı şekilde ayrı repo açılır (`vimics/<ad>`, Pages açık, CNAME dosyası **yok**) ve `vimics.vet/<ad>` altında yayınlanır. Bu repoda o adla klasör açılmaz.
- Alt siteler bu repodaki `/assets/site.css`, fontlar ve ikonları mutlak yolla kullanabilir. `assets/` içinde dosya silme ya da yeniden adlandırmadan önce alt sitelerde kullanılıp kullanılmadığını kontrol et.
