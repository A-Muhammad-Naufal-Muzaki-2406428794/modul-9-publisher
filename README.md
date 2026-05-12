# Module 09: Event-Driven Architecture - Publisher

Repository ini berisi program **publisher** untuk tutorial Event-Driven Architecture pada Module 09. Program ini bertugas mengirimkan event bertipe `UserCreatedEventMessage` ke RabbitMQ sebagai message broker. Event tersebut kemudian akan diterima dan diproses oleh program **subscriber** melalui queue yang sama, yaitu `user_created`.

## Publisher and Message Broker

### a. How much data will the publisher program send to the message broker in one run?

Program **publisher** ini akan mengirimkan sebanyak **5 data event** ke message broker dalam satu kali eksekusi `cargo run`.

Hal ini terlihat dari adanya 5 pemanggilan fungsi `p.publish_event(...)` secara berurutan di dalam fungsi `main()`. Setiap pemanggilan fungsi tersebut mengirimkan satu `UserCreatedEventMessage` yang berisi `user_id` dan `user_name`.

Dengan demikian, setiap kali program publisher dijalankan, RabbitMQ akan menerima 5 event baru pada queue `user_created`.

### b. The URL `amqp://guest:guest@localhost:5672` is the same as in the subscriber program, what does it mean?

Kesamaan URL `amqp://guest:guest@localhost:5672` berarti program **publisher** dan **subscriber** terhubung ke instance message broker yang sama, yaitu RabbitMQ yang berjalan di local machine.

Penjelasan URL tersebut adalah sebagai berikut:

- `amqp://` menunjukkan bahwa koneksi menggunakan protokol AMQP.
- `guest` pertama adalah username RabbitMQ.
- `guest` kedua adalah password RabbitMQ.
- `localhost` berarti RabbitMQ berjalan pada komputer lokal.
- `5672` adalah port default RabbitMQ untuk komunikasi AMQP.

Karena publisher dan subscriber menggunakan URL yang sama, publisher dapat mengirim event ke RabbitMQ, lalu subscriber dapat mengambil event tersebut dari broker yang sama. Dengan cara ini, kedua program dapat berkomunikasi tanpa harus saling memanggil secara langsung.

---

## Running RabbitMQ as Message Broker

RabbitMQ dijalankan menggunakan Docker dengan image `rabbitmq:3.13-management`. RabbitMQ menggunakan port `5672` untuk koneksi AMQP dari program publisher dan subscriber, sedangkan port `15672` digunakan untuk membuka dashboard RabbitMQ melalui browser.

Dashboard RabbitMQ berhasil dibuka melalui `localhost:15672` menggunakan username dan password default, yaitu `guest`.

![Dashboard Awal RabbitMQ](images/dashboard.png)

---

## Sending and Processing Event

Setelah RabbitMQ berjalan, program subscriber dijalankan terlebih dahulu agar siap menerima message dari queue `user_created`. Setelah itu, program publisher dijalankan menggunakan `cargo run`.

Dalam satu kali eksekusi, publisher mengirimkan 5 event ke RabbitMQ. Event tersebut kemudian diterima dan diproses oleh subscriber. Hal ini menunjukkan bahwa komunikasi event-driven berhasil berjalan, karena publisher dan subscriber tidak berkomunikasi secara langsung, melainkan melalui RabbitMQ sebagai message broker.

![Sending and Processing Event](images/sending-processing.png)

---

## Monitoring Chart Based on Publisher

Ketika program publisher dijalankan, RabbitMQ dashboard menunjukkan adanya aktivitas message pada chart. Setiap eksekusi publisher mengirimkan 5 event ke queue `user_created`, sehingga grafik RabbitMQ dapat menunjukkan lonjakan atau spike pada message rate.

Spike tersebut menunjukkan bahwa message broker menerima event dari publisher. Setelah message diproses oleh subscriber, jumlah message pada queue akan kembali turun. Hal ini membuktikan bahwa RabbitMQ berperan sebagai perantara yang menerima, menyimpan sementara, dan mendistribusikan message ke subscriber.

![Monitoring Chart Based on Publisher](images/slow.png)

---

## Bonus: Cloud Experiment

Selain menjalankan eksperimen secara lokal, saya juga menjalankan eksperimen event-driven architecture di cloud environment. Pada eksperimen ini, RabbitMQ, publisher, dan subscriber dijalankan di environment cloud sehingga alur komunikasi tetap sama seperti pada local machine.

### Cloud RabbitMQ Dashboard

RabbitMQ berhasil dijalankan di cloud menggunakan Docker. Dashboard RabbitMQ dapat diakses melalui forwarded port `15672`, sedangkan koneksi AMQP tetap menggunakan port `5672`.

![Cloud: Dashboard](images/cloud-dashboard.png)

### Cloud Publisher

Program publisher berhasil dijalankan di cloud dan mengirimkan event ke RabbitMQ. Event yang dikirim tetap berjumlah 5 event dalam satu kali eksekusi, sesuai dengan jumlah pemanggilan `publish_event(...)` di dalam kode publisher.

![Cloud: Publisher](images/cloud-publisher.png)

### Cloud Slow Subscriber Simulation

Pada simulasi slow subscriber di cloud, subscriber dibuat lebih lambat dengan mengaktifkan `thread::sleep(ten_millis);`. Setelah publisher dijalankan beberapa kali dengan cepat, RabbitMQ sempat menampung message di queue karena message masuk lebih cepat daripada kemampuan subscriber untuk memprosesnya.

![Cloud: Slow Event](images/cloud-slow.png)

### Cloud Multiple Publishers

Eksperimen juga dilakukan dengan menjalankan beberapa publisher. Dengan lebih dari satu pulisher, message dapat akan menumpuk dan membuat beban subscriber semakin berat untuk memprosesnya.

![Cloud: Three Event](images/cloud-three.png)

---

## Reflection

Dari tutorial ini, saya memahami bahwa event-driven architecture memungkinkan dua program terpisah untuk berkomunikasi melalui message broker. Publisher tidak perlu mengetahui secara langsung bagaimana subscriber bekerja, karena publisher hanya bertugas mengirim event ke RabbitMQ. Subscriber kemudian mengambil dan memproses event tersebut dari queue.

RabbitMQ membantu sistem menjadi lebih fleksibel karena message dapat disimpan sementara ketika subscriber belum siap atau sedang lambat. Hal ini terlihat pada simulasi slow subscriber, ketika queue sempat meningkat karena publisher mengirim message lebih cepat daripada subscriber memprosesnya. Ketika jumlah subscriber ditambah, message dapat diproses lebih cepat karena beban kerja dibagi ke beberapa consumer.

Salah satu hal yang dapat ditingkatkan dari kode publisher adalah data event masih ditulis secara hardcoded di dalam fungsi `main()`. Untuk sistem yang lebih besar, data sebaiknya berasal dari input pengguna, database, atau service lain. Selain itu, error handling juga dapat dibuat lebih baik agar program tidak hanya mengabaikan hasil dari `publish_event(...)`.