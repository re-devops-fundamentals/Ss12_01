# BÁO CÁO THỰC HÀNH: XÂY DỰNG QUY TRÌNH CI ĐA PHIÊN BẢN JDK (MATRIX STRATEGY) CHO JAVA SPRING BOOT GRADLE

---

- **Môn học**: DevOps Fundamentals (IT-209)
- **Session**: 12 - Tối ưu hóa CI/CD Pipeline
- **Bài tập**: Bài 1 - Xây dựng quy trình CI đa phiên bản JDK (Matrix Strategy) cho Java Spring Boot Gradle
- **Học viên**: Hồ Quốc Khải
- **Email**: khaiho6040@gmail.com
- **Đường dẫn gợi ý nộp bài**: `homework/session_12/ex1/`

---

## 1. MỤC TIÊU VÀ YÊU CẦU BÀI TOÁN

### 1.1. Mục tiêu
- Cấu hình ma trận kiểm thử tự động (**Matrix Strategy**) trong GitHub Actions để kiểm thử song song ứng dụng Java Spring Boot trên nhiều phiên bản JDK (JDK 17 LTS và JDK 21 LTS).
- Tối ưu hóa tốc độ thực thi của toàn bộ ma trận kiểm thử thông qua cơ chế lưu bộ đệm phụ thuộc (**Gradle Dependency Caching**).
- Đảm bảo mã nguồn dự án tương thích tốt và chạy thành công trên cả hai phiên bản môi trường Java phổ biến nhất hiện nay.

### 1.2. Yêu cầu kỹ thuật chi tiết
1. Trong thư mục mã nguồn dự án Java Spring Boot (Gradle), tạo tệp kịch bản `.github/workflows/ci-matrix.yml`.
2. Khai báo sự kiện kích hoạt khi `push` vào nhánh `main`.
3. Định nghĩa Job `test-matrix` sử dụng từ khóa `strategy.matrix`:
   - Định nghĩa mảng phiên bản Java: `java-version: ['17', '21']`.
   - Sử dụng `actions/checkout@v5`.
   - Cài đặt Java với `actions/setup-java@v5`, truyền biến `${{ matrix.java-version }}`, `distribution: 'temurin'` và `cache: 'gradle'`.
   - Cấp quyền thực thi: `chmod +x gradlew`.
   - Thực thi lệnh kiểm thử: `./gradlew test`.
4. Khi đẩy code lên GitHub, xác nhận Job `test-matrix` tự động rẽ nhánh thành 2 sub-job chạy song song (`Test on Java 17` và `Test on Java 21`) và đều đạt trạng thái tích xanh (**Success**).

---

## 2. CẤU TRÚC THƯ MỤC DỰ ÁN

```
Ss12/1/
├── .github/
│   └── workflows/
│       └── ci-matrix.yml        # Kịch bản GitHub Actions Matrix Strategy CI workflow
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar   # Gradle wrapper executable archive
│       └── gradle-wrapper.properties # Cấu hình phiên bản Gradle (9.5.1)
├── src/
│   ├── main/
│   │   ├── java/com/example/tasks/
│   │   │   ├── controller/
│   │   │   │   └── TaskController.java        # REST Controller quản lý công việc
│   │   │   ├── model/
│   │   │   │   └── Task.java                  # JPA Entity Task
│   │   │   ├── repository/
│   │   │   │   └── TaskRepository.java        # JpaRepository giao tiếp Database
│   │   │   └── TaskApiApplication.java       # Spring Boot Main Application
│   │   └── resources/
│   │       ├── db/
│   │       │   └── taskdb.sql                 # DDL khởi tạo cơ sở dữ liệu
│   │       └── application.properties        # Cấu hình Spring Datasource & JPA
│   └── test/
│       ├── java/com/example/tasks/
│       │   ├── controller/
│       │   │   └── TaskControllerTest.java    # Unit test JUnit 5 & Mockito cho Controller
│       │   └── TaskApiApplicationTests.java   # Spring Boot Context load test
│       └── resources/
│           └── application.properties        # Cấu hình H2 in-memory DB cho môi trường test
├── .env.example.properties       # Mẫu biến môi trường
├── .gitattributes                # Quy định xử lý line endings
├── .gitignore                    # Loại trừ build artifacts, cache, logs
├── build.gradle                  # Cấu hình Gradle build & sourceCompatibility 17
├── docker-compose.yml            # Docker Compose service cho MySQL và App
├── Dockerfile                    # Multi-stage Dockerfile đóng gói ứng dụng
├── gradlew                       # Gradle wrapper script (Linux/macOS)
├── gradlew.bat                   # Gradle wrapper script (Windows)
├── settings.gradle               # Tên root project (session-12-task-api-practical)
└── README.md                     # Báo cáo thực hành chi tiết
```

---

## 3. NỘI DUNG KỊCH BẢN GITHUB ACTIONS WORKFLOW

Tệp kịch bản được lưu tại: `.github/workflows/ci-matrix.yml`

```yaml
name: Java Spring Boot Matrix CI (Gradle)

on:
  push:
    branches: [ "main" ]

jobs:
  test-matrix:
    name: Test on Java ${{ matrix.java-version }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        java-version: ['17', '21']
    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5

      - name: Cài đặt JDK ${{ matrix.java-version }} (Temurin) & Caching Gradle
        uses: actions/setup-java@v5
        with:
          java-version: ${{ matrix.java-version }}
          distribution: 'temurin'
          cache: 'gradle'

      - name: Cấp quyền thực thi cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Chạy kiểm thử tự động với Gradle
        run: ./gradlew test
```

### Phân tích chi tiết từng khối chỉ thị:

