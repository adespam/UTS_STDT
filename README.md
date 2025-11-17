# UTS sistem terdistribusi dan terdesentralisasi
Nama: Ade setya pambudi

NIM: 245410044

1. Jelaskan teorema CAP dan BASE dan keterkaitan keduanya. Jelaskan menggunakan contoh yang pernah anda gunakan. 
2. Jelaskan keterkaitan antara GraphQL dengan komunikasi antar proses pada sistem terdistribusi. Buat diagramnya. 
3. Dengan menggunakan Docker / Docker Compose, buatlah streaming replication di PosthreSQL yang bisa menjelaskan sinkronisasi. Tulislah langkah-langkah pengerjaannya dan buat penjelasan secukupnya.

Jawab:
#### nomor 1
**CAP**

**C - Consistency (Konsistensi):**

Semua klien (pengguna) yang mengakses sistem akan melihat data yang sama pada waktu yang sama, di mana pun mereka terhubung.Secara teknis, setiap operasi read (baca data) harus menerima hasil dari operasi write (tulis data) terbaru atau mengalami error.

**A - Availability (Ketersediaan):**

Sistem selalu merespons setiap permintaan (request) yang masuk, baik itu read atau write. Sistem tidak boleh menolak permintaan hanya karena sedang sibuk atau ada masalah di bagian lain. Sistem selalu online dan tidak mengembalikan error (meskipun datanya mungkin belum yang terbaru).

**P - Partition Tolerance (Toleransi Partisi):**

Sistem tetap dapat beroperasi meskipun terjadi network partition (partisi jaringan). Partisi jaringan adalah kondisi di mana komunikasi antara beberapa node (server) dalam sistem terputus atau sangat lambat. Ini adalah fakta yang tidak bisa dihindari dalam sistem terdistribusi modern (misalnya, koneksi internet yang putus).

sistem tidak dapat menggunakan 3 theorema  CAP harus memilih  consitency (memastikan semua tampilan sama sesuai update terbaru)  atau availability (selalu menampilkan respon)

sebagai contoh sistem yang memilih consistency yaitu sistem bank, dimana ketika kita menyetorkan uang pada cabang bank A maka data kita di cabang  bank B akan ikut berubah atau update.

dan  sebagai contoh sistem yang memilih availability yaitu pada marcket place saat kita memesan atau menambahakan barang pada keranjang kita dapat menambahkannya lagi dengan barang selanjutnya tanpa harus menunggu barang sebelumnya sinkron disistem.

**BASE**

**BA - Basically Available (Pada Dasarnya Tersedia):**

Sistem menjamin Availability (sesuai dengan 'A' dalam CAP). Sistem akan selalu merespons permintaan.

**S - Soft State (Status Lunak):**

Status data dalam sistem dapat berubah seiring waktu, bahkan tanpa input baru dari pengguna. Ini terjadi karena sistem terus-menerus mencoba menyinkronkan data di latar belakang. Data belum "final" atau "kaku" sampai proses sinkronisasi selesai.

**E - Eventually Consistent (Konsisten Pada Akhirnya):**

Sistem tidak menjamin konsistensi seketika (strong consistency). Namun, sistem menjamin bahwa jika tidak ada input baru, semua node pada akhirnya akan memiliki data yang sama (konsistensi pada akhirnya).

jadi teorema BASE merupakan implementasi dari teorema CAP jadi untuk contoh kita melanjutkan contoh pada contoh CAP yang marcket place misal kita terhubung ke server indonesia lalu check out barang dan disaat yang bersamaan terhubung ke server di luar indonesia dan menchek out barang lagi.maka proses tersebut dilakukan secara bersamaan tanpa menunggu salah satu proses sinkron. 

**Keterkaitan CAP dan BASE**

Teorema CAP adalah teorema yang berisi tentang prisip sistem penyimpanan data terdistribusi yang terdiri dari consistency, availabiliy, dan partion tolerance. dan BASE adalah solusi atau filosofi desain dari teorema CAP yang terdiri dari basically available, soft state, dan eventually consistent. jadi keterkaitan dari CAP dan BASE adalah sistem BASE adalah implementasi dari CAP terutaman pada bagian AP (availability). ketika kita memilih mementingkan availability dalam merancang sistem maka kita perlu membangun sistem BASE

#### nomor 2
**GraphQl**

Secara sederhana, GraphQL adalah bahasa kueri (query language) untuk API dan sebuah runtime untuk memenuhi kueri tersebut.GraphQL dirancang untuk membuat API menjadi lebih fleksibel, efisien, dan mudah digunakan, terutama untuk aplikasi klien seperti aplikasi web dan seluler. 

**IPC**

Secara sederhana, IPC (Inter-Process Communication) adalah sekumpulan mekanisme atau aturan yang memungkinkan proses-proses komputer yang terpisah untuk saling berkomunikasi dan berbagi data

**Keterkaitan antara GraphQl dan IPC**

GraphQL adalah bahasa kueri level aplikasi yang membutuhkan mekanisme IPC (biasanya HTTP) untuk dikirimkan dari klien ke server.dan Dalam arsitektur microservices, sebuah server GraphQL sering bertindak sebagai gateway yang menerima satu panggilan IPC (via HTTP) dan kemudian mengoordinasikan banyak panggilan IPC internal lainnya (bisa HTTP, gRPC, dll.) untuk mengumpulkan data.

**Gambar diagramnya**

![alt text](GraphQl-1.png)

#### nomor 3

1. membuat file docker-compose 

version: '3.8'

    services:
  
     postgres-primary:
  
    image: postgres:15
    container_name: postgres-primary
    hostname: postgres-primary
    ports:
      - "5432:5432" # Port primary (host:container)
    volumes:
      - primary_data:/var/lib/postgresql/data
      - ./primary/postgresql.conf:/etc/postgresql/postgresql.conf
      - ./primary/pg_hba.conf:/etc/postgresql/pg_hba.conf
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    environment:
      - POSTGRES_DB=app_db
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=admin_pass
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d app_db"]
      interval: 10s
      timeout: 5s
      retries: 5
    command: postgres -c config_file=/etc/postgresql/postgresql.conf -c hba_file=/etc/postgresql/pg_hba.conf

 
    postgres-standby:
    image: postgres:15
    container_name: postgres-standby
    hostname: postgres-standby
    ports:
      - "5433:5432" # Port standby (host:container)
    depends_on:
      postgres-primary:
        condition: service_healthy # Menunggu primary sehat
    volumes:
      - standby_data:/var/lib/postgresql/data
      - ./standby/entrypoint.sh:/entrypoint.sh
    environment:
      - PGDATA=/var/lib/postgresql/data
      # Variabel untuk skrip entrypoint
      - PRIMARY_HOST=postgres-primary
      - REPL_USER=repl_user
      - REPL_PASS=repl_pass
    entrypoint: ["/bin/bash", "/entrypoint.sh"]
    command: ["postgres"] # Perintah ini diteruskan ke entrypoint.sh

        volumes:
        primary_data:
        standby_data:

2. eksekusi docker-compose.yml

![alt text](gambar1.png)

3. membuat table 





