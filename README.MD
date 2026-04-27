### JMeter Performance Testing Results
1. Endpoint `/all-student`
![JMeter All Student](images/jmeter_all_student.png)

2. Endpoint `/all-student-name`
![JMeter All Student Name](images/jmeter_all_student_name.png)

3. Endpoint `/highest-gpa`
![JMeter Highest GPA](images/jmeter_highest_gpa.png)

### JMeter CLI Execution
![JMeter CLI](images/jmeter_cli.png)

### Profiling & Optimization Analysis

1. Endpoint `/all-student` (Before Optimization)
![VisualVM Profiling - All Student Before](images/profiling_all_student_before.png)

2. Endpoint `/all-student-name` (Before Optimization)
![VisualVM Profiling - All Student Name Before](images/profiling_all_student_name_before.png)

3. Endpoint `/highest-gpa` (Before Optimization)
![VisualVM Profiling - Highest GPA Before](images/profiling_highest_gpa_before.png)

### Kesimpulan Optimasi (After Optimization)

Setelah melakukan profiling dan menemukan bottleneck, dilakukan refactoring pada ketiga metode di `StudentService`. Hasilnya, performa aplikasi meningkat secara drastis dengan rincian perbaikan sebagai berikut:

1. Endpoint `/all-student` (N+1 Query Problem):
   Mengganti *looping query* dengan satu pemanggilan `studentCourseRepository.findAll()`. Hal ini menghilangkan ribuan query berlebih ke database.

2. Endpoint `/all-student-name` (Memory Overhead):
   Mengganti operasi String concatenation dengan Java Stream API dan `Collectors.joining()`. Hal ini mencegah pembuatan puluhan ribu objek String baru di memori yang membebani Garbage Collector.

3. Endpoint `/highest-gpa` (Inefficient Data Fetching):
   Mendelegasikan proses pencarian nilai tertinggi langsung ke database menggunakan custom query `findFirstByOrderByGpaDesc()`. Hal ini mencegah penarikan seluruh data mahasiswa (puluhan ribu baris) ke dalam memori aplikasi (RAM).

Bukti Profiling Sesudah Optimasi:
![VisualVM Profiling - After Optimization](images/profiling_highest_gpa_after.png)

## Reflection

1. What is the difference between the approach of performance testing with JMeter and profiling with IntelliJ Profiler in the context of optimizing application performance?
JMeter melakukan black-box testing yang berfokus pada performa aplikasi dari sudut pandang pengguna (luar), seperti mengukur response time, throughput, dan error rate saat diberi beban tertentu. Sebaliknya, Profiler (IntelliJ Profiler/VisualVM) melakukan *white-box analysis* yang melihat ke dalam internal aplikasi untuk memantau penggunaan CPU, memori, dan durasi eksekusi setiap metode/fungsi secara spesifik. JMeter memberi tahu kita *bahwa* aplikasi lambat, sedangkan Profiler memberi tahu kita *bagian kode mana* yang membuatnya lambat.

2. How does the profiling process help you in identifying and understanding the weak points in your application?
Proses profiling membantu dengan cara memetakan penggunaan sumber daya CPU ke baris kode tertentu. Melalui fitur seperti Call Tree atau Flame Graph, kita bisa melihat metode mana yang memakan waktu eksekusi paling besar (hotspots). Misalnya, dalam tugas ini, profiling berhasil menunjukkan bahwa inefisiensi pada `joinStudentNames` dan `findStudentWithHighestGpa` adalah penyebab utama tingginya beban CPU.

3. Do you think IntelliJ Profiler is effective in assisting you to analyze and identify bottlenecks in your application code?
Ya, sangat efektif. Dengan profiler, kita tidak perlu menebak-nebak bagian mana yang bermasalah. Data visual yang disajikan mempermudah identifikasi *bottleneck* seperti *N+1 query problem* atau penggunaan memori yang tidak efisien akibat pembuatan objek String yang berulang-ulang, sehingga proses perbaikan kode menjadi lebih terukur.

4. What are the main challenges you face when conducting performance testing and profiling, and how do you overcome these challenges?
Tantangan utamanya adalah memastikan lingkungan pengujian tetap konsisten agar data tidak bias oleh aplikasi lain yang berjalan di komputer. Selain itu, keterbatasan fitur pada IntelliJ Community Edition mengharuskan penggunaan alat eksternal seperti VisualVM. Saya mengatasinya dengan mempelajari integrasi VisualVM dan memastikan aplikasi berjalan dalam kondisi beban yang serupa (menggunakan JMeter CLI) di setiap sesi pengujian.

5. What are the main benefits you gain from using IntelliJ Profiler for profiling your application code?
Manfaat utamanya adalah kemampuan untuk melakukan deep dive ke dalam eksekusi thread secara real-time. Kita bisa mendapatkan metrik presisi mengenai metode yang paling sering dipanggil dan durasi eksekusinya, yang sangat membantu dalam membuktikan apakah optimasi yang dilakukan benar-benar memberikan dampak signifikan pada level kode.

6. How do you handle situations where the results from profiling with IntelliJ Profiler are not entirely consistent with findings from performance testing using JMeter?
Jika terjadi ketidakkonsistenan (misalnya JMeter menunjukkan lambat tapi Profiler terlihat normal), saya akan memeriksa faktor eksternal seperti latensi jaringan, koneksi database, atau konfigurasi server-side (seperti limit pada Tomcat). Saya juga akan melakukan *profiling* ulang dengan beban yang lebih besar di JMeter untuk memastikan *bottleneck* internal benar-benar muncul di bawah tekanan tinggi.

7. What strategies do you implement in optimizing application code after analyzing results from performance testing and profiling? How do you ensure the changes you make do not affect the application's functionality?
(1) Mengurangi akses berulang ke database (menyelesaikan N+1 query), 
8. (2) Menggunakan struktur data yang lebih efisien (seperti `StringBuilder` atau Stream API)
9. (3) Mendelegasikan komputasi berat ke database jika memungkinkan. 