1. **`name: Java Spring Boot Matrix CI (Gradle)`**:
   - Đặt tên hiển thị cho quy trình CI, xuất hiện rõ ràng trên giao diện tab Actions của GitHub repository.

2. **`on: push: branches: [ "main" ]`**:
   - Bộ kích hoạt (Trigger): Workflow sẽ tự động thực thi mỗi khi có thao tác đẩy code (`git push`) vào nhánh `main`.

3. **`jobs: test-matrix:`**:
   - Định nghĩa công việc chính mang mã định danh `test-matrix`.
   - `name: Test on Java ${{ matrix.java-version }}`: Sử dụng biểu thức nội suy để hiển thị tên trực quan cho từng sub-job, tương ứng là `Test on Java 17` và `Test on Java 21`.
   - `runs-on: ubuntu-latest`: Chỉ định runner máy ảo Ubuntu mới nhất do GitHub cung cấp.

4. **`strategy.matrix.java-version: ['17', '21']` (Ma trận kiểm thử)**:
   - Đây là trọng tâm kỹ thuật của bài toán. Thay vì phải viết lặp lại nhiều job riêng lẻ, GitHub Actions sẽ tự động nhân bản (fan-out) job `test-matrix` thành 2 tiến trình độc lập chạy song song:
     - Nhánh 1: `${{ matrix.java-version }} = '17'`
     - Nhánh 2: `${{ matrix.java-version }} = '21'`
   - Nếu một trong hai phiên bản gặp lỗi không tương thích, pipeline sẽ báo đỏ ngay tại phiên bản đó, giúp lập trình viên phát hiện sớm các vấn đề tương thích ngược (backward compatibility) và tính năng mới.

5. **`actions/setup-java@v5` với `cache: 'gradle'`**:
   - Cài đặt bộ phát triển Java Eclipse Temurin OpenJDK theo đúng phiên bản được cấp phát từ ma trận (`17` hoặc `21`).
   - Tham số `cache: 'gradle'` kích hoạt cơ chế tự động hash các tệp build (`build.gradle`, `gradle/wrapper/gradle-wrapper.properties`) và lưu trữ thư mục `~/.gradle/caches` cũng như `~/.gradle/wrapper`.
   - Khi chạy nhiều sub-job hoặc các lần push tiếp theo, runner sẽ tái sử dụng cache các dependencies và wrapper đã tải từ các lần chạy trước, giúp giảm đáng kể thời gian tải gói qua mạng và tăng tốc độ thực thi toàn bộ pipeline.

6. **`run: chmod +x gradlew`**:
   - Cấp quyền thực thi (`execute`) cho script wrapper `gradlew` trên môi trường Linux runner nhằm tránh lỗi `Permission denied`.

7. **`run: ./gradlew test`**:
   - Chạy toàn bộ các bài kiểm thử JUnit (`TaskApiApplicationTests` và `TaskControllerTest`) với cơ sở dữ liệu in-memory H2.

---

## 4. CẤU HÌNH DỰ ÁN GRADLE TƯƠNG THÍCH ĐA PHIÊN BẢN

Để đảm bảo Gradle chạy kiểm thử chính xác bằng đúng phiên bản Java do runner cung cấp (thay vì cố định một toolchain duy nhất), tệp `build.gradle` được cấu hình sử dụng `sourceCompatibility` và `targetCompatibility`:

```groovy
plugins {
	id 'java'
	id 'org.springframework.boot' version '4.1.0'
	id 'io.spring.dependency-management' version '1.1.7'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

java {
	sourceCompatibility = JavaVersion.VERSION_17
	targetCompatibility = JavaVersion.VERSION_17
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-webmvc'
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	runtimeOnly 'com.mysql:mysql-connector-j'
	testImplementation 'org.springframework.boot:spring-boot-starter-webmvc-test'
	testImplementation 'com.h2database:h2'
	testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

tasks.named('test') {
	useJUnitPlatform()
}
```

- **Mục đích**: Biên dịch mã nguồn với tương thích Java 17. Khi chạy trên JDK 17, JVM 17 sẽ biên dịch và chạy test. Khi chạy trên JDK 21, JVM 21 hoàn toàn tương thích và chạy test thành công.

---

## 5. HƯỚNG DẪN TỰ ĐẨY CODE LÊN GITHUB VÀ KIỂM TRA

Theo yêu cầu, học viên sẽ tự khởi tạo Git repository và đẩy code lên GitHub theo các bước sau:

### Bước 1: Mở Terminal tại thư mục `Ss12/1`
```bash
cd "d:\\[IT-209\\] DevOps Fundamentals\\Ss12\\1"
```

### Bước 2: Khởi tạo Git và thêm remote repository
```bash
git init
git checkout -b main
git add .
git commit -m "feat: setup matrix strategy CI for Java 17 and 21 with gradle caching"
```

Thêm remote repository của bạn (ví dụ tạo repo trên GitHub đặt tên là `Ss12_01`):
```bash
git remote add origin https://github.com/<your-username>/<your-repo-name>.git
```

### Bước 3: Đẩy mã nguồn lên GitHub
```bash
git push -u origin main
```

### Bước 4: Kiểm tra kết quả trên tab Actions
1. Truy cập tab **Actions** trên GitHub repository.
2. Bạn sẽ thấy Workflow **`Java Spring Boot Matrix CI (Gradle)`** tự động kích hoạt.
3. Khi click vào phiên chạy, Job `test-matrix` tự động rẽ nhánh thành 2 sub-job chạy song song:
   - :white_check_mark: **`Test on Java 17`**
   - :white_check_mark: **`Test on Java 21`**
4. Chụp ảnh màn hình hiển thị 2 sub-job tích xanh và lưu tệp ảnh vào thư mục `Ss12/1/` (ví dụ `matrix_success.png`) để nộp bài.
