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