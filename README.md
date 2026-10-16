# ⚡ Microsoft SQL Server (MSSQL) DBA Cockpit

> **Tamamen İstemci Taraflı (Zero-Knowledge), Güvenli ve GitHub Pages Üzerinde 0 TL Maliyetle Çalışan Modern SQL Server DBA Kokpiti.**

Microsoft SQL Server veritabanı yöneticileri (DBA), Sistem Mühendisleri, Veritabanı Geliştiricileri ve DevOps ekipleri için tasarlanmış tek sayfalık (`index.html`) modern bir yönetim, tuning, DMV teşhis ve komut üretim aracıdır.

---

## 🚀 Öne Çıkan 7 Ana Modül

### 1. 📦 Backup, Restore & BCP Sihirbazı
- **Backup (T-SQL)**: `FULL`, `DIFFERENTIAL` ve `TRANSACTION LOG` yedekleme scriptleri.
  - `COMPRESSION`, `CHECKSUM`, `COPY_ONLY` (Liman zincirini bozmayan bağımsız yedek), `INIT / NOINIT`, `FORMAT` ve `STATS = 5` parametreleri.
  - `BUFFERCOUNT = 64` ve `MAXTRANSFERSIZE = 4194304` ile yüksek I/O bant genişliği optimizasyonu.
  - `RESTORE VERIFYONLY` ve `RESTORE HEADERONLY` kontrol komutları.
- **Restore & WITH MOVE Sihirbazı**:
  - `RECOVERY`, `NORECOVERY` (Log shipping ve Always On secondary replica hazırlığı) ve `STANDBY` (salt okunur undo dosyası ile) modları.
  - Mantıksal veri (`.mdf / .ndf`) ve log (`.ldf`) dosyalarını yeni disklere taşıyan dinamik `WITH MOVE` eşleştirme tablosu.
  - `WITH REPLACE` ve `RESTORE FILELISTONLY` yardımcı komutu.
- **BCP (Bulk Copy Program) Komut Üretici**:
  - `out`, `queryout` ve `in` yönleri.
  - Karakter (CSV, Tab, Pipe), Native (`-n`) ve Unicode (`-w`) formatları.
  - Windows Auth (`-T`) veya SQL Auth (`-U -P`).
  - `-F 2` (header atla), `-k` (null koru), `-E` (identity koru), `-C 65001` (UTF-8) ve hata loglama bayrakları.

### 2. 👤 Login, User & Yetkilendirme (Security)
- **Server Login Oluşturma**: `CHECK_POLICY = ON`, `CHECK_EXPIRATION = ON`, `DEFAULT_DATABASE`.
- **Web Crypto API**: Tarayıcı içinde donanımsal rastgelelikle kriptografik güçlü şifre üretici.
- **Database User Eşleştirme**: `CREATE USER [usr] FOR LOGIN [usr]`.
- **Rol Şablonları (Presets)**:
  - **Read-Only**: `db_datareader` + `GRANT VIEW DEFINITION`.
  - **Read-Write**: `db_datareader` + `db_datawriter` + `GRANT EXECUTE`.
  - **DDL Admin**: `db_ddladmin` + okuma/yazma izinleri.
  - **Granular Özel Yetki**: Belirli bir Stored Procedure (`OBJECT::[dbo].[usp_...]`) veya Şemaya (`SCHEMA::[sales]`) özel `EXECUTE`, `SELECT`, `DML` yetkilendirme.
- **Orphaned User (Yetim Kullanıcı)**: Restore sonrası oluşan SID uyumsuzluklarını tespit eden ve `ALTER USER ... WITH LOGIN` ile tek tıkla onaran script.

### 3. 💾 Database, Filegroup & Storage Yönetimi
- **Yeni Dosya Ekleme**: İkincil Veri Dosyası (`.ndf`) veya Log Dosyası (`.ldf`).
  - Filegroup belirleme (`PRIMARY` veya özel filegroup).
  - Sabit büyüme boyutu (`FILEGROWTH = 1024MB`) ile yüzde bazlı büyümenin getirdiği kilitlenme risklerini önleme.
