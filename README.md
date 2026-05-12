# Modul 9: Event-Driven Architecture - Publisher


**a. How much data your publisher program will send to the message broker in one run?**
Program *publisher* ini akan mengirimkan sebanyak 5 data *event* ke *message broker* dalam satu kali eksekusi (`cargo run`). Hal ini terlihat dari adanya 5 pemanggilan fungsi `p.publish_event(...)` secara berurutan di dalam fungsi `main()`.

**b. The url of: "amqp://guest:guest@localhost:5672" is the same as in the subscriber program, what does it mean?**
Kesamaan URL ini berarti program *publisher* dan *subscriber* terhubung ke *instance Message Broker* (RabbitMQ) yang sama persis. *Publisher* akan mempublikasikan *event/message* ke *broker* di alamat tersebut, dan *subscriber* akan "mendengarkan" (listen) dan mengambil *message* dari antrean (queue) pada *broker* di alamat yang sama agar komunikasi data antara dua program yang terpisah ini dapat terjadi.