# Elmas Voyages

Türkçe, mobil uyumlu kişisel seyahat planlama sitesi. Büyük fotoğraflı ana sayfa, paketler, yazdırılabilir örnek kitapçık, üç adımlı talep formu ve Node.js kayıt API'si.

## Yerel çalışma

Node.js 22+ gerekir. Harici npm bağımlılığı yoktur.

```bash
npm start
```

http://localhost:3000 adresinden açın. Kontroller: `npm run check` ve `npm test`.

## VPS kurulumu (Docker Compose + mevcut Nginx)

Docker Engine, Compose eklentisi ve Nginx önceden kurulmuş olmalı. Diğer sitelerin yapılandırmalarını değiştirmeyin. DNS'te `elmasvoyages.com` ve `www` kayıtlarını VPS'ye yönlendirin; kullanılmayan eski AAAA kaydını kontrol edin.

GitHub deposu oluşturulup kod gönderildikten sonra:

```bash
git clone https://github.com/insanevladionurelmas/Elmasvoyages.git
cd Elmasvoyages
cp .env.example .env
docker compose up -d --build
curl -f http://127.0.0.1:8092/healthz
sudo cp deploy/nginx.conf /etc/nginx/sites-available/elmasvoyages
sudo ln -s /etc/nginx/sites-available/elmasvoyages /etc/nginx/sites-enabled/elmasvoyages
sudo nginx -t
sudo systemctl reload nginx
sudo certbot --nginx -d elmasvoyages.com -d www.elmasvoyages.com
```

Depo kullanılmadan ZIP açılarak aynı `cd` sonrası komutlarla da kurulabilir. 8092 kullanımdaysa `.env` APP_PORT ve Nginx proxy_pass portunu birlikte değiştirin. `PUBLIC_ORIGIN` ziyaretçilerin kullandığı kesin HTTPS kök adres olmalı; sonuna `/` eklemeyin. www yönlendirmesini Certbot sonrasında kontrol edin: tüm www istekleri https://elmasvoyages.com adresine gitmeli. API farklı origin'lerden form gönderimini reddeder. TLS kurulmadan üretim formu çalışmaz.

## Gelen talepleri görüntüleme

Talepler Docker `inquiries` volume'ünde, her talep ayrı JSON dosyası olarak saklanır; web üzerinden erişilemez. Yönetim paneli veya e-posta bildirimi bu sürümde yoktur. Operatör SSH üzerinden okuyabilir:

```bash
docker compose exec web sh -c 'ls -lt /app/data'
docker compose exec web sh -c 'cat /app/data/EV-TALEP-NUMARASI.json'
```

Talep bilgileri kişisel veri içerir. Yetkili erişimle sınırlandırın. Silme talebinde ilgili dosyayı silin ve yedek politikasına göre işlem yapın. `docker compose down -v` kullanmayın; kayıtları siler.

Yedek:

```bash
docker compose exec -T web tar -czf - -C /app/data . > inquiries-backup.tar.gz
```

Yedek dosyasını güvenli, erişimi sınırlı bir konumda tutun ve GitHub'a eklemeyin.

## Güncelleme

```bash
git pull --ff-only
docker compose up -d --build
```

## Yayın öncesi işletme ayarları

- `public/index.html`: paket kapsamları önerilen başlangıç kapsamıdır. Hizmet bedelleri ve teslim süreleri uydurulmadı; teklif yöntemi kullanılıyor.
- Veri kullanımı açıklamasına gerçek işletme unvanı, doğrulanmış iletişim adresi ve belirlediğiniz saklama süresini ekleyin. Şu anda bu bilgiler kullanıcı tarafından sağlanmadığı için metinde yer almıyor.
- Form gönderimlerini düzenli takip edecek yetkiliyi belirleyin; otomatik e-posta gönderimi yoktur.
- Gerçek ödeme ve rezervasyon sistemi yoktur. Form yalnızca talep toplar. Kitapçık önizlemesi temsili olarak işaretlenmiştir.
- Kaydedilmiş gerçek talepler, .env ve yedekler Git deposuna dahil edilmez.

## Doğrulama

Syntax kontrolü ve HTTP entegrasyon testi geçti: kayıt oluşturma, diskte kalıcılık, geçersiz tarih/kişi/e-posta, onay, origin denetimi, özel veri erişiminin engellenmesi ve yerel görsel sunumu. Canlı VPS, Docker build ve tarayıcı görsel testi bu ortamda çalıştırılmadı.

## GitHub

Kaynak depo: https://github.com/insanevladionurelmas/Elmasvoyages

## Dosyalar

- `public/`: site ve yerel fotoğraf
- `server.mjs`: güvenlik başlıkları, form doğrulama, hız sınırı ve dosya kaydı
- `compose.yaml`, `Dockerfile`, `deploy/nginx.conf`: VPS dağıtımı
- `test.mjs`: HTTP entegrasyon testi
- `CREDITS.md`: fotoğraf kaynağı