- **DBCC SHRINKFILE & Resize**:
  - `TRUNCATEONLY` modu (sayfa taşımadan yalnızca dosya sonundaki boş alanı işletim sistemine iade eder, indeksleri parçalamaz).
  - `EMPTYFILE` modu (dosyayı silmeden önce içindeki tüm veriyi aynı filegroup'taki diğer dosyalara boşaltır).
  - Üretim ortamlarında neden rutin shrink yapılmaması gerektiğine dair kritik DBA güvenlik uyarıları.
- **Recovery Model Değiştirme**:
  - `FULL`, `SIMPLE` ve `BULK_LOGGED` geçişleri.
  - SIMPLE'dan FULL'e geçerken log zincirini başlatmak için zorunlu olan anlık ilk FULL backup scripti.

### 4. ⚡ Oturum Yönetimi & Kill Helper
- **SPID Sonlandırma**: `KILL <spid>;`.
- **Canlı Rollback İzleme**: `KILL <spid> WITH STATUSONLY;` (Büyük bir transaction iptal edildiğinde rollback ilerleme yüzdesini ve tahmini kalan süreyi izler).
- **Kill Öncesi Teşhis**: SPID'nin hangi sorguyu çalıştırdığını, açık transaction sayısını, CPU ve I/O bekleme durumunu analiz eden T-SQL.
- **SINGLE_USER Modu**: Bakım öncesi veritabanına bağlı tüm oturumları zorla kapatıp tek kullanıcıya alan cankurtaran: `ALTER DATABASE [DB] SET SINGLE_USER WITH ROLLBACK IMMEDIATE;`.
- **Uyuyan Oturumları Kapatma**: X dakikadan eski sleeping transaction'ları dinamik cursor ile sonlandıran script.

### 5. 🔍 Acil Durum & Kritik DMV Deposu (10 Test Edilmiş T-SQL)
- **Blocking & Kilitlenme Ağacı**: Kim kimi bekletiyor, bekleme tipi (`LCK_M_X`, `PAGEIOLATCH_SH`) ve sorgu metinleri.
- **system_health Deadlock Grafikleri**: Varsayılan Extended Events oturumundan canlı XML deadlock raporlarını çıkarma.
- **En Çok CPU Tüketen Top 10 Sorgu**: `sys.dm_exec_query_stats` üzerinden statement bazlı analiz ve XML execution plan.
- **En Çok Disk / Bellek I/O Yapan Top 10 Sorgu**: Mantıksal ve fiziksel okumalarla Buffer Pool'u kirleten sorgular.
- **Otomatik Index Fragmantasyon & Bakım Üretici**: %30 üzerine `REBUILD WITH (ONLINE = ON)`, %10-%30 arasına `REORGANIZE`.
- **Eksik İndeksler (Missing Indexes)**: İyileştirme puanına göre hazır `CREATE NONCLUSTERED INDEX` scriptleri.
- **TempDB Şişiren Sorgular**: Geçici tablolar ve dahili çalışma alanı (sort/hash) tahsislerini yakalama.
- **Always On Senkronizasyon ve Kuyruk Durumu**: `log_send_queue_size`, `redo_queue_size` ve replika gecikmesi.
- **VLF (Virtual Log File) Sayısı Analizi**: 1000'den fazla VLF'i tespit eden log parçalanma kontrolü.
- **Temizlenmiş Wait Statistics**: Arka plan zararsız beklemeleri elenmiş net sunucu darboğaz analizi.

### 6. 🌐 Connection String & Bağlantı Formatlayıcı
- Parametreler: Host, Port (1433), Instance (MSSQLSERVER/SQLEXPRESS), Database, SQL Auth / Windows Auth / Azure MFA, `Encrypt=True`, `TrustServerCertificate=True`, `ApplicationIntent=ReadOnly`, `MultiSubnetFailover`.
- Desteklenen Çıktılar:
  - **ADO.NET / C#**
  - **SQLCMD CLI**
  - **Java / JDBC Thin URL**
  - **Python (pyodbc)**
  - **Python (pymssql)**
  - **Node.js (mssql / tedious)**
  - **PowerShell (Invoke-Sqlcmd)**
  - **ASP.NET Core appsettings.json**

### 7. 🧮 Max Server Memory & MAXDOP Hesaplayıcısı
- **Max Server Memory (MB)**:
  - Microsoft ve Jonathan Kehayias endüstri standardı OS rezervasyon algoritması (İlk 4 GB için 1 GB, 4-16 GB arası her 4 GB için 1 GB, 16 GB üzeri her 8 GB için 1 GB).
  - Görsel bellek dağılım çubuğu ve anlık `sp_configure 'max server memory (MB)'` çıktısı.
- **MAXDOP & Cost Threshold for Parallelism**:
  - NUMA düğüm mimarisi ve çekirdek sayısına göre resmi Microsoft MAXDOP kural motoru.
  - SQL Server'ın eski varsayılanı olan 5 yerine modern işlemciler için önerilen 50 değeri.
  - `RECONFIGURE WITH OVERRIDE` T-SQL scripti.

---

## 🔒 Güvenlik & Zero-Knowledge Garantisi

- **Sıfır Veri İletimi**: Bu araç hiçbir sunucuya bağlanmaz, telemetri göndermez veya API isteği yapmaz.
- **%100 İstemci Taraflı (Client-Side)**: Şifreler ve connection string girdileri dahil tüm veriler yalnızca tarayıcınızın belleğinde üretilir. Sayfa yenilendiğinde tamamen silinir.
- **Denetlenebilir Açık Kaynak**: Tek bir `index.html` dosyasından oluştuğu için kaynak kodunu dilediğiniz gibi inceleyebilirsiniz.

---

## 💻 Nasıl Çalıştırılır?

### 1. Yerel Olarak Çalıştırma (Sıfır Kurulum)
`index.html` dosyasına çift tıklayarak herhangi bir modern web tarayıcısında (Edge, Chrome, Firefox, Safari) doğrudan açabilirsiniz.

### 2. GitHub Pages ile 0 TL Maliyetle Yayınlama
1. GitHub üzerinde `sql-server-dba-cockpit` isimli yeni bir repo oluşturun.
2. Yerel projenizi bağlayın ve push edin:
   ```bash
   git init
   git add index.html README.md
   git commit -m "feat: initial release of SQL Server DBA Cockpit"
   git branch -M main
   git remote add origin https://github.com/alisvo/sql-server-dba-cockpit.git
   git push -u origin main
   ```
3. GitHub reponuzda **Settings > Pages** sekmesine gidin.
4. **Build and deployment > Source** seçeneğini `Deploy from a branch` yapın, `main` dalını ve `/ (root)` dizinini seçip **Save** butonuna tıklayın.
5. Birkaç saniye içinde siteniz `https://alisvo.github.io/sql-server-dba-cockpit/` adresinde canlıya geçecektir!

---

## 📜 Lisans

MIT License - Dilediğiniz gibi kullanabilir, değiştirebilir ve şirket içi sunucularınızda barındırabilirsiniz.
