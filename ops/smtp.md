# Bangun server ngirim surat SMTP anjeun sorangan

## bubuka

SMTP tiasa langsung mésér jasa ti padagang awan, sapertos:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Ali awan email push](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Anjeun ogé tiasa ngawangun server surat anjeun nyalira - ngirim tanpa wates, biaya sadayana rendah.

Di handap ieu, urang nunjukkeun léngkah-léngkah kumaha ngawangun server mail urang sorangan.

## Pilihan server

Pangladén SMTP anu di-host sorangan ngabutuhkeun IP umum kalayan palabuhan 25, 456, sareng 587 kabuka.

Awan umum anu biasa dianggo parantos ngablokir palabuhan ieu sacara standar, sareng tiasa waé dibuka ku cara ngaluarkeun pesenan kerja, tapi éta pisan nyusahkeun.

Abdi nyarankeun mésér ti host anu ngagaduhan palabuhan ieu kabuka sareng ngadukung nyetél nami domain sabalikna.

Di dieu, kuring nyarankeun [Contabo](https://contabo.com) .

Contabo nyaéta panyadia hosting dumasar di Munich, Jérman, diadegkeun dina 2003 kalawan harga pisan kalapa.

Upami anjeun milih Euro salaku mata uang pameseran, hargana langkung mirah (pangladén kalayan mémori 8GB sareng 4 CPU hargana sakitar 529 yuan per taun, sareng biaya instalasi awal gratis salami sataun).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

Nalika nempatkeun pesenan, nyarios `prefer AMD` , sareng server nganggo CPU AMD bakal gaduh kinerja anu langkung saé.

Di handap ieu, kuring bakal nyandak VPS Contabo sabagé conto pikeun nunjukkeun kumaha ngawangun server mail anjeun nyalira.

## Konfigurasi sistem Ubuntu

Sistem operasi di dieu nyaéta Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Lamun server on ssh mintonkeun `Welcome to TinyCore 13!` (sakumaha ditémbongkeun dina gambar di handap), eta hartina sistem teu acan dipasang. Punten pegatkeun sambungan ssh sareng antosan sababaraha menit kanggo log in deui.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Nalika `Welcome to Ubuntu 22.04.1 LTS` mucunghul, initialization geus réngsé, tur anjeun bisa neruskeun kalawan léngkah di handap ieu.

### [Opsional] Initialize lingkungan ngembangkeun

Léngkah ieu opsional.

Pikeun genah, kuring nempatkeun pamasangan sareng konfigurasi sistem parangkat lunak ubuntu di [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

Jalankeun paréntah di handap ieu pikeun masang sareng hiji klik.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Pamaké Cina, punten nganggo paréntah di handap ieu, sareng basa, zona waktos, jsb bakal otomatis disetel.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo ngamungkinkeun IPV6

Aktipkeun IPV6 supados SMTP ogé tiasa ngirim email nganggo alamat IPV6.

édit `/etc/sysctl.conf`

Robah atawa tambahkeun garis handap

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Turutan [tutorial contabo: Nambahkeun konektipitas IPv6 ka server anjeun](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Edit `/etc/netplan/01-netcfg.yaml` , tambahkeun sababaraha garis ditémbongkeun saperti dina gambar di handap ieu (Contabo VPS file konfigurasi standar geus boga garis ieu, ngan uncomment aranjeunna).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Lajeng `netplan apply` sangkan konfigurasi dirobah mawa pangaruh.

Saatos konfigurasi suksés, anjeun tiasa nganggo `curl 6.ipw.cn` pikeun ningali alamat ipv6 tina jaringan éksternal anjeun.

## Kloning ops gudang konfigurasi

```
git clone https://github.com/wactax/ops.soft.git
```

## Ngahasilkeun sertipikat SSL gratis kanggo nami domain anjeun

Ngirim surat butuh sertipikat SSL pikeun énkripsi sareng nandatanganan.

Kami nganggo [acme.sh](https://github.com/acmesh-official/acme.sh) pikeun ngahasilkeun sertipikat.

acme.sh mangrupikeun alat tandatangan sertipikat otomatis open source,

Lebetkeun konfigurasi gudang ops.soft, ngajalankeun `./ssl.sh` , sarta folder `conf` bakal dijieun dina **diréktori luhur** .

Teangan panyadia DNS anjeun ti [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , edit `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Teras jalankeun `./ssl.sh 123.com` pikeun ngahasilkeun sertipikat `123.com` sareng `*.123.com` pikeun nami domain anjeun.

Run kahiji bakal otomatis install [acme.sh](https://github.com/acmesh-official/acme.sh) tur nambahkeun tugas dijadwalkeun pikeun pembaharuan otomatis. Anjeun tiasa ningali `crontab -l` , aya garis sapertos kieu.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

Jalur pikeun sertipikat anu dihasilkeun nyaéta sapertos `/mnt/www/.acme.sh/123.com_ecc。`

Pembaruan sertipikat bakal nelepon `conf/reload/123.com.sh` skrip, ngédit naskah ieu, Anjeun bisa nambah paréntah saperti `nginx -s reload` pikeun refresh cache sertipikat aplikasi patali.

## Ngawangun server SMTP kalawan chasquid

[chasquid](https://github.com/albertito/chasquid) mangrupikeun server SMTP open source anu ditulis dina basa Go.

Salaku gaganti pikeun program mail server kuna kayaning Postfix na Sendmail, chasquid basajan tur gampang ngagunakeun, sarta eta oge gampang pikeun ngembangkeun sekundér.

Jalankeun `./chasquid/init.sh 123.com` bakal dipasang sacara otomatis ku hiji klik (ganti 123.com ku ngaran domain ngirim anjeun).

## Konpigurasikeun Email Signature DKIM

DKIM dianggo pikeun ngirim tanda tangan email pikeun nyegah hurup tina dianggap spam.

Saatos paréntah dijalankeun suksés, anjeun bakal dipenta pikeun nyetél rékaman DKIM (sapertos anu dipidangkeun di handap).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Ngan nambahan rékaman TXT kana DNS anjeun (sakumaha ditémbongkeun di handap).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Nempo status & log jasa

 `systemctl status chasquid` Témbongkeun status jasa.

Kaayaan operasi normal sapertos anu dipidangkeun dina gambar di handap ieu

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` atanapi `journalctl -xeu chasquid` tiasa ningali log kasalahan.

## Konfigurasi ngaran domain sabalikna

Ngaran domain sabalikna nyaéta pikeun ngidinan alamat IP bisa direngsekeun kana ngaran domain pakait.

Nyetél ngaran domain sabalikna bisa nyegah surelek ti keur diidentifikasi minangka spam.

Nalika suratna ditampi, pangladén panampi bakal ngalakukeun analisa ngaran domain sabalikna dina alamat IP tina server anu ngirim pikeun ngonfirmasi naha server anu ngirim ngagaduhan nami domain sabalikna anu valid.

Lamun server ngirim teu boga ngaran domain sabalikna atawa lamun ngaran domain sabalikna teu cocog alamat IP tina server ngirim, server panarima bisa mikawanoh email salaku spam atawa nampik eta.

Didatangan [https://my.contabo.com/rdns](https://my.contabo.com/rdns) sareng konpigurasikeun sapertos anu dipidangkeun di handap ieu

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Saatos netepkeun nami domain sabalikna, émut pikeun ngonpigurasikeun resolusi maju tina nami domain ipv4 sareng ipv6 ka server.

## Edit hostname of chasquid.conf

Ngaropéa `conf/chasquid/chasquid.conf` kana nilai ngaran domain sabalikna.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Teras ngajalankeun `systemctl restart chasquid` pikeun ngamimitian deui jasa.

## Cadangan conf ka git Repository

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Contona, kuring nyadangkeun folder conf kana prosés github kuring sorangan saperti kieu

Jieun gudang pribadi heula

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Lebetkeun diréktori conf sareng kirimkeun ka gudang

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Tambahkeun pangirim

lumpat

```
chasquid-util user-add i@wac.tax
```

Bisa nambahan pangirim

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Pastikeun kecap akses disetel leres

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Saatos nambihan pangguna, `chasquid/domains/wac.tax/users` bakal diropéa, émut pikeun ngirimkeun ka gudang.

## DNS nambahkeun catetan SPF

SPF (Sender Policy Framework) mangrupikeun téknologi verifikasi email anu dianggo pikeun nyegah panipuan email.

Éta verifikasi identitas pangirim surat ku mariksa yén alamat IP pangirim cocog sareng rékaman DNS tina nami domain anu diklaim, nyegah panipuan ngirim email palsu.

Nambahkeun rékaman SPF bisa nyegah surelek ti keur diidentifikasi minangka spam saloba mungkin.

Upami server ngaran domain anjeun henteu ngadukung jinis SPF, tambahkeun catetan jinis TXT.

Contona, SPF tina `wac.tax` nyaéta kieu

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF pikeun `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Catet yén kuring geus `include:_spf.google.com` dieu, ieu sabab kuring bakal ngonpigurasikeun `i@wac.tax` salaku alamat ngirim dina kotak surat Google engké.

## Konfigurasi DNS DMARC

DMARC nyaéta singketan tina (Auténtikasi Pesen basis Domain, Pelaporan & Conformance).

Hal ieu dipaké pikeun néwak mantul SPF (panginten disababkeun ku kasalahan konfigurasi, atanapi batur anu nyamar janten anjeun ngirim spam).

Tambahkeun catetan TXT `_dmarc` ,

eusina kieu

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

Harti unggal parameter nyaéta kieu

### p (Kawijakan)

Nunjukkeun cara nanganan surelek nu gagal verifikasi SPF (Sender Policy Framework) atawa DKIM (DomainKeys Identified Mail). Parameter p tiasa disetel ka salah sahiji tina tilu nilai:

* euweuh: Taya aksi dicokot, ngan hasil verifikasi fed deui pangirim ngaliwatan mékanisme ngalaporkeun email.
* Karantina: Pasang surat anu teu lulus verifikasi kana polder spam, tapi moal nampik surat langsung.
* nampik: Langsung nampik surelek nu gagal verifikasi.

### fo (Pilihan Gagal)

Nangtukeun jumlah inpormasi anu dipulangkeun ku mékanisme ngalaporkeun. Éta tiasa disetel ka salah sahiji nilai ieu:

* 0: Laporan hasil validasi pikeun sadaya pesen
* 1: Ngan laporkeun pesen anu gagal verifikasi
* d: Ngan ngalaporkeun gagal verifikasi ngaran domain
* s: ngan ngalaporkeun gagal verifikasi SPF
* l: Ngan ngalaporkeun gagal verifikasi DKIM

### rua & ruf

* rua (Ngalaporkeun URI pikeun laporan agrégat): Alamat surélék pikeun narima laporan agrégat
* ruf (Ngalaporkeun URI pikeun laporan Forensik): alamat surélék pikeun nampa laporan lengkep

## Tambahkeun rékaman MX pikeun neraskeun surelek ka Google Mail

Kusabab kuring henteu mendakan kotak surat perusahaan gratis anu ngadukung alamat universal (Catch-All, tiasa nampi email anu dikirim ka nami domain ieu, tanpa larangan awalan), kuring nganggo chasquid pikeun neraskeun sadaya email ka kotak surat Gmail kuring.

**Upami Anjeun gaduh kotak surat bisnis mayar sorangan, punten ulah ngaropéa MX jeung skip hambalan ieu.**

Édit `conf/chasquid/domains/wac.tax/aliases` , setel kotak surat diteruskeun

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` nunjukkeun sadaya email, `i` mangrupikeun awalan alamat email pangguna anu ngirim anu didamel di luhur. Pikeun neraskeun surat, unggal pangguna kedah nambihan garis.

Lajeng nambahkeun catetan MX (Kuring nunjuk langsung ka alamat tina ngaran domain sabalikna dieu, ditémbongkeun saperti dina garis kahiji dina gambar di handap).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Saatos konfigurasi réngsé, anjeun tiasa nganggo alamat email anu sanés pikeun ngirim email ka `i@wac.tax` sareng `any123@wac.tax` pikeun ningali naha anjeun tiasa nampi email dina Gmail.

Upami henteu, pariksa log chasquid ( `grep chasquid /var/log/syslog` ).

## Kirim surélék ka i@wac.tax kalawan Google Mail

Saatos Google Mail nampi suratna, kuring ngarep pisan ngabales nganggo `i@wac.tax` tinimbang i.wac.tax@gmail.com.

Buka [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) teras klik "Tambahkeun alamat surélék anu sanés".

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Teras, lebetkeun kodeu verifikasi anu ditampi ku email anu diteruskeun.

Tungtungna, éta tiasa disetél salaku alamat pangirim standar (sareng pilihan pikeun ngabales sareng alamat anu sami).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Ku cara kieu, kami parantos ngabéréskeun ngadegna pangladén surat SMTP sareng dina waktos anu sami nganggo Google Mail pikeun ngirim sareng nampi email.

## Kirim surélék test pikeun pariksa naha konfigurasi geus suksés

Lebetkeun `ops/chasquid`

Jalankeun `direnv allow` masang dependensi (direnv parantos dipasang dina prosés inisialisasi hiji konci sateuacana sareng pancing parantos ditambah kana cangkang)

tuluy lumpat

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

Harti paraméter nyaéta kieu

* pamaké: ngaran pamaké SMTP
* lulus: sandi SMTP
* ka: panarima

Anjeun tiasa ngirim email tés.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Disarankeun make Gmail pikeun nampa surelek test pikeun mariksa naha konfigurasi geus suksés.

### enkripsi standar TLS

Ditémbongkeun saperti dina gambar di handap ieu, aya konci leutik ieu, nu hartina sertipikat SSL geus suksés diaktipkeun.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Teras klik "Témbongkeun Email Asli"

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Sapertos dina gambar di handap, halaman surat asli Gmail nunjukkeun DKIM, anu hartosna konfigurasi DKIM parantos suksés.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Pariksa Narima dina lulugu tina email aslina, tur anjeun tiasa ningali yén alamat pangirim nyaeta IPV6, nu hartina IPV6 ogé ngonpigurasi junun.